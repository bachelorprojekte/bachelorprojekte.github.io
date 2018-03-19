---
layout: post
title: Latex in Gitlab
category: bpbahn
author: Marcus Ding
tags:
- gitlab-ci
- latex
- python
---

{% highlight python %}
def compile_pdfs(self):
    """compile every TeX file in each repository subfolder"""
    second_level_items = {}
    self.first_level_dirs = self.remove_exempt_folders(self.first_level_dirs)

    for first_level_dir in self.first_level_dirs:
        dir_path = join(self.relative_path, first_level_dir)
        files_folders = get_files_folders(dir_path)
        second_level_items[first_level_dir] = files_folders

    for dir_name, data in second_level_items.items():
        self.enforce_compliance(dir_name, data)

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
