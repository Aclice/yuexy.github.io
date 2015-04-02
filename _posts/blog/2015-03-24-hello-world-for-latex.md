---
layout: post
title: Hello World —— LaTeX
description: 因为某些原因觉得有必要改进下自己写东西的方式，所以开始尝试用LaTeX, so ~ Hello World
category: blog
---

>This article refer to LaTeX2e reference manual(May 2013).

## 1. Overview of LaTeX

LaTeX typesets a file of text using the TeX program and the LaTeX"macro package" for TeX. That is, it processes an input file containing the text of a document with interspersed commands that describe how the text should be formatted. LaTeX files are plain text that can be written in any reasonable editor. It produces at least three files as output:

### 1. The main output file, whice is one of:
- .dvi: when invoked as *latex*.
- .pdf: when invoked as *pdflatex*.
- .other

### 2. The "transcript" or .log file that contains summary information and diagnostic messages for any errors discovered in the input file.

### 3. An "auxiliary" or .aux file. This is uesd by LaTeX itself, for things such as cross-references;

In the LaTeX input file, a command name starts with a '\', followed by a string of letters or a single non-letter. Arguments contained in square brackets [], are optional while arguments contained in braces {}, are required.

LaTeX is case sensitive. Enter all commands in lower case unless explicitly directed to do otherwise.

## 2. Starting & ending

A minimal input file looks like the following:

\documentclass{class}
\begin{document}
Hello World!
\end{document}

where the *class*  is a valid document class for LaTeX. We will introduce *Document classes* in the next section.

You may include other LaTeX commands between the *\documentclass* and the *\begin{document}* commands, **this area is called the preamable**.

## 3. Document classes

The class of a given document is defined with the command: \documentclass[option]{class}

The \documentclass command must be the first command in a LaTeX source file.

Built-in LaTeX document clases are:
- article
- report
- book
- letter
- slides
Of course, many other document classes are avaliable as add-ons.

### 3.1 Document class options

You can specify so-called *global options* or *class options* to the \documentclass command by enclosing them in square brackets as usual. To specify more than one *option*, separate them with a comma:

\documentclass[option1,option2,...]{class}

Here is the list of the standard class options.

#### Typeface size

All of the standard classes except **slides** accept the following options for selecting the typeface size (default is 10pt):

10pt 11pt 12pt

#### Paper size

All of the standard classes accept these options for selecting the paper size (default is letterpaper):

a4paper a5paper b5paper executivepaper legalpaper letterpaper

#### Minscellaneous other options

##### *draft, final*

mark/do not mark overfull boxes with a big black box; default is final.

##### *fleqn*

Put displayed formaulas flush left; default is centered.

##### *landscape*

Select landscape format; default is portrait.

##### *leqno*

Put equation numbers on the left side of equations; default is the right side.

##### *openbib*

Use "open" bibliography format.

##### *titlepage, notitlepage*

Specifise whether the title page is separate; default depends on the class. These options are not available with the slides class;

##### *onecolumn, twocolumn*

Typeset in one or two columns; default id onecolumn.

##### *oneside, twoside*

Select one- or two-sided layout; default is oneside, except for the *book* class.

##### *openright, openany*

Determines if a chapter should start on a right-hand page; default is *openright* for book. 

--
-
The *slides* class offers the option *clock* for printing the time at the bottom of each note. 

#### Additionl packages are loaded like this:

\usepackage[options]{pkg}

To specify more than one pkg, you can separate them with a comma, or use multiple *\usepackage* commands.

Any options given in the \documentclass command that are unknown by the selected document class are passed on to the packages loader with \usepackage.

## 4. Typefaces