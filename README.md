pandoc-opml
===========

pandoc-opml generates [OPML] files from [Markdown] with the help of [pandoc].

[OPML]: http://dev.opml.org/spec2.html
[Markdown]: http://johnmacfarlane.net/pandoc/README.html#pandocs-markdown
[pandoc]: http://johnmacfarlane.net/pandoc/

Demo
----

Imagine this Markdown document:

```markdown
---
title: Demo Document
author: Eric Davis
---

# Hello World!

This is a child of the "Hello World!" header.
```

After running it through `pandoc-opml`, you'd have this OPML document:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<opml version="2.0">
  <!-- OPML generated by pandoc-opml v0.1 on Tue, 13 Jan 2015 04:21:33 GMT -->
  <head>
    <title>Demo Document</title>
    <ownerName>Eric Davis</ownerName>
    <dateModified>Tue, 13 Jan 2015 04:21:33 GMT</dateModified>
    <generator>https://github.com/edavis/pandoc-opml</generator>
    <docs>https://github.com/edavis/pandoc-opml#docs</docs>
  </head>
  <body>
    <outline level="1" name="hello-world" text="Hello World!">
      <outline text="This is a child of the &quot;Hello World!&quot; header."/>
    </outline>
  </body>
</opml>
```

Alright, so I've taken the simplicity of Markdown and turned it into a
jumble of XML. What's so great about this?

Well, think of what an XML version of your Markdown now enables.

Say you wanted to grab all level 1 and level 2 headlines from a
Markdown document to put together a table of contents.

All the widely used Markdown libraries seem to focus primarily on
transforming Markdown into HTML, so no help there. Beyond that, you
could try writing a regex to extract the headers but [that path is
brittle and full of pain][regex quote].

What if instead you could transform your Markdown into XML and gain
with it all the tools and libraries that natively work with XML? Then
your "grab all level 1 and level 2 headers" task would be a breeze.

pandoc-opml is the tool to do just that.

[regex quote]: http://blog.codinghorror.com/regular-expressions-now-you-have-two-problems/

Installation
------------

I'll eventually toss this up on PyPI, but for now:

```bash
$ pip install https://github.com/edavis/pandoc-opml
```

The only external requirement is [pandoc].

Running
-------

```bash
$ pandoc-opml [-o <output.opml>] <input.txt>
```

If `-o/--output` is not provided, the output is written to stdout.

Docs
----

pandoc-opml makes every effort to follow the [OPML v2.0][OPML]
specification as closely as possible.

However, Markdown is a rich format so some additional information
about the source elements are stored as attributes.

A good OPML parser should ignore anything it doesn't understand, so
none of this should be a problem. Please file a bug report if any
problems do arise.

### Headers

The OPML of Markdown [headline elements][headlines] includes two
attributes: `level` and `name`.

The `level` attribute is the HTML level for the given header
element. For example `1` for h1, `2` for h2, etc.

The `name` attribute is the unique identifier assigned according to
[these rules][unique ids].

[headlines]: http://johnmacfarlane.net/pandoc/README.html#headers
[unique ids]: http://johnmacfarlane.net/pandoc/README.html#extension-auto_identifiers

To override the `name` attribute, explicitly set the unique identifier:

```markdown
# Hello World {#custom-id}
```

```xml
<outline level="1" name="custom-id" text="Hello World"/>
```

### Attributes

If you specify [header attributes], pandoc-opml will include them in
the resulting OPML:

```markdown
# Hello World {#custom-id .draft category=demo}
```

```xml
<outline level="1" name="custom-id" text="Hello World" draft="true" category="demo"/>
```

Class header attributes have the value of "true" while key/value
header attributes are included as-is.

Later attributes overwrite earlier ones. For example:

```markdown
# Hello World {#unique-id .name name=example}
```

First, `name=unique-id`. Then, the class attribute sets
`name=true`. Then, the key/value attribute sets `name=example`. In the
resulting OPML, `name` will equal `example`.

[header attributes]: http://johnmacfarlane.net/pandoc/README.html#extension-header_attributes

### Lists

[Unordered list items][unordered lists] have a `list` attribute set to
`unordered`.

[Ordered list items][ordered lists] have a `list` attribute set to
`ordered` and an `ordinal` attribute set to the ordinal number of the
list item.

Example:

```markdown
- Hello World
- This is a test

1) Hello World
2) This is a test
```

```xml
<outline list="unordered" text="Hello World"/>
<outline list="unordered" text="This is a test"/>

<outline list="ordered" ordinal="1" text="Hello World"/>
<outline list="ordered" ordinal="2" text="This is a test"/>
```

[list elements]: http://johnmacfarlane.net/pandoc/README.html#lists
[unordered lists]: http://johnmacfarlane.net/pandoc/README.html#bullet-lists
[ordered lists]: http://johnmacfarlane.net/pandoc/README.html#ordered-lists

### Metadata

If `description` is included in the [metadata], it is included as a
`<description>` element in the OPML's `<head>`.

If `date` is included, it is included as the `<dateCreated>` element
in the OPML's `<head>`.

The `<dateModified>` element is the timestamp of when `pandoc-opml`
created the OPML.

All the other metadata (e.g., title, author, email, etc.) maps to the
standard OPML `<head>` elements.

If more than one author is provided, a single `<ownerName>` element is
created with the names comma delimited.

[metadata]: http://johnmacfarlane.net/pandoc/README.html#metadata-blocks

### HTML

If the source Markdown contains formatting, the respective OPML `text`
attribute will contain encoded HTML markup:

```markdown
This paragraph contains *emphasis* and **strong** formatting along
with `code` and H~2~O (subscripts) and 2^10^ (superscripts) and last,
but not least, ~~deleted text~~.
```

```xml
<outline text="This paragraph contains &lt;em&gt;emphasis&lt;/em&gt; and &lt;strong&gt;strong&lt;/strong&gt; formatting along with &lt;code&gt;code&lt;/code&gt; and H&lt;sub&gt;2&lt;/sub&gt;O (subscripts) and 2&lt;sup&gt;10&lt;/sup&gt; (superscripts) and last, but not least, &lt;del&gt;deleted text&lt;/del&gt;."/>
```

Background
----------

I've long been interested in OPML as a file format, but I was always
more comfortable using a text editor than any of the available OPML
editors.

So I started toying with the idea of using a regular text editor and
exporting plain text files to OPML instead of editing OPML
directly.

I knew the hardest part was going to be parsing the plain text input
files. Looking for alternatives to writing that code myself, I found
pandoc and was thrilled to see it provided access to the abstract
syntax tree (AST) that represented the input file's headers,
paragraphs, list items, etc. Plus, by using pandoc, I could write the
input files in any of the [many file formats it understands][inputs].

[inputs]: http://johnmacfarlane.net/pandoc/README.html#description
