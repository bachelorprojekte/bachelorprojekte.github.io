---
layout: post
title: Automating LaTeX compilation in Gitlab-CI
category: bpbahn
author: Marcus Ding
tags:
- gitlab-ci
- latex
- python
---

## The problem

You are pushing LateX documents into a repository with all of your friends. Your friends compile the PDF files locally and sometimes push garbage into the repo. Also you want to email your assignments to your tutors. But you are already annoyed by writing commit messages. Opening your email client and dragging and dropping that PDF, for you it's pain in it's purest form. This makes you very sad. You ask yourself: What could I do about it?

<!--more-->

## The solution

Combining some Python scripting and GitLab-CI configs results in a nice pipeline that processes LaTeX documents automatically while enforcing a certain folder structure. No more garbage and no more pain. Find the [example repo here](https://gitlab.com/kryptokommunist/latexpipeline).

{% include image.html url="/bpbahn/images/gitlab-ci.png" description='The working <a href="https://gitlab.com/kryptokommunist/latexpipeline/pipelines">GitLab CI pipeline</a>' %}

{% include image.html url="/bpbahn/images/gitlab-ci-out.png" description="The script output" %}

{% include image.html url="/bpbahn/images/gitlab-ci-artifacts.png" description='The uploaded resulting artifacts <a href="https://gitlab.com/kryptokommunist/latexpipeline/-/jobs/57720189/artifacts/browse">accessible in GitLab</a>' %}

### Email

For emailing a document for example to your tutors the [scripts/pipeline/latexpipeline.py](https://gitlab.com/kryptokommunist/latexpipeline/blob/master/scripts/pipeline/latexpipeline.py) submodule will parse the commit message and search for the `--final` flag. In our example the commit message

    Add my assignment no 5 --final assignment_papers/documentfoldername --comment Sorry for my late submission, it will not happen again! 😇💋

would result in an raw text email being sent to the recipient specified in [`scripts/pipeline_script.py`](https://gitlab.com/kryptokommunist/latexpipeline/blob/master/scripts/pipeline_script.py) with the email of the GitLab User being CCed as Reply-To.

{% include image.html url="/bpbahn/images/gitlab-ci-mail.png" description='The mail being sent to the address defined in <a href="https://gitlab.com/kryptokommunist/latexpipeline/blob/master/scripts/pipeline_script.py">scripts/pipeline_script.py</a>' %}

### Telegram and IFTT

With a little help by [IFTTT.com](https://ifttt.com) it's very easy to integrate Telegram notifications into the system using a bot. One benefit is certainly shaming users into testing their TeX files before submitting them knowing that their utter failure will be public.

{% include image.html url="/bpbahn/images/gitlab-ci-telegram.png" description='Exemplary Telegram notifications about successes and failures' %}

{% include image.html url="/bpbahn/images/gitlab-ci-ifttt.png" description='The applets for the bachelorprojects Telegram group including the Webhook based applet forwarding messages to the Telegram bot.' %}

### A short intro to GitLab CI

[GitLab CI](https://docs.gitlab.com/ce/ci/README.html) is like any other continuous integration platform with the additional feature of [job artifacts](https://docs.gitlab.com/ee/user/project/pipelines/job_artifacts.html). This means a job can produce files as output that will be stored and and can be directly linked to on the GitLab platform. Of course this is ideal for our LaTeX pipeline. Let's take a look at the `compile_pdf` stage:

{% highlight yml %}
compile_pdf:
  stage: compile_pdf
  before_script:
    - rm -rf htmlcov
  script:
    - python3 scripts/pipeline_script.py run "$GITLAB_USER_NAME" $CI_JOB_ID "$(git log -1 --pretty=%B)" $GITLAB_USER_EMAIL $CI_PROJECT_URL
  cache:
    key: single_pdfs
    paths:
      - assignment_papers/latex_output
      - letters/latex_output
  artifacts:
    paths:
      - assignment_papers/*.pdf
      - letters/*.pdf
  only:
    - master
{% endhighlight %}

First it executes the [`scripts/pipeline_script.py`](https://gitlab.com/kryptokommunist/latexpipeline/blob/master/scripts/pipeline_script.py) script. The necessary arguments are the [GitLab variables](https://docs.gitlab.com/ce/ci/variables/README.html) `$GITLAB_USER_NAME`, `$CI_JOB_ID`, `$GITLAB_USER_EMAIL` and `$CI_PROJECT_URL` as well as the current commit message: `$(git log -1 --pretty=%B)`. The executed [scripts/pipeline_script.py](https://gitlab.com/kryptokommunist/latexpipeline/blob/master/scripts/pipeline_script.py) will write all files resulting from document compilation to `assignment_papers/latex_output/*` and `letters/latex_output/*` and copy the PDFs to `assignment_papers/*.pdf` and `letters/*.pdf/.

Setting the key `cache` results in all files specified under `paths` being cached between job runs given you're using a dedicated [GitLab runner](https://docs.gitlab.com/ce/ci/runners/README.html). Alos it will speed up compilation because documents will only be recompiled if there's a change. In our example case it reduced the runtime from 5 minutes to just 1 minute. Unfortunately the free shared runners on gitlab.com don't allow this. But what about the [job artifacts](https://docs.gitlab.com/ee/user/project/pipelines/job_artifacts.html)?

The [`scripts/pipeline_script.py`](https://gitlab.com/kryptokommunist/latexpipeline/blob/master/scripts/pipeline_script.py) script will copy the resulting PDFs from `foldername/latex_output/*pdf` to `foldername/*.pdf`. This happens at the `PIPELINE.move_pdfs()` function call. Thus we specify under `artifacts` the `paths` where GitLab CI will look for matching files to store them as artifacts.

{% include image.html url="/bpbahn/images/gitlab-ci-caching.png" description='Processing time without caching and with enabled caching.' %}

### What does the script do?

Ultimately after checking that the folder structures are compliant the script will execute the following lines in the [scripts/pipeline/latexpipeline.py](https://gitlab.com/kryptokommunist/latexpipeline/blob/master/scripts/pipeline/latexpipeline.py) submodule:

{% highlight python %}
        for dir_name, data in second_level_items.items():
            main_filename = dir_name + ".tex"
            folderpath = join(self.relative_path, dir_name)
            cmd = ('cd "{fp}" && latexmk -output-directory=../latex_output -pdf "{f}"'\
                " -e '$pdflatex=q/pdflatex %O -shell-escape %S/'")\
                .format(fp=folderpath, f=main_filename)
            if call(cmd, shell=True) == 0:
                sccmsg = "\033[92mCompiled \033[94m{d}.tex\033[92m successfully\033[0m"\
                    .format(d=dir_name)
                print(sccmsg)
            else:
                errmsg = "\033[91mError with compiling \033[94m{d}.tex\033[0m\ncmd: {c}"\
                    .format(d=dir_name, c=cmd)
                self.error_notify_and_exit(errmsg)
{% endhighlight %}

The dictionary `second_level_items` contains all but the exempted subfolders in a valid root subdirectory. Now all it does for a folder called `documentfoldername` residing in `assignment_papers` is executing these commands:

{% highlight shell %}
cd documentfoldername
latexmk -output-directory=../latex_output -pdf "documentfoldername.tex" -e '$pdflatex=q/pdflatex %O -shell-escape %S/'
{% endhighlight %}

After executing the script will print a success message. If the command fails it will notify about this and exit with an error so that the GitLab job will fail.

## Enforcing folder rules

The script [`scripts/pipeline_script.py`](https://gitlab.com/kryptokommunist/latexpipeline/blob/master/scripts/pipeline_script.py) enforces a folder structure like this:

```
.
|
+--assignment_papers
|  +--documentfoldername
|  |  +--figures
|  |  |  +--example.png
|  |  +--documentfoldername.tex
|  |  +--preamble.tex
|  |  +--content.tex
|  +--otherdocumentfoldername
|  |  +--figures
|  |  |  +--example.pdf
|  |  +--otherdocumentfoldername.tex
|  |  +--preamble.tex
|  |  +--content.tex
+--letters
|  +--documentfoldername
|  |  +--figures
|  |  |  +--example.jpg
|  |  +--documentfoldername.tex
|  +--otherdocumentfoldername
|  |  +--figures
|  |  |  +--example.jpg
|  |  +--otherdocumentfoldername.tex
```

 - there can be multiple folders the root directory
    - `assignment_papers`
      - subfolders must contain `.tex` files with the same name
      - subfolders must contain `preamble.tex` file
      - subfolders must contain `content.tex` file
      - subfolders may contain a figures folder
    - `letters`
      - subfolders must contain `.tex` files with the same name
      - subfolders may contain a figures folder

These rules are determined in the [`scripts/pipeline_script.py`](https://gitlab.com/kryptokommunist/latexpipeline/blob/master/scripts/pipeline_script.py) in the `FOLDER_STRUCTURES` dictionary. It maps the root level subfolder names to folder structure rules formatted as dictionaries. For example if a subfolders name begins with a string in the list referenced by the `"exempt_dirs_start_with"` key it will be exempted from being processed by the pipeline.

{% highlight python %}
#root level directories we won't process
EXEMPT_DIRS = ["scripts", "__pycache__"]
#dirs starting with given string in LateXPipeline.relative_path will be ignored
#also applied to root level dirs
EXEMPT_DIRS_START_WITH = ["."]
#files mandatory in each LaTeX folder
MANDATORY_FILES_ASSIGNMENT = ["preamble.tex", "references.bib", "content.tex"]
#directories in LateXPipeline.relative_path the pipeline will ignore
EXEMPT_DIRS_ASSIGNMENT = ["latex_output", "BP_COMBINED", "__pycache__"]
#folders allowed but not mandatory in subdirectories
ALLOWED_SUBDIRS = ["figures"]
#allowed folder structure for folder assignment_papers
FOLDER_STRUCTURE_ASSIGNMENT = {"exempt_dirs": EXEMPT_DIRS_ASSIGNMENT,\
    "exempt_dirs_start_with": EXEMPT_DIRS_START_WITH,\
    "mandatory_files": MANDATORY_FILES_ASSIGNMENT,\
    "allowed_subdirs": ALLOWED_SUBDIRS}
#allowed folder structure for folder letters
FOLDER_STRUCTURE_LETTERS = dict(FOLDER_STRUCTURE_ASSIGNMENT)
FOLDER_STRUCTURE_LETTERS["mandatory_files"] = []
#different folder structures for each root level dir
#test_root_folder(_one/two) is necessary for pytest testing
FOLDER_STRUCTURES = {"test_root_folder": FOLDER_STRUCTURE_ASSIGNMENT,\
    "test_script_root_folder": FOLDER_STRUCTURE_ASSIGNMENT,\
    "test_folder_one": FOLDER_STRUCTURE_ASSIGNMENT, "test_folder_two": FOLDER_STRUCTURE_ASSIGNMENT,\
    "assignment_papers": FOLDER_STRUCTURE_ASSIGNMENT, "letters": FOLDER_STRUCTURE_LETTERS}
{% endhighlight %}

### Known issues

There is still a problem with calculating the code coverage. As seen in the current [example repo ](https://gitlab.com/kryptokommunist/latexpipeline) the displayed code coverage is just about 80% when it should be very close to 100 percent. Unfortunately the tests in [scripts/tests/test_latexpipeline.py](https://gitlab.com/kryptokommunist/latexpipeline/blob/master/scripts/tests/test_latexpipeline.py) calling the script with `subprocess.run` won't be measured by `codecov`:

{% highlight python %}
def test_console_test_usage(self):
     """Check that Latexmk actually works and the module can be invoked as script from command line"""
     ret = subprocess.run("python3 " + RELATIVE_SCRIPT_PATH + ' test test@test.de "Test --meta lol"',\
         shell=True, cwd=SCRIPT_TEST_ROOT_DIR_PATH, stderr=subprocess.PIPE, stdout=subprocess.PIPE)
     print(ret.stderr)
     assert b"YAY" in ret.stdout
     assert b"Latexmk" in ret.stderr
{% endhighlight %}
