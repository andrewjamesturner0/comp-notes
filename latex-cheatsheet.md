---
title: LaTeX Cheatsheet
author: Andrew Turner
date: 2015-12-05
---

**Just use markdown and pandoc**

# Formatting

* ... = \ldots

* Indented quotes = \begin{quote} WORDS \end{quote}

* _italics_ = \emph{WORDS}


# Citations

#### Preamble

\usepackage{natbib}

#### Add bibliography

\bibliography{references}{ \bibliographystyle{agsm}}

#### In text citation


| Citation command        | Output                                    |
|-------------------------|-------------------------------------------|
| \citet{goossens93}      | Goossens et al. (1993)                    |
| \citep{goossens93}      | (Goossens et al., 1993)                   |
| \citet*{goossens93}     | Goossens, Mittlebach, and Samarin (1993)  |
| \citep*{goossens93}     | (Goossens, Mittlebach, and Samarin, 1993) |
| \citeauthor{goossens93} | Goossens et al.                           |
| \citeauthor*{goossens93}| Goossens, Mittlebach, and Samarin         |
| \citeyear{goossens93}   | 1993                                      |
| \citeyearpar{goossens93}| (1993)                                    |
| \citealt{goossens93}    | Goossens et al. 1993                      |
| \citealp{goossens93}    | Goossens et al., 1993                     |
| \citetext{priv.\ comm.} | (priv. comm.)                             |


# Building

#### Sample Makefile

    PDF = pdflatex
    BIB = bibtex
    REFS = references.bib
    VERSION = 0.0.1
    FILENAME = My-awesome-paper

    all: $(FILENAME)_v$(VERSION).pdf

    %.pdf: %.latex $(REFS)
        $(PDF) $^
        $(BIB) $(FILENAME)_v$(VERSION).aux
        $(PDF) $^
        $(PDF) $^
        rm *$(VERSION).aux
        rm *$(VERSION).bbl
        rm *$(VERSION).blg
        rm *$(VERSION).log

    clean:
        rm *$(VERSION).pdf
