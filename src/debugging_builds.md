# Debugging Builds

Building simple documentation projects with Foliant is usually straightforward. But Foliant and its extensions are designed to build complex projects. Foliant with its extensions is a powerful, customizable, and very flexible tool. If you understand what do you exactly want to do, and you can formalize it at the project config level, Foliant will perform your task efficiently and accurately.

The problem is that it sometimes may be difficult to configure all preprocessors and backends correctly for the first time. Some settings are pretty subtle and unobvious. The order of applying the preprocessors matters. Some preprocessors may work unexpectedly when paired with others. It may be necessary to apply some preprocessor twice — before and after some another preprocessor. Fetching data from external sources may also become a bottleneck.

Fortunately, Foliant will not ask you to diagnose problems with the car engine without opening the hood. Foliant provides advanced diagnostic facilities such as the following:

* detailed event logging in the special debug mode;
* the backend `pre` that does nothing, i.e. does not convert preprocessed Markdown into any target;
* keeping the temporary working directory that is used during builds, for further analysis.

## Notes on Docker Use

In this documentation, examples of shell commands in most cases represent the native use of Foliant, i.e. as an applicaion that is installed directly on your local operating system.

In practice, Foliant is more commonly used with Docker. Public images of Foliant with different sets of extensions are available [at Docker Hub](https://hub.docker.com/r/foliant/foliant). The actual tags are:

* `slim` — minimal image of Foliant with no extensions;
* `latest` — same as `slim` but with the `foliant init` command support;
* `pandoc` — image of Foliant with Pandoc backend, Pandoc itself, and LaTeX (`texlive-full` Ubuntu package);
* `full` — most complete image of Foliant with all released extensions and dependencies required for them.

You can find Dockerfiles for each image in [this GitHub repository](https://github.com/foliant-docs/docker/).

If you need to build PDFs with Pandoc/LaTeX, and some additional functionality is required, e.g. drawing diagrams with PlantUML, it’s recommended to use the image with the tag `full`. This image is widely used, frequently updated, and well supported.

To get or update the Foliant Docker image, run the command:

```bash
$ docker pull foliant/foliant:full
```

It’s not necessary but recommended to use [Docker Compose](https://docs.docker.com/compose/) utility to build images for Foliant projects and run containers based on them.

To prepare your Foliant project to be built within Docker container, add the files `Dockerfile` and `docker-compose.yml` into the project’s “root” directory.

If you decided to use the image with the `full` tag, the content of `Dockerfile` should be the following:

```
FROM foliant/foliant:full
```

The file `docker-compose.yml` should define the `foliant` service in this way:

```yaml
version: '3'

services:
  foliant:
    build:
      context: ./
      dockerfile: ./Dockerfile
    volumes:
      - ./:/usr/src/app/
```

For debug purposes, it’s useful to add one more service, `bash`, to run containers with an interactive shell. Recommended full content of the file `docker-compose.yml` is the following:

```yml
version: '3'

services:
  foliant:
    build:
      context: ./
      dockerfile: ./Dockerfile
    volumes:
      - ./:/usr/src/app/
  bash:
    build:
      context: ./
      dockerfile: ./Dockerfile
    volumes:
      - ./:/usr/src/app/
    entrypoint: /bin/bash
```

To run Foliant project build, first, you have to build an image for the project:

```bash
$ docker-compose build
```

This command should be executed in the project’s “root” directory.

The image for the project should be rebuilt after pulling a newer version of Foliant image. Also, if you use a custom `Dockerfile`, and some local files are mentioned in it, the image for the project should be rebuilt after any updates of these files.

To perform the project build with Foliant, run the command like the following:

```bash
$ docker-compose run --rm foliant make <target> --with <backend>
```

You need to specify the certain target and the certain backend. So, if you want to build PDF with Pandoc, the command should be:

```bash
$ docker-compose run --rm foliant make pdf --with pandoc
```

This command corresponds to the native command:

```bash
$ foliant make pdf --with pandoc
```

Note that by default the commands within Docker containers run as root. So, after running a container, you can get some files owned by root in your local file system. To avoid this, run commands inside Docker containers as a user with the user ID and group ID of your local user on the host machine:

```bash
$ docker-compose run --user="$(id -u):$(id -g)" --rm foliant make pdf --with pandoc
```

If you described the service `bash` in your `docker-compose.yml` file, you may run a container based on the project’s image, with an interactive shell. To open shell for root, run:

```bash
$ docker-compose run --rm bash
```

To open shell for a user with the same user ID and group ID as your current user on the host machine:

```bash
$ docker-compose run --user="$(id -u):$(id -g)" --rm bash
```

All examples below will represent native commands. Add `docker-compose run --rm ` or `docker-compose run --user="$(id -u):$(id -g)" --rm ` to their beginnings to run the corresponding commands within Docker containers.

## Running Foliant in the Debug Mode

The usual command to build some target with some backend, `foliant make <target> --with <backend>`, runs Foliant in the regular mode. In this mode, Foliant and its extensions will log only events of the levels `critical`, `error`, and `warning`. Normally, there should not be critical cases (i.e. fatal crashes) and errors (they mean, for example, fails of some preprocessors, unavailability of some external services, etc.). Warnings are almost acceptable. Some preprocessors generate large numbers of specific warnings, and such behavior doesn’t mean that something is wrong.

Foliant provides the `--debug` or `-d` command line option that enables the debug mode. In the debug mode, Foliant and its extensions also log events of the levels `debug` and `info`. How detailed the logging of such events is, depends on the implementation of a particular extension. Complex preprocessors like Includes usually log their actions in great detail. Usually the messages of the `info` level are informative: they indicate, for example, the beginning of some preprocessor’s work. The messages with the `debug` level generally show the status of atomic operations, e.g. reading data from the certain file. These messages often contain the values of variables that are important in this context: paths to files, external commands that are called, etc.

The variant of the command that tells Foliant to build PDF with Pandoc in the debug mode, looks like:

```bash
$ foliant make pdf --with pandoc --debug
```

Log files will be written to the project’s “root” directory with names `*.log`, where `*` represents UNIX timestamps of the moments when the certain files were created. We were asked to implement optional overriding of logs location—it will be done in future release of Foliant Core. But it’s so easy to move logs to any custom place with a simple shell script; fixed location of logs is a really imaginary inconvenience!

Each log is a text file that contains a number of lines (records). Each record represents a single event and consists of 4 separated fields:

* date and time of registration of the event;
* context (“place,” module) that the event is registered in;
* event log level: one of `CRITICAL`, `ERROR`, `WARNING`, `DEBUG`, `INFO`;
* message text that explains the essence of the event.

For example, the first record of a log is usually looks like that:

```
2020-06-25 09:40:54,419 |                  flt |     INFO | Build started.
```

The string `flt` in the second field means Foliant itself, i.e. Foliant Core.

The context is hierarchical. The following record represents an event that is registered in the preprocessor Includes that is called from the preprocessor Flatten, that is called during project build with Pandoc backend.

```
2020-06-25 09:40:54,678 | flt.pandoc.flatten.includes |    DEBUG | Processing Markdown file: /usr/src/app/__folianttmp__/__all__.md
```

In this example, the message text contains the path to the file that is currently processed.

In the next example, Pandoc backend logs the external command that is called to build needed target:

```
2020-06-25 09:40:54,684 |           flt.pandoc |    DEBUG | PDF generation command: pandoc --template="/foliant_stuff/pandoc_templates/tex_templates/main.tex" --output "My_Awesome_Project-1.0-2020-06-25.pdf" --variable title="My Awesome Project" --variable version="1.0" --variable subtitle="Description Of My Awesome Project" --variable logo="/foliant_stuff/pandoc_templates/logos/logo.png" --variable year="2020" --variable title_page --variable toc --variable tof  --pdf-engine=xelatex --listings -f markdown __folianttmp__/__all__.md
```

If you suspect that the command executes wrong, you can copy it and try to run directly in an interactive shell.

Detailed logging in the debug mode allows you to quickly localize problems accurate to a specific Foliant extension, specific source Markdown file, specific syntax construction, and solve them with minimal time.

## The `pre` Backend and Target, and Keeping the Working Directory

Every Foliant backend takes preprocessed Markdown content and passes it to an external command. So it would be nice to see what content the backend gets, and how the backend additionally modifies this content.

During build, source files of Foliant project are moved to the temporary working directory. By default, it is called `__folianttmp__/` and located in the “root” directory of the project. Source files of the project are kept unchanged. Any transformations are applied only to the files located in the temporary working directory.

Foliant Core provides the built-in backend `pre` that does nothing. More precisely, this backend makes the `pre` target. The `pre` target is obtained simply by copying the temporary working directory to the project directory as the result of build.

The `pre` target is the content that comes after all preprocessors are applied, but before any backend of another kind than `pre` is called.

If the backend is not the location of your problem, and your problem occurs in an extension of another type (most often in a preprocessor), you don’t need to wait each time when the backend will complete to make the requested target.

The `pre` target is convenient for debugging extensions of all types excluding backends, and especially preprocessors.

To build Foliant project to the `pre` target, run the command:

```bash
$ foliant make pre
```

In additional to the `pre` backend and target, Foliant Core supports the `--keep_tmp` or `-k` command line option. By default, the temporary working directory is removed after the project build. But if the `--keep_tmp` or `-k` option is specified, the temporary working directory will be kept after build.

After the project build, this directory will contain the files that are modified by all preprocessors and the chosen backend.

The following command tells Foliant to build PDF with Pandoc, keeping the temporary working directory after build:

```bash
$ foliant make pdf --with pandoc --keep_tmp
```

Tip: if Pandoc doesn’t make PDF due to errors in LaTeX markup, you can build the target `tex` and first debug the LaTeX source. Also, you may call Pandoc directly from the command line to build PDF from LaTeX source.

## Killing Two Birds With One Stone

Now you know what debugging facilities are provided by Foliant. But we strongly recommend you to make it a rule to start debugging Foliant projects with one universal shell command:

```bash
$ foliant make pre --debug
```

This command tells Foliant to build the `pre` target in the debug mode. And this is a very effective way to get closer to understanding what is wrong with your project.
