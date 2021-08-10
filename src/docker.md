Public images of Foliant with different sets of extensions are available [at Docker Hub](https://hub.docker.com/r/foliant/foliant). The actual tags are:

* `slim`—minimal image of Foliant with no extensions;
* `latest`—same as `slim` but with the `foliant init` command support;
* `pandoc`—image of Foliant with Pandoc backend, Pandoc itself, and LaTeX (`texlive-full` Ubuntu package);
* `full`—most complete image of Foliant with all released extensions and dependencies required for them.

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

