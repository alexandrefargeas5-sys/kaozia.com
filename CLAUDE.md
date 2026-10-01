# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The marketing site for Kaozia (kaozia.com), an agency selling AI voice-receptionist agents to independent
businesses and restaurants. It is a static, no-build, no-dependency site: plain HTML/CSS/JS files served
as-is. There is no `package.json`, no bundler, no test suite, and no framework.

## Running it locally

There's no dev server or build step. To preview changes in a real browser (recommended before calling
anything done — this is a visual marketing site):

```bash
npx --yes serve -l 8935 .
```

Then open `http://localhost:8935/`. Plain Python `http.server` also works. To just open the file directly
without a server: `open index.html` — fine for layout/CSS checks.

There is no lint or test command — verify changes by loading the page in a browser and checking the
relevant flow (see "conventions" below for what usually needs re-checking).

## Deployment

Static GitHub Pages deployment: pushing to `main` on `alexandrefargeas5-sys/kaozia.com` publishes the
site directly (`CNAME` pins the custom domain `kaozia.com`). No CI/build pipeline — whatever is committed
in the HTML files is what goes live. A Pages build takes a few minutes after the push, and the CDN caches
pages for ~10 min (`cache-control: max-age=600`), so the live site can lag behind — check
`gh api repos/alexandrefargeas5-sys/kaozia.com/pages/builds/latest` before assuming a deploy failed.

## Architecture

### Pages
- **`index.html`** — the only page of the site (hero, solution/features, FAQ, about, final CTA, footer).
  Everything is in this one file: all CSS lives in a single `<style>` block in `<head>`, all JS lives in
  a single `<script>` block at the end of `<body>`, and both are internally organized into clearly
  labeled sections via `/* ==== SECTION NAME ==== */` (CSS) and `<!-- ==== SECTION NAME ==== -->` (HTML)
  comment banners — use these as anchors when navigating the file rather than reading it top to bottom.

A secondary `agent.html` page ("Découvrir l'agent"), a "Notre agent" side panel and an `<audio>` call-demo
player (`call-demo.mp3`) used to exist; they were removed on purpose to simplify the site. Don't
reintroduce them unless asked.

### Demo phone number
The demo line is **09 74 06 33 60** (`tel:+33974063360`). It appears in several places in `index.html`
(navbar CTA, hero CTA, final CTA section, displayed `.phone-number`). When the number changes, grep for
both the `tel:` form and the spaced display form and replace every occurrence.

### Loss calculator panel
A fixed-position floating tab (`calcTab`) toggles an `.open` class on `calcOverlay` and `calcPanel`, which
slides in from the right via `transform: translateX()`. Its JS is a self-contained IIFE at the end of the
`<script>` block; it closes on overlay click, the close button, Escape, and its CTA link.

### Icons
Most icons are inline `<svg>` (Lucide-style path data, `stroke="#0f2044"` matching `--navy`, hand-copied
into the HTML) rather than external files. A handful of icons also exist as standalone `.svg` files in the
repo root (`zap.svg`, `message-circle-more.svg`, `sliders-horizontal.svg`, `square-chart-gantt.svg`,
`calendar-check-2.svg`, `phone-incoming.svg`, `calculator.svg`) and are referenced via `<img src="...">` —
some are no longer used since the "Notre agent" panel was removed; check which ones are wired up before
adding a new SVG file.

### Navbar
`.navbar` is `display:flex; justify-content:space-between` with two children: the logo and the phone CTA.
The two-line CTA (`.navbar-cta-sub` + `.navbar-cta-main`) has aggressive font/padding reductions under
`@media (max-width: 768px)` — it collapses to a single compact pill and the sub-line is hidden. The
`.navbar-link` / `.navbar-links` styles are leftovers from removed nav links. When adding anything to
the navbar, re-test at a narrow width (~500px) — it has broken this way before.

### Other utility scripts
- **`generate_card.py`** — a standalone script (run via `python3 generate_card.py`) that generates
  Alexandre's business card (HTML + print-ready PDF) into `business_card_output/`, using Chrome headless
  for rendering. Configuration is a block of variables at the top of the file (name, phone, colors, etc.)
  — edit those directly rather than passing args. Unrelated to the main site; don't assume it shares
  anything with `index.html` beyond `logo.png`.
