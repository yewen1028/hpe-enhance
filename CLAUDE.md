# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page marketing website for **HPE Solutions Sdn Bhd** (a Malaysian IT support / managed-services company). There is no build system, no package manager, no tests, and no backend — each page is one self-contained HTML file with all CSS and JS inlined. Only third-party assets (fonts, icons, Leaflet, photos) are fetched from CDNs at runtime.

## Files

- `hpe-enhanced-editing.html` — the **active working copy**; make edits here. Most recently modified.
- `hpe-enhanced.html` — a near-duplicate earlier/baseline version. Not auto-synced; only touch it if a change is explicitly meant for it too.

The two files are ~12k lines each and diverge slightly. Treat them as independent — a change in one does **not** propagate to the other.

## Running

No compile step. Either open the HTML file directly in a browser, or serve it (needed so the Leaflet CDN and relative behavior work cleanly):

```bash
python -m http.server 3000    # then open http://localhost:3000/hpe-enhanced-editing.html
```

The `.claude/launch.json` "hpe-website" config runs exactly this.

To verify a visual change end-to-end without a GUI, headless Chrome screenshots work on this machine:

```bash
"/c/Program Files/Google/Chrome/Application/chrome.exe" --headless=new --disable-gpu \
  --hide-scrollbars --window-size=1440,1400 --virtual-time-budget=6000 \
  --screenshot="<out>.png" "file:///C:/Users/Sysadm/Downloads/HPE - refined/hpe-enhanced-editing.html"
```

Note: headless static capture cannot exercise scroll events, so scroll-driven effects (reveal animations, parallax drift) won't show motion — it only confirms layout and that images load.

## File layout within a single HTML file

Everything lives in one file in this order (line numbers approximate, from `hpe-enhanced-editing.html`):

- **`<style>` block (~29–7318)** — ALL CSS. `:root` (~46–74) defines the theme tokens (`--orange #f47920`, `--navy #0d1b2a`, `--text`, `--muted`, `--light`, `--radius`, `--shadow`, `--nav-h`). Use these variables; do not hardcode the brand colors.
- **`<noscript>` fallback (~7326)** — force-reveals JS-hidden elements when scripting is off. Keep it in sync when adding new scroll-hidden content.
- **Body content (~7334+)** — sections in DOM order: navbar/scroll-bar/section-dots, `hero`, `keyfacts`, `about`, `services`, `why`, `industries`, `process`, `casestudy`, `centers` (Leaflet map), `testimonials`, `bizpartners`, `contact-section`, `footer`, off-canvas case-study pages, back-to-top FAB.
- **Main script (~10410–11792)** — nav state, scroll progress bar, section dots, count-up (`requestAnimationFrame`), IntersectionObservers, the parallax engine, modal/case-page controls, contact form (opens a `mailto:`).
- **Leaflet scripts (~11800+)** — the Leaflet CDN `<script src>` plus **two** separate map init blocks (dark Carto tiles) for the service-centers map.

## Architecture patterns to preserve

- **Reveal-on-scroll**: elements start hidden (`opacity:0; transform:translateY(...)`) and are revealed by `IntersectionObserver`s that add classes / set inline `transform`. Each observer has a **safety-net `setTimeout`** that force-reveals content after a few seconds so nothing stays invisible if the observer never fires. Match this pattern (hidden start + observer + safety net) for new animated content.
- **Parallax engine** (in the main script): a single rAF-throttled, `passive` scroll handler drifts a `--py` CSS custom property on every `[data-parallax]` element. CSS consumes `--py`. Sections use the `.px-photo` class, whose `::before` renders a photo at `z-index:-1` behind the content, tinted by an inline `--px-tint` gradient with the photo URL in inline `--px-img`. Drift is bounded so oversized layers never reveal edges. It bails out entirely under `prefers-reduced-motion: reduce`. When adding a parallax layer, follow this: add `class="... px-photo" data-parallax="1"` plus the two inline custom properties; use a navy tint on dark sections and a light tint on light sections so the existing theme color still dominates and text stays readable.
- **Scroll handlers must stay `{ passive: true }` and rAF-throttled** — the shared scroll listener already does layout reads (progress bar, dots); don't add un-throttled work to it.

## External dependencies (all CDN, runtime-only)

Font Awesome 6.5.1, Google Fonts "Inter", Leaflet 1.9.4 (+ Carto dark basemap tiles). Photos/logos are hot-linked from `hpe.com.my`, Unsplash, and Pexels (`images.pexels.com/photos/<id>/...`). Images use `onerror` fallbacks where a source may fail; keep that resilience when adding imagery, and prefer Pexels/Unsplash URLs you have confirmed resolve (HTTP 200).

## Editing gotcha

Both HTML files are formatted with a **blank line between nearly every source line** (double-spaced). When using exact-match string edits, include those intervening blank lines in your match, or the match will fail.
