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
brittle and a bit of a hassle][jwz].

What if instead you could transform your Markdown into XML and gain
with it all the tools and libraries that natively work with XML? Then
your "grab all level 1 and level 2 headers" task would be a breeze.

pandoc-opml is the tool to do just that.

[jwz]: http://blog.codinghorror.com/regular-expressions-now-you-have-two-problems/

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

pandoc-opml tries to follow the [OPML v2.0][OPML] specification as
closely as possible.

That said, a few things to note:

- Header elements include a `level` and `name` attribute. See below for
  more information on these.
- Unordered list items have a `list="unordered"` attribute.
- Ordered list items have a `list="ordered"` attribute along with an
  `ordinal` attribute specifying the ordinal number of the list item.
- If `description` is provided in the metadata, it is included in the
  OPML `head` element.
- `dateCreated` uses the `date` metadata (if provided) while
  `dateModified` is the date and time the conversion took place.
- A `text` attribute can contain encoded HTML markup.

### Headers

The `level` attribute is the HTML level for the given header
element. For example `1` for h1, `2` for h2, etc.

Each header is assigned a unique identifier according to
[these rules][unique-ids] and that identifier is stored as the `name`
attribute.

To override the `name` attribute, explicitly set the unique identifier:

```markdown
# Hello World {#custom-id}
```

which produces:

```xml
<outline level="1" name="custom-id" text="Hello World"/>
```

### Attributes

If you specify [header attributes], pandoc-opml will include them in
the resulting OPML:

```markdown
# Hello World {#custom-id .draft category=demo}
```

would produce:

```xml
<outline level="1" name="custom-id" text="Hello World" draft="true" category="demo"/>
```

Class header attributes have the value of "true" while key/value
header attributes are included as-is.

[header attributes]: http://johnmacfarlane.net/pandoc/README.html#extension-header_attributes
[unique-ids]: http://johnmacfarlane.net/pandoc/README.html#extension-auto_identifiers

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
