# Readium 2 Authoring Guide

Version 0.3.0

## Introduction

This document aims to provide authoring guidelines for Reading Applications using the Readium 2 (R2) SDK, and serve as a documentation boilerplate developers can use for their own R2-based apps.

## Fundamentals and global guidelines

### Overview

The R2 SDK supports both EPUB 2.0.1 and EPUB 3. They can be used to create reflowable and fixed-layout ebooks.

Reflowable is ideal for text-heavy publications, its text can be resized and its layout adapted to different needs. Images, audio, video, and interactivity are supported.

A Fixed Layout ebook can be created if the publication needs a pixel-perfect design or layout, its text can’t be resized nor its layout adapted. Full-bleed images, audio, video, and interactivity are supported.

### EPUB 3 Features

This section describes the following features supported in EPUB 3 ebooks:

- footnotes
- pagelist and page numbers
- page progression direction
- writing mode
- extended language support (internationalization)
- interactivity
- audio & video
- media overlays
- MathML

#### Footnotes

In EPUB 3 ebooks, you can label footnotes with the correct `epub:type` values: `noteref` and `footnote`. It is up to Reading Apps to choose how to handle them visually e.g. pop-up, pop-over, slide-over card, modal, etc.

You will need to use an anchor (`<a>`) with an `epub:type` value of `noteref` and an `href` attribute pointing to the corresponding footnote. The footnote consequently needs a matching `id` and an `epub:type` value of `footnote`.

```
<a epub:type="noteref" href="#footnote-1">1</a>

<aside epub:type="footnote" id="footnote-1">The footnote.</aside>
```

If the text of the footnote requires a right-to-left direction, add a `dir="rtl"` attribute to make sure the footnote will be handled properly.

```
<aside epub:type="footnote" id="footnote-1" dir="rtl">الحاشية</aside>
```

It is worth noting that the styling of footnotes may not be preserved when displayed in popups, popovers, modals, etc.

#### Pagelist and page numbers

TBD

#### Page progression direction

By default, the R2 SDK paginates from left to right but supports right to left as well. If you need a right to left progression – Arabic script, Hebrew, Chinese, and Japanese –, you must apply a `page-progression-direction` value of `rtl` to the `<spine>` of the OPF:

```
<spine page-progression-direction="rtl">
  <itemref idref="cover-page" />
  <itemref idref="titlepage" />
  <itemref idref="chapter1" />
</spine>
```

This page progression is applied to the whole publication and will serve as a reference. 

**Note:** You should make sure to add a `dir="rtl"` attribute to the root (`<html>`) of every content document when needed (Arabic script, Hebrew). For maximum compatibility, if your publication contains entire chapters in a left-to-right script, you should add a `dir="ltr"` to a `<section>` or `<div>` wrapper instead of `<html>` or `<body>`. 

#### Writing mode

The R2 SDK currently supports only one writing mode for the entire publication. It will be applied based on the `page-progression-direction` and [primary language](#primary-language) found in the OPF.

For bilingual ebooks in which writing modes differ e.g. a Japanese-English publication, we recommend using the `horizontal-tb` writing mode and a `page-progression-direction` of left to right for best compatibility.

#### Extended language support (internationalization)

In order to get the best language support possible, you should make sure to declare the language (`xml:lang` attribute) in each content document. The value will indeed be used as a reference for default fonts, typographic rules e.g. hyphenation and line-breaking, text-to-speech, etc.

In case the language is not declared, the R2 SDK will use [the primary language in the OPF](#primary-language).

#### Interactivity

TBD – related to r2-glue-js

#### Audio & video

TBD

#### Media Overlays

TBD – related to Daniel’s work (r2 desktop app)

#### MathML

TBD – WIP in all implems

### Metadata

This section provides guidelines, describes how some metadata will be used and lists non-standard metadata the R2 SDK implemented.

#### Primary language

In case there are several `<dc:language>` in the OPF, the first instance will be considered the primary language of the publication.

For instance, if you have:

```
<dc:language>en</dc:language>
<dc:language>hi</dc:language>
```

The primary language will be English and the secondary Hindi.

You should make sure to order those items in the correct order as the primary language can be used to provide users with language-specific fonts and preferences, and trigger the correct direction or writing-mode in some cases.

In case content documents don’t have an `xml:lang` attribute (either on `<html>` and/or `<body>`), this primary language will be added automatically.

**Note:** It is recommended to specify the language and the script for Chinese (`zh-Hans` for simplified, `zh-Hant` for traditional) and Mongol (`mn-Cyrl` or `mn-Mong`) ebooks. 

#### Title

The `<dc:title>` will be used for the running head by default.

#### Non-standard metadata

TBD – related to https://github.com/readium/readium-2/issues/70 

### Navigation

#### Table of Contents

TBD

#### Landmarks

TBD

#### Page list

TBD

### Testing

As a rule of thumb, authors should make sure their HTML, CSS, JavaScript and fonts are supported properly in all major rendering engines – i.e. Safari’s Webkit, Chrome’s Blink, Microsoft’s Trident/EdgeHTML, and Firefox’ Gecko/Quantum. 

The R2 SDK can indeed be used in combination with Webviews – a lite browser without a User Interface – that platforms and/or the web development community provide to app developers. HTML, CSS, and JavaScript support consequently depend on the rendering engines powering those webviews.

TBD: debugging in R2-desktop?

## Guidelines for reflowable ebooks

### Overview

This section provides guidelines and best practices for reflowable ebooks.

#### Structure (HTML)

It is strongly recommended that authors use the proper semantic elements to structure their contents. 

For instance, using a paragraph (`<p>`) instead of a heading (`<h1>`, `<h2>` … `<h6>`) for titles will provide users with a subpar experience, with incorrect typographic rules enforced by default or when setting some preferences. Using `<figure>` and `<figcaption>` will help too.

As another example using `<button>` for buttons will help achieve a better experience for interactive ebooks on touch devices.

#### Styling (CSS)

You should make sure to use all the necessary properties (prefixed and standard) for the `-epub-*` ones as some rendering engines won’t support them otherwise. This is especially critical for `writing-mode` but can also create other significant issues e.g. hyphenation not being applied in long strings of text on some platforms.

For maximum compatibility with `page-break-*` properties, you may want to use the older `column-break-*` properties too – e.g. `-webkit-column-break-inside: avoid`. Indeed, the aliasing of both properties is not guaranteed and depends on the Reading App.

#### Fonts 

- It is recommended that embedded font only be used to achieve an intended effet e.g. handwritten note, stylistic heading, snippets of code, etc.
- In case default fonts don’t provide all the glyphs you need, embedded fonts can be used to improve support. You might want to add a warning telling readers the book may have rendering issues if they change the typeface.
- Don’t use icon fonts if possible, and prefer images or SVG. Indeed, we can’t guarantee your icons and/or decorations won’t be overridden when the user changes the typeface. In case you can’t use images or SVG, make sure the icon and decoration glyphs are mapped to relevant characters i.e. unicode’s miscellaneous symbols and pictographs, and not letters of the alphabet.
- Font sizes should be declared using `em`, `%` or `rem` values. Don’t use absolute values – `pt`, `px`, including keywords e.g. `small`, `medium`, `x-large`, etc. – as they will offer users with a subpar experience on some platforms.
- The main text should either not have a defined font size or should have a font size of `1em` or `100%`. This will ensure ideal readability and font scaling.

#### Images

- Make sure your images, especially those with a transparent background, will still be visible in night mode.

TBD – reaching consensus on handling first (modal, etc.)

#### Tables

TBD – reaching consensus on handling first (modal, etc.)

## Guidelines for fixed-layout ebooks

### Overview

This section provides guidelines and best practices for fixed-layout ebooks.

TBD – WIP in all implems

#### Fonts 

- Font sizes should be declared using `px` values.
- Authors must make sure their embedded fonts render properly on all platforms – Android, iOS, Mac, Windows, etc. – and that inconsistencies don’t impact the user experience negatively e.g. overlapping words or letters, bad word and letter spacing, etc. There is indeed nothing reading app developers can do in this regard since the onus lies with the platforms in the case of font rendering.

## Revision history

Date – Version                        | Summary                                 
--------------------------------------|--------------------------------------
June 4, 2018 – Version 0.3.0          | Added page-breaks, fonts guidelines, and details about primary language and content documents’ xml:lang.
May 29, 2018 – Version 0.2.0          | Added writing mode handling and fonts guidelines for fixed layout.
May 28, 2018 – Version 0.1.0          | First release of the guide, which contains global and reflowable guidelines.