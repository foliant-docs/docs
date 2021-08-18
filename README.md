#   Foliant Documentation

[![Build Status](https://api.travis-ci.com/foliant-docs/docs.svg?branch=master)](https://travis-ci.com/github/foliant-docs)

[https://foliant-docs.github.io/docs/](https://foliant-docs.github.io/docs/)

##  Build Locally

With Docker Compose:

```bash
$ git clone git@github.com:foliant-docs/foliant.git
$ cd foliant/docs
# Site:
$ docker-compose run --rm site
# PDF:
$ docker-compose run --rm pdf
```

With pip and stuff (requires Python 3.6+, Pandoc, and TeXLive):

```bash
$ git clone git@github.com:foliant-docs/foliant.git
$ cd foliant/docs
$ pip install -r requirements
# Site:
$ foliant make site
# PDF:
$ foliant make pdf
```
