# Creating the Gibberish Generator

There are already several Python packages present on PyPi which generate placeholder texts like [loremipsum](https://github.com/monkeython/loremipsum) but we won't deprive ourselves of the fun of creating our own.

Let's define some requirements:

- The generated text should consist of sentences which start with a capital letter and end with a dot.
- There should be a way of controlling the size of the sentence and the number of sentences in the resulting text.
- The words in the text should have at least slight resemblance with real language words.

The last requirement is a bit tricky: we don't want words like `q` or `zxd`, or at least not too many of those for the text look a bit more real. So what we will do is create a simple `gen_word` function which will generate a word with random number of random letters, but the letters will be picked in a more controlled way by another function `pick_letter`:

```python
from random import randint

def gen_word():
    word_len = randint(2, 9)  # [1]
    return ''.join(pick_letter() for _ in range(word_len))  # [2]
```

1. We've restricted the length of the words to 2 to 9 letters so we could avoid too short and too long words.
2. The `pick_letter` function will be supplying us with random letters.

Now to the `pick_letter` function. To make the words look real we don't want this function to return too many of the letters q, w, x and z, which don't appear in words often. We also want to get more vowels than consonants. `kiobe` looks more like a word than `lknsd`.

Here's what I've come up with

```python
from random import choice, random

def pick_letter():
    rare_letters = 'qwxz'
    vowels = 'aeiouy'
    consonants = 'cdfghjklmnprstv'

    pick = random()  # [1]
    if pick > 0.9:  # [2]
        return choice(rare_letters) # [3]
    elif pick > 0.25: # [4]
        return choice(vowels) # [5]
    else:
        return choice(consonants) # [6]
```

1. Get a random float number with [`random`](https://docs.python.org/3/library/random.html#random.random) function.
2. Since `random` returns random floats from 0.0 to 1.0, there's about 10% chance of getting a float which is larger than 0.9.
3. In this case we will randomly pick one of the rare letters: q, w, x or z with [`choice`](https://docs.python.org/3/library/random.html#random.choice) function.
4. The chance of getting a float between 0.25 and 0.9 is about 65%.
5. In this case we will return a vowel.
6. Finally, with a chance of about 25% we will be returning one of the remaining consonants.

Let's put it all together and test our `gen_word` function

```python
>>> gen_word()
'eojuo'
>>> gen_word()
'soe'
>>> gen_word()
'qwiim'
>>> gen_word()
'itookao'
```

Oh my god, I think we've just created Finnish language! Jokes aside, it seems that our words generator works fine.

Now we need to create functions for generating sentences and putting them together into a text.

```python
def gen_sentence(num_words=7):  # [1]
    words = (gen_word() for _ in range(num_words))  # [2]
    return ' '.join(words).capitalize()  # [3]
```

1. The number of words in the sentence is determined by the `num_words` parameter with a sensible default of 7 words.
2. Creating a generator which will yield a new word required number of times.
3. Joining the generated words into a single string, separated by spaces. We are also making the first word capitalized in our sentence.

A quick test to make sure it works

```python
>>> gen_sentence()
'Oveecyyi tukzgoli zvo uqyi ujiffrl viivu odui'
>>> gen_sentence(3)
'Ioyieyug ie hkeepnyo'
```

And now to the whole text generator

```python
def gen_text(num_sentences=10):  # [1]
    sizes = (randint(3, 12) for _ in range(num_sentences))  # [2]
    sentences = (gen_sentence(size) for size in sizes)  # [3]
    return '. '.join(sentences) + '.'  # [4]
```

1. Text generator will accept one parameter `num_sentences` with a default of 10.
2. Creating a generator which will yield a number of words in each sentence required number of times. We are limiting the sentence size here to 3 to 12 words.
3. Creating a generator which will yield a new sentence required number of times.
4. Joining the generated sentences into a single string, separated by dots. We are also adding a dot at the end of the text.

Time for the final test!

```python
>>> gen_text()
'Eeaidmt cznm aeoiemino ivjuyauq exieh aoioayif yavfkoa tasojm xuz qizxiyum iyoi fajo. Anuipcauo uac eunjtou oiy hougqf tulztiawk qooulu eiewewaii. Lxi isoxuau ooovox wtopuodu oom ougvoeyy ou calxja io reicye yaioyzx. Usmyuavq yoyu xioqei iiu ateuyau yeroueut gucuifuth tiazkkgc. Oyqzuy rnzouq ajiof qaxewxufo. Utiselorc qpoaoydoi kyvyiuao ofxaoiy ueyaoi azdacy lieaiiy au vteccye. Lopgygsz efixuio gi eyzeuxoa eea qwaycx impoetvy eoyijaum uoiighcq lyaxa xy. Yo yazd oio yyn gvyifzaeo eyz iewueuqze yy yeadvtx dqmdiy. Agiiorixk yae tvmu eeeoe aqjy eqnsouejn. Szejaae yl vuoaewt aujc nvkols auokud reaqopae.'
>>> gen_text(2)
'Uyayu xpriicoe usao yua duleekayi loqk iop saiy iuys sciyaihs. Onacrtog ual iei nuuoaz gdgia yyoui.'
```

Our gibberish generator turned out quite decent. Now it's time to make it a Foliant preprocessor.

<if backend="mkdocs">
??? example "Complete source of the generator module"
    ```python
    from random import choice
    from random import randint
    from random import random


    def pick_letter() -> str:
        """
        Pick a random letter.
        Vowels have a higher chance of picking.
        Letters q, w, x and z have the lowest chance of picking.
        """

        rare_letters = 'qwxz'
        vowels = 'aeiouy'
        consonants = 'cdfghjklmnprstv'

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
    ```
</if>

Next: <link src="preprocessor.md"></link>
Previous: <link src="intro.md"></link>
