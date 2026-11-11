---
name: beamer-dark-presentation
description: Create dark-themed ("Noir") LaTeX Beamer presentations (16:9, amber accents, translucent cards over images, progress-square navigator) and add slides to them with an explicit, selectable layout. Use when the user asks to create a new dark Beamer/LaTeX presentation or deck, or to add, insert or lay out a slide in one (agenda, image + cards, columns, process flow, code/prompt box, diagram, results table, bullets, closing/questions).
---

# Dark Beamer presentations (Noir theme)

This skill does two things:

1. **Create** a new, empty presentation from `assets/presentation-template.tex`. The template contains the full preamble and a title slide.
2. **Add** a slide with a chosen layout from `layouts/`, inserted at a chosen position.

Read `reference/design-system.md` before improvising anything a layout doesn't cover: new grids, extra rows or columns, custom diagrams. It documents the palette roles, both positioning systems, the grids and the known pitfalls.

## Files

| Path | Purpose |
|---|---|
| `assets/presentation-template.tex` | Preamble (palette, fonts, progress-square footline, background macros) and a title slide |
| `layouts/*.tex` | One self-contained slide block per layout. Each file's header comment holds its geometry and variants |
| `reference/design-system.md` | How the theme works: colors, positioning, grids, type scale, pitfalls |

## Layout catalog

| Layout | Background | Use for | Knobs (see the file header) |
|---|---|---|---|
| *title* (in template) | canvas | Opening slide: kicker, title, subtitle, hex motif, byline | Core keyword. Swap the motif for `\includegraphics` |
| `agenda` | canvas | Numbered sections with one-line descriptions | Rows 1–6 (pitch 1.08) |
| `image-stack` | full-bleed image | Image on the right, 2–3 stacked definition cards on the left | Card count. Left or right column |
| `image-flow` | full-bleed image | Process of 2–4 steps, cards joined by amber arrows | Step count. Sub-label line |
| `banner-columns` | black + banner | Illustrated banner under the title, 2–4 tall cards below | Card count. Optional `\framesubtitle` |
| `columns` | void (black) | 2–4 key ideas or contributions, no image | Card count |
| `image-callout` | full-bleed image | Illustration with one wide explanatory card at the bottom | — |
| `code-window` | full-bleed image (or void) | Prompt, code or I/O example in a monospace box | Token coloring |
| `diagram` | void | Hand-placed TikZ pipeline or architecture (13.5 × 6.3 canvas) | Panels: dashed, solid, strong. Arrows |
| `table` | void | Results table with a highlighted best row and best/worst coloring | Lead paragraph on/off. Rows. Columns (formula) |
| `bullets` | canvas | Text-heavy slide, optional image column | One or two columns |
| `closing` | void | Big call-to-action ("Questions?") + references at the bottom | CTA size. Reference count |

If the user doesn't name a layout, pick one from this table based on the content shape: how many parallel items, whether there's an image, whether it's a sequence or a table. Tell them which layout you picked in one line, then proceed. Ask only when two layouts fit equally well and the choice changes the result.

## A. Create a new presentation

1. Collect the title, short title (the amber kicker on the title slide), subtitle, author, institute or course, language (babel name: `english`, `spanish`, …) and output path. Use sensible defaults for anything the user hasn't given. Ask only if the title is missing.
2. Copy `assets/presentation-template.tex` to the target path. Fill every `<<PLACEHOLDER>>`:
   - `<<BABEL_LANGUAGE>>`, `<<SHORT_TITLE>>`, `<<TITLE>>` (use `\\` for manual line breaks), `<<SUBTITLE>>` (may be empty), `<<AUTHOR>>`, `<<INSTITUTE_OR_COURSE>>`.
   - `<<CORE_KEYWORD>>`: one or two short words for the centre hexagon. Or replace the whole motif block with an image node.
3. Add only the slides the user asked for, using section B. An "empty presentation" means the title slide only.
4. Build and verify (section C).

## B. Add a slide

1. **Open the target deck** and check that it uses this theme: it defines `\NoirBackgroundReset` and the `Noir*` colors.
   - If it's an older deck without the macros (e.g. one that switches backgrounds with raw `\usebackgroundtemplate` or uses its own color names), copy the `% ── Per-slide backgrounds` block and any missing color definitions from the template into its preamble. Tell the user you did this. Don't rewrite their existing slides.
2. **Choose the layout** (see the catalog) and read that file in `layouts/`. Its header comment lists the geometry and the variants (2/3/4 cards, number of rows, column formula).
3. **Choose the position.** Slides are delimited by `% === Slide: <name> ===` banners. A block runs from its banner to the matching `\end{frame}`, plus a trailing `\NoirBackgroundReset` if present.
   - Default: before the closing slide if one exists. Otherwise just above `% ── Insert new slides above this line`, or above `\end{document}`.
   - "After X" or "before X": find X's banner (or its frame title if banners are missing) and insert at the block boundary, never inside a block.
4. **Fill the block.**
   - Replace every `<<PLACEHOLDER>>`. Set the banner to `% === Slide: <short name> ===`.
   - For a different card or row count, use the coordinates listed in the file header. Don't eyeball them. For tables, recompute column centres with the formula, along with the separators, outer border and footnote y.
   - Delete unused optional parts (the lead paragraph, `\framesubtitle`, the image column) instead of leaving them empty.
   - Escape `_ % & #` in the content. Keep the color roles: amber headings or key phrases, `NoirText` body, `NoirMuted` secondary, cyan or pink for best/worst only.
   - Put images next to the `.tex` file and reference them by relative name. A missing image renders a labelled placeholder, so the build still succeeds. Mention it to the user.
5. Run `grep -n '<<' <file>.tex`. No placeholders may remain.
6. Build and verify (section C).

## C. Build and verify

Run these in the deck's directory:

```bash
latexmk -pdf -interaction=nonstopmode -halt-on-error <file>.tex
grep -nE '^!|Overfull' <file>.log          # errors / overflow
pdftoppm -r 80 -f <N> -l <N> -png <file>.pdf /tmp/<file>-slide   # N = page of the new slide
```

Two passes are mandatory: the progress squares and every `remember picture` overlay depend on the `.aux` file. `latexmk` handles this, and a single `pdflatex` run gives misplaced cards. Then **look at the rendered PNG** with the Read tool. Check that:
- no text overflows its card, and no card spills into the footline or the title band;
- the cards don't cover the focal point of the background image;
- the progress squares show the right count.

Fix any problem and rebuild before reporting. When you report, list the layout used, the slide position, and any missing images.

## Rules

- Every slide is self-contained: background macro, frame, then `\NoirBackgroundReset`. **Never use `\usebackgroundtemplate{}` as a reset.** It makes the following slides render white.
- Don't change the preamble or footline to fix one slide. Fix it locally.
- Don't add slides, speaker notes or packages the user didn't ask for.
