---
tags:
  - preprocessor
  - yaml
  - config
---

# General Notes on Usage

Most simple preprocessors apply unconditionally to the whole content of each Markdown file in the Foliant project. But usually preprocessors look for some specific pseudo-XML tags in Markdown content. Each preprocessor registers its own set of tags.

Tags can have attributes and a body. Attributes are usually used to specify some required or optional parameters. Body is the content that is enclosed between opening and closing tags; preprocessors usually do something with this content:

```xml
<tag attribute_1="value_1" ... attribute_N="value_N">body</tag>
```

Foliant under 1.0.8 tries to convert each attribute value into a boolean value, a number, or a string. Attribute values must be enclosed in double quotes (`"`).

Since Foliant 1.0.9, attribute values are processed as YAML. Scalar values are also converted into boolean values, numbers and strings, but you may specify composite values that should be transformed into lists or dictionaries. You may also use modifiers (i.e. YAML tags) that are available in the Foliant project’s config.

`!path`
:   The string preceded by this modifier should be converted into an existing path relative to the Foliant project’s top-level (“root”) directory.

`!project_path`
:   The string preceded by this modifier should be converted into a path relative to the Foliant project’s top-level (“root”) directory. This path may be nonexistent.

`!rel_path`
:   The string preceded by this modifier should be converted into a path relative to the currently processed Markdown file. This path may be nonexistent.

If you develop a preprocessor that accepts some path, by default it is better to be a path relative to the currently processed Markdown file.

Also, since Foliant 1.0.9, attribute values may be enclosed into double (`"`) or single (`'`) quotes.
