---
tags:
  - overview
  - developing
---

# Developer’s Reference

The power of Foliant is in its extensions. Foliant's ecosystem consists of many beautiful tools for technical writers, but there is still a lot to be done. You are welcome to contribute to Foliant and its extensions.

This article contains the reference of the main classes and functions available in Foliant Core. As an extension developer, you will be using them to write your own preprocessors, backends, CLI- and config-extensions.

If you are new to extending Foliant, we suggest you to take a look at the <link src="tutorials/preprocessor/intro.md">Creating a Preprocessor</link> tutorial first.

Official Foliant extensions live in Git repositories inside the [foliant-docs](https://github.com/foliant-docs/) GitHub group. Check out their source code to find out different approaches to solving techwriters' problems.

The repo of Foliant Core is called [foliant](https://github.com/foliant-docs/foliant/). The names of Foliant extensions' repositories start with the `foliantcontrib.` prefix. The repo of this documentation project is called [docs](https://github.com/foliant-docs/docs/).

## Core Modules

Core modules live in the [foliant](https://github.com/foliant-docs/foliant) GitHub repository. Foliant Core itself does not build documentation projects, this job is delegated to extensions. But it defines the base classes for all types of extensions. Each base class offers useful attributes and methods which are described later in this article. For more info on how Foliant works check the <link src="architecture.md"></link> article.

This section lists all modules in the Foliant Core package.

* `foliant`:
    * `backends`:
        * [`base`](https://github.com/foliant-docs/foliant/blob/develop/foliant/backends/base.py) — defines the base class for all backends;
        * [`pre`](https://github.com/foliant-docs/foliant/blob/develop/foliant/backends/pre.py) — simplest backend that returns Markdown content processed by specified preprocessors as a build result;
    * `preprocessors`:
        * [`base`](https://github.com/foliant-docs/foliant/blob/develop/foliant/preprocessors/base.py) — defines the base class for all preprocessors;
        * [`_unescape`](https://github.com/foliant-docs/foliant/blob/develop/foliant/preprocessors/_unescape.py) — simple preprocessor that escapes pseudo-XML tags (which are normally recognized by other preprocessors as control sequences) in code examples. If you want an opening tag to be ignored by any preprocessor, precede this tag with the `<` character. The `_unescape` preprocessor removes these characters before build. Instead of the `_unescape` preprocessor, you may use more flexible [EscapeCode and UnescapeCode](https://foliant-docs.github.io/docs/preprocessors/escapecode/) preprocessors;
    * [`cli`](https://github.com/foliant-docs/foliant/blob/develop/foliant/cli/__init__.py) — defines the Foliant’s root class `Foliant()` and the `entry_point()` method that is used as a starting point for calling Foliant. Nested modules:
        * [`base`](https://github.com/foliant-docs/foliant/blob/develop/foliant/cli/base.py) — defines the base class for all CLI extensions;
        * [`make`](https://github.com/foliant-docs/foliant/blob/develop/foliant/cli/make.py) — provides the main Foliant’s `make` command;
    * `config`:
        * [`base`](https://github.com/foliant-docs/foliant/blob/develop/foliant/config/base.py) — defines the base class for all config extensions;
        * [`include`](https://github.com/foliant-docs/foliant/blob/develop/foliant/config/include.py) — resolves the `!include` YAML tag that allows to include the content of additional YAML-files in Foliant config. More info in the <link src="config.md" title="'!include'">Project Configuration</link> article;
        * [`path`](https://github.com/foliant-docs/foliant/blob/develop/foliant/config/path.py) — resolves the `!path`, `!project_path` and `!rep_path` YAML tags. These tags are useful for specifying file paths in Foliant config or tag attributes. More info in the <link src="config.md" title="'!path, !project_path, !rel_path'">Project Configuration</link> article;
    * [`utils`](https://github.com/foliant-docs/foliant/blob/develop/foliant/utils.py) — defines basic methods that may be used in different types of extensions.

### The `make()` Method Arguments

The `make()` method is defined in the [`foliant.cli.make`](https://github.com/foliant-docs/foliant/blob/develop/foliant/cli/make.py) module. This method is called when the user runs `foliant make ...` command. For more info on how `make` command works check the <link src="architecture.md" title="Project Build Process"></link> article.

The `make()` method accepts a number of arguments; some of them are then passed to the backends and preprocessors in the build *context*:

* `target` (string) — required resulting target of the current build;
* `backend` (string, defaults to an empty string) — the name of the backend that is used for the current build;
* `project_path` (path, defaults to the current directory path) — the path of top-level, “root” directory of the current Foliant project;
* `config_file_name` (string, defaults to `foliant.yml`) — the file name of the Foliant project’s config;
* `quiet` (boolean, default to `False`) — a flag that prohibits writing to `STDOUT`;
* `keep_tmp` (boolean, defaults to `False`) — a flag that tells Foliant and its extensions to preserve the temporary working directory, which is used during the build;
* `debug` (boolean, defaults to `False`) — a flag that tells Foliant and its extensions to log events of `info` and `debug` levels in addition to messages of `warning`, `error`, and `critical` levels.

## Base Classes

Foliant Core provides 4 base classes—one per each type of extension.

* `BaseBackend()` is defined in the [`foliant.backends.base`](https://github.com/foliant-docs/foliant/blob/develop/foliant/backends/base.py) module. It is the base class for all backends. Each newly developed backend should:
    * be a module or a package `foliant.backends.<your_backend_name>`;
    * import the class `BaseBackend()` from the `foliant.backends.base` module;
    * define its own class called `Backend()` that is inherited from `BaseBackend()`;
    * define the method called `make()` within the `Backend` class.
* `BasePreprocessor()` is defined in the [`foliant.preprocessors.base`](https://github.com/foliant-docs/foliant/blob/develop/foliant/preprocessors/base.py) module. It is the base class for all preprocessors. Each newly developed preprocessor should:
    * be a module or a package `foliant.preprocessors.<your_preprocessor_name>`;
    * import the class `BasePreprocessor()` from the `foliant.preprocessors.base` module;
    * define its own class called `Preprocessor()` that is inherited from `BasePreprocessor()`;
    * define the method called `apply()` within the class `Preprocessor()`.
* `BaseCli()` is defined in the [`foliant.cli.base`](https://github.com/foliant-docs/foliant/blob/develop/foliant/cli/base.py) module. It is the base class for all CLI extensions. Each newly developed CLI extension should:
    * be a module or a package `foliant.cli.<your_cli_extension_name>`;
    * import the class `BaseCli()` from the `foliant.cli.base` module;
    * define its own class called `Cli()` that is inherited from `BaseCli()`.
* `BaseParser()` is defined in the [`foliant.config.base`](https://github.com/foliant-docs/foliant/blob/develop/foliant/config/base.py) module. It is the base class for all config extensions. Each newly developed config extension should:
    * be a module or a package `foliant.config.<your_config_extension_name>`;
    * import the class `BaseParser()` from the `foliant.config.base` module;
    * define its own class called `Parser()` that is inherited from `BaseParser()`.

### The `BaseBackend()` Attributes

* Class attributes:
    * `targets` (tuple of strings) — names of the targets that the backend can build;
    * `required_preprocessors_before` (tuple of strings) — names of the preprocessors that should be applied before all other preprocessors when this backend is used;
    * `required_preprocessors_after` (tuple of strings) — names of preprocessors that should be applied after all other preprocessors when this backend is used;
* instance variables:
    * `context` — a dictionary that contains the build context:
        * `project_path` (path) — path to the currently built Foliant project;
        * `config` (dictionary) — full config of the currently built Foliant project;
        * `target` (string) — the name of the resulting target;
        * `backend` (string) — the name of the backend that is used in the current build;
    * `config` — full config of the currently built Foliant project. The same as `context['config']`;
    * `project_path` — path to the currently built Foliant project. The same as `context['project_path']`;
    * `working_dir` (path) — the path to the temporary working directory that is used during the build. It is defined as `self.project_path / self.config['tmp_dir']`;
    * `logger` — the [Logger](https://docs.python.org/3/library/logging.html#logging.Logger) instance of the current build;
    * `quiet` (boolean) — if `True`, the backend should not write anything to stdout;
    * `debug` (boolean) — if `True`, the backend should log the messages of `info` and `debug` levels.

### The `BasePreprocessor()` Attributes

* Class attributes:
    * `defaults` (dictionary) — default values of options that may be overridden in config;
    * `tags` (tuple of strings) — names of pseudo-XML tags that are recognized by the preprocessor, without `<` and `>` characters;
* instance variables:
    * `context` — a dictionary that contains the build context:
        * `project_path` (path) — path to the currently built Foliant project;
        * `config` (dictionary) — full config of the currently built Foliant project;
        * `target` (string) — the name of the resulting target;
        * `backend` (string) — the name of the backend that is used in the current build;
    * `config` — full config of the currently built Foliant project. The same as `self.context['config']`;
    * `project_path` — path to the currently built Foliant project. The same as `self.context['project_path']`;
    * `working_dir` (path) — the path to the temporary working directory that is used during the build. It is defined as `self.project_path / self.config['tmp_dir']`;
    * `logger` — the [Logger](https://docs.python.org/3/library/logging.html#logging.Logger) instance of the current build;
    * `quiet` (boolean) — if `True`, the backend should not write anything to stdout;
    * `debug` (boolean) — if `True`, the backend should log the messages of `info` and `debug` levels.
    * `options` (dictionary) — the preprocessor’s options. Is defined as `{**self.defaults, **options}`, where `options` is the data that is read from the preprocessor's config in foliant.yml;
    * `pattern` — the regular expression [pattern](https://github.com/foliant-docs/foliant/blob/develop/foliant/preprocessors/base.py#L53) that is used to get components of a pseudo-XML tag in an easy way. Defined if `self.tags` is not empty. Provides the RegEx groups with the following names:
        * `tag` — captured tag name;
        * `options` — captured tag attributes (options) as a string; this string may be converted into a dictionary by using the [`self.get_options()` method](https://github.com/foliant-docs/foliant/blob/develop/foliant/preprocessors/base.py#L17), which is provided by the base class;
        * `body` — captured tag body, i.e. the content between the opening and closing tags.

### `BaseCli()` Attributes

* Instance attributes:
    * `logger` — the [Logger](https://docs.python.org/3/library/logging.html#logging.Logger) instance of the current build.

### `BaseConfig()` Attributes

* Instance attributes:
    * `project_path` (path) — the path to the currently built Foliant project;
    * `config_path` (path) — the path to the config file of the currently built Foliant project;
    * `logger` — the [Logger](https://docs.python.org/3/library/logging.html#logging.Logger) instance of the current build;
    * `quiet` (boolean) — if `True`, the config extension should not write anything to stdout.
