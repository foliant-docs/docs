# Documenting API with Foliant

In this tutorial we will show you a way to document your database using Foliant. Right now there are options available for those of you who use **PostgreSQL**, **Oracle** and **DBML**.

## DBML

Quote from the official website: *[DBML](https://dbml.org) (Database Markup Language) is an open-source DSL language designed to define and document database schemas and structures. It is designed to be simple, consistent and highly-readable.* And that makes it a perfect choice for designing your database. You can create your table structure without messing with cumbersome SQL in a more human-readible way like this:

```
Table users {
  id integer
  username varchar
  role varchar
  created_at timestamp
}

Table posts {
  id integer [primary key]
  title varchar
  body text [note: 'Content of the post']
  user_id integer
  status post_status
  created_at timestamp
}

Enum post_status {
  draft
  published
  private [note: 'visible via URL only']
}

Ref: posts.user_id > users.id // many-to-one
```

As you may have noticed, DBML also has tools to document pieces of you schema using *notes* (`body text [note: 'Content of the post']`) and comments (`Ref: posts.user_id > users.id // many-to-one`).

So how can we convert DBML schema descriptions into a human-readible document? Well, the idea is pretty simple: we parse the DBML definitions and pass them to a Jinja template, which renders markdown for us.

You won't need to do it all manually, of course, we have a preprocessor for that.

### DBMLDoc preprocessor

[DBMLDoc](https://foliant-docs.github.io/docs/preprocessors/dbmldoc/) preprocessor for Foliant accepts a DBML schema file on the input and outputs the Markdown documentation based on it.

In this tutorial we will use [this sample schema](https://github.com/holistics/dbml/blob/master/packages/dbml-core/__tests__/parser/dbml-parse/input/general_schema.in.dbml) from the official repo for practicing. But first, let's make sure we have all the required packages installed.

#### Installing prerequisites

First, you will need Foliant, of course. If you don't have it yet, please, refer to the [installation guide](https://foliant-docs.github.io/docs/installation/).

Next, let's install [Foliant Init](https://github.com/foliant-docs/foliantcontrib.init/) to facilitate the task of creating new project:

```bash
$ pip3 install foliantcontrib.init
```

If you are going to use our full Docker image, you don't need to install anything else. If you will run Foliant natively, install preprocessors DBMLDoc, PlantUML and the Slate backend:

```bash
$ pip3 install foliantcontrib.dbmldoc foliantcontrib.slate, foliantcontrib.plantuml
```

We are going to use [Slate](https://github.com/slatedocs/slate) backend for building a static website with documentation, so you will need to [install Slate dependencies](https://github.com/slatedocs/slate/wiki/Using-Slate-Natively).

Finally, [install PlantUML](https://plantuml.com/ru/starting), we will use it to draw database scheme.

#### Creating project

Let's create a Foliant project for our experiments. `cd` to the directory where you want your project created and run the `init` command:

```bash
$ cd ~/foliant_projects
$ foliant init
Enter the project name: Database Docs
Generating project... Done
────────────────────
Project "Database Docs" created in database-docs

$ cd database-docs
```

Next, let's download the sample DBML spec and save it into file `schema.dbml`:

```bash
$ wget https://raw.githubusercontent.com/holistics/dbml/master/packages/dbml-core/__tests__/parser/dbml-parse/input/general_schema.in.dbml -O schema.dbml
```

Now it's time to set up our config. Open `foliant.yml` and add the following lines:

```diff
title: Database Docs

chapters:
  - index.md

+preprocessors:
+  - dbmldoc:
+      spec_path: schema.dbml
+  - plantuml
+
```

We've added the plantuml DBMLDoc preprocessor to pipeline and specified path to our DBML sample schema in the comments.

> Note: if plantuml is not available under `$ plantuml` in your system, you will also need to specify path to platnum.jar in settings like this:
> ```yaml
>   - plantuml:
>       plantuml_path: /usr/bin/plantuml.jar
> ```


Finally, we need to tell Foliant where to put the generated documentation. Since we already have an `index.md` chapter created for us by `init` command, let's put it in there. Open the `src/index.md` file and make it look like this:

```diff
# Welcome to Database Docs

-Your content goes here.
+<dbmldoc></dbmldoc>
+
```

All preparations done, let's build our site:

```
$ foliant make site -w slate
Parsing config... Done
Applying preprocessor dbmldoc... Done
Applying preprocessor mkdocs... Done
Applying preprocessor _unescape... Done
Making site with MkDocs... Done
────────────────────
Result: Database_Docs-2020-06-03.mkdocs
```

Now open `Database_Docs-2020-06-03.mkdocs/index.html` and look what you've got:




Generally we want to keep our docs as close as possible to the code. When documenting source code we usually can utilize the power of Swagger to generate docs from comments in the sources. Since there's no Swagger for databases, we had to invent something similar.

We are going to document our database using *comments*. Comment (not to be confused with SQL script `--comments`) is a special entity which in one way or another is present in most DBMS's. We use comments to describe tables, columns and often many more entities (in PostgreSQL you can describe almost anything with a comment). The syntax of adding a comment may be following:

```sql
COMMENT ON TABLE "Clients" IS "Table holding info about the clients"
```

These comments don't affect data or table structure in any way, they are only present for documentation purposes.

The general idea is that you supply a specification file path (`json` or `yaml` for OpenAPI, `raml` for RAML) to a preprocessor which will generate a Markdown document out of it. Markdown is what Foliant is good at, so after that you can do anything with it: convert to PDF, partially include in other documents, etc. In this guide we will concentrate on building a static website for your API documentation.

> Please note that in this article we cover only the basic usage of the tools. For detailed information on features and customizing output refer to each component's doc page.

## OpenAPI

### Installing prerequisites

Besides Foliant you will need to install some additional packages on your system. If you are using our full docker image `foliant/foliant:full`, you can skip this chapter.

First, install the [SwaggerDoc](https://foliant-docs.github.io/docs/preprocessors/swaggerdoc/) preprocessor which will convert spec file to Markdown.

```shell
pip3 install foliantcontrib.swaggerdoc
```

SwaggerDoc preprocessor uses [Widdershins](https://github.com/Mermade/widdershins) under the hood, so you will need to install that too.

```shell
npm install -g widdershins
```

Finally, to build the static website we will be using [Slate backend](https://foliant-docs.github.io/docs/backends/slate/):

```shell
pip3 install foliantcontrib.slate
```

Also note that Slate requires [Ruby](https://www.ruby-lang.org/en/) and [Bundler](https://bundler.io/) to work (that's a lot of dependencies, I know).

### Creating project

Let's create Foliant project. The easiest way is to use [foliant init](https://github.com/foliant-docs/foliantcontrib.init/) command. After running the command Foliant will ask you about your project name. We've chosen "OpenAPI docs", but it may be anything:

```shell
cd ~/projects
foliant init
    Enter the project name: OpenAPI docs
    Generating project... Done
    ────────────────────
    Project "OpenAPI docs" created in openapi-docs
```

In the output Foliant informs us that the project was created in a new folder `openapi-docs`. Let's copy your OpenAPI spec file into this folder:

```shell
cp ~/Downloads/my_api.yaml ~/projects/openapi-docs
```

In the end you should get the following directory structure:

```
└── openapi-docs
    ├── Dockerfile
    ├── README.md
    ├── docker-compose.yml
    ├── foliant.yml
    ├── my_api.yaml
    ├── requirements.txt
    └── src
        └── index.md
```

If you wish to use Docker with full Foliant image, which is the recommended way to build Foliant projects, then open generated `Dockerfile` and replace its contents with the following line:

```Dockerfile
FROM foliant/foliant:full
```

### Configuring project

Now let's set up `foliant.yml`. Right now it looks like this:

```yaml
title: OpenAPI docs

chapters:
  - index.md

```

First add and fill up the `preprocessors` section at the bottom:

```yaml
preprocessors:
    - swaggerdoc:
        spec_path: !path my_api.yaml # path to your API spec file, relative to project root
```

At this stage you may also specify path to custom templates dir in `environment: {user_tempaltes: path/to/custom/templates}` parameter. Templates describe the exact way of how to convert structured specification file into a Markdown document. For this tutorial we will be using default templates because they are perfect for our static site. Check [Widdershins docs](https://github.com/Mermade/widdershins#templates) for detailed info on templates.

The last thing we need to do is point Foliant where to insert the generated Markdown from the spec file. We already have a source file created for us by `init` command, called `index.md`, so let's use it to store our API docs.

Open `openapi-docs/src/index.md` with text editor and replace its contents with the following:

```html
<swaggerdoc></swaggerdoc>
```

Foliant will insert generated markdown on the place of this tag during build. You may even add some kind of introduction for the API docs before the tag, if you don't have such inside your spec file.

That's it! All is left to do is run `make` command to build your site.

```shell
foliant make site --with slate
    Parsing config... Done
    Applying preprocessor swaggerdoc... Done
    Applying preprocessor slate... Done
    Applying preprocessor _unescape... Done
    Making site...
...
    Done
    ────────────────────
    Result: OpenAPI_docs-2019-11-29.slate/
```

If you use docker, the command is:

```shell
docker-compose run --rm foliant make site --with slate
```

Now if you open the `index.html` from just created `OpenAPI_docs-2019-11-29.slate` folder, you should see something like this:

![Slate static site](img/slate.png)

You can customize the page styles, add or remove language example tabs and tune other options. Check the [Slate backend](https://foliant-docs.github.io/docs/backends/slate/) documentation for details.

## RAML

Building API docs from RAML specification is quite similar to that of OpenAPI, the difference is that instead of `swaggerdoc` preprocessor you use `ramldoc`. We will go through all the steps anyway.

### Installing prerequisites

Besides Foliant you will need to install some additional packages on your system. If you are using our full docker image `foliant/foliant:full`, you can skip this chapter.

First, install the [RAMLdoc](https://foliant-docs.github.io/docs/preprocessors/ramldoc/) preprocessor which will convert spec file to Markdown.

```shell
pip3 install foliantcontrib.ramldoc
```

RAMLdoc preprocessor uses [raml2html](https://github.com/raml2html/raml2html) with [full-markdown-theme](https://github.com/Vanderhoof/raml2html-full-markdown-theme) under the hood, so you will need to install those too.

```shell
npm install -g raml2html raml2html-full-markdown-theme
```

Finally, to build the static website we will be using [Slate backend](https://foliant-docs.github.io/docs/backends/slate/). If you don't have it, run:

```shell
pip3 install foliantcontrib.slate
```

Also note that Slate requires [Ruby](https://www.ruby-lang.org/en/) and [Bundler](https://bundler.io/) to work.

### Creating project

Let's create Foliant project. The easiest way is to use [foliant init](https://github.com/foliant-docs/foliantcontrib.init/) command. After running the command Foliant will ask you about your project name. We've chosen "API docs", but it may be anything:

```shell
cd ~/projects
foliant init
    Enter the project name: API docs
    Generating project... Done
    ────────────────────
    Project "API docs" created in api-docs
```

In the output Foliant informs us that the project was created in a new folder `api-docs`. Now let's copy your RAML spec file to this folder:

```shell
cp ~/Downloads/my_api.raml ~/projects/api-docs
```

In the end you should get the following directory structure:

```
└── api-docs
    ├── Dockerfile
    ├── README.md
    ├── docker-compose.yml
    ├── foliant.yml
    ├── my_api.raml
    ├── requirements.txt
    └── src
        └── index.md
```

If you wish to use Docker with full Foliant image, which is the recommended way to build Foliant projects, then open generated `Dockerfile` and replace its contents with the following line:

```Dockerfile
FROM foliant/foliant:full
```

### Configuring project

Now let's set up `foliant.yml`. Right now it looks like this:

```yaml
title: API docs

chapters:
  - index.md

```

First add and fill up the `preprocessors` section at the bottom:

```yaml
preprocessors:
    - ramldoc:
        spec_path: !path my_api.yaml # path to your API spec file, relative to project root
```

At this stage you may also specify path to custom templates dir in the `template_dir` parameter. Templates describe the exact way of how to convert structured specification file into a Markdown document. raml2html uses [Nunjucks](https://mozilla.github.io/nunjucks/) templates, which are stored in the theme. So the easiest way to create your own templates is to copy [default](https://github.com/Vanderhoof/raml2html-full-markdown-theme/tree/master/templates) ones and adjust them to your needs. But we will use the default template which works great with Slate.

The last thing we need to do is point Foliant where to insert the generated Markdown from the spec file. We already have a source file created for us by `init` command, called `index.md`, so let's use it to store our API docs.

Open `api-docs/src/index.md` with text editor and replace its contents with the following:

```html
<ramldoc></ramldoc>
```

Foliant will insert generated markdown on the place of this tag during build. You may even add some kind of introduction for the API docs before the tag, if you don't have such in your spec file.

That's it! All is left to do is run `make` command to build your site.

```shell
foliant make site --with slate
    Parsing config... Done
    Applying preprocessor ramldoc... Done
    Applying preprocessor slate... Done
    Applying preprocessor _unescape... Done
    Making site...
...
    Project built successfully.

    Done
    ────────────────────
    Result: API_docs-2019-11-29.slate/
```

If you use docker, the command is:

```shell
docker-compose run --rm foliant make site --with slate
```

Now if you open the `index.html` from just created `API_docs-2019-11-29.slate` folder, you should see something like this:

![Slate static site](img/slate_raml.png)

You can customize the page styles, add or remove language example tabs and tune other options. Check the [Slate backend](https://foliant-docs.github.io/docs/backends/slate/) documentation for details.

## Blueprint

API Blueprint is a Markdown-based API specification format. That's why the build process differs from that of OpenAPI or RAML: we skip the converting step and just add the specification file as a source.

### Installing prerequisites

To build a static site we will use [Aglio](https://foliant-docs.github.io/docs/backends/aglio/) backend which is designed specifically for rendering API Blueprint. So first install the backend and [Aglio renderer](https://github.com/danielgtaylor/aglio) itself:

```shell
pip3 install foliantcontrib.aglio
npm install -g aglio
```

### Creating project

Let's create a Foliant project. The easiest way is to use [foliant init](https://github.com/foliant-docs/foliantcontrib.init/) command. After running the command Foliant will ask you about your project name. We've chosen "API docs", but it may be anything:

```shell
cd ~/projects
foliant init
    Enter the project name: API docs
    Generating project... Done
    ────────────────────
    Project "API docs" created in api-docs
```

In the output Foliant informs us that the project was created in a new folder `api-docs`. Now copy your Blueprint spec file into the `src` subfolder (it's better to change the extension to `.md` too), replacing "index.md":

```shell
cp ~/Downloads/spec.abip ~/projects/api-docs/src/index.md
```

In the end you should get the following directory structure:

```
└── openapi-docs
    ├── Dockerfile
    ├── README.md
    ├── docker-compose.yml
    ├── foliant.yml
    ├── requirements.txt
    └── src
        └── index.md
```

If you wish to use Docker with full Foliant image, which is the recommended way to build Foliant projects, then open generated `Dockerfile` and replace its contents with the following line:

```Dockerfile
FROM foliant/foliant:full
```

### Configuring project

Now check your `foliant.yml`. Right now it looks like this:

```yaml
title: API docs

chapters:
  - index.md  # this should be your API Blueprint specification

```

It may be hard to believe, but no other configuration is required! Let's build our project:

```shell
foliant make site --with aglio
    Parsing config... Done
    Applying preprocessor flatten... Done
    Applying preprocessor _unescape... Done
    Making site... Done
    ────────────────────
    Result: OpenAPI_docs-2019-11-29.aglio
```

If you use docker, the command is:

```shell
docker-compose run --rm foliant make site --with aglio
```

Now if you open the `index.html` from just created `API_docs-2019-11-29.aglio` folder, you should see something like this:

![Aglio static site](img/aglio1.png)

It's not near as attractive as the Slate site we had in previous examples. But don't worry, Aglio supports styling with CSS and layout control with [Jade](http://jade-lang.com/) templates. It also has several built-in themes, which look much better than the default one.

Open your `foliant.yml` again and add following lines at the end:

```yaml
backend_config:
    aglio:
        params:
            theme-variables: streak
            theme-template: triple
```

Now run the same build command:

```shell
foliant make site --with aglio
```

And look at the result:

![Aglio more beautiful static site](img/aglio2.png)

Much better!
