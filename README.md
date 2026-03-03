#   Foliant Documentation

[![Build Status](https://github.com/foliant-docs/docs/actions/workflows/deploy-site-to-pages.yml/badge.svg)](https://github.com/foliant-docs/docs/actions/workflows/deploy-site-to-pages.yml)

##  Build Locally

With Docker Compose:

```bash
$ git clone git@github.com:foliant-docs/foliant.git
$ cd foliant/docs
# Site:
$ docker compose run --rm site
# PDF:
$ docker compose run --rm pdf
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
