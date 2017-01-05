---
title: Vim cheatsheet
author: Andrew Turner
date: 2014-12-03
version: 2
---

## Vim keys

### Navigation/Editing

^ = Go the the first character of the line

0 = Go to the first column of the line

I = Insert at the beginning of the line

A = Insert at the end of the line

o = Insert on the line below current

O = Insert on the line above current

C = Delete all text to end of line and enter insert mode

D = Delete all text to end of line

J = Put a subsequent line on the current line

H = high (top of screen)

M = mid (medium of screen)

L = low (bottom of screen)

x = delete character under cursor

w = skip to next word

b = skip to prev word


### Spelling

:set spell

:set unspell

z= = suggest alternate spellings

zg = add word to vim's dictionary

zug = undo add word to vim's dictionary


### Formatting

gggqG = Apply text wrapping to whole file

gqG = Apply text wrapping from cursor to end of file

gqap = Apply text wrapping to the current paragraph

gUU = Change line to UPPER CASE

guu = Change line to lower case


## Vim commands

:sp. = split window and open file explorer on working folder

### Wrap text

:set textwidth=80


### Vimdiff keys

do = When the cursor is on a (highlighted) difference, copies the changes from
the other window to the current one. 

dp = Inverse of diff obtain; copies the changes from present window to the
other one.


### Line Endings

set fileformat=unix  =  Remove windows line endings (^M)
