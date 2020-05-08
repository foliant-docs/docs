# Architecture And Basic Design Concepts

Foliant is an open-source software written in Python.

Foliant has modular architecture. Foliant itself, also known as [Foliant Core](https://github.com/foliant-docs/foliant/), is a relatively compact and rarely updated package. Foliant Core centrally manages extensions and provides base classes for developing extensions according to a common pattern.

Suddenly, Foliant Core doesn’t build documentation projects. All application functionality is provided by Foliant extensions. There are almost 50 extensions for Foliant. If you don’t need some extensions, you may not to install them.

There are 4 types of Foliant extensions.

* **Backends** are modules that prepare your Markdown content in accordance to requirements of third-party software that builds some final **targets**, e.g. PDF files or static sites. Also backends manage external commands that call such software. Usually one backend manages one external command, for example, `pandoc` or `mkdocs`. A backend may not to call any external command but upload your content to some external service, e.g. Confluence. A single backend may generate multiple targets. Different backends may build the same target. For example, the target `site` (a static site) can be built with 3 backends: MkDocs, Slate, and Aglio. If more than one of them are installed, you should choose the certain backend to build the required target: it may be specified in the command line or asked interactively.
* **Preprocessors** are modules that perform various transformations of your source Markdown content before passing the content to a backend. Preprocessors are the most numerous type of Foliant extensions. They may be very simple like [RemoveExcess](https://github.com/foliant-docs/foliantcontrib.removeexcess/blob/develop/foliant/preprocessors/removeexcess.py), or pretty complicated like [Includes](https://github.com/foliant-docs/foliantcontrib.includes/blob/develop/foliant/preprocessors/includes.py). Each Foliant project may use any number of preprocessors that are applied sequentially, one after another. Preprocessors provide many kinds of content transformations. Preprocessors may:
    * replace something in your content according to some rules;
    * draw diagrams and schemes from code;
    * include content from external files into your sources;
    * get data for your documentation project from external services, e.g. remote Git repositories, Swagger, Testrail, Figma, Sympli, etc.;
    * set high-level semantic relations between different parts of you content to provide smart cross-target links, or to restructure your single-source documentation automatically, context-dependently;
    * and even run arbitrary external commands that can do anything with your files.
* **CLI extensions** extend Foliant’s command-line interface and provide additional actions that may be called from the command line.
* **Config extensions** provide additional actions that are performed during reading the project’s config. For example, MultiProject extension allows to include multiple nested Foliant projects into a parent Foliant project’s structure.

Foliant’s architecture is shown at the following diagram.

![Architecture of Foliant](https://github.com/foliant-docs/docs/blob/master/src/images/basic-architecture.png)
