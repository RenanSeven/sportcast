# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This repo is a single self-contained HTML file, `proposta.html`, that implements the Sport Cast commercial pitch deck as a client-side "slide deck" web page (no build step, no dependencies, no server). It is sent to leads/clients as a link so they can view the proposal on desktop or mobile.

`sportcast-pitch.pptx` is a generated PowerPoint export of the same deck (not the source of truth — regenerate from the HTML if content changes, see below).

## Commands

There is no build, lint, or test tooling in this repo — it's a single static HTML file. To preview locally, just open `proposta.html` in a browser (note: opening as a raw file on iOS shows a non-interactive Quick Look preview instead of running the JS — must be opened via a real browser/URL to work).

### Deploy

Changes go live automatically on push:

```
git add proposta.html
git commit -m "..."
git push
```

- GitHub repo: `https://github.com/RenanSeven/sportcast` (branch `main`)
- Vercel auto-deploys from that repo/branch
- Live URL: `https://sportcastmg.vercel.app/proposta.html`

### Regenerating the PPTX export

The `.pptx` was generated with a one-off Node script (using `pptxgenjs` + `cheerio` to parse `proposta.html` and rebuild each slide). That script is not checked into this repo. If the deck content changes and a new PPTX is needed, rewrite/rerun a similar script rather than editing the `.pptx` by hand.

## Architecture

Everything lives in one file: `<style>` in `<head>`, markup in `<body>`, logic in a `<script>` at the end.

**Slide deck structure**: `#deck` is a flex container; each page is a `<section class="slide">` (currently 12) laid out horizontally via `flex:0 0 100vw`. Navigation moves between them by setting `deck.style.transform = translateX(-N * 100vw)` in JS — there's no routing/hash, no framework, and no per-slide `id`. Slides are identified only by **document order**, so to target a specific page in CSS you must use `#deck .slide:nth-of-type(N)`.

Page count is tracked in two places that must stay in sync when slides are added/removed:
- `const totalPages = 12;` in the `<script>`
- the initial `<span class="page-number">01 / 12</span>` in the footer markup

**Navigation**: prev/next buttons (`.footer-btn`) in `.container-footer` (a fixed element, sibling of `#deck`, not inside it), plus swipe (`touchstart`/`touchend` on `#deck`) and arrow-key handling, all funnel into `updatePage()`.

**Per-slide layout patterns** (reused via shared classes, not per-slide CSS):
- `.container-main` is the white card inset from the frame; add `.has-side-img` when a slide has a photo, which reflows to a two-column layout (text + `.side-img`).
- `.side-img` / `.fan-wrap` + `.fan-card`/`.fan-slot-0..2`: the "fan of photos" layout used on the visibility/stats slide (3 rotated overlapping images).
- Images are embedded inline as base64 `data:` URIs (this is why the file is large, ~1.5MB) — there are no external image assets.

**Mobile responsiveness**: a single `@media (max-width:720px)` block near the top of `<style>` handles all mobile overrides (reflowing `.has-side-img`, hiding `.side-img` for specific slides via `#deck .slide:nth-of-type(N) .side-img{display:none}`, shrinking the footer/table, etc.). `--frame-gap` is redefined smaller inside this block. `.deck`/`.slide` height uses a `100vh` fallback overridden by `100svh` to avoid mobile browser chrome clipping the fixed footer.

**Design tokens**: colors are CSS custom properties on `:root` (`--navy`, `--red`, `--white`, `--mist`, `--mist-dim`, `--line`, `--frame-gap`) — reuse these instead of hardcoding colors when adding new slides/content.

**Link previews**: `<title>` and `og:title`/`og:description` in `<head>` control how the link looks when shared (e.g. in WhatsApp). Update these if the pitch's framing changes. WhatsApp caches previews per-URL, so a changed preview may need a `?v=N` cache-buster on the link to show up for recipients who already saw the old one.
