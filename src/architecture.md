# Architecture And Basic Design Concepts

Foliant is an open-source software written in Python.

[![Architecture of Foliant](https://raw.githubusercontent.com/foliant-docs/docs/master/src/images/basic-architecture.png)(https://raw.githubusercontent.com/foliant-docs/docs/master/src/images/basic-architecture.png)

Foliant has modular architecture. [Foliant Core](https://github.com/foliant-docs/foliant/) is a relatively compact and rarely updated package. Foliant Core is a dispatcher which manages installed extensions according to the configuration. Foliant Core package also defines base classes for all types of extensions.

Foliant Core itself does not build documentation projects, this job is delegated to extensions.

There are 4 types of base Foliant extensions.

* **Backends** are responsible for building the project's Markdown content into final formats which we call *targets*, e.g. PDF files or static sites. Backends manage external commands that call third party software to produce the final documentation. Usually one backend manages one external command, for example, `pandoc` or `mkdocs`. A backend may not to call any external command but upload your content to some external service, e.g. Confluence. A single backend may generate multiple targets. Different backends may build the same target. For example, the target `site` (a static site) can be built with 3 official backends: MkDocs, Slate, and Aglio. If more than one of them are installed, you should choose the certain backend to build the required target: it may be specified in the command line or asked interactively.
* **Preprocessors** are modules that perform various transformations of your source Markdown content before passing it to a backend. Preprocessors are the most numerous type of Foliant extensions. They may be very simple like [RemoveExcess](https://github.com/foliant-docs/foliantcontrib.removeexcess/blob/develop/foliant/preprocessors/removeexcess.py), or pretty complicated like [Includes](https://github.com/foliant-docs/foliantcontrib.includes/blob/develop/foliant/preprocessors/includes.py). Each Foliant project may use any number of preprocessors that are applied sequentially, one after another. Preprocessors provide many kinds of content transformations. Preprocessors may:
    * replace something in your content according to some rules;
    * draw diagrams and schemes from code;
    * include content from external files into your sources;
    * get data for your documentation project from external services, e.g. remote Git repositories, Swagger, Testrail, Figma, Sympli, SQL Database etc.;
    * set high-level semantic relations between different parts of you content to provide smart cross-target links, or to restructure your single-source documentation automatically, context-dependently;
    * and even run arbitrary external commands that can do anything with your files.
* **CLI extensions** extend Foliant’s command-line interface and provide additional actions that may be called from the command line. This is always the topmost component of any Foliant's action. *foliant* ***make*** is in fact a CLI extension which builds projects.
* **Config extensions** allow to customize the project configuration parsing, add custom YAML tags and new configuration options. For example, MultiProject extension adds a YAML tag `!from` which allows to include multiple nested Foliant projects into a parent Foliant project’s structure.

# Project Build Process

The project build process is governed by Foliant CLI extension called *make*. make is a part of Foliant core package.

The build process is described below:

1. User calls a `make` command specifying the backend and the target he wants to build, for example:
    
        $ foliant make site --with mkdocs

    In this example the target is `site` and backend is `mkdocs`. `--with` argument is optional, `make` will assume the backend or ask for user to input the name interactively.

2. make launches the project build in the following stages:
    1. **Configuration parsing**. The project configuration file (`foliant.yml` if not customized in `--config` argument) is processed by each installed *Config extension* and saved into internal context.
    2. **Copying sources**. The `src` folder which holds Markdown source files of the project is copied into a temporary folder, which is called `__folianttmp__` by default. Preprocessors will only work with the copies, leaving the sources intact.
    3. **Preprocessing**. Each *preprocessor* defined in the project configuration file is applied to the copy of Markdown sources. The preprocessors run in an order in which they are specified in the `preprocessors` list, but each backend may implicitly add specific preprocessors to the beginning or to the end of this list.
    4. **Producing output format**. The chosen *backend* takes the Markdown sources in the temporary dir and converts them into output format.
