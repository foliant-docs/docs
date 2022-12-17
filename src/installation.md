---
tags:
  - tutorial
  - install
---

# Installation

Installation of Foliant is split into three stages: installing Python with your system’s package manager, installing Foliant with pip, and optionally installing Pandoc and TeXLive bundle. Below you’ll find the instructions for three popular platforms: macOS, Windows, and Ubuntu.

Alternatively, you can avoid installing Foliant and its dependencies on your system by using <link title="Docker">Docker and Docker Compose</link>.


## macOS

1.  Install Python 3 with Homebrew:

        $ brew install python3

2.  Install Foliant with pip:

        $ pip3 install foliant foliantcontrib.init

3.  If you plan to bake PDF or DOCX, install Pandoc and MacTeX with Homebrew:

        $ brew install pandoc mactex librsvg

    Finally, install the Pandoc backend:

        $ pip3 install foliantcontrib.pandoc

## Windows

0.  Install [Scoop package manager](https://scoop.sh/) in PowerShell:

        $ iex (new-object net.webclient).downloadstring('https://get.scoop.sh')

1.  Install Python 3 with Scoop:

        $ scoop install python

2.  Install Foliant with pip:

        $ python -m pip install foliant foliantcontrib.init

3.  If you plan to bake pdf or DOCX, install Pandoc and MikTeX with Scoop:

        $ scoop install pandoc latex

    Finally, install the Pandoc backend:

        $ pip3 install foliantcontrib.pandoc

## Ubuntu

1.  Install Python 3 with apt.

    On 18.04 or higher Python 3 will already be installed. Check that by running:

        $ python3

    If it is not installed, here's a way to install the latest version:

        $ sudo apt update
        $ sudo apt install software-properties-common
        $ sudo add-apt-repository ppa:deadsnakes/ppa
        $ sudo apt install python3.9 python3-pip

2.  Install Foliant with pip:

        $ pip3 install foliant foliantcontrib.init

3.  If you plan to bake pdf or DOCX, install Pandoc and TeXLive with apt and wget:

        $ sudo apt update
        $ sudo apt install -y texlive-full librsvg2-bin pandoc

    Finally, install the Pandoc backend:

        $ pip3 install foliantcontrib.pandoc


## Docker

There is a selection of Docker images for Foliant in the [Docker hub](https://hub.docker.com/r/foliant/foliant):

* `foliant/foliant:slim` — minimal image of Foliant with no extensions;
* `foliant/foliant` — the basic image with just Foliant core and the `init` command;
* `foliant/foliant:pandoc` — asic image with the addition of TexLive and Pandoc for building PDF and DOCX;
* `foliant/foliant:full` — the full image with all official Foliant extensions and third-party tools required for them to work.

Choose the image you want and run the `docker pull command`

```bash
$ docker pull foliant/foliant
```

If you are new to Docker, check our tutorial on <link src="tutorials/docker.md">using Foliant with Docker</link>.
