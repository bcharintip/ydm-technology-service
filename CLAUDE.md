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

## Page flow & section IDs (vertical order)
`#top` (hero) → `#about` → `.solution-stack` (single sticky stage holding `#services`, `#data`, `#ai`, `#web` as overlapping slots) → `#contact` → `#unstoppable` (bottom marquee) → footer.

`#services` and the three solutions are siblings inside `.solution-stage` — there is no longer a standalone services section in normal flow.

## Solution stack — 4-slot pinned stage
- Structure: `<div class="solution-stack"><div class="solution-stage"><nav class="solution-nav">...</nav><section class="solution overview-slot" id="services">...</section><section class="solution" id="data">...</section><section class="solution" id="ai">...</section><section class="solution" id="web">...</section></div></div>`.
- `.solution-stage` is `position: sticky; top: 0; height: 100vh`. Each `<section class="solution">` inside is `position: absolute; inset: 0; opacity: 0; transition: opacity .3s`. Only the `.active` slot is visible.
- `initSolutionStack()` IIFE measures: `slotH = innerHeight` per slot; stack height = `4 × innerHeight + innerHeight` so each slot occupies one viewport of vertical scroll.
- `setActive(idx)` toggles `.active` on the target section, removes `cards-revealed` from the previously active slot (so reveal replays on revisit), and after `DELAY_MS` (300ms) adds `cards-revealed` to the new slot to trigger the intro/cards stair-up animation.
- Slot 0 (services) reveal trigger: when `idx === 0 && stack.getBoundingClientRect().top <= 0` — required because viewport-snap jumps directly to slot offsets so the older `progress > 0.04` heuristic never fires.

### Click-skip vs. scroll
- `.solution-nav` (top-right of stage) renders four dots `00 / 01 / 02 / 03`. Clicking a dot OR a `.pillar-big` anchor inside services (`#data` / `#ai` / `#web`) calls `skipToSlot(idx)`.
- `skipToSlot` sets a `skipMode` flag, calls `setActive(idx)` directly (so only services → target crossfade plays), and instant-scrolls the page to the destination slot's offset. To force instant scroll despite `html { scroll-behavior: smooth }`, the function temporarily flips `documentElement.style.scrollBehavior = 'auto'` and restores it on the next `requestAnimationFrame`.
- `onScroll` early-returns when `skipMode` is true so the smooth crossfade isn't disturbed by transient scroll events.
- Natural wheel/touch scroll still walks anchors slot-by-slot (services → data → ai → web). Only click-skip bypasses intermediates.

### Cards-list (drag-pan + page snap)
- Cards inside `.solution-stage .cards-list` are NOT scroll-driven by vertical page scroll anymore. Vertical scroll only switches slots; horizontal browsing happens via pointer drag.
- `attachDrag(list)` uses `pointerdown/move/up` with `setPointerCapture` and updates `list.scrollLeft` directly during drag. On release, `pageSnap` (now using `card.scrollIntoView({ inline: 'start' })`) aligns the nearest card based on drag direction (`lastDx`).
- A 4px move threshold + capture-phase click guard prevents accidental link navigation when ending a drag on a card.
- `scroll-snap-type: x mandatory` + `scroll-snap-stop: always` is the CSS backup for wheel/touch inertia; `.dragging` class disables snap during the active drag so manual `scrollLeft` writes don't fight the browser snap.
- `← DRAG →` hint is rendered next to the `Capabilities (NN)` label on each slot's `.cards-header`. Arrows pulse via `drag-pulse-left/right` keyframes.
- Mobile ≤768px: stack/stage become non-sticky and all slots get `opacity: 1 !important` so they stack vertically with native horizontal scroll inside each cards-list.

## Viewport-snap (page scroll)
- `initViewportSnap()` IIFE intercepts `wheel` (preventDefault, debounce 200ms, cooldown 1000ms) and `touchend` to advance to the next anchor instead of free-scrolling.
- Anchors are recomputed each call: `[0, aboutTop, stackTop + i*vh for i 0..3, contactTop + j*vh while inside contact, maxScroll]`. The marquee between sections is intentionally NOT an anchor — wheel scroll skips it.
- Skipped targets (drag/scroll handled natively): `.cards-list` (horizontal drag area), inputs/textareas/selects/`[contenteditable]` (so the contact form remains usable). Keyboard `PageUp/PageDown/Space` also snap.
- `SNAP_DURATION = 1000ms` covers the 700ms smooth scroll + the in-slot crossfade (300ms) + reveal animation (~500ms) so users can see each slot fully before the next snap is allowed.

## Section reveal animations
- `IntersectionObserver` adds `.in-view` to every `section` and `.hero` once 15%+ visible. `.in-view` removes the base `opacity: 0; transform: translateY(32px)` and lets staggered child rules apply.
- `#about` and `#contact` have explicit per-child stagger (`nth-child(N) { transition-delay: ... }`) so heading, h2, tagline/description, info rows, and pillars/right-column tile in sequentially.
- Solution slots are NOT controlled by `.in-view` — they use the `.active` + `.cards-revealed` mechanism described above instead.

## Display headline blinking cursor
- Every `h2.display` (NOT `h1.display`) ends with a blinking yellow cursor via `::after` pseudo-element using `arrow-yellow.svg` (viewBox `17.61 × 84.95`, ratio ~0.21). Sized at `0.15em × 0.7em` with `transform: translateY(-4px)`. `display-cursor-blink` keyframes toggle opacity at 1Hz.
- For solutions with bg video (`.solution:not(.overview-slot)` = data/ai/web) the cursor uses a slightly larger `margin-left` (0.156em vs 0.13em) so it reads cleanly over the video.
- Period at the end of each h2 was removed in the markup so the cursor visually replaces it. The contact h2 ends with `?` and is left untouched.

## Gotchas
- **Working directory is in Google Drive and has spaces** — quote every shell argument (`"$WORKDIR/og-image.png"`, not `$WORKDIR/og-image.png`).
- **SVG `<use href="#ydm-logo">` needs explicit `viewBox="0 0 620.07 240.36"` on the outer `<svg>`** — otherwise width defaults to the user-agent's 300px and the logo stretches/misaligns.
- **Nav is transparent at all scroll positions** — the `.scrolled` class still toggles past 50px but its style block is a no-op. Do not reintroduce `mix-blend-mode: difference` (conflicts with `backdrop-filter`).
- **Page-level scroll-snap is removed.** `html { scroll-snap-type: y proximity }` and `section { scroll-snap-align: start }` were removed because they bounced when scrolling through the absolute-positioned solution slots inside `.solution-stage`. Don't reintroduce them; viewport-snap is JS-driven now.
- **Solution slot 0 reveal must use `rect.top <= 0`, not progress threshold** — viewport-snap jumps the page directly to slot offsets so progress is always 0 at landing. Without the rect-based check, services content stays hidden after a snap-in.
- **The marquee between About and the solution-stack was removed.** The `id="unstoppable"` lives on the bottom marquee (after Contact). Keep the hero `<a href="#unstoppable">` pointing there; do not re-add the middle marquee.
- **Contact form is taller than 100vh.** `#contact` overrides the base `section { justify-content: center; min-height: 100vh }` with `justify-content: flex-start; min-height: auto; padding: 140px 0 120px` so the form doesn't overflow above the section. Inside contact, viewport-snap adds an anchor every viewport so the long form is paginated cleanly.
- **About video aspect-ratio is `4 / 3`.** `.media-placeholder` carries `aspect-ratio: 4/3` and `width: 100%`; the actual `about-video.mp4` is portrait (1080×1280) so `object-fit: cover` center-crops top/bottom. Switch to `contain` if letterboxing is preferred.
- **Bottom darkening gradient is only on slots with video.** `.solution:not(.overview-slot) .section-bg` carries the bottom-up dark fade for text legibility; the services slot has `background: transparent` on its `.section-bg`.
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
- Read about-video metadata: `mdls -name kMDItemPixelHeight -name kMDItemPixelWidth about-video.mp4`.
