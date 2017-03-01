title: Beamer
tags:
  - Beamer
categories:
  - Latex
  - Presentation
  - Beamer
date: 2016-11-18 17:02:00
---
## Simple Overlay
Use `\pause` command.

## Overlay Specification
`\somecommand<overlay specification>[optional argument]{arguments}`

`overlay specification` = 1-3 (range), 5 (number), -3 (same as 1-3), 1- (from 1 on), ...

## Commands with Overlay Specifications

if `<o.s.>` appears twice, it can be put at either place.

`<text>` can even be a command, like `\draw (0,0) -- (1,0);`

`only<o.s.>{text}<o.s.>` insert the text only in specified slides, for other slides, it takes up no space (throw text away).

`item<o.s.>{text}<o.s.>` show the text in a list environment in the specified slides.

`alt<o.s.>{default text}{alternative text}<o.s.>` display the *default text* in specified slides, otherwise show the *alternative text*.

`temporal<o.s.>{before slide text}{default text}{after text}` alternate between three different texts.

`visible<o.s.>{text}` only shown in specified slides, otherwise it doesn't show up completely.

`uncover<o.s.>{text}` compared to `visible`, in unspecified slides the text can be transparent.

`onslide`