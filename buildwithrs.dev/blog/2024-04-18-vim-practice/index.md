---
slug: vim-practice
title: Vim Practice
authors: [forfd8960]
tags: [rust, error]
---

## Delete words backward

- delete 1 word: `db`
- delete n words: `d<number>b`

## Delete word forward

- `dw`: delete one word.

## Change the current word inpalce

- `cw`: chagne the current word

<!-- truncate -->

## Move forward by sentence or paragraph

- `)`: Move forward one sentence.
- `}`: Move forward one paragraph.

## Move in the sceen

- Move to Top of screen: `H`.
- Move to Bottom of screen: `L`.
- Move to Middle of screen: `M`.
- Move to top of the **file**: `gg`.
- Move to bottom of the **file**: `G`

## Move in line

- Move forward one word: `w`
- Move backwrd one word: `b`.

## Insert at the begining of multipe lines.

1. Ctrl + V enter Visual Model
2. Use j or k to select the lines you want change.
3. Use `I` to insert text at the beginning of the line.
4. Press `Esc` to enter Normal Mode, the text will apear for every line selected.

## Append text to the end of multiple lines.

1. `Ctrl + v` enter Visual Mode.
2. Use j or k to select the lines you want change.
3. Press `$` to navigate to the end of each line.
4. Press `A` to append text to end of line.
5. Insert text u want to add.
6. Press `Esc` to enter Normal Mode, the text will appear for every line selected.
