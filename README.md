# toml-sort

[![pypi-version](https://img.shields.io/pypi/v/toml-sort.svg)](https://python.org/pypi/toml-sort)
[![license](https://img.shields.io/pypi/l/toml-sort.svg)](https://python.org/pypi/toml-sort)
[![image-python-versions](https://img.shields.io/badge/python->=3.7-blue)](https://python.org/pypi/jedi-language-server)
[![image-pypi-downloads](https://pepy.tech/badge/toml-sort)](https://pepy.tech/project/toml-sort)
[![readthedocs-status](https://readthedocs.org/projects/toml-sort/badge/?version=latest)](https://toml-sort.readthedocs.io/en/latest/?badge=latest)

A command line utility to sort and format your toml files.

Read the latest documentation here: <https://toml-sort.readthedocs.io/en/latest/>

## Installation

```bash
pip install toml-sort
```

## Motivation

This library sorts TOML files, providing the following features:

- Sort tables and Arrays of Tables (AoT)
- Option to sort non-tables / non-AoT's, or not
- Preserve comments, where possible
- Standardize whitespace and indentation

I wrote this library/application because I couldn't find any "good" sorting utilities for TOML files. Now, I use this as part of my daily workflow. Hopefully it helps you too!

## Command line usage

This project can be used as either a command line utility or a Python library. Read the docs for an overview of its library capabilities. For command line usage, see below:

```console
$ toml-sort --help
usage: toml-sort [-h] [--version] [-o OUTPUT] [-i] [-I] [-a] [--no-sort-tables] [--sort-table-keys]
                 [--sort-inline-tables] [--sort-inline-arrays] [--no-header] [--no-comments] [--no-header-comments]
                 [--no-footer-comments] [--no-inline-comments] [--no-block-comments]
                 [--spaces-before-inline-comment {1,2,3,4}] [--check]
                 [F [F ...]]

Toml sort: a sorting utility for toml files.

positional arguments:
  F                     filename(s) to be processed by toml-sort (default: -)

optional arguments:
  -h, --help            show this help message and exit
  --version             display version information and exit
  -o OUTPUT, --output OUTPUT
                        output filepath (default: '-')
  -i, --in-place        overwrite the original input file with changes
  --check               silently check if an original file would be changed by the formatter

sort:
  change sorting behavior

  -I, --ignore-case     ignore case when sorting
  -a, --all             sort ALL keys. This implies sort table-keys, inline-tables and inline arrays. (default: only
                        sort non-inline 'tables and arrays of tables')
  --no-sort-tables      Disables the default behavior of sorting tables and arrays of tables by their header value.
                        Setting this option will keep the order of tables in the toml file the same.
  --sort-table-keys     Sort the keys in tables and arrays of tables (excluding inline tables and arrays).
  --sort-inline-tables  Sort inline tables.
  --sort-inline-arrays  Sort inline arrays.

comments:
  exclude comments from output

  --no-header           Deprecated. See --no-header-comments
  --no-comments         remove all comments. Implies no header, footer, inline, or block comments
  --no-header-comments  remove a document's leading comments
  --no-footer-comments  remove a document's trailing comments
  --no-inline-comments  remove a document's inline comments
  --no-block-comments   remove a document's block comments
  --spaces-before-inline-comment {1,2,3,4}
                        the number of spaces before an inline comment (default: 1)

Examples:

  - **Stdin -> Stdout**: cat input.toml | toml-sort
  - **Disk -> Disk**: toml-sort -o output.toml input.toml
  - **Linting**: toml-sort --check input.toml input2.toml input3.toml
  - **Inplace Disk**: toml-sort --in-place input.toml input2.toml

Return codes:

  - 0 : success.
  - 1 : errors were found

Notes:

  - You cannot redirect from a file to itself in Bash. POSIX shells process
    redirections first, then execute commands. --in-place exists for this
    reason
```

## Configuration file

toml-sort can also be configured by using the `pyproject.toml` file. If the file exists and has a `tool.tomlsort` section, the configuration is used. If both command line arguments and the configuration are used, the options are merged. In the case of conflicts, the command line option is used.

In short, the names are the same as on the command line (and have the same meaning), but `-` is replaced with `_`. Please note, that only the below options are supported:

```toml
[tool.tomlsort]
all = true
in_place = true
no_comments = true
no_header_comments = true
no_footer_comments = true
no_inline_comments = true
no_block_comments = true
spaces_before_inline_comment = 2
check = true
ignore_case = true
```

## Comments

Due to the free form nature of comments, it is hard to include them in a sort in a generic way that will work for everyone. `toml-sort` deals with four different types of comments. They are all enabled by default, but can be disabled using CLI switches, in which case comments of that type will be removed from the output.

### Header

The first comments in a document, that are followed by a blank line, are treated as a header, and will always remain at the top of the document. If there is no blank line, the comment will be treated as a block comment instead.

```toml
# This is a header
# it can be multiple lines, as long as it is followed with a blank line
# it will always stay at the top of the sorted document

title = "The example"
```

### Footer

Any comments at the end of the document, after the last item in the toml, will be treated as a footer, and will always remain at the bottom of the document.

```toml
title = "The example"

# this is a footer comment
```

### Inline

Inline comments are comments that are at the end of a line where the start of the line is a toml item.

```toml
title = "The example" # This is a inline comment
```

### Block

Block comments, are any comments that are on their own line. These comments are treated as _attached_ to the item in the toml that is directly below them, not seperated by whitespace. These comments can be multiple lines. Inline comments will appear in the sorted output above the item they were attached to in the input toml.

```toml
# Comment attached to title
title = "The example"

# This comment is an orphan because it
# is seperated from a-section by whitespace

# This comment is attached to a-section
# attached comments can be multiple lines
[a-section]
# This comment is attached to date
date = "2019"
```

### Orphan

Orphan comments are any comments that don't fall into the above categories, they will be removed from the output document.

```toml
# Header comment

# Orphan comment, not attached to any item
# because there is whitespace before title

title = "The example"

# This comment is an orphan because it
# is seperated from a-section by whitespace

# This comment is attached to a-section
[a-section]
```

## Example

The following example shows the input, and output, from the CLI with default options.

### Unformatted, unsorted input

```toml
# My great TOML example

  title = "The example"

[[a-section.hello]]
ports = [ 8001, 8001, 8002 ]
dob = 1979-05-27T07:32:00Z # First class dates? Why not?


     # Attached to b-section
  [b-section]
  date = "2018"
  name = "Richard Stallman"

[[a-section.hello]]
ports = [ 80 ]
    #Attched to dob
dob = 1920-05-27T07:32:00Z # Another date!

                          [a-section]
                          date = "2019"
                          name = "Samuel Roeca"
```

### Formatted, sorted output

```toml
# My great TOML example

title = "The example"

[a-section]
date = "2019"
name = "Samuel Roeca"

[[a-section.hello]]
ports = [ 8001, 8001, 8002 ]
dob = 1979-05-27T07:32:00Z # First class dates? Why not?

[[a-section.hello]]
ports = [ 80 ]
# Attched to dob
dob = 1920-05-27T07:32:00Z # Another date!

# Attached to b-section
[b-section]
date = "2018"
name = "Richard Stallman"
```

## Local Development

Local development for this project is quite simple.

**Dependencies**

Install the following tools manually.

- [Poetry>=1.0](https://github.com/sdispater/poetry#installation)
- [GNU Make](https://www.gnu.org/software/make/)

_Recommended_

- [asdf](https://github.com/asdf-vm/asdf)

**Set up development environment**

```bash
make setup
```

**Run Tests**

```bash
make test
```

## Written by

Samuel Roeca, *samuel.roeca@gmail.com*
