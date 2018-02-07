# Repo for [kryptokommunist.github.io](https://kryptokommunist.github.io) [![Build Status](https://travis-ci.org/kryptokommunist/kryptokommunist.github.io.svg?branch=jekyll)](https://travis-ci.org/kryptokommunist/kryptokommunist.github.io)

This is the repo for my personal blog powered by static site generator [jekyll](https://jekyllrb.com) and [github pages](https://pages.github.com).

## Auto deployment on [Travis-CI](https://travis-ci.org)

The [.travis.yml file](https://github.com/kryptokommunist/kryptokommunist.github.io/blob/jekyll/.travis.yml) is configured to execute the [build.sh](https://github.com/kryptokommunist/kryptokommunist.github.io/blob/jekyll/build.sh) with each push to the jekyll branch. The jekyll site will be generated and pushed to the master branch of the repository.

If you want to use this as a template just generate an github access token.  Then

- `git clone git@github.com:kryptokommunist/kryptokommunist.github.io.git`
- `git checkout jekyll`
- remove the last three lines from the [.travis.yml file](https://github.com/kryptokommunist/kryptokommunist.github.io/blob/jekyll/.travis.yml)
- generate a personal [github token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/)
- `gem install travis`
- `travis encrypt GH_TOKEN=$YOUR_TOKEN_HERE --add env.global`
- replace github paths/names in the [build.sh](https://github.com/kryptokommunist/kryptokommunist.github.io/blob/jekyll/build.sh)
- replace articles in _posts folder
- create personal repository named $YOUR_GH_USERNAME.github.io
- activate Travis-CI for repo
- push repo to your personal repository named $YOUR_GH_USERNAME.github.io

and you're good to go!
