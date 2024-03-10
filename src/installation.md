---
tags:
  - tutorial
  - install
---

# Installation

The Foliant set up stages include the apps further: 

- Python through your system’s package manager (mandatory),
- Foliant through the pip manager (mandatory),
- Pandoc and the TeXLive bundle (optional). 

!!! warning "Warning"
    Foliant is configured to run best on Python 3.9. Select this version on installation.

!!! tip "Pro tip"
    Install only <link title="Docker">Docker</link> instead.

## Foliant

### macOS: OS X Mavericks 10.9 or higher
0.  Launch Terminal
   
        Open **Launchad** from the Doc

        Type `terminal` in the search bar

        Click on **Terminal**

1.  Install Python 3 with Homebrew:

        $ brew install python3

2.  Install Foliant with pip:

        $ pip3 install foliant foliantcontrib.init

3.  To bake PDF or DOCX, install Pandoc and MacTeX with Homebrew:

        $ brew install pandoc mactex librsvg

    Finally, install the Pandoc backend:

        $ pip3 install foliantcontrib.pandoc

### Windows 8 or higher

0.  Launch PowerShell
   
        Press `⊞ Win + R`  
   
        Type `powershell` and hit `↵ Enter`   

    Install [Scoop package manager](https://scoop.sh/) in PowerShell:

        $ iex (new-object net.webclient).downloadstring('https://get.scoop.sh')

1.  Install [Python 3.9.0](https://www.python.org/downloads/release/python-390/) with
    selected optional features: pip and pylauncher

2.  Return to PowerShell and install Foliant with pip:
   
        $ pip install PyYAML

        $ pip install PyYAML==5.4.1 (if previous command failed)

        $ python -m pip install foliant foliantcontrib.init

3.  To bake PDF or DOCX, install Pandoc and MikTeX with Scoop:

        $ scoop install pandoc latex

    Finally, install the Pandoc backend:

        $ pip3 install foliantcontrib.pandoc

### Ubuntu

0.  Open launch terminal pressing `Ctrl + Alt + T`

1.  Install Python 3 with apt.

    On 18.04+ version Python 3 comes by default. Run to check:

        $ python3

    If missing, install the latest version:

        $ sudo apt update
        $ sudo apt install software-properties-common
        $ sudo add-apt-repository ppa:deadsnakes/ppa
        $ sudo apt install python3.9 python3-pip

2.  Install Foliant with pip:

        $ pip3 install foliant foliantcontrib.init

3.  To bake PDF or DOCX, install Pandoc and TeX Live with apt and wget:

        $ sudo apt update
        $ sudo apt install -y texlive-full librsvg2-bin pandoc

    Finally, install the Pandoc backend:

        $ pip3 install foliantcontrib.pandoc


## Docker

Check the Foliant images on [Docker hub](https://hub.docker.com/r/foliant/foliant):

* `foliant/foliant:slim` — the core Foliant set;
* `foliant/foliant` — the core Foliant set and the `init` command;
* `foliant/foliant:pandoc` — the core Foliant set, plus, the Tex Live and Pandoc extensions for making PDF and DOCX;
* `foliant/foliant:full` — the full Foliant set with all actual extensions.

Choose the image and run the `docker pull` command like shown below

```bash
$ docker pull foliant/foliant
```

Being anew to Docker, use the <link src="tutorials/docker.md">tutorial on using Foliant with Docker</link>.
