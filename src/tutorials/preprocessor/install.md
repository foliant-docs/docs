# Installing and Testing

Right now our preprocessor's folder looks like this

```bash
$ tree
.
└── foliant
    └── preprocessors
        └── gibberish.py

2 directories, 1 file
```

To make it an installable Python package we need to add a `setup.py` file to the root folder.

Here's an [article on creating the setup file](https://docs.python.org/3/distutils/setupscript.html) from the official docs. Usually we just take one of the setup.py's from an existing preprocessor as a template or use this official Foliant [snippet](https://github.com/foliant-docs/foliantcontrib.templates.preprocessor/blob/develop/setup.py).

Here's what your `setup.py` may look like

```python
from setuptools import setup


SHORT_DESCRIPTION = 'Gibberish preprocessor for Foliant.'  # [*]

try:
    with open('README.md', encoding='utf8') as readme:
        LONG_DESCRIPTION = readme.read()

except FileNotFoundError:
    LONG_DESCRIPTION = SHORT_DESCRIPTION


setup(
    name='foliantcontrib.gibberish',  # [*]
    description=SHORT_DESCRIPTION,
    long_description=LONG_DESCRIPTION,
    long_description_content_type='text/markdown',
    version='1.0.0',
    author='Simon Garfunkel',  # [*]
    author_email='simong@example.com',  # [*]
    url='https://github.com/foliant-docs/foliantcontrib.gibberish',  # [*]
    packages=['foliant.preprocessors'],
    license='MIT',
    platforms='any',
    install_requires=[
        'foliant>=1.0.8',
    ],
    classifiers=[
        "Development Status :: 5 - Production/Stable",
        "Environment :: Console",
        "Intended Audience :: Developers",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
        "Programming Language :: Python",
        "Topic :: Documentation",
        "Topic :: Utilities",
    ]
)
```

Lines marked with commented asteriks you would probably want to change to suit your preprocessor. Also note that we supplied the contents of the `README.md` as a full description of the package. It's a good time to add the readme for your preprocessor. Explain what your preprocessor does and what options it has. You may use one of the official preprocessors for possible readme structure.

Now the folder structure should look like this

```bash
$ tree
.
├── foliant
│ └── preprocessors
│     └── gibberish.py
├── README.md
└── setup.py

2 directories, 3 files
```

Now it's time to test if the preprocessor actually works. First install by running this command inside the preprocessor folder.

```bash
$ pip3 install .
```

Create an empty Foliant project with `init` command:

```bash
$ foliant init  # creating the empty project
Enter the project name: Gibberish Test                                                                                                          
Generating project... Done
────────────────────
Project "Gibberish Test" created in gibberish-test
$ cd gibberish-test  # entering the created project folder
$ tree  # inspecting the project file structure
.
├── docker-compose.yml
├── Dockerfile
├── foliant.yml
├── README.md
├── requirements.txt
└── src
    └── index.md

1 directory, 6 files
```

First let's add our preprocessor to the `foliant.yml`

```diff
title: Gibberish Test

+ preprocessors:
+   - gibberish
+
chapters:
  - index.md
```

Now let's edit the `index.md` and use the `<gibberish><gibberish>` tag a few times.

```html
# Welcome to Gibberish Test

Here's some gibberish:

<gibberish></gibberish>

Here are just two sentences of gibberish:

<gibberish size="2"></gibberish>
```

Let's run a build into `pre` target. This target doesn't create a PDF or a DOCX, it just returns the preprocessed Markdown text, which perfectly suits our testing needs.

```bash
$ foliant make pre   
Parsing config... Done
Applying preprocessor gibberish... Done
Applying preprocessor _unescape... Done
────────────────────
Result: Gibberish_Test-2021-08-16.pre
```

Inspect the results

```
$ cat Gibberish_Test-2021-08-16.pre/index.md 
# Welcome to Gibberish Test

Here's some gibberish:

Yxz izyuo sjo iir tewo qvqc etosaeeuo iecaizaso aaeoeuo iyey. Apavaiqfu eqaaa eecyo ioiiyuoay ah caou iets. Yooyofa iiynndea yiuqehlq uizu yca. Pi iuld ixuaeqei ousogp yu ushggxyq yiia uiuyjo. Ofoemct ciyfuup uufiy avkfeqa ehtjoj ietwohoo xqgif. Iwohqoeao snf uozlw qeasoqzu gevuywxui ou xypikyyqu on hrx. Ruagoisia ivga ovzho da oziazioic. Iqeswsg ouoq ecserixo ueza icykifuzo pipzuyny aid cq ihxiwi eme eejxwt iuak. Oui goido yduz eeyfahxil dyiya mezifeo iym xuuvyiy. Iii yucnyyyq eono qyqu uu ioo sqwcjuhip.

Here are just two sentences of gibberish:

Lof peuoy iiouy yyau qggedo evuoucaig. Pziqgsg ekiqepyu laeiridyc.
```

Looks like everything worked fine. Now let's set the `default_size` parameter to check if preprocessor options work too. Edit the `foliant.yml`

```diff
title: Gibberish Test

preprocessors:
-   - gibberish
+   - gibberish:
+       default_size: 1

chapters:
  - index.md

```

And run the build

```bash
$ foliant make pre   
Parsing config... Done
Applying preprocessor gibberish... Done
Applying preprocessor _unescape... Done
────────────────────
Result: Gibberish_Test-2021-08-16.pre
```

Now the first `<gibberish></gibberish>` tag should be replaced with just one line of text. Let's check that

```
cat Gibberish_Test-2021-08-16.pre/index.md
# Welcome to Gibberish Test

Here's some gibberish:

Jy si zhwtyu acneec qeugeya ax qqofaiu ydyyyxz.

Here are just two sentences of gibberish:

Wnuhocx uqny ns. Iu ieuiaea iogyjyfy kl eyyeex agayii aioaac yacjume.
```

Everything works as expected. Now you can add the `LICENSE` file and the `changelog.md` to your preprocessor folder and publish it github and pypi so others could use our wonderful creation!

The repository with full code of Gibberish preprocessor is available here.

# Summary

Now you know the basics of creating preprocessors for Foliant. But there's a lot more to learn. Study the code of different preprocessors created by our team to find out different approaches to solving techwriters' problems.

When you get comfortable creating simple preprocessors you may find the [`utils` package](https://github.com/foliant-docs/foliantcontrib.utils) useful. It contains different tools which perform common tasks in preprocessors like [dealing with `chapters` section](https://github.com/foliant-docs/foliantcontrib.utils/blob/master/docs/chapters.md) in foliant.yml or efficiently [combining options](https://github.com/foliant-docs/foliantcontrib.utils/blob/master/docs/combined_options.md) from foliant.yml and XML tags. There's also a powerful [`BasePreprocessorExt` class](https://github.com/foliant-docs/foliantcontrib.utils/blob/master/docs/preprocessor_ext.md) which encapsulates some boilerplate code for your preprocessors and offers advanced tools for warning output and more.

That's all for now. We wish you luck in extending Foliant! Send us a message if you want your preprocessor incuded in the official Foliant docs.
