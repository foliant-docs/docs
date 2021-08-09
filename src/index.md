# Welcome to Foliant!

Foliant is an all-in-one single-source documentation authoring tool. It lets you produce standalone documents in **pdf** and **docx**, build **static websites**, upload pages to **Confluence**, all from single Markdown source.

Foliant is a higher order tool, which means it uses other programs to do its job. For pdf and docx, it can use [Pandoc](https://pandoc.org/) or [md-to-pdf](https://github.com/simonhaenisch/md-to-pdf), for websites there are available integrations with [MkDocs](https://www.mkdocs.org/), [Aglio](https://github.com/danielgtaylor/aglio) and [Slate](https://github.com/slatedocs/slate).

Foliant preprocessors let you reuse parts of your documents, show and hide content with flags, render diagrams from text, and much more.

Foliant is highly extensible, so if it lacks some of the functions or output formats you can always make a plugin for it or request one from our team.

> Logo made by [Hand Drawn Goods](http://handdrawngoods.com/) from [flaticon.com](https://www.flaticon.com/).

## Who Is It for?

You’ll love Foliant if you:

* need to ship documentation as pdf, docx, and websites
* want to use Markdown with consistent extension system instead of custom syntax for every new bit of functionality
* like reStructuredText’s extensibility and AsciiDoc’s flexibility, but would rather use Markdown
* want a tool that you can extend with custom plugins without dealing with something as overengineered as Sphinx
* want to work with docs as code and make them a part of your CI pipeline
* to standardize the documenting approach among the segregated repositories with sources of your texts

## Changelog

Here is the changelog of [Foliant Core](https://github.com/foliant-docs/foliant/), the main and only strictly required package. See also the <link src="releases.md" title="History of Releases">history of releases of numerous Foliant extensions</link>.

<include repo_url="https://github.com/foliant-docs/foliant.git" path="changelog.md" sethead="3"></include>
