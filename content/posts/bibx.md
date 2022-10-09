---
title: "bibextract: What, why and how?"
date: 2022-10-09T11:52:22+05:30
draft: true
type: "post"
tags: ["latex", "bibtex", "python", "opensource"]
showTableOfContents: true
---

# [**bibextract**](https://github.com/CodePurble/bibextract)?
`bibextract` is a small program that I have written. Given a set of LaTeX
sources and a BibTeX file (that acts as a database), it generates a BibTeX file
containing just the ones that are used in those sources, provided they are also
present in the database.

Now if all that is going over your head, in brief,
[LaTeX](https://www.latex-project.org/about/) is a typesetting system for
documents[^1] and [BibTeX](http://www.bibtex.org/) is a format used for
reference management, that is used along with LaTeX.

[^1]: [A nice LaTeX
  playlist](https://www.youtube.com/watch?v=NwnYHoNtfJ0&list=PL-p5XmQHB_JSQvW8_mhBdcwEyxdVX0c1T)

# Why?
I use LaTeX a _lot_. When using LaTeX, it is common to maintain a set of
bibliography entries as BibTeX files for citation. Since I use a
monolithic BibTeX file with all the entries I will ever need, I run into
problems when I need to share my LaTeX project with others. Rather than ship a
~2000 line BibTeX file containing mostly irrelevant entries, it is much better
to ship a BibTeX file only with entries that are used in that project. Now,
it can be easily done for small articles, but for research papers and theses,
it will be quite the task. Manually, one would have to look at the generated
bibliography in the document output and get the required entries from the
database BibTeX. This process can be easily automated given the structured
nature of LaTeX.

# How?
This is accomplished in two stages:
* Extract
* Search and write

## Extraction
We look to the ever-reliable `coreutils` for help here. `grep` can be used to
search for `\cite{}` in the LaTeX sources. `sed` can then be used to extract
the BibTeX entry name. The resulting one-liner is:
```sh
# ext.sh f1.tex
grep -Eo '\\cite{.*}' $1 | sed -E 's/\\cite\{(.*)\}/\1/g'
```

* `grep -Eo '\\cite{.*}' $1` searches for the text `\cite{}` in
  the file provided as the first argument to the script (`$1`). So if the file
  contains both `\cite{123}` and `\cite{abc}`, the `grep` command will return:
  ```
  \cite{123}
  \cite{abc}
  ```
* `sed -E 's/\\cite\{(.*)\}/\1/g'` processes this output by removing the
  unwanted `\cite` and the brackets, leaving us with just what we need: the
  BibTeX entry names. For the discussed example, this command will return:
  ```
  123
  abc
  ```

I learnt something new about `sed` and [RegEx](https://regexone.com/)
sub-expressions while doing this. When using `sed`, you can use `\1` through
`\9` as references to the corresponding matching sub-expression. In our case,
the sub-expression is the part within the braces in `\cite{}`, indicated by the
use of `()` in the RegEx.

## Search and write
The rest of the work is done using Python, because I am **_not_** writing a
BibTeX parser in `bash` 🤣.

I used the
[`python-bibtexparser`](https://github.com/sciunto-org/python-bibtexparser)
library for this. The API is straightforward, and offers enough control. It
uses dictionaries to store the parsed BibTeX. Once the database BibTeX has been
read, it can be queried, and matches can be stored in a separate dictionary for
writing. In order to prevent duplicate entries, I used Python `sets` instead of
`dictionaries` directly.

# Please contribute!
This project is completely free and open-source! [Hacktoberfest
2022](https://hacktoberfest.com/) is live, and I invite you to take this
opportunity to contribute to this project. So head over to
[GitHub](https://github.com/CodePurble/bibextract) and get started!

Happy hacking!
