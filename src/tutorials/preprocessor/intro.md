# Creating a Preprocessor

Creating preprocessors for Foliant is quite straightforward, because they are essentially just Python scripts wrapped into a `Preprocessor` class, which is provided by Foliant core. In this tutorial we will go through all steps of creating a new preprocessor.

First of all we need to decide what our preprocessor will do. Let's say you need a preprocessor which will generate some placeholder gibberish text for your documentation, somewhat like [Lorem Ipsum](https://lipsum.com/).

We need a way to tell Foliant to insert the placeholder in a specific part of our document and the Foliant way of doing that is putting an XML-tag like the following:

```html
<gibberish></gibberish>
```

After the preprocessor is applied this tag should be transformed into some placeholder text:

```
Hiteap zoiouxwaf jyrcaay yty xuzuapo eyuigouu. Ysseotaeq ytuiio qqyy yehiiy koyiyoky uul. Pan osfu zoiz oy ikcya tcsxecy qxiiyo. Gryxeye ogeelaee atprwm mjioy eigyyoov nx qe tayoiaud jodmaofue yvo ieyuunyrq eaowu. Jyqnr aej elqj wuytjcae oy igy ak.
```

We would also want to specify the size of the generated text so our tag should accept the `size` parameter which will define the number of generated sentences:

```html
<gibberish size="15"></gibberish>
```

The tutorial will be split into three stages:

1. <link src="generator.md">Writing the gibberish generator code</link>,
1. <link src="preprocessor.md">Wrapping it into a `Preprocessor` class</link>,
1. <link src="install.md">Installing and testing the preprocessor</link>.

So let's get started!

Next: <link src="generator.md"></link>
