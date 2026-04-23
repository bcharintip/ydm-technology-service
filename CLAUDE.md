# YDM Technology Service — Landing Page

Single-file static HTML landing page. Deployed to Vercel via GitHub auto-deploy.

## Architecture
- `index.html` — all CSS in a single inline `<style>` block, all JS in a single inline `<script>` at the end. No build step, no framework.
- Assets in repo root: `favicon.svg`, `logo.svg`, `og-image.png`, `arrow-yellow.svg`, `ydmspecial.ttf`, `vercel.json`.
- `ydm-tech-v3.html` was renamed to `index.html` during initial setup; originals (`YDM-LOGO.svg`, `YDM-Tech-Landing-Content.docx`, `ydmspecial 2.ttf`) are kept locally as source material and are not tracked.

## Deploy
- Repo: `bcharintip/ydm-technology-service` (public GitHub).
- Push to `main` → Vercel auto-deploys production. Push to any other branch → Vercel preview URL.
- No build command / output directory (framework preset: **Other**). Vercel serves repo root.

## Brand
- Accent `#FFFF00` (`var(--accent)`) · BG `#0A0A0A` (`var(--bg)`) · FG `#F5F5F0`.
- Fonts: Google `Prompt` (default) + `YDMSpecial` (loaded via `@font-face` from `./ydmspecial.ttf`, used in the marquee).

## Section IDs (for nav scroll targets)
`#top` · `#about` · `#services` · `#work` · `#contact` · `#unstoppable` (marquee)

## Gotchas
- **Working directory is in Google Drive and has spaces** — quote every shell argument (`"$WORKDIR/og-image.png"`, not `$WORKDIR/og-image.png`).
- **SVG `<use href="#ydm-logo">` needs explicit `viewBox="0 0 620.07 240.36"` on the outer `<svg>`** — otherwise width defaults to the user-agent's 300px and the logo stretches/misaligns.
- **Nav has scroll-triggered background** (`.scrolled` class added by JS past 50px). Don't reintroduce `mix-blend-mode: difference` — it conflicted with `backdrop-filter` and the scroll state.
- **OG image is generated via Chrome headless** from a temporary `og-template.html` (deleted after render). Regenerate with: `"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --disable-gpu --hide-scrollbars --no-sandbox --window-size=1200,630 --virtual-time-budget=5000 --screenshot="$PWD/og-image.png" "file://$PWD/og-template.html"`.

## Workflow
- Design iterates rapidly via small commits. User reviews in a local Live Server (auto-reload) — do NOT call `open <file>` after each edit.
- Commits should be meaningful; each push auto-deploys to production.
- Spec + implementation plan live under `docs/superpowers/`.

## Hero ASCII canvas (`#hero-ascii`)
- Single IIFE `initHeroAscii()` at the end of the inline `<script>` drives a Canvas 2D grid. Per-cell state lives in Float32Arrays sized `cols*rows` (`rState`, `trailAge`, `shiftX/Y`, `tempX/Y`, `delay`, etc.) — reallocated in `resize()`.
- `grid[idx]` comes from rasterizing `logo.svg` via `drawImage` into an offscreen canvas and reading the alpha channel (`data[i*4+3] > 64`). `logo.svg` has internal `.cls-1 { fill: #ff0 }` — keep it, `drawImage` preserves it.
- Fluid deformation uses `shiftX/Y` as a displacement field: snapshot → `tempX/Y`, diffuse (4-neighbor avg) → decay → cursor drag/bulge → `grid[shiftedIdx]` resample. Prefer `Math.round(shift/CELL)` over `| 0` for sub-cell responsiveness.

## Dev shortcuts
- Syntax-check the inline `<script>` without running it: `node -e "const fs=require('fs');const h=fs.readFileSync('index.html','utf8');const m=h.match(/<script>([\s\S]*)<\/script>/);try{new Function(m[1]);console.log('OK')}catch(e){console.log('ERR:',e.message)}"`
- No `ffmpeg`, `opencv`, or PyObjC on this machine. For `.mov` references: `qlmanage -t -s 1200 -o /tmp file.mov` (one thumbnail), or Swift + `AVAssetImageGenerator` + `NSBitmapImageRep` for multiple frames.
