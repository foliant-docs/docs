# Quickstart

If you don't have Foliant installed, please follow <link src="installation.md">the instructions</link> first.

**Step 1.** Create a new project

```bash
$ foliant init
```

Or with Docker

```bash
$ docker run --rm -it --user $(id -u):$(id -g) -v $(pwd):/usr/src/app -w /usr/src/app foliant/foliant init
```

**Step 2.** cd into the folder created by command

```bash
$ cd my-project
```

**Step 3.** Edit the Markdown source of your documentation located in `src/index.md`.

To build a static site with [MkDocs](https://www.mkdocs.org/), install the <link src="backends/mkdocs.md">MkDocs backend</link> (skip this step if you are using Docker)

```bash
pip3 install foliantcontrib.mkdocs
```

**Step 4.** Build the site with `foliant make` command

```bash
$ foliant make site
```

Or with Docker

```bash
$ docker-compose run --rm foliant make site
```

**Done!** Your site is generated in the `My_Project-2020-05-25.mkdocs` folder, crank up a webserver to take a look at it

```bash
$ python3 -m http.server -d My_Project-2020-05-25.mkdocs
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Now let's build a DOCX out of the same source. You will need [Pandoc](https://pandoc.org/) and <link src="backends/pandoc.md">Pandoc backend</link> for that, the instructions for installing them are in <link src="installation.md">the installation guide</link>.

**Step 5.** Build docx

```bash
$ foliant make docx
```

With Docker you will need to adjust the Dockerfile first, replace the first line with the following

```diff
- FROM foliant/foliant
+ FROM foliant/foliant:pandoc
```

and rebuild the image

```bash
$ docker-compose build
```

Finally, run the make command inside the container

```bash
$ docker-compose run --rm foliant make docx
```

**Done!** The `My_Project-2020-05-25.docx` is created in the project dir.

***

If you want to know more about how Foliant works, check out the <link src="architecture.md"></link> or just dive straight into <link src="tutorials/first_project.md"></link>.
