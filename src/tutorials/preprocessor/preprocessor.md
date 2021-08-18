# Formalizing the Preprocessor

A Foliant preprocessor is a Python package of a certain structure. Here's a list of requirements for a package to be considered by Foliant a preprocessor:

1. After installing the preprocessor package should appear inside the Foliant package folder at a path `foliant.preprocessors.your_preprocessor`.
2. It must be possible to import a class named `Preprocessor` from your package:

    ```python
    from foliant.preprocessor.your_preprocessor import Preprocessor
    ```

3. The `Preprocessor` class should (but is not required to) be a subclass of a `foliant.preprocessors.BasePreprocessor` class.
    * in any case the `Preprocessor` class must accept the same `__init__` arguments as the `BasePreprocessor` class.
4. The `Preprocessor` class must define at least the `apply` method.


Let's take our `gibberish` module which we've created in the previous chapter and make it work with Foliant. We will be adding the code into the same module.

According to the requirements above, we first need to create a proper directory structure. It should look like this:

```bash
$ tree
.
└── foliant
    └── preprocessors
        └── gibberish.py

2 directories, 1 file
```

This is the first requirement satisfied. Now to the rest of them.

We should define a `Preprocessor` class and ideally subclass it from `BasePreprocessor`.

Usually, we start out a new preprocessor from a template containing the boilerplate code. You can find the full template [here](https://github.com/foliant-docs/foliantcontrib.templates.preprocessor/blob/develop/foliant/cli/init/templates/preprocessor/foliant/preprocessors/%24slug.py).

But to understand the boilerplate code you have to write it at least once, so let's start from scratch.

We start by defining the `Preprocessor` class.

```python
from foliant.preprocessors.base import BasePreprocessor


class Preprocessor(BasePreprocessor):
```

The `BasePreprocessor` parent class offers some useful attributes and methods, go ahead and take a look at its [source code](https://github.com/foliant-docs/foliant/blob/develop/foliant/preprocessors/base.py).

Let's start writing the class by adding the `__init__` method and several class attributes.


```python
class Preprocessor(BasePreprocessor):
    tags = ('gibberish', )  # [1]
    defaults = {'default_size': 10}  # [2]

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)  # [3]

        self.logger = self.logger.getChild('gibberish')  # [4]

        self.logger.debug(f'Preprocessor inited: {self.__dict__}')  # [5]
```

1. First, we define the tags which will be captured in the Markdown source. As we've decided in the beginning, we want to process tags that look like `<gibberish></gibberish>`, so our tag name is `gibberish`. We put that into the `tags` class attribute which must be a sequence. `tuple` or `list` are equally good choices.
1. We will allow the user to define the default size of the generated text in the preprocessor options. Here we provide the default value of `10` to this option, in case the user hasn't supplied it.
1. Running the parent's `__init__` method first. It will populate our class with useful attributes.
1. Using the `logger` attribute to set up a logger. This line embeds our preprocessor into the main log file under the name of `gibberish`.
1. Posting our first log message, which will contain all preprocessor's attributes for inspection.

Now let's write the `apply` method. As mentioned above, this method must be present in all preprocessors. This is the method that Foliant will call to apply the preprocessor. Usually, it subsequently opens each file from the temporary directory and calls the main processing method to transform their content in the desired way. It's a good practice to start and end this method with log messages.

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

1. Scan the temporary directory (the `working_dir` attribute, which is a `pathlib.Path` object, created for us by the parent class) and find all Markdown files in it.
1. Get the source content of each Markdown file.
1. Process the content with the `_process_tags` method which we are about to write next.
1. This step is important. We check if the main processing method actually returned any content. If the string is empty, it usually means that something went wrong. Foliant won't interrupt the build process if one of the preprocessors fails to run. We don't want to write empty or broken content into Markdown files, because other preprocessors still may run after ours even if ours failed.
1. If everything is OK, and our preprocessing function returned some content, we overwrite the original Markdown file with it.

The `apply` method defers the actual preprocessing work to the `_process_tags` method, so now let's write it.

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

1. The `pattern` is another attribute created by the base class. It is a RegEx pattern object which will capture the XML tags in the Markdown source. Remember the `tags` class attribute we've defined in the beginning? `pattern` will use it to capture the appropriate tags for our preprocessor. We use the [`re.sub`](https://docs.python.org/3/library/re.html#re.sub) method of the pattern which will replace our tag definitions (`<gibberish></gibberish>`) in the `content` with whatever string returns the `sub_tag` local function.
1. Next, we define the `sub_tag` local function. This function accepts one argument: the [`Match`](https://docs.python.org/3/library/re.html#match-objects) object which was captured by the pattern.
1. We use the handy `get_options` method of the base class, which takes the options string of the tag found in the source, and turns it into a dictionary of options. For example, if the captured tag was `<gibberish size="15"></gibberish>`, the options string is `size="15"`. It will be turned into `{'size': 15}` by the `get_options` method.
1. Getting the value of the `default_size` parameter from the preprocessor options. The options are stored in `self.options` dictionary by the base class. The dictionary is first prepopulated by values from the `defaults` attribute that we've defined earlier. According to the `defaults`, if the user hasn't stated any options, the `default_size` will have the value of `10`.
1. Getting the `size` option from the tag options. If options were not stated, we are using the `default_size` value as a fallback.
1. Finally, we are using the `get_text` function from our gibberish generator, which we've written in the previous part of the tutorial. We are returning the string returned from the `gen_text` function as the result of our `sub_tag` local function. This is the text which will replace the `<gibberish></gibberish>` tag in the processed Markdown file.

And that's it! We have all the code required for the preprocessor to work. All is left to do is to make our package installable and test its work.


<if backend="mkdocs">
??? example "Complete source of the preprocessor module"
    ```python
    '''
    Gibberish preprocessor for Foliant documenation authoring tool.
    '''

    import re

    from random import choice
    from random import randint
    from random import random

    from foliant.preprocessors.base import BasePreprocessor


    def pick_letter() -> str:
        """
        Pick a random letter.
        Vowels have a higher chance of picking.
        Letters q, w, x and z have the lowest chance of picking.
        """

        rare_letters = 'qwxz'
        vowels = 'aeiouy'
        consonants = 'cdfghjklmnpqrstv'

        pick = random()
        if pick > 0.9:
            return choice(rare_letters)
        elif pick > 0.3:
            return choice(vowels)
        else:
            return choice(consonants)

    def gen_word() -> str:
        """Return a word consisting of 2 to 9 letters."""
        word_len = randint(2, 9)
        return ''.join(pick_letter() for _ in range(word_len))


    def gen_sentence(num_words: int = 7) -> str:
        """Generate a sentence consisting of `num_words` words."""
        words = (gen_word() for _ in range(num_words))
        return ' '.join(words).capitalize()


    def gen_text(num_sentences: int = 10) -> str:
        """
        Generate a paragraph of gibberish consisting of `num_sentences`
        senteces, each consisting of 3 to 12 words.
        """

        sizes = (randint(3, 12) for _ in range(num_sentences))
        sentences = (gen_sentence(size) for size in sizes)
        return '. '.join(sentences) + '.'


    class Preprocessor(BasePreprocessor):
        defaults = {
            'default_size': 10
        }
        tags = ('gibberish',)

        def __init__(self, *args, **kwargs):
            super().__init__(*args, **kwargs)

            self.logger = self.logger.getChild('gibberish')

            self.logger.debug(f'Preprocessor inited: {self.__dict__}')

        def _process_tags(self, content):
            def sub_tag(match):
              tag_options = self.get_options(match.group('options'))
              default_size = self.options['default_size']
              size = tag_options.get('size', default_size)
              return gen_text(size)

            return self.pattern.sub(sub_tag, content)

        def apply(self):
            self.logger.info('Applying preprocessor Gibberish')
            for markdown_file_path in self.working_dir.rglob('*.md'):
                with open(markdown_file_path, encoding='utf8') as markdown_file:
                    content = markdown_file.read()

                processed_content = self._process_tags(content)

                if processed_content:
                    with open(markdown_file_path, 'w') as markdown_file:
                        markdown_file.write(processed_content)
            self.logger.info('Preprocessor Gibberish applied')
    ```
</if>

Next: <link src="install.md"></link>

Previous: <link src="generator.md"></link>
