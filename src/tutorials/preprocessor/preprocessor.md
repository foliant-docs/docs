# Formalizing the Preprocessor

Foliant preprocessor is a Python package of a certain structure. Here's a list of requirements for a package to be considered by Foliant a preprocessor:

1. After installing the preprocessor package it should appear inside the Foliant package folder at a path `foliant.preprocessors.your_preprocessor`.
2. It must be possible to import a class named `Preprocessor` from your package:

    ```python
    from foliant.preprocessor.your_preprocessor import Preprocessor
    ```

3. The `Preprocessor` class should (but is not required to) be a subclass of a `foliant.preprocessors.BasePreprocessor` class.
    * in any case the `Preprocessor` class must accept the same `__init__` arguments as the `BasePreprocessor` class.
4. The `Preprocessor` class must define at least `apply` method.


Let's take our `gibberish` module which we've created in the previous chapter and make it work with Foliant.

According the the requirements above we first need to create a proper directory structure. It should look like this:

```bash
$ tree
.
└── foliant
    └── preprocessors
        └── gibberish.py

2 directories, 1 file
```

This is the first requirement satisfied. Now to the rest of them.

We sould define a `Preprocessor` class and ideally subclass it from `BasePreprocessor`.

Usually we start out a new preprocessor from a template containing the boilerplate code. You can find the full template [here](https://github.com/foliant-docs/foliantcontrib.templates.preprocessor/blob/develop/foliant/cli/init/templates/preprocessor/foliant/preprocessors/%24slug.py).

But to understand the boilerplate code you have to write it at least once, so let's start from scratch.

Let's start by defining the `Preprocessor` class:

```python
from foliant.preprocessors.base import BasePreprocessor


class Preprocessor(BasePreprocessor):
```

The `BasePreprocessor` parent class offers some useful attributes and methods, go ahead and check its [source code](https://github.com/foliant-docs/foliant/blob/develop/foliant/preprocessors/base.py). For our `__init__` method we are going to use one of those:


```python
class Preprocessor(BasePreprocessor):
    tags = ('gibberish', )  # [1]
    defaults = {'default_size': 10}  # [2]

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)  # [3]

        self.logger = self.logger.getChild('gibberish')  # [4]

        self.logger.debug(f'Preprocessor inited: {self.__dict__}')  # [5]
```

1. First we define the tags which will be captured in the Markdown source. As we've decided in the beginning, we want to process tags which look like `<gibberish></gibberish>`, so our tag name is `gibberish`. We put that into the `tags` class attribute which must be a sequence. `tuple` or `list` are equally good choices.
1. We will allow user to define the default size of the generated text in the preprocessor options. Here we provide the default value of `10` to this option, in case user haven't supplied the option.
1. Running the parent's `__init__` method first, which will populate our class with useful attributes.
1. Using the `logger` attribute to set up a logger. This line embeds our preprocessor into the main log file under a name of `gibberish`.
1. Posting our first log message, which will contain all preprocessor's attributes for inspection.

Now let's write the `apply` method. As said above, this method should be present in all preprocessors. This is the method which Foliant will call to apply the preprocessor. Usually this method subsequently opens each file in the temporary directory and calls the main processing method to transform its content in the desired way. It's a good practice to start and end this method with log messages.

```python
    def apply(self):
        self.logger.info('Applying preprocessor Gibberish')
        for markdown_file_path in self.working_dir.rglob('*.md'):  # [1]
            with open(markdown_file_path, encoding='utf8') as markdown_file:
                content = markdown_file.read()  # [2]

            processed_content = self._process_tags(content)  # [3]

            if processed_content:  # [4]
                with open(markdown_file_path, 'w') as markdown_file:
                    markdown_file.write(processed_content)  # [5]
        self.logger.info('Preprocessor Gibberish applied')
```

1. Scan the temporary directory (the `working_dir` attribute, which is a `pathlib.Path` object, defined for us by the parent class) and find all Markdown files in it.
2. Get the source content of each Markdown file.
3. Process the content with `_process_tags` method which we are about to write next.
4. This step is important. We check if the main processing method actually returned processed content. If the string is empty it usually means that something went wrong. Foliant won't interrupt the build process if one of the preprocessors failed to run, and we don't want to write empty or broken content into the Markdown file, because other preprocessors still may run after ours even if ours failed.
5. If everything is OK and our preprocessing function returned some content, we overwrite the original Markdown file with it.

That was pretty straightforward. The `apply` method defers the actual preprocessing work to the `_process_tags` method, so now let's write it.

```python
    def _process_tags(self, content):
        def sub_tag(match):  # [2]
          tag_options = self.get_options(match.group('options'))  # [3]
          default_size = self.options['default_size']  # [4]
          size = tag_options.get('size', default_size)  # [5]
          return gen_text(size)  # [6]

        return self.pattern.sub(sub_tag, content)  # [1]
```

Note the order of the bullet points: we start with the last line of the code above:

1. The `pattern` is another attribute defined by the base class. It is a RegEx pattern object which will capture the XML tags in the Markdown source. Remember the `tags` class attribute we've defined in the beginning? `pattern` will use it to capture the appropriate tags for our preprocessor. We use the [`re.sub`](https://docs.python.org/3/library/re.html#re.sub) method of the pattern which will replace our tag definitions (`<gibberish></gibberish>`) with whatever content returns the `sub_tag` local function.
1. Next we define the `sub_tag` local function. This function accepts one argument: the [`Match`](https://docs.python.org/3/library/re.html#match-objects) object which was captured by the pattern.
1. We use the handy `get_options` method of a base class which will take the options string of the tag found in the source and turn it into a dictionary of options. For example, if the tag in the source was `<gibberish size="15"></gibberish>` then the options string is `size="15"` and it will be turned by the `get_options` method into `{'size': 15}`.
1. Getting the value of the `default_size` parameter from the preprocessor options. The options are stored in the `self.options` dictionary by the base class. The dictionary is first prepopulated by values from the `defaults` attribute that we defined earlier. If the user hasn't stated any options, the `default_size` will have the value of `10`.
1. Getting the `size` option from the tag options. If options were not stated, we are using the `default_size` value.
1. Finally, we are using the `get_text` function from our gibberish generator, which we've written in the previous part of the tutorial. We are returning the string got from the `gen_text` function as a result of our `sub_tag` local function. This is the text which will replace the `<gibberish></gibberish>` tag in the processed Markdown file.

And that's it! We have all the code needed for the preprocessor to work. All is left to do is to make our package installable and test its work.

Next: <link src="install.md"></link>