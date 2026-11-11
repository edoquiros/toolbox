# Noir Beamer design system

This file explains how the theme works. You need it when you go beyond the layouts in `layouts/`.

## 1. Page and theme

- `\documentclass[aspectratio=169]{beamer}` gives a **16cm × 9cm** page (`\paperwidth` × `\paperheight`).
  `\textwidth` is about 14cm (398pt), with roughly 1cm margins.
- `\usetheme{default}` + `\usefonttheme{professionalfonts}`. The look comes from colors,
  fonts and two templates (footline and per-slide backgrounds), not from a theme package.
- Font: Montserrat if `montserrat.sty` exists, otherwise Helvetica scaled to 0.95.
  `\familydefault` is `\sfdefault`.
- Frame title: bold `\large`, `NoirText` on a full-width `NoirPanel` band (the
  default outer theme fills the frametitle colorbox with its `bg`). The band takes
  about the top **13% of the page height**. Overlay content should therefore start
  at 0.14–0.16ph or lower.
- `\setbeameroption{hide notes}`: speaker notes are never rendered.

## 2. Palette and color roles

| Color | Hex | Role |
|---|---|---|
| `NoirBg` | `#111317` | Default canvas. Card fill (at 0.65–0.82 opacity). Dark zebra rows |
| `NoirPanel` | `#1A1F26` | Title band, table header, light zebra rows, agenda odd rows |
| `NoirText` | `#F2EEE6` | Body text, card bodies |
| `NoirMuted` | `#CFC8BB` | Secondary text, sub-labels, captions, borders (`!30`–`!80`), table values |
| `NoirAccent` | `#D39A3C` | **Amber**: card headings, bullets, agenda badges, progress squares, key labels |
| `NoirCyan` | `#00D4C8` | Best results in tables, code tokens/variables, diagram hexes |
| `NoirPink` | `#D81B9F` | Worst results (`NoirPink!80!NoirMuted`), decorative relay dots |
| `NoirNavy` | `#0A2540` | Fill for diagram shapes (title hexes) |
| `NoirLink` | `#00ACC1` | Thin rules and outlines (title rule, byline box) |
| `NoirFlow` | `blue!55!white` | Diagram arrows and connectors |
| `black` | — | "Void" slides (diagrams, tables, closing). Slightly darker than `NoirBg`, so `NoirBg` panels read as raised surfaces |
| `NoirAccent!15!black` | — | Highlighted table row |

Rules:
- Use only **one** amber element type per hierarchy level: amber headings, *or* an amber
  key-phrase in body text. Don't make everything amber.
- Reserve cyan and pink for meaning (best/worst, variables). They are not decoration on text slides.
- Text inside cards always carries an explicit color (`\color{NoirText}` etc.), so each
  line's role is then visible in the source, independent of the background.

## 3. Navigation: progress squares

There are no beamer navigation symbols and no header. The footline is a full-width
colorbox, right-aligned with `rightskip=0.8em`, holding a TikZ row of 0.10cm squares at a
0.15cm pitch, **one per frame**. Frames up to the current one are `NoirAccent`; the rest are
`NoirMuted!35!NoirBg`.

Consequences:
- `\inserttotalframenumber` is only correct on the **second** compile (it is 1 on the
  first pass, hence the `max(1,...)` guard). Always build with `latexmk`.
- The row is 0.15·N cm wide, so ~60 frames still fit comfortably. Past ~90 frames,
  switch to a fraction bar.
- Overlays (`\pause`, `\only<2>`) don't add squares, because squares count frames, not pages.
- `\begin{frame}[noframenumbering]` stops a frame from adding a square. Use it for backup slides.

## 4. Backgrounds: the per-slide block convention

Every slide is a self-contained block:

```latex
% === Slide: <name> ===
\NoirBackground...{...}      % optional; omit for the plain NoirBg canvas
\begin{frame}{...} ... \end{frame}
\NoirBackgroundReset          % only when a background macro was used
```

| Macro | Effect | Used by |
|---|---|---|
| *(none)* | `NoirBg` canvas | title, agenda, bullets |
| `\NoirBackgroundImage{file}` | Image stretched to the full 16:9 page | image-stack, image-flow, image-callout, code-window |
| `\NoirBackgroundBanner{file}` | Black page, image at full width, top edge at 0.13ph (just under the title band) | banner-columns |
| `\NoirBackgroundVoid` | Pure black page | columns, diagram, table, closing |
| `\NoirBackgroundReset` | Restores the default canvas template | after every block that set a background |

Why these macros exist: writing `\usebackgroundtemplate` by hand costs 4–8 lines per slide,
and the obvious reset, `\usebackgroundtemplate{}`, is wrong. That empty template is **not a
reset**: it removes the canvas fill and the next slide without its own background renders
**white**. `\NoirBackgroundReset` uses `\setbeamertemplate{background canvas}[default]`,
which paints `NoirBg` again.

Image guidance:
- Full-bleed images are **stretched** to 16:9. Create or crop them at 16:9 (e.g. 1920×1080)
  to avoid distortion.
- Image slides need **negative space** where the cards go: the left half for image-stack,
  the bottom 40% for image-callout, and the centre for code-window.
- If a file doesn't exist, the macros draw a labelled `NoirPanel` placeholder instead of
  failing the build.

## 5. Two positioning systems

### A. Page-relative overlay (cards on top of images)

```latex
\begin{tikzpicture}[remember picture, overlay, mystyle/.style={...}]
  \node[mystyle] at ([xshift=<fx>\paperwidth, yshift=-<fy>\paperheight] current page.north west) {...};
\end{tikzpicture}
```
- The origin is the **top-left of the page**. `fx` goes right and `fy` goes down, both as page fractions.
- The node is ignored by the frame's flow, so beamer's vertical centering doesn't move it.
- It needs two compiles (`remember picture`).

**Card anatomy** (shared by all card layouts):
```latex
fill=NoirBg, fill opacity=0.72, text opacity=1, rounded corners=6pt,
text width=<w>\paperwidth, align=left, inner sep=10pt, anchor=<a>, minimum height=<h>\paperheight
content: {\small\bfseries\color{NoirAccent} Heading}\\[5pt]
         {\scriptsize\color{NoirText} Body}
```
Opacity scale: 0.65 (image should show through) → 0.72 (default) → 0.82 (dense text/code).

**Column grids** (card centre x and text width; `inner sep` adds about 0.044pw per card):

| Cards | Centres (pw) | `text width` (pw) |
|---|---|---|
| 1 | 0.50 | 0.61 (code) / 0.86 (callout, anchored NW at 0.07) |
| 2 | 0.27, 0.73 | 0.34–0.38 |
| 3 | 0.17, 0.50, 0.83 *(or 0.14, 0.50, 0.86 with narrower 0.22 cards)* | 0.22–0.26 |
| 4 | 0.13, 0.38, 0.62, 0.87 | 0.17–0.18 |

Vertical anchors: centred cards at y = 0.54ph. Top-anchored columns at 0.44ph (below a banner).
Stacked rows at 0.16 / 0.43 / 0.70ph (pitch 0.27ph).

### B. Flow canvas (diagrams, tables, agenda)

```latex
\begin{frame}{Title}
\vspace{-0.10cm}
\centering
\begin{tikzpicture}[x=1cm, y=-1cm, baseline=(current bounding box.north)] ... \end{tikzpicture}
```
- `y=-1cm` makes **y grow downward**, so coordinates read like a page layout.
- The canvas is **13.5cm wide**, starting at x = 0. The usable height is about 6.3cm when the
  slide has a title (about 5.5cm with a lead paragraph or a footnote line).
- The picture is part of the normal flow: beamer centres the frame vertically, and text can
  come before it (lead paragraph) or after it (`\par\vspace{2pt}` + footnote).
- Small text uses explicit sizes: `\fontsize{8.5}{10.5}` row titles, `{7}{8.5}` table cells,
  `{6}{7.5}` descriptions, `{5.5}{7}` footnotes.

## 6. Typographic scale

| Element | Size |
|---|---|
| Title-slide kicker (short title) | `\LARGE\bfseries` NoirAccent |
| Title-slide title | `\large\bfseries` NoirText |
| Frame title | `\large\bfseries` |
| Card heading | `\small\bfseries` NoirAccent |
| Card body | `\scriptsize` (`\footnotesize` in the single wide callout) |
| Card sub-label | `\tiny\itshape` NoirMuted |
| Table cell | `\fontsize{7}{8.5}` |
| Footnote / source | `\fontsize{5.5}{7}` or `\tiny`, NoirMuted |
| Closing call-to-action | `\fontsize{40}{48}\bfseries` NoirAccent |

## 7. Pitfalls

- Every special character inside nodes needs escaping: `\_`, `\%`, `\&`, `\#`. `<`/`>` are fine with T1.
- Tables and diagrams use hard-coded coordinates. When you add a row or column, recompute
  **every** dependent coordinate: separators, outer border bottom, footnote y.
- Card text that is too long silently grows the card past its `minimum height` and can
  overlap the footline. Check the rendered PNG.
- Keep `babel` and the content in the same language.
