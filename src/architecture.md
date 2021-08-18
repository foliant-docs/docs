# Architecture And Basic Design Concepts

Foliant is an open-source application written in Python.

[![Architecture of Foliant](https://raw.githubusercontent.com/foliant-docs/docs/master/src/images/basic-architecture.png)](https://raw.githubusercontent.com/foliant-docs/docs/master/src/images/basic-architecture.png)

Foliant has a modular architecture. [Foliant Core](https://github.com/foliant-docs/foliant/) is a relatively compact and rarely updated package. Foliant Core is a dispatcher which manages installed extensions according to the configuration. The Core package also defines base classes for all types of extensions.

Foliant Core itself does not build documentation projects, this job is delegated to extensions.

There are 4 types of base Foliant extensions.

* **Backends** build the project's Markdown content into final formats which we call *targets*, e.g. PDF files or static sites. Backends may call third-party software to produce the final documentation or upload your content to an external service, e.g. Confluence. A single backend may generate multiple targets. Different backends may build the same target. For example, a static site (the `site` target) can be built with 3 official backends: MkDocs, Slate, and Aglio. If several of them are installed, user may specify the certain backend in the `foliant make` command or it will be asked interactively.
* **Preprocessors** are modules which apply various transformations to the source Markdown content before passing it to a backend. The transformations include:

    * replacing parts of content according to specific rules;
    * rendering diagrams and schemes from source code;
    * embedding content from external files;
    * getting data for your documentation project from external services, e.g. remote Git repositories, Swagger, Testrail, Figma, Sympli, SQL Databases etc.;
    * seting high-level semantic relations between different parts of content to provide smart cross-target links, or restructure single-source documentation automatically and context-dependently;
    * running arbitrary external commands.

    Each Foliant project may use any number of preprocessors. Preprocessors are applied sequentially, one after another. The same preprocessor may appear more than once in the pipeline.

* **CLI extensions** extend Foliant’s command-line interface and provide additional actions that may be called from the command line. This is always the topmost component of any Foliant's action. *foliant* ***make*** is in fact a CLI extension that builds projects.
* **Config extensions** allow to customize the project configuration parsing, add custom YAML tags, and new configuration options. For example, MultiProject extension adds a YAML tag `!from` which allows to include multiple nested Foliant projects into a single parent project.

# Project Build Process

The project build process is operated by Foliant CLI extension called *make*, which is a part of Foliant Core package.

**The steps of the build process**

1. User calls a `make` command specifying the backend and the target he wants to build, for example:
    
        $ foliant make site --with mkdocs

    In this example, the target is `site` and the backend is `mkdocs`. `--with` argument is optional, `make` will assume the backend or ask for user input if there are several options for the target.

2. `make` launches the project build in the following stages:
    1. **Configuration parsing**. The project configuration file (`foliant.yml` by default) is processed by each installed *Config extension* and saved into the internal context.
    2. **Copying sources**. The `src` folder which holds Markdown source files of the project is copied into a temporary folder (`__folianttmp__` by default). Preprocessors will only affect the copies, leaving the sources intact.
    3. **Preprocessing**. Each *preprocessor* defined in the project configuration file is subsequently applied to the temporary folder with copies of Markdown sources. The preprocessors run in an order in which they are specified in the `preprocessors` list, but each backend may implicitly add specific preprocessors to the beginning or the end of this list.
    4. **Producing output format**. The chosen *backend* takes the Markdown files from the temporary folder and converts them into the target format.
    5. **Removing temporary files**. If `make` wasn't run with `--keep-tmp|-k` argument, the temporary folder with preprocessed Markdown sources is removed from the project dir.
