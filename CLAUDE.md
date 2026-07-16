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

Then open `http://localhost:8935/`. Plain Python `http.server` also works for static files, but its
default handler doesn't support HTTP Range requests, which breaks in-page `<audio>` playback/seeking —
use `serve` (or any Range-capable static server) whenever testing the audio players.

To just open the file directly without a server: `open index.html`. This works for layout/CSS checks but
audio elements and any fetch-based behavior should be verified through a server instead.

There is no lint or test command — verify changes by loading the page in a browser and checking the
relevant flow (see "conventions" below for what usually needs re-checking).

## Deployment

Static GitHub Pages deployment: pushing to `main` on `alexandrefargeas5-sys/kaozia.com` publishes the
site directly (`CNAME` pins the custom domain `kaozia.com`). No CI/build pipeline — whatever is committed
in the HTML files is what goes live.

## Architecture

### Pages
- **`index.html`** — the main landing page (hero, solution/features, FAQ, about, final CTA, footer).
  Everything is in this one file: all CSS lives in a single `<style>` block in `<head>`, all JS lives in
  a single `<script>` block at the end of `<body>`, and both are internally organized into clearly
  labeled sections via `/* ==== SECTION NAME ==== */` (CSS) and `<!-- ==== SECTION NAME ==== -->` (HTML)
  comment banners — use these as anchors when navigating the file rather than reading it top to bottom.
- **`agent.html`** — a secondary standalone page ("Découvrir l'agent") reached from the navbar. It
  duplicates the shared design tokens and components it needs (`:root` variables, navbar, buttons,
  footer, feature-card styles, the audio player) rather than importing/sharing CSS with `index.html`.
  This is a deliberate choice for this codebase: there's no shared stylesheet or templating, so
  duplicating a self-contained `<style>`/`<script>` block per page avoids coupling two static files
  together with no build step to keep them in sync safely. When editing shared visual language (colors,
  fonts, button styles, navbar), check whether the same rule exists in both files.

### The "two floating/sliding panels" pattern
Both `index.html` and its navbar-triggered content use a repeated interaction pattern: a trigger (a
navbar button, or a fixed-position floating tab like `.calc-tab`) toggles an `.open` class on a
`{name}-overlay` div and a `{name}-panel` div that slides in from the right via `transform: translateX()`.
Two instances of this exist side by side in `index.html`:
- `calcTab` / `calcOverlay` / `calcPanel` — the loss calculator.
- `agentTab` / `agentOverlay` / `agentPanel` — the "Notre agent" audio + qualities showcase.

These are intentionally **not** sharing CSS classes (each has its own `.calc-*` / `.agent-*` rules,
copy-pasted from one another) to avoid any risk of one feature's styling changes breaking the other.
Each panel's JS is a self-contained IIFE at the end of the `<script>` block; the calculator's and the
agent panel's IIFE each close the other panel when opened, so only one is ever visible at a time — keep
that cross-close behavior when touching either one.

### Audio demo player
The `<audio>`-based custom player (play/pause button, seekable progress bar, elapsed/duration labels) is
implemented twice: once inside the `.agent-panel` in `index.html`, once inside `agent.html`'s dedicated
audio section. Both point at the same `call-demo.mp3` file (a compressed, voice-optimized MP3 — see
below) and share the same element IDs (`audioPlayBtn`, `audioDemoEl`, `audioProgressTrack`, etc.) and the
same vanilla-JS wiring. If you change the player's behavior in one file, mirror the change in the other.

`call-demo.mp3` was produced from a `.mov`/`.m4a` voice recording via ffmpeg
(`ffmpeg -i input -vn -codec:a libmp3lame -q:a 4 -ar 44100 call-demo.mp3`) to keep it small and
voice-clear. Regenerate it the same way if the source recording changes.

### Icons
Most icons are inline `<svg>` (Lucide-style path data, `stroke="#0f2044"` matching `--navy`, hand-copied
into the HTML) rather than external files, for the feature cards and quality tags. A handful of icons
also exist as standalone `.svg` files in the repo root (`zap.svg`, `message-circle-more.svg`,
`sliders-horizontal.svg`, `square-chart-gantt.svg`, `calendar-check-2.svg`, `phone-incoming.svg`,
`calculator.svg`) and are referenced via `<img src="...">` — check which ones are already wired up before
adding a new SVG file, since several were pre-staged and only later matched to a feature.

### Navbar
`.navbar` is `display:flex; justify-content:space-between` with three children: the logo, a
`.navbar-links` group (wraps the "Notre agent" panel trigger and the "Découvrir l'agent" page link
together so they sit as one visual cluster), and the phone CTA. Both nav text items and the CTA have
fairly aggressive font/padding reductions under `@media (max-width: 768px)` — the two-line CTA
(`.navbar-cta-sub` + `.navbar-cta-main`) collapses to a single compact pill and the sub-line is hidden
entirely on narrow screens. This breakpoint is tight by design (two nav links + a labeled CTA in a fixed
70px-tall bar); when adding anything to the navbar, re-test at a narrow width (~500px) — it's broken this
exact way twice already during development.

### Other utility scripts
- **`generate_card.py`** — a standalone script (run via `python3 generate_card.py`) that generates
  Alexandre's business card (HTML + print-ready PDF) into `business_card_output/`, using Chrome headless
  for rendering. Configuration is a block of variables at the top of the file (name, phone, colors, etc.)
  — edit those directly rather than passing args. Unrelated to the main site; don't assume it shares
  anything with `index.html`/`agent.html` beyond `logo.png`.
