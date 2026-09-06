---
title: "Markdown Feature Showcase"
subtitle: "Every rendering feature on one page"
summary: "A reference post that exercises every Markdown feature so I can see how PaperMod and Hugo render them on this site."
tags: ["meta", "markdown", "hugo"]
date: 2026-09-06
lastmod: 2026-09-06
draft: true
showToc: true
TocOpen: true
math: true
lightCode: true
---

This post is a rendering test. It uses every Markdown feature I care about so I can
check how the theme and Hugo's Goldmark renderer treat them. Notes about the site
configuration are marked with **Config note**.

## Headings

The lines below are H2 to H6. The page title above is the only H1.

## H2 heading

### H3 heading

#### H4 heading

##### H5 heading

###### H6 heading

## Paragraphs and line breaks

A normal paragraph. It has several sentences so wrapping is visible. Lorem ipsum
dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut
labore et dolore magna aliqua.

This line ends with two trailing spaces,  
so the next line is a hard break inside the same paragraph.

This line ends with a backslash,\
which is the other way to force a hard break.

## Inline text formatting

- *Italic* with asterisks and _italic with underscores_.
- **Bold** with asterisks and __bold with underscores__.
- ***Bold and italic*** together.
- ~~Strikethrough~~ text.
- Inline `code span` with backticks.
- A literal backtick inside code: `` `backtick` ``.
- Escaped characters: \*not italic\*, \# not a heading, \[not a link\].
- Subscript and superscript: H~2~O and E = mc^2^.

**Config note:** subscript and superscript are enabled on this site via
`markup.goldmark.extensions.extras` (off by default), so `H~2~O` and `mc^2^`
typeset with real `sub` / `sup` tags. This required turning off the GFM
`strikethrough` extension (single `~` clashes with subscript); `~~strike~~`
still works through `extras.delete`.

## Typographer

Smart punctuation is on by default: "curly double quotes", 'curly singles',
an en dash 10--20, an em dash -- like this -- and an ellipsis... done.

## Blockquotes

> A single-level blockquote.
> It spans two source lines but one paragraph.

> Top level.
>
> > Nested one level deeper.
> >
> > > And a third level.

> Blockquotes can contain other blocks:
>
> 1. an ordered item
> 2. another item
>
> ```js {style=github}
> const inside = "a fenced block within a quote";
> ```
>
> — and a closing line.

### GitHub-style alerts

> [!NOTE]
> Highlights information that users should take into account.

> [!TIP]
> Optional information to help a user be more successful.

> [!IMPORTANT]
> Crucial information.

> [!WARNING]
> Critical content demanding attention.

> [!CAUTION]
> Negative potential consequences of an action.

**Config note:** Hugo renders these alert blockquotes only when a
`blockquote` render hook is present. If they look like plain quotes, the theme
does not ship one.

## Lists

### Unordered

- First item
- Second item
  - Nested item
  - Another nested item
    - Deeper still
- Third item

### Ordered

1. First
2. Second
   1. Nested ordered
   2. Second nested
3. Third
7. The source number is 7, but output continues in sequence.

### Ordered list with a custom start

5. Starts at five
6. Six
7. Seven

### Task list

- [x] Completed task
- [ ] Open task
- [ ] Task with **bold** and a [link](https://gohugo.io)

### Definition list

Term one
: Definition of the first term.

Term two
: First definition of the second term.
: Second definition of the second term.

**Config note:** definition lists are on by default in Hugo's Goldmark.

## Links

- Inline link to [the Hugo docs](https://gohugo.io/documentation/).
- Inline link [with a title attribute](https://gohugo.io "Hover over me").
- Reference-style link to [PaperMod][papermod].
- Bare autolink: <https://github.com/hwalinga>.
- Linkified plain URL: https://gohugo.io (converted automatically).
- Link to a heading on this page: [jump to Tables](#tables).
- Link to an internal page: [About](/about/).
- Email autolink: <hielkewalinga@gmail.com>.

[papermod]: https://github.com/adityatelange/hugo-PaperMod

## Images

Plain Markdown image with alt text:

![Placeholder image with an orange background](/images/markdown-showcase.svg)

Image as a link:

[![Same image, now clickable](/images/markdown-showcase.svg)](https://gohugo.io)

Using the theme `figure` shortcode with a caption:

{{< figure
    src="/images/markdown-showcase.svg"
    alt="Placeholder image rendered through the figure shortcode"
    caption="Figure 1. The `figure` shortcode adds a semantic `<figcaption>`."
    align="center"
>}}

## Code

Inline code: run `hugo server -D` to preview drafts.

Fenced block without a language:

```text {style=github}
plain preformatted text
    indentation is preserved
no syntax highlighting
```

Fenced block with a language:

```python {style=github}
from dataclasses import dataclass


@dataclass
class Point:
    x: float
    y: float

    def norm(self) -> float:
        return (self.x**2 + self.y**2) ** 0.5


print(Point(3, 4).norm())  # 5.0
```

Fenced block with line numbers and highlighted lines:

```go {style=github,linenos=inline,hl_lines=[3,"6-7"]}
package main

import "fmt"

func main() {
    for i := 0; i < 3; i++ {
        fmt.Println("line", i)
    }
}
```

A shell block:

```bash {style=github}
#!/usr/bin/env bash
set -euo pipefail
for f in *.md; do
    printf 'Building %s\n' "$f"
done
```

**Config note:** this site renders code blocks dark (monokai) by default. This
post opts into white blocks by setting `lightCode: true` in its front matter and
adding `{style=github}` to every fence (see also the hashing post). The
`lightCode` front matter injects a small CSS override; the per-fence
`{style=github}` picks the light token palette.

A diff block:

```diff {style=github}
 unchanged line
-removed line
+added line
```

## Tables

| Feature        | Supported | Notes                          |
| -------------- | :-------: | ------------------------------ |
| Tables         |    yes    | GFM pipe tables                |
| Alignment      |    yes    | left, center, right per column |
| Inline markup  |    yes    | **bold**, `code`, [links](/)   |

Column alignment:

| Left | Center | Right |
| :--- | :----: | ----: |
| a    |   b    |     c |
| aaaa |  bbbb  |  cccc |

## Horizontal rule

Text above.

---

Text below.

## Footnotes

Here is a statement that needs a source.[^1] Here is another one, with an inline
reference.[^longnote]

[^1]: The first footnote definition.
[^longnote]: Footnotes can contain multiple paragraphs and code.

    ```text {style=github}
    they stay part of the footnote as long as they are indented
    ```

## Raw HTML

Inline HTML such as <kbd>Ctrl</kbd> + <kbd>C</kbd>, <mark>highlighted text</mark>,
and <sub>sub</sub> / <sup>sup</sup>:

<div style="padding: 0.5rem; border: 1px dashed currentColor;">
  A raw <code>&lt;div&gt;</code> block with inline styling.
</div>

**Config note:** with `markup.goldmark.renderer.unsafe = false` (the Hugo
default) all of the raw HTML above is dropped from the output. Set it to `true`
in `hugo.yaml` to let it through.

## Emoji

Shortcode form: :tada: :rocket: :+1: — and a raw Unicode emoji: 🎉.

**Config note:** `:shortcode:` emoji need `enableEmoji: true` in `hugo.yaml`;
without it the colons render literally. Unicode emoji always work.

## Math

Inline: $E = mc^2$ and $\sum_{i=1}^{n} i = \frac{n(n+1)}{2}$. Display:

$$
\int_0^\infty e^{-x^2}\,dx = \frac{\sqrt{\pi}}{2}
$$

**Config note:** this needs the `passthrough` extension in `hugo.yaml` plus a
KaTeX partial, and the post must set `math: true` in its front matter. Both are
configured on this site, so the expressions above should typeset.

## Details / collapsible

{{< collapse summary="Click to expand this section" >}}
This content is hidden until the reader opens the `<details>` element.

It can contain **Markdown**, lists, and code:

```python {style=github}
print("hidden until expanded")
```
{{< /collapse >}}

## Automatic dark and light

Toggle the theme with the header switch. Body text, links, tables, and
blockquotes all have separate light and dark palettes. Code blocks stay dark
(monokai) in both themes unless a post opts into `lightCode`.
