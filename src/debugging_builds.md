# Debugging Builds

Building simple documentation projects with Foliant is usually straightforward. But Foliant is a powerful, customizable, and very flexible tool, capable of turning your most complex ideas into beautiful documents. If you understand exactly what you want to achieve, you can formalize it at the project config level, and Foliant will perform your task efficiently and precisely.

But sometimes it is difficult to configure all preprocessors and backends properly in one go. Some settings are pretty subtle and some preprocessors are quite complicated. The order of applying the preprocessors matters. Some preprocessors may work unexpectedly when paired with others. Fetching data from external sources may also become a bottleneck. The list goes on.

Fortunately, Foliant will not ask you to diagnose problems with the car engine without opening the hood. Foliant provides advanced diagnostic facilities such as:

* detailed event logging in <link anchor="debug_mode">the debug mode</link>;
* <link title="The pre backend">the pre backend</link> which does nothing, i.e. just returns the preprocessed Markdown;
* an option to <link title="Keeping the project sources">keep the temporary working directory</link> for further analysis.

## Notes on Docker Use

In practice, Foliant is more commonly <link src="tutorials/docker.md">used with Docker</link>.

Here's a tip for debugging Foliant projects with docker.

It’s useful to add one more service to your project's default `docker-compose.yml`. We will call it `bash` and it will run containers with an interactive shell:

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

Now you can run a container based on the project’s image with an interactive shell. To open the shell for root, run:

```bash
$ docker-compose run --rm bash
```

To open shell for a user with the same user ID and group ID as your current user on the host machine:

```bash
$ docker-compose run --user="$(id -u):$(id -g)" --rm bash
```

All debugging approaches which we will discuss next are represented as native Foliant commands, but they are applicable for the Docker way too. Just start your commands with `docker-compose run --rm ` or `docker-compose run --user="$(id -u):$(id -g)" --rm ` to run them within Docker containers.

## Logging

The `foliant make ...` command runs Foliant in the regular logging mode. In this mode, Foliant and its extensions will only log events with levels of `critical`, `error`, and `warning`. Note that some preprocessors may generate a lot of specific warnings which may or may not indicate that something went wrong. These messages are usually worth studying anyway though.

The new log file is created for each build, unless there were no errors and warnings. The logs are stored by default in the project root under the name `<unix timestamp of the build>.log`, for example, `1628582527.log`. With such a naming convention the log file for the latest build will always be last in alphabetical order. The location of the log files may be customized by the `--logs|-l` command-line option.  

**Debugging Mode**
<anchor>debug_mode</anchor>

Foliant provides the `--debug` or `-d` command-line option which enables the debugging mode. In this mode, Foliant and its extensions log not just events with levels `critical`, `error` and `warning`, but also events with levels `debug` and `info`. The amount of information you will get from such events depends on the implementation of a particular extension. Complex preprocessors like Includes usually log their actions in great detail. The messages of the `info` level are usually informative: they may mark the beginning or end of some preprocessor’s work, for example. The messages of the `debug` level generally show the status of atomic operations, for example, reading data from a certain file. These messages often contain the values of the variables which are important in the current context: paths to files, external commands that are called, etc. But to make sense of these values prepare to get your hands dirty, or, in other words, read and understand the code of the corresponding extension.

Here's an example of a command that tells Foliant to build PDF with Pandoc in debug mode:

```bash
$ foliant make pdf --with pandoc --debug
```

Each log is a text file that contains a number of lines (records). Each record represents a single event and consists of 4 separate fields:

* date and time of the event registration;
* context (module name) in which the event was registered;
* event log level: one of `CRITICAL`, `ERROR`, `WARNING`, `DEBUG`, `INFO`;
* message text that explains the essence of the event.

For example, the first record of a log usually looks like that:

```
2020-06-25 09:40:54,419 |                  flt |     INFO | Build started.
```

The string `flt` in the second field means Foliant itself (Foliant Core).

The context is hierarchical. The following record represents an event that is registered in the Includes preprocessor which was implicitly called by the Flatten preprocessor, which was implicitly called during project build by Pandoc backend.

```
2020-06-25 09:40:54,678 | flt.pandoc.flatten.includes |    DEBUG | Processing Markdown file: /usr/src/app/__folianttmp__/__all__.md
```

In the next example, Pandoc backend logs the external command that is called to build needed target:

```
2020-06-25 09:40:54,684 |           flt.pandoc |    DEBUG | PDF generation command: pandoc --template="/foliant_stuff/pandoc_templates/tex_templates/main.tex" --output "My_Awesome_Project-1.0-2020-06-25.pdf" --variable title="My Awesome Project" --variable version="1.0" --variable subtitle="Description Of My Awesome Project" --variable logo="/foliant_stuff/pandoc_templates/logos/logo.png" --variable year="2020" --variable title_page --variable toc --variable tof  --pdf-engine=xelatex --listings -f markdown __folianttmp__/__all__.md
```

If you suspect that the command executes wrong, you can run it directly in an interactive shell and study the results.

Detailed logging in debug mode allows you to quickly localize problems zooming in from Foliant itself to a specific Foliant extension, a specific Markdown source file, or a specific line of code. This takes effort but with practice allows one to solve complex problems in minimal time.

## Debugging extensions

Each Foliant backend takes preprocessed Markdown content and passes it to an external command (see <link src="architecture.md"></link>). For debugging backends it's essential to see the content which the backend actually gets. 

During the build source files of Foliant project are copied to a temporary working directory. By default, it is called `__folianttmp__/` and located in the “root” directory of the project. Source Markdown files of the project are kept unchanged during the build. Any transformations are applied only to the files located in the temporary working directory.

### The pre backend

Foliant Core provides the built-in backend `pre` which does nothing. More precisely, this backend makes the `pre` target. The `pre` target is obtained simply by copying the temporary working directory to a subdirectory inside the project root as the result of the build.

The `pre` target is essentially the content that comes after all preprocessors are applied, but before any backend (other than `pre`) is called.

**Determining the cause of the problem**

`pre` backend is convenient to determine the stage of a build which causes problems:

* configuration stage (reading the configuration file),
* preprocessing stage (transforming the Markdown content with extensions)
* or backend stage (producing the output format).

If you build a `pre` target and the results seem fine, then there is a problem with your *backend* and you have to debug that. You may start with the <link title="Keeping the project sources">keep-tmp</link> which we will discuss next.

If the problem persists in the results, produced by `pre`, then it's one of the preprocessors causing trouble or the configuration parser not working properly. In this case, it's a nice idea to stick with the `pre` target during your experiments so you won't need to wait each time for the backend to complete producing the target while you are debugging the build.

To build a Foliant project to the `pre` target, run the command:

```bash
$ foliant make pre
```

### Keeping the project sources

In addition to the `pre` backend, Foliant Core supports the `--keep_tmp` or `-k` command-line option. By default, the temporary working directory (`__folianttmp__`) is removed after the project build. But if the `--keep-tmp` or `-k` option is specified, the temporary working directory will stay in the project root after build.

This directory will contain the files that are modified by all preprocessors and the chosen backend.

If you have determined that the backend causes issues, <link title="The pre backend">pre</link> won't help you anymore. Run the build with `-k` argument and study the working dir. If it seems fine, then the problem may be with the command that the backend runs to convert Markdown to a target format. Time to <link title="Logging">study logs</link>!

The following command tells Foliant to build PDF with Pandoc, keeping the temporary working directory after build:

```bash
$ foliant make pdf --with pandoc --keep_tmp
```

<!-- **Tip:** if Pandoc doesn’t make PDFs due to errors in LaTeX markup, you can build the target `tex` and then debug the LaTeX source. Also, you may call Pandoc directly from the command line to build a PDF from LaTeX source. -->

## Killing Two Birds With One Stone

Now you know what debugging facilities are provided by Foliant. But we strongly recommend you make it a rule to start debugging Foliant projects with one universal shell command:

```bash
$ foliant make pre --debug
```

This command tells Foliant to build the `pre` target in the debug mode. And this is a very effective way to get closer to understanding what is wrong with your project.
