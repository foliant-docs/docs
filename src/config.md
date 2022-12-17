---
tags:
  - overview
  - config
  - yaml
---

# Project Configuration

Configuration for Foliant is kept in a [YAML](https://yaml.org/) file in the project root. The default filename is `foliant.yml` but you can pick a different name by specifying the `--config` option:

```bash
$ foliant make pdf --config myconf.yml
```

## Config Sections

* <link title="Root Options"></link>
* <link title="chapters"></link>
* <link title="preprocessors"></link>
* <link title="backend_config"></link>

### Root Options

These are the options that are placed at the root of the config file. There are several built-in options, which are described below, but extensions may introduce their own root options (for example, <link src="config/alt_structure.md"></link> or <link src="preprocessors/escapecode.md"></link>). Refer to each extension’s respective docs for details.

Here are all built-in root options:

```yaml
title: My Awesome Project
slug: myproj
src_dir: src
tmp_dir: __folianttmp__
```

`title` *(string)*
:    Project title. It will be used to generate the resulting file name, if `slug` option is not defined.

`slug` *(string)*
:    Slug is a string which will be used to name the output file or folder after build. For example, if slug is `myproj`, the output PDF will be saved into `myproj.pdf`. If not defined — `title` will be used to generate filename.

`src_dir` *(string)*
:    Name of the directory with your project’s Markdown source files. Default: `src`.

`tmp_dir` *(string)*
:    Name of the directory where the intermediate files will be stored during preprocessor pipeline execution. Default: `__folianttmp__`.

### chapters

*(list)*

`chapters` is a list of paths to the Markdown sources which you want to be used in the project. The paths are specified relative to your `src` dir.

Here’s a basic chapters list:

```yaml
chapters:
    - intro.md
    - definitions.md
    - tutorial.md
```

Chapters may be nested with mappings and sublists. These complex structures may be treated differently by different backends: some may ignore nesting, some may use it to alter the resulting build. But usually, these two ideas are shared between all backends:

- only those Markdown files which are mentioned in the chapters list will appear in the resulting build;
- the order in which chapters are mentioned in the list will be preserved in the resulting build.

Consider this example chapters list:

```yaml
chapters:
    - intro.md  # list item
    - definitions.md  # list item
    - How To Use This Tutorial: tutorial_help.md  # mapping with one element
    - Creating Documentation With Foliant:  # mapping with nested list
        - preprequisites.md
        - Preparing Config:
            - root options.md
            - chapters.md
            - preprocessors.md
            - backend_config.md
        - create_sources.md
        - building_project.md
```

In this example first two chapters are defined as simple list items, the third chapter is a mapping with one element, and after that, we see several mappings with nested lists.

If we were building a PDF document with <link src="backends/pandoc.md">Pandoc backend</link> or a static site with <link src="backends/slate.md">Slate backend</link>, this complex chapter structure will be ignored, as if we have supplied a simple flat list:

```yaml
chapters:
    - intro.md
    - definitions.md
    - tutorial_help.md
    - preprequisites.md
    - root options.md
    - chapters.md
    - preprocessors.md
    - backend_config.md
    - create_sources.md
    - building_project.md
```

In any case, we would get a one-file PDF or a one-page site with data from listed Markdown files in the provided order.

But if we were building a site with <link src="backends/mkdocs.md" title="MkDocs">MkDocs backend</link>, mappings would become meaningful.

For example, this element:

```yaml
    - How To Use This Tutorial: tutorial_help.md
```

means "take the source from `tutorial_help.md` but change its title to `How To Use This Tutorial`" in the sidebar.

And this element:

```yaml
    - Creating Documentation With Foliant:
        - preprequisites.md
        - Preparing Config:
            - root options.md
            - chapters.md
            - preprocessors.md
            - backend_config.md
```

means "create a subsection **Creating Documentation With Foliant** in the sidebar and nest the `preprequisites.md` chapter inside. Then nest another subsection **Preparing Config** within the first one, and nest four other chapters inside of it".

Refer to each backend’s respective docs for details on how they work with chapters.

### preprocessors

*(list)*

All preprocessors which you want to be used in your project, should be listed under the `preprocessors` section:

```yaml
preprocessors:
    - macros:  # options are adjusted
        macros:
            ref: <if backends="pandoc">{pandoc}</if><if backends="mkdocs">{mkdocs}</if>
    - flags  # all options are set to defaults
    - includes
    - blockdiag
    - plantuml:
        params:
            config: !path configs/plantuml.cfg
    - graphviz:
        format: svg
        as_image: false
        params:
            Gdpi: 0
```

Each preprocessor has to be put in a separate list item. If you don’t need to set any options, just put the preprocessor’s name in the item (`flags`, `includes`, and `blockdiag` in the example above). If you are setting preprocessor options, then make it a mapping, with key being the preprocessor name, and value — another mapping, with preprocessor options. (`macros`, `plantuml`, and `graphviz` in the example above).

Refer to each preprocessor’s respective docs for details on which options they have and how to set them.

There are several things you have to keep in mind when building the preprocessors section:

**The order matters**

The order, in which the preprocessors are defined in the list, is the order they are run during the build. For example, if you are using <link src="preprocessors/includes.md">Includes preprocessor</link> to get source code for a PlantUML scheme like this:

```html
<plantuml>
    <include url="http://example.com/scheme.puml"></include>
</plantuml>
```

then `includes` must be defined before `plantuml` in the preprocessor list. Otherwise, you will get an error from PlantUML when it tries to process `<include>` tag instead of the scheme code.

Some preprocessors are especially sensitive to their position in the list (for example, <link src="preprocessors/superlinks.md" title="SuperLinks">SuperLinks</link>) and there may even be situations when you will have to put the same preprocessor in the list twice.

**Preprocessors are applied to all files**

Generally, preprocessors just ignore the chapters list and apply to *all Markdown files* in the src dir. Usually, this is not an issue, but sometimes preprocessor may spend a long time on the files, which may not even get into the resulting build.

We suggest you keep your src dir tidy and only put there files that are actually getting into the project. The other solution is to use <link src="preprocessors/removeexcess.md">RemoveExcess</link> preprocessor, which removes all Markdown files, which are not mentioned in the chapters list, from the temporary directory.

### backend_config

*(mapping)*

Keep all your backend settings in `backend_config` section:

```yaml
backend_config:
    pandoc:
        template: !path template/docs.tex
        vars:
            title: *title
            subtitle: User’s Manual
            logo: !path template/octopus-black-512.png
        params:
            pdf_engine: xelatex
            listings: true
    mkdocs:
        use_title: true
        use_chapters: true
        use_headings: true
        mkdocs.yml:
            repo_name: foliant-docs/docs
            theme:
                name: material
                custom_dir: !path ./theme/
```

Unlike `preprocessors` section, `backend_config` is not a list but a mapping. Hence, the order in which you define backends is not important.

Moreover, you can even skip adding a backend into `backend_config` and still be able to build a project with it. It will just mean that you are using default settings.

## Modifiers

Foliant defines several custom YAML-modifiers, some of which you have already met in the examples above.

### !include

The `!include` modifier allows inserting content from another YAML-file.

For example, if your chapters list has grown so big, that you want to keep it separately from the main config, you can put it into `chapters.yml` file and include it in `foliant.yml`:

```yaml
chapters: !include chapters.yml
```

### !path, !project_path, !rel_path

When used in foliant.yml, `!path`, `!project_path`, `!rel_path` all do the same thing: they resolve the path to an absolute path to make sure the preprocessor or backend processes this file properly.

It is recommended, that whenever you supply a path to any file in options, to precede it with the `!path` modifier:

```yaml
preprocessors:
    - swaggerdoc:
        spec_path: !path swagger.yml
        environment:
            user_templates: !path widdershins_templates
    - plantuml:
        params:
            config: !path configs/plantuml.cfg

backend_config:
    pandoc:
        template: !path pandoc/tex_templates/main.tex
        reference_docx: !path pandoc/docx_references/basic.docx
```

Why there are three of them then, would you ask? The reason is that all *foliant tag options* in Markdown source files are in fact also YAML-strings, which means that you can supply a list in tag option like this:

```html
<jinja2 vars="[a1, a2, a3]">
Received the variables!

{% for var in vars %}
    I’ve got a var {{ var }}
{% endfor %}
</jinja2>
```

And that’s where `!project_path`, `!rel_path` modifiers come in really handy. Now you can refer to a file that is sitting in the project root, no matter where inside the src dir your current file is:

```html
Here are the contents of this project’s config:

<include src="!project_path foliant.yml"></include>
```

By convention, all tag parameters, which accept paths to external files, are considered to be paths relative to the current file. But if you want to make things more explicit, you may add the `!rel_path` tag, which ensures that the path the preprocessor will get, will be relative to the current file:

```html
Here are the contents of the adjacent chapter:

<include src="!rel_path chapter2.md"></include>
```

`!path` modifier, if used in tag parameters, works the same as `!project_path` modifier: it returns the absolute path to the file, relative to the project root.

### `!env`

The `!env` modifier allows you to access environment variables in the config, as well as in tag options.

It is useful if you don’t want to keep credentials in your config files, for example:

```yaml
# foliant.yml

preprocessors:
    dbdoc:
        host: localhost
        user: admin
        password: !env DBA_PASSWORD
```

Now to build this project add the variable to your command:

```bash
DBA_PASSWORD=WQHsaio901SY foliant make pdf
```

Or, if you are using docker:

```bash
docker-compose run --rm -e DBA_PASSWORD=WQHsaio901SY foliant make pdf
```
