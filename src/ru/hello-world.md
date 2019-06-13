# "Hello, World!". Первые шаги с Foliant

## Предварительная установка необходимых компонентов

**Ubuntu:**

Все команды вводятся в терминале. Терминал открывается нажатием сочетания клавиш CTRL+ALT+T.

### Устанавливаем Python 3.6 и pip3

**Для Ubuntu 14.04 and 16.04:**

```sh
$ add-apt-repository ppa:jonathonf/python-3.6
$ apt update && apt install -y python3 python3-pip
```

**Для более новых версий:**

```sh
$ apt update && apt install -y python3 python3-pip
```

### Устанавливаем Foliant с помощью Python:

```sh
$ python3.6 -m pip install foliant foliantcontrib.init
```

### Устанавливаем Pandoc и TeXLive:

```sh
$ apt update && apt install -y wget texlive-full librsvg2-bin
$ wget https://github.com/jgm/pandoc/releases/download/2.0.5/pandoc-2.0.5-1-amd64.deb && dpkg -i pandoc-2.0.5-1-amd64.deb
```

## Установка бэкендов

* Бэкенд Mkdocs используется для генерации сайта;
* Бэкенд Pandoc используется для генерации PDF, DOCX, TEX.

### Устанавливаем Mkdocs

```sh
$ pip3 install foliantcontrib.mkdocs

```

### Устанавливаем Pandoc

```sh
$ pip3 install foliantcontrib.pandoc

```

## Испытание Foliant в деле

### Создаём локальный Foliant-проект

**Команда:**

```sh
$ foliant init
```

Тут Foliant просит указать название проекта. Мы в примере укажем "Hello Foliant":

```sh
Enter the project name: Hello Foliant
```

**Ожидаемый результат:**

```sh
$ foliant init
Enter the project name: Hello Foliant
✔ Generating Foliant project
─────────────────────
Project "Hello Foliant" created in hello-foliant
```

В результате Foliant создал на нашем компьютере папку Hello Foliant, содержащую компоненты Foliant-проекта, вот с такой структурой:

```
├── docker-compose.yml
├── Dockerfile
├── foliant.yml
├── README.md
├── requirements.txt
└── src
    └── index.md
```

### Генерируем сайт из Markdown

**Команда:**

```sh
$ foliant make site
```

**Ожидаемый результат:**

```sh
$ foliant make site
✔ Parsing config
✔ Applying preprocessor mkdocs
✔ Making site with MkDocs
─────────────────────
Result: My_Project-2017-12-04.mkdocs
```

Сайт готов. Чтобы посмотреть на него, перейдём в терминале в папку проекта и там создадим с помощью Python локальный HTTP-сервер.

**Команды:**

```sh
$ cd Hello_Foliant-2018-01-23.mkdocs
$ python3 -m http.server
```

**Ожидаемый результат:**

```sh
$ cd Hello_Foliant-2018-01-23.mkdocs
$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Python создал нам HTTP-сервер на порту 8000. Открываем в браузере адрес [http://localhost:8000/](http://localhost:8000/) и видим наш сайт.
