---
layout: post
title:  "Identifying numbers"
date:   2024-11-16 16:59:00 +0100
categories: css html5 wat
excerpt_separator: <!--more-->
---

Fun learning about CSS today.

Since HTML5, element IDs can start with numbers. But in the CSS selectors spec, and therefore in `document.querySelector`, identifiers *can't*. So how do you target an element with a numeric ID?

<!--more-->

```html
<div id="123">Hello</div>

<style>
#123 { color: red } /* No */
</style>
```

CSS has a single method of escaping characters for identifiers, and it's not one of the common ones: you can replace a character with a \ followed by the unicode codepoint for that character in hexadecimal.

```css
#\123 { color: red } /* No */
#"123" { color: red } /* No */
```

The Unicode codepoints for the digit characters run from 0x30 to 0x39, which means the escape sequence for a leading `1` is `\31`.

But if you try escaping this way, you'll find you still aren't all of the way there yet.

```css
#\3123 { color: red } /* No */
```

This is because CSS thinks you're escaping the codepoint 3123 here ("ㄣ"). Instead, you have to either a) pad the Unicode codepoint to 6 hexadecimal characters, or b) add a space after the escape sequence to finish the escape:

```css
#\00003123 { color: green } /* YES */
#\31 23 { color: green } /* YES */
```

That's right, if you want to target id="123" in CSS (or in Javascript's querySelector) you end up with this ABOMINATION(!) At that point, if you need this (because you're generating IDs) you should just add a prefix.

```html
<div id="123">Hello, world!</div>

<style>
#\31 23 { color: green }
</style>
```

Javascript's CSSOM library (in browsers by default) has a CSS.escape method which deals with this:

```js
CSS.escape('123') // =>  "\31 23"
```

But actually trying to use this in practice makes your Javascript feel as clunky as your CSS is.
