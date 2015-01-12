pandoc-opml
===========

pandoc-opml is a utility that generates [OPML][] files with the help
of [pandoc][].

I've long been interested in OPML as a file format, but was never
happy with the available desktop OPML editors. They were either
outdated, didn't run on a Mac, or wouldn't let me set outline
attributes. Dave Winer's pair of web-based outliners (LittleOutliner
and Fargo) weren't bad, but I was looking for something to run on the
desktop as opposed to the browser.

So I started toying with the idea of using a regular text editor and
exporting plain text files to OPML instead of editing OPML
directly.

The hardest part I knew was going to be parsing the plain text input
files. Looking for alternatives to writing that code myself, I found
pandoc and was thrilled to see it provided access to the abstract
syntax tree (AST) that represented the input file's headers,
paragraphs, list items, etc. Plus, by using pandoc, I could write the
input files in any of the [many file formats it understands][inputs].

After a weekend of hacking, `pandoc-opml.py` was born. Here's how to
run it:

    $ pandoc -t json </path/to/input.txt> | pandoc-opml.py -o </path/to/output.opml>

In short, parse the input file and generate a JSON version of the
AST. Then pipe that to `pandoc-opml.py` and write the output to
`-o/--output` (or stdout if not provided).

Here's a super simple demo:

```bash
$ cat > test.md
% Demo for the README
% Eric Davis
  
# Hello World!

This paragraph is a child of the "Hello World!" header.
EOF
$ pandoc -t json test.md | ./pandoc-opml.py -o test.opml && xmllint -format test.opml
<?xml version="1.0" encoding="UTF-8"?>
<opml version="2.0">
  <!-- OPML generated by pandoc-opml v0.1 on Mon, 12 Jan 2015 00:13:45 GMT -->
  <head>
    <title>Demo for the README</title>
    <ownerName>Eric Davis</ownerName>
    <dateModified>Mon, 12 Jan 2015 00:13:45 GMT</dateModified>
    <generator>https://github.com/edavis/pandoc-opml</generator>
    <docs>http://dev.opml.org/spec2.html</docs>
  </head>
  <body>
    <outline name="hello-world" text="Hello World!">
      <outline text="This paragraph is a child of the &quot;Hello World!&quot; header."/>
    </outline>
  </body>
</opml>
$
```

Generating the OPML is still a bit rough but I hope to eventually
polish it up.

[OPML]: http://dev.opml.org/spec2.html
[pandoc]: http://johnmacfarlane.net/pandoc/
[inputs]: http://johnmacfarlane.net/pandoc/README.html#description
