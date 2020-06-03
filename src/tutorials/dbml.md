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


Finally, we need to tell Foliant where to insert the generated documentation. Since we already have an `index.md` chapter created for us by `init` command, let's put it in there. Open the `src/index.md` file and make it look like this:

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
Applying preprocessor plantuml... Done
Applying preprocessor flatten... Done
Applying preprocessor _unescape... Done
Making site... Done
...
────────────────────
Result: Database_Docs-2020-06-03.slate/
```

Now open `Database_Docs-2020-06-03.slate/index.html` and look what you've got:

![](img/dbml.png)
