# Running Foliant in Docker

Foliant's design philosophy says that everybody should do what they do best. We don't aim to create a universal text processing combine which covers all needs of a technical writer by itself. Instead Foliant introduces integrations with different beautiful open source tools which specialize on a little chunk of work and do it perfectly.

This approach comes with a disadvantage that you have to install each one of the tools you are using in the project for Foliant to work. You may do it once on your machine but Foliant projects usually need to work as well for other people if they decide to clone your project's repository. And these people may not have the right tool installed, or they may have another version of it, or even an operating system which doesn't have the tool at all.

Docker solves this problem by creating a virtual environment which will be consistent among different machines and even different operating systems. All the required tools will be installed and configured inside this virtual environment so all it's left to do is to run the build.

Working with Docker may seem complicated for non-programmers, but we will try to make it simple. We will concentrate on practical examples and keep the technical details out of this tutorial. If you want them — check the [Docker documentation](https://docs.docker.com/).

## Getting Docker

The first step is to download and install Docker.

**Windows**

Go to [https://www.docker.com/get-started](https://www.docker.com/get-started) and download Docker installer.

Follow the instructions of the installer. In the end, it may ask you to restart the computer. After restarting, run Docker by the shortcut in your Start menu.

You can also use Docker within [WSL](https://learn.microsoft.com/windows/wsl/install).

**Linux**

Follow the [instructions](https://docs.docker.com/engine/install/) for your Linux distribution on the official website.

After that [install Docker Compose](https://docs.docker.com/compose/install/#install-compose-on-linux-systems).

**MacOS**

Download and install Docker from [this page](https://hub.docker.com/editions/community/docker-ce-desktop-mac/).

## Creating a Test Project

Now that we’ve got Docker, we can create our test project.

If you have Foliant <link src="!path src/installation.md">installed</link> on your system, run the `init` command

```bash
$ foliant init
```

Type the name of the project and `cd` into the freshly created folder.

But that's the beauty of the Docker way, you don't even need to have Foliant installed on you computer to build Foliant projects. Instead of using `init` you can clone the [Foliant Project template](https://github.com/foliant-docs/foliant_project_template). It’s an empty Foliant project with the required file and directory structure, including necessary Docker configs. It's similar to what you get by running `foliant init`.

Now `cd` to the cloned directory and remove the `.git` folder, which still points to the template repository.

## Setting up Docker configs

We've got a basic project which we already can build with Docker. 

Inside the project dir run:

```bash
$ docker-compose run --rm foliant make site
```

Right now our project can only build an MkDocs site. If we try to build a pdf we will get

```bash
$ docker-compose run --rm foliant make pdf
No backend available for pdf.
```

If you are already familiar with Foliant you know that to build PDFs you need to install [Pandoc](https://pandoc.org/) with TeXLive and <link src="!path src/backends/pandoc.md">Pandoc backend</link>. So how do we install that in a Docker container?

In your project root you have a `Dockerfile`. This file describes the steps required to set up your virtual container.

If you open the `Dockerfile`, you will see that apart from comments it contains three lines:

```
FROM foliant/foliant

COPY requirements.txt .

RUN pip3 install -r requirements.txt
```

The first line means that we start out with the base Foliant image, more on that later. The second one copies the file `requirements.txt` from your project folder into the container and the last one installs all Python dependencies from this file with `pip` from inside the container.

`requirements.txt` is a file where all your Python dependencies for the project live. It means that adding the Pandoc backend to the virtual environment is a matter of adding it into `requirements.txt`. Let's do this now:

```diff
foliantcontrib.mkdocs
+ foliantcontrib.pandoc

```

With Pandoc and TeXLive it's not that easy, because they are not Python packages. But don't worry, it's still easy enough.

Your virtual container is based on `foliant/foliant` image, which is in turn based on Ubuntu operating system. So all you need to do is to find the right commands to install the required packages as if you were on Ubuntu.

These commands are

```bash
apt update
apt install -y texlive-full librsvg2-bin
apt install -y pandoc
```

So we take this commands and put them in our `Dockerfile`, but we will put `RUN` before each one to explain Docker our intentions. The order in which lines appear in the `Dockerfile` is important. Docker does a nice job of caching stages of container build, so make a rule of putting the commands which less prone to change at the start.

In our case we will probably be editing our `requirements.txt` further down the line, but the pandoc installation commands are unlikely to change so we put them first:

```diff
FROM foliant/foliant

+ ENV DEBIAN_FRONTEND=noninteractive
+ RUN apt update
+ RUN apt install -y texlive-full librsvg2-bin
+ RUN apt install -y pandoc

COPY requirements.txt .

RUN pip3 install -r requirements.txt
```

We've also added an environment variable `DEBIAN_FRONTEND` which is required to install texlive-full inside a Docker container. Consider it magic, just don't forget to add it each time you install texlive in Docker.

Now we need to rebuild our container. If we were to run the `docker-compose run` command now, it would still run in the old container, which doesn't have pandoc. So let's build it

```bash
$ docker-compose build
```

This command will now take time to complete because of the TeXLive engine which is HUGE. Don't worry, you will need to wait for so long just once.

Now, as it's starting to get dark and you can hear the workers coming home from the factories, our image has finished building.

Let's make a PDF!

```bash
$ docker-compose run --rm foliant make pdf
```

If all went right you will see a PDF file in your project folder.

## Using different Foliant Docker images

You've noticed that `docker-compose build` command took a lot of time to complete because it needed to download and install the massive TexLive engine. It would be a pain to repeat this for each new Foliant project.

Luckily, Foliant offers [a selection of Docker images](https://hub.docker.com/r/foliant/foliant/tags), each of which offers different number of tools preinstalled. One of the images is called `pandoc` and has the same packages which we've installed in the previous section.

The full list is:

* `slim` — minimal image of Foliant with no extensions;
* `latest` — same as `slim` but with the `foliant init` command support;
* `pandoc` — image of Foliant with Pandoc backend, Pandoc itself, and LaTeX (`texlive-full` Ubuntu package);
* `full` — most complete image of Foliant with all released extensions and dependencies required for them.

Let's update our project to use the `pandoc` image instead of manually installing the dependencies.

The image to use for the project is specified on the first line of the `Dockerfile`. Open it and replace the first line with:

```diff
- FROM foliant/foliant
+ FROM foliant/foliant:pandoc
```

Now remove the lines which we've added previouslly so your `Dockerfile` looks like this:

```
FROM foliant/foliant:pandoc

RUN pip3 install -r requirements.txt
```

Next, remove the Pandoc backend from `requirements.txt` as it is also preinstalled in the `pandoc` image.

Finally, rebuild the image and run the PDF making command:

```
$ docker-compose build
$ docker-compose run --rm foliant make pdf
```

Once the `pandoc` image is downloaded, the build commands will always run very fast.

## Working with Foliant full image

We've learned how to use Foliant with Docker, how to install dependencies inside the container and how to use different Foliant images.

Now it's time to learn about the `full` Foliant image. This is the most powerful one of all. It has all official Foliant extensions and all their dependencies preinstalled.

Once you base your `Dockerfile` on this image you will have whole power of Foliant at your disposal whenever you need it.

To use it replace the first line of your `Dockerfile` with

```diff
- FROM foliant/foliant
+ FROM foliant/foliant:full
```

and run the build command

```
$ docker-compose build
```

Now you can for instance make a [Slate](https://github.com/slatedocs/slate) static website with your docs instead of MkDocs.

The command is

```bash
$ docker-compose run --rm foliant make site --with slate
```

We had to add a `--with slate` argument to our command to specify the backend to build `site` target with. `full` Foliant docker image contains all available official backends for Foliant and several of them are capable of building the `site` target. Without the `--with` argument (or `-w` for short) Foliant would prompt you for the specific backend name interactively.

# Summary

That's all you need to know to work with Foliant the Docker way. Just remember the steps:

- put your Python dependencies in the `requirements.txt` file,
- add commands for installing your non-Python dependencies into the `Dockerfile`, preceding them with the `RUN`,
- rebuild your image with `docker-compose build` command every time you edit `Dockerfile` or `requirements.txt`. No need to run it after editing your Markdown sources or Foliant-related configuration files.

And one last note for the `full` image users. We keep constantly updating Foliant, adding and updating its extensions. To use all the fresh features update the image every once in a while with command

```bash
$ docker pull registry.itv.restr.im:5000/foliant:full
```

And don't forget to rebuild your project's image after updating:

```bash
$ docker-compose build
```
