# Developer’s Reference

You’re welcome to contribute to Foliant and its extensions. Foliant ecosystem, of course, needs a lot of new useful extensions for all occasions.

Foliant and its extensions are developed in Git repositories that belong to the [foliant-docs](https://github.com/foliant-docs/) GitHub group.

The repo of Foliant Core is called [foliant](https://github.com/foliant-docs/foliant/). Repositories of Foliant extensions have names starting with the `foliantcontrib.` prefix. The repo of this documentation is called [docs](https://github.com/foliant-docs/docs/).

Foliant Core modules and base classes that should be used in all newly developed extensions, are described at this page.

## Core Modules

Foliant Core includes a number of modules:

* `foliant`:
    * `backends`:
        * [`base`](https://github.com/foliant-docs/foliant/blob/develop/foliant/backends/base.py)—defines the base class for all backends, see below;
        * [`pre`](https://github.com/foliant-docs/foliant/blob/develop/foliant/backends/pre.py)—simplest backend that returns Markdown content processed by specified preprocessors as build result;
    * `preprocessors`:
        * [`base`](https://github.com/foliant-docs/foliant/blob/develop/foliant/preprocessors/base.py)—defines the base class for all preprocessors, see below;
        * [`_unescape`](https://github.com/foliant-docs/foliant/blob/develop/foliant/preprocessors/_unescape.py)—simple preprocessor that allows to use pseudo-XML tags that are recoginzed by other preprocessors as control sequences, in code examples. If you want an opening tag not to be interpreted by any preprocessor, precede this tag with the `<` character. The preprocessor `_unescape` removes such characters. Instead of the `_unescape` preprocessor, it’s recommended to use more flexible [EscapeCode and UnescapeCode](https://github.com/foliant-docs/foliantcontrib.escapecode/) preprocessors;
    * [`cli`](https://github.com/foliant-docs/foliant/blob/develop/foliant/cli/__init__.py)—defines the Foliant’s root class `Foliant()` and the `entry_point()` method that is used as a starting point when calling Foliant; nested modules:
        * [`base`](https://github.com/foliant-docs/foliant/blob/develop/foliant/cli/base.py)—defines the base class for all CLI extensions, see below;
        * [`make`](https://github.com/foliant-docs/foliant/blob/develop/foliant/cli/make.py)—provides the main Foliant’s command `make`;
    * `config`:
        * [`base`](https://github.com/foliant-docs/foliant/blob/develop/foliant/config/base.py)—defines the base class for all config extensions;
        * [`include`](https://github.com/foliant-docs/foliant/blob/develop/foliant/config/include.py)—resolves the `!include` YAML tag that allows to reuse the data of one YAML file in another;
        * [`path`](https://github.com/foliant-docs/foliant/blob/develop/foliant/config/path.py)—resolves the `!path` YAML tag. The string preceded by this modifier should be converted into an existing path relative to the Foliant project’s top-level (“root”) directory;
    * [`utils`](https://github.com/foliant-docs/foliant/blob/develop/foliant/utils.py)—defines some basic methods that may be used in extensions of different types.

### The `make()` Method Arguments

To build any Foliant project, the method `make()` that is defined in the module [`foliant.cli.make`](https://github.com/foliant-docs/foliant/blob/develop/foliant/cli/make.py) should be called in one or another way.

The method takes a number of arguments; some of them pass onwards to backends and preprocessors:

* `target` (string)—required resulting target of the current build;
* `backend` (string, defaults to an empty string)—backend that is used for the current build;
* `project_path` (path, defaults to the current directory path)—path of top-level, “root” directory of the current Foliant project;
* `config_file_name` (string, defaults to `foliant.yml`)—Foliant project’s config file name;
* `quiet` (boolean, default to `False`)—flag that prohibits writing to `STDOUT`;
* `keep_tmp` (boolean, defaults to `False`)—flag that tells Foliant and its extensions not to delete a temporary working directory that is used during build;
* `debug` (boolean, defaults to `False`)—flag that tells Foliant and its extensions to log events of `info` and `debug` levels in additional to messages of `warning`, `error`, and `critical` levels.

## Base Classes

Foliant Core provides 4 base classes—one per each type of extensions.

* `BaseBackend()` that is defined in the [`foliant.backends.base`](https://github.com/foliant-docs/foliant/blob/develop/foliant/backends/base.py) module, is the base class for all backends. Each newly developed backend should:
    * be a module `foliant.backends.<your_backend_name>`;
    * import the class `BaseBackend()` from the `foliant.backends.base` module;
    * define its own class called `Backend()` that is inherited from `BaseBackend()`;
    * define the method called `make()` within the class `Backend()`.
* `BasePreprocessor()` that is defined in the [`foliant.preprocessors.base`](https://github.com/foliant-docs/foliant/blob/develop/foliant/preprocessors/base.py) module, is the base class for all preprocessors. Each newly developed preprocessor should:
    * be a module `foliant.preprocessors.<your_preprocessor_name>`;
    * import the class `BasePreprocessor()` from the `foliant.preprocessors.base` module;
    * define its own class called `Preprocessor()` that is inherited from `BasePreprocessor()`;
    * define the method called `apply()` within the class `Preprocessor()`.
* `BaseCli()` that is defined in the [`foliant.cli.base`](https://github.com/foliant-docs/foliant/blob/develop/foliant/cli/base.py) module, is the base class for all CLI extensions. Each newly developed CLI extension should:
    * be a module `foliant.cli.<your_cli_extension_name>`;
    * import the class `BaseCli()` from the `foliant.cli.base` module;
    * define its own class called `Cli()` that is inherited from `BaseCli()`.
* `BaseParser()` that is defined in the [`foliant.config.base`](https://github.com/foliant-docs/foliant/blob/develop/foliant/config/base.py) module, is the base class for all config extensions. Each newly developed config extension should:
    * be a module `foliant.config.<your_config_extension_name>`;
    * import the class `BaseParser()` from the `foliant.config.base` module;
    * define its own class called `Parser()` that is inherited from `BaseParser()`.

Quick reference on important variables that are provided by base classes, is below.

### The `BaseBackend()` Variables

* Class variables:
    * `targets` (tuple of strings)—names of targets that the backend can build;
    * `required_preprocessors_before` (tuple of strings)—names of preprocessors that should be applied before all other preprocessors when this target is used;
    * `required_preprocessors_after` (tuple of strings)—names of preprocessors that should be applied after all other preprocessors when this target is used;
* instance variables:
    * `self.context`—dictionary that contains the keys:
        * `project_path` (path)—path to the currently built Foliant project;
        * `config` (dictionary)—currently built Foliant project’s full config;
        * `target` (string)—name of the resulting target;
        * `backend` (string)—name of the backend that is used in the current build;
    * `self.config`—exactly the same as `self.context['config']`;
    * `self.project_path`—exactly the same as `self.context['project_path']`;
    * `self.working_dir` (path)—exactly the same as `self.project_path / self.config['tmp_dir']`, the path to the temporary working directory that is used during build;
    * `self.logger`—logger instance;
    * `self.quiet` (boolean)—if `True`, the backend should not write anything to `STDOUT`;
    * `self.debug` (boolean)—if `True`, the backend should log the messages of `info` and `debug` levels.

### The `BasePreprocessor()` Variables

* Class variables:
    * `defaults` (dictionary)—default values of options that may be overridden in config;
    * `tags` (tuple of strings)—names of pseudo-XML tags that are recognized by the preprocessor, without `<` and `>` characters;
* instance variables:
    * `self.context`—dictionary that contains the keys:
        * `project_path` (path)—path to the currently built Foliant project;
        * `config` (dictionary)—currently built Foliant project’s full config;
        * `target` (string)—name of the resulting target;
        * `backend` (string)—name of the backend that is used in the current build;
    * `self.config`—exactly the same as `self.context['config']`;
    * `self.project_path`—exactly the same as `self.context['project_path']`;
    * `self.working_dir` (path)—exactly the same as `self.project_path / self.config['tmp_dir']`, the path to the temporary working directory that is used during build;
    * `self.logger`—logger instance;
    * `self.quiet` (boolean)—if `True`, the preprocessor should not write anything to `STDOUT`;
    * `self.debug` (boolean)—if `True`, the preprocessor should log the messages of `info` and `debug` levels.
    * `self.options` (dictionary)—preprocessor’s options, i.e. `{**self.defaults, **options}` where `options` is data that is read from config;
    * `self.pattern`—regular expression [pattern](https://github.com/foliant-docs/foliant/blob/develop/foliant/preprocessors/base.py#L53) that is used to get components of a pseudo-XML tag in easy way. Defined if `self.tags` is not empty. Provides the groups with the following names:
        * `tag`—for tag name;
        * `options`—for tag attributes (options) as a string; this string may be converted into a dictionary by using the [`self.get_options()` method](https://github.com/foliant-docs/foliant/blob/develop/foliant/preprocessors/base.py#L17) provided by the basic class;
        * `body`—for tag body, i.e. a content between the opening and closing tags.

### The `BaseCli()` Variables

* Instance variables:
    * `self.logger`—logger instance.

### The `BaseConfig()` Variables

* Instance variables:
    * `self.project_path` (path)—path to the currently built Foliant project;
    * `self.config_path` (path)—path to the currently built Foliant project’s config file;
    * `self.logger`—logger instance;
    * `self.quiet` (boolean)—if `True`, the config extension should not write anything to `STDOUT`.
