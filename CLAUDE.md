# YDM Technology Service — Landing Page

Single-file static HTML landing page. Deployed to Vercel via GitHub auto-deploy.

## Architecture
- `index.html` — all CSS in a single inline `<style>` block, all JS in a single inline `<script>` at the end. No build step, no framework.
- Assets in repo root: `favicon.svg`, `logo.svg`, `og-image.png`, `arrow-yellow.svg`, `ydmspecial.ttf`, `vercel.json`.
- Section-BG videos (served statically; keep names URL-safe): `bg-data.mp4`, `bg-ai.mp4`, `bg-web.mp4` (one per solution sub-section), `about-video.mp4` (About media slot).
- `ydm-tech-v3.html` was renamed to `index.html` during initial setup; originals (`YDM-LOGO.svg`, `YDM-Tech-Landing-Content.docx`, `ydmspecial 2.ttf`) are kept locally as source material and are not tracked.

## Deploy
- Repo: `bcharintip/ydm-technology-service` (public GitHub).
- Push to `main` → Vercel auto-deploys production. Push to any other branch → Vercel preview URL.
- No build command / output directory (framework preset: **Other**). Vercel serves repo root.

## Brand
- Accent `#FFFF00` (`var(--accent)`) · BG `#000` (`var(--bg)`) · FG `#F5F5F0`.
- Fonts: Google `Prompt` (default) + `YDMSpecial` (loaded via `@font-face` from `./ydmspecial.ttf`, used in the marquee).

## Section IDs (for nav scroll targets)
`#top` · `#about` · `#services` (overview) · `#data` / `#ai` / `#web` (solutions 03.1/02/03) · `#contact` · `#unstoppable` (marquee).

## Gotchas
- **Working directory is in Google Drive and has spaces** — quote every shell argument (`"$WORKDIR/og-image.png"`, not `$WORKDIR/og-image.png`).
- **SVG `<use href="#ydm-logo">` needs explicit `viewBox="0 0 620.07 240.36"` on the outer `<svg>`** — otherwise width defaults to the user-agent's 300px and the logo stretches/misaligns.
- **Nav is transparent at all scroll positions** — the `.scrolled` class still toggles past 50px but its style block is a no-op. Do not reintroduce `mix-blend-mode: difference` (conflicts with `backdrop-filter`).
- **Solution sections use scroll-pinned horizontal slide** — each `<section class="solution">` is wrapped in `<div class="solution-pin">`. The section is `position: sticky; top:0; height:100vh`; the pin has JS-computed height (`innerHeight + max(60vh, cards.scrollWidth - viewportW + 40)`). `initSolutionPins()` IIFE maps vertical scroll progress → `translateX` on `.cards-list`. Cards start with `translateY(120%) opacity:0`; a 800ms timer after entry adds `.cards-revealed` to trigger stair-up (stagger via `--idx`). `list.transform` is held at 0 until `.cards-revealed` to prevent first-card overflow on fast scroll. Mobile ≤768px falls back to native horizontal scroll.
- **Scroll-snap is `proximity`, not `mandatory`** — mandatory conflicted with the sticky pin.
- **OG image is generated via Chrome headless** from a temporary `og-template.html` (deleted after render). Regenerate with: `"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --disable-gpu --hide-scrollbars --no-sandbox --window-size=1200,630 --virtual-time-budget=5000 --screenshot="$PWD/og-image.png" "file://$PWD/og-template.html"`.

## Workflow
- Design iterates rapidly via small commits. User reviews in a local Live Server (auto-reload) — do NOT call `open <file>` after each edit.
- Commits should be meaningful; each push auto-deploys to production.
- Spec + implementation plan live under `docs/superpowers/`.

## Hero ASCII canvas (`#hero-ascii`)
- **Three visual layers, one canvas**: (1) BG wall — faint `.·:'` chars at `rgba(255,255,255,0.16)`; (2) YDM logo — bright white `CHAR_ON` chars shaped by rasterized `logo.svg`; (3) yellow rain — matrix-style drops cycling `CHAR_RAIN = ':|'` rendered on top.
- **Fixed viewport background**: Canvas lives at `<body>` root (sibling of `<nav>`), CSS `position:fixed; width:100vw; height:100vh; z-index:-1`. `<html>` owns `background: var(--bg)` (not `<body>`) so the negative z-index canvas renders above the page base color. `.marquee` must stay `background: transparent` — any solid bg section will cover the canvas.
- Single IIFE `initHeroAscii()` at the end of the inline `<script>` drives a Canvas 2D grid. Per-cell state lives in Float32Arrays sized `cols*rows` (`rState`, `trailAge`, `shiftX/Y`, `tempX/Y`, `delay`, `logoBlink`, etc.) — reallocated in `resize()`.
- `grid[idx]` comes from rasterizing `logo.svg` via `drawImage` into an offscreen canvas and reading the alpha channel (`data[i*4+3] > 64`). `logo.svg` has internal `.cls-1 { fill: #ff0 }` — keep it, `drawImage` preserves it.
- Fluid deformation uses `shiftX/Y` as a displacement field: snapshot → `tempX/Y`, diffuse (4-neighbor avg) → decay → cursor drag/bulge → `grid[shiftedIdx]` resample. Prefer `Math.round(shift/CELL)` over `| 0` for sub-cell responsiveness.
- **Scroll-driven logo dissolve**: `logoOpacity = 1 - clamp(scrollY / (0.9 * innerHeight), 0, 1)` — fade completes just before `#about`. Each logo cell has a random `logoBlink[idx]` threshold; when `blinkT > logoOpacity` the cell "dissolves back" — it must still draw as a BG-wall `pickOff()` glyph, NOT skip rendering (leaving holes breaks the effect). Edge zone (`edge < 0.08`) uses 55% random flicker between bright/wall for a blinking transition.
- **Pointer listeners bind to `window`**, not `hero` — canvas is viewport-sized so `clientX/Y` map directly to canvas coords. `IntersectionObserver` pause was removed (canvas is always visible); only `visibilitychange` pauses the render loop.

## Dev shortcuts
- Syntax-check the inline `<script>` without running it: `node -e "const fs=require('fs');const h=fs.readFileSync('index.html','utf8');const m=h.match(/<script>([\s\S]*)<\/script>/);try{new Function(m[1]);console.log('OK')}catch(e){console.log('ERR:',e.message)}"`
- No `ffmpeg`, `opencv`, or PyObjC on this machine. For `.mov` references: `qlmanage -t -s 1200 -o /tmp file.mov` (one thumbnail), or Swift + `AVAssetImageGenerator` + `NSBitmapImageRep` for multiple frames.
