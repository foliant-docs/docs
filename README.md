#   Foliant Documentation

[![Build Status](https://travis-ci.org/foliant-docs/docs.svg?branch=master)](https://travis-ci.org/foliant-docs/docs)

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
