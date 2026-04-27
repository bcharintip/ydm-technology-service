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

## Z-stack (viewport-fixed layers, back → front)

```
z:-2  .solution-bg-stack > video.bg-video.active   (BG video, fixed full viewport)
      .solution-bg-stack::after                     (rgba(0,0,0,0.65) overlay above video)
z:-1  #hero-ascii                                   (BG wall + rain + cursor deformation)
       html background (#000)
auto  body content (sections, slot text, etc.)
```

The fixed-position BG video stack lives BEHIND the hero canvas so the hero's ASCII wall + cursor deformation paint OVER the video. Everything in normal-flow body (text, cards, etc.) paints above both. See "Solution BG videos" below for why videos can't live inside the slots anymore.

## Solution stack — 4-slot pinned stage
- Structure: `<div class="solution-stack"><div class="solution-stage"><nav class="solution-nav">...</nav><canvas class="solution-ascii">...</canvas><section class="solution overview-slot" id="services">...</section><section class="solution" id="data">...</section><section class="solution" id="ai">...</section><section class="solution" id="web">...</section></div></div>`.
- `.solution-stage` is `position: sticky; top: 0; height: 100vh`. Each `<section class="solution">` inside is `position: absolute; inset: 0; opacity: 0; transition: opacity .3s`. Only the `.active` slot is visible.
- `initSolutionStack()` IIFE measures: `slotH = innerHeight` per slot; stack height = `4 × innerHeight + innerHeight` so each slot occupies one viewport of vertical scroll.
- `setActive(idx)` toggles `.active` on the target section, removes `cards-revealed` from the previously active slot (so reveal replays on revisit), and after `DELAY_MS` (300ms) adds `cards-revealed` to the new slot to trigger the intro/cards stair-up animation.
- `updateBgState(idx, inStack, dir)` — separate helper called from BOTH `onScroll` and `skipToSlot` — owns BG video activation, hero rain enable, hero wall alpha, and triggers the wall sweep when `bgKey` changes. Lives outside `setActive` because `skipMode` suppresses `onScroll` during click-driven jumps.
- Slot 0 (services) reveal trigger: when `idx === 0 && stack.getBoundingClientRect().top <= 0` — required because viewport-snap jumps directly to slot offsets so the older `progress > 0.04` heuristic never fires.
- `wasInStack` tracks whether the user is inside the sticky stage range. Re-entering from outside (e.g. scrolling back up from Contact while active slot is still 3) explicitly fires the ASCII burst because `setActive` is a no-op when `idx === activeIdx`.

### Click-skip vs. scroll
- `.solution-nav` (top-right of stage) renders four dots `00 / 01 / 02 / 03`. Clicking a dot OR a `.pillar-big` anchor inside services (`#data` / `#ai` / `#web`) calls `skipToSlot(idx)`.
- `skipToSlot` captures `fromIdx = activeIdx` BEFORE setting active, sets `skipMode`, calls `setActive(idx)` directly (so only services → target crossfade plays), and instant-scrolls the page to the destination slot's offset. To force instant scroll despite `html { scroll-behavior: smooth }`, the function temporarily flips `documentElement.style.scrollBehavior = 'auto'` and restores it on the next `requestAnimationFrame`.
- After `setActive`, `skipToSlot` calls `updateBgState(idx, true, idx >= fromIdx ? 1 : -1)` so click-driven sweeps travel in the matching direction (forward → top→bottom, backward → bottom→up).
- `onScroll` early-returns when `skipMode` is true so the smooth crossfade isn't disturbed by transient scroll events.
- Natural wheel/touch scroll still walks anchors slot-by-slot (services → data → ai → web). Only click-skip bypasses intermediates.

### Cards-list (drag-pan + page snap)
- Cards inside `.solution-stage .cards-list` are NOT scroll-driven by vertical page scroll anymore. Vertical scroll only switches slots; horizontal browsing happens via pointer drag.
- `attachDrag(list)` uses `pointerdown/move/up` with `setPointerCapture` and updates `list.scrollLeft` directly during drag. On release, `pageSnap` (now using `card.scrollIntoView({ inline: 'start' })`) aligns the nearest card based on drag direction (`lastDx`).
- A 4px move threshold + capture-phase click guard prevents accidental link navigation when ending a drag on a card.
- `scroll-snap-type: x mandatory` + `scroll-snap-stop: always` is the CSS backup for wheel/touch inertia; `.dragging` class disables snap during the active drag so manual `scrollLeft` writes don't fight the browser snap.
- `← DRAG →` hint is rendered next to the `Capabilities (NN)` label on each slot's `.cards-header`. Arrows pulse via `drag-pulse-left/right` keyframes.
- Mobile ≤640px: stack/stage become non-sticky and all slots get `opacity: 1 !important` so they stack vertically with native horizontal scroll inside each cards-list.

## Solution BG videos (fixed stack at z:-2)
- HTML: `<div class="solution-bg-stack">` is a sibling of `<canvas class="hero-ascii">`, NOT a descendant of `.solution-stage`. It contains three `<video class="bg-video" data-slot="data|ai|web">` elements.
- CSS: stack is `position: fixed; inset: 0; z-index: -2; overflow: hidden; pointer-events: none`. Each `.bg-video` is absolute, `object-fit: cover`, `opacity: 0`, `transition: opacity 0.4s ease`. `.bg-video.active` flips to `opacity: 1`. `::after` paints `rgba(0,0,0,0.65)` over the videos for text-contrast.
- Why fixed and outside the stage: `.solution-stage` has `position: sticky` which creates a stacking context. A video inside the stage cannot be at viewport z:-2 — its z-index would be local to the stage's stacking context, leaving it ABOVE the hero canvas (viewport z:-1). Moving the videos out lets them sit visually behind the hero ASCII canvas.
- The `<video>` elements still inside each slot (`<section class="solution"> > <video class="section-video">`) are HIDDEN on desktop/tablet via the `@media (min-width: 641px)` rule in the TEMP block (see Gotchas). Mobile keeps the in-slot videos because the stage isn't sticky there and the layout is vertical-stacked.
- The dark gradient overlay element `<div class="section-bg">` inside slots is ALSO hidden globally by the same TEMP block — the `.solution-bg-stack::after` 0.65 black overlay replaces its purpose.
- The `.solution-ascii` overlay canvas inside `.solution-stage` is also hidden by the same TEMP block; its rendering code is dead but kept in source in case the wall + cursor logic gets repurposed later.

## Hero ASCII canvas (`#hero-ascii`)
- **Three visual layers, one canvas**: (1) BG wall — faint `.·:'` chars at `rgba(255,255,255, wallAlpha)` (default 0.16, dimmed to 0.08 over BG-video slots); (2) YDM logo — bright white `CHAR_ON` chars shaped by rasterized `logo.svg`; (3) yellow rain — matrix-style drops cycling `CHAR_RAIN = ':|'` rendered on top.
- **Fixed viewport background**: Canvas lives at `<body>` root (sibling of `<nav>`), CSS `position:fixed; width:100vw; height:100vh; z-index:-1`. `<html>` owns `background: var(--bg)` (not `<body>`) so the negative z-index canvas renders above the page base color. `.marquee` must stay `background: transparent` — any solid bg section will cover the canvas.
- Single IIFE `initHeroAscii()` at the end of the inline `<script>` drives a Canvas 2D grid. Per-cell state lives in Float32Arrays sized `cols*rows` (`rState`, `trailAge`, `shiftX/Y`, `tempX/Y`, `delay`, `logoBlink`, etc.) — reallocated in `resize()`.
- `grid[idx]` comes from rasterizing `logo.svg` via `drawImage` into an offscreen canvas and reading the alpha channel (`data[i*4+3] > 64`). `logo.svg` has internal `.cls-1 { fill: #ff0 }` — keep it, `drawImage` preserves it.
- **Logo intro cascade-blink**: instead of random per-cell delays, `assignCascadeDelays()` finds the logo's vertical bbox (top→bot row) from `grid` and distributes `delay[i]` across just those rows over `cascadeMs = 1200ms` (jitter ±150ms). In `render()`, each LOGO cell whose `localElapsed` is in `[0, FLICKER_MS=360]` renders a bright `CHAR_ON` in place at `baseY` — random char for the first 55% of the window, then settles to the stable `chars[idx]` glyph (matches the post-flicker steady state so there's no visible swap). Non-logo cells never participate, so the wave is exactly the width of the logo. Recomputed in both `resize()` and `logoImg.onload` because the grid changes both times.
- Fluid deformation uses `shiftX/Y` as a displacement field: snapshot → `tempX/Y`, diffuse (4-neighbor avg) → decay → cursor drag/bulge → `grid[shiftedIdx]` resample. Prefer `Math.round(shift/CELL)` over `| 0` for sub-cell responsiveness.
- **Scroll-driven logo dissolve**: `logoOpacity = 1 - clamp(scrollY / (0.9 * innerHeight), 0, 1)` — fade completes just before `#about`. Each logo cell has a random `logoBlink[idx]` threshold; when `blinkT > logoOpacity` the cell "dissolves back" — it must still draw as a BG-wall `pickOff()` glyph, NOT skip rendering (leaving holes breaks the effect). Edge zone (`edge < 0.08`) uses 55% random flicker between bright/wall for a blinking transition.
- **Pointer listeners bind to `window`**, not `hero` — canvas is viewport-sized so `clientX/Y` map directly to canvas coords. `IntersectionObserver` pause was removed (canvas is always visible); only `visibilitychange` pauses the render loop.

### Hero canvas external toggles (driven by solution stack)
The hero IIFE exposes three setters on `window` so the solution-stack `updateBgState` can react when the user enters/leaves a video-backed slot:

| Function | Effect |
|---|---|
| `window.setHeroRainEnabled(bool)` | Pauses yellow-rain spawn + clears in-flight drops when `false`. |
| `window.setHeroWallAlpha(0..1)` | Replaces the hardcoded 0.16 wall alpha. Used to dim the wall to `0.08` over BG-video slots so the video remains the dominant texture. |
| `window.triggerHeroWallSweep(dir)` | Starts the wall-sweep transition. `dir > 0` → top→down; `dir < 0` → bottom→up. |

Solution stack always calls all three from `updateBgState` so they stay in sync with `bgKey` changes.

### ASCII wall sweep transition
- `SWEEP_DURATION = 1700ms`; band height `h * 0.30`; eased with cubic in-out.
- During the active window, cells inside the moving band get a per-frame chance to render `CHAR_ON` (alphabet) instead of the usual `pickOff()` wall glyph. Spawn probability = `proximity² × 0.55` (0% at the band edge, ~55% at the center). Selected cells pick a random `CHAR_ON` char each frame so the band looks scattered + flickering, not a solid bar.
- Triggered from `updateBgState` only when `bgKey !== lastBgKey && bgKey !== null` — i.e. entering or changing video slots. Direction comes from scroll direction (`onScroll`) or click-driven idx delta (`skipToSlot`).

## Viewport-snap (page scroll)
- `initViewportSnap()` IIFE intercepts `wheel` (preventDefault, debounce 200ms, cooldown 1000ms) and `touchend` to advance to the next anchor instead of free-scrolling.
- Anchors are recomputed each call: `[0, aboutTop + j*vh while inside about, stackTop + i*vh for i 0..3, stackTop + 4*vh (slot-3 hold), contactTop, maxScroll]`. The marquee between sections is intentionally NOT an anchor — wheel scroll skips it. Contact has only its entry anchor — once the probe is inside contact, `inFreeScrollZone` suppresses snap entirely.
- The `stackTop + 4*vh` "hold" anchor was added so snapping from slot-03 entry doesn't leap straight to Contact (a 2vh jump that visually skipped slot 3).
- **Free-scroll zones**: `inFreeScrollZone()` returns true when the mid-viewport probe lies inside a section that should scroll natively. Contact is **always** free-scroll (long form + expandable items pair poorly with viewport snap). About is free-scroll **below 1700px** because the two-column layout overflows on common laptop widths (1440/1536/1600); only ≥1700px monitors fit it in one viewport, where snap is fine. Both `wheel`/`touchend`/`keydown` early-return when the probe is in a free-scroll zone. Snap stays active for the solution stack at all widths.
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

## Custom cursor
- `.cursor` element is a fixed yellow filled circle (`background: var(--accent); border: 1px solid var(--accent); border-radius: 50%`) with `mix-blend-mode: difference` — so it inverts against whatever is below.
- Hover state (`.cursor.hover`) grows to 48px + stays yellow.
- View-service mode (`.cursor.mode-view`, set on `.solution` hover) grows to 120px with `WATCH SERVICE` label.
- Hidden + native cursor restored at `≤768px`.

## Responsive breakpoints
| Breakpoint | What changes |
|---|---|
| `<1700px` | About switches to free-scroll (snap suppressed inside about). Two-column layout still applies — only the snap behavior changes. |
| `≤1199px` | Nav links collapse to logo + sub + CTA only. Pillar-big `<p>` drops `max-width: 320px`. Contact stacks left-on-top, right-below. About `.about-grid` switches from flex space-between to a single-column flex stack (left's `display: contents` flows children into the grid). About-media moves to order 4 with `aspect-ratio: 4/3` landscape, `width: 100%`, `align-self: stretch` (no max-width cap on tablet/mobile). About loses `min-height: 100vh` (becomes content-sized). |
| `≤900px` | Pillar grid collapses to 1 column. About card spacing tightens. |
| `≤768px` | Hero-bottom becomes 2-row centered. Nav padding/logo size shrinks. Hero ASCII logo enlarges 20% (`computeGrid` uses `0.897` vs desktop `0.749` — both bumped +30% from earlier `0.69 / 0.576`). Contact form spacing tightens. |
| `≤640px` | **JS mobile threshold.** `.solution-stack` height auto, `.solution-stage` non-sticky, slots become `position: relative` and stack vertically with `opacity: 1 !important`. `.solution-bg-stack` hidden (in-slot section-video re-shows for mobile). `.solution-nav` hidden. ASCII burst skipped on this size. |
| `≤380px` | Nav `.sub` (TECHNOLOGY/SERVICES) hides too — only logo + CTA. |

## Gotchas
- **Working directory is in Google Drive and has spaces** — quote every shell argument (`"$WORKDIR/og-image.png"`, not `$WORKDIR/og-image.png`).
- **SVG `<use href="#ydm-logo">` needs explicit `viewBox="0 0 620.07 240.36"` on the outer `<svg>`** — otherwise width defaults to the user-agent's 300px and the logo stretches/misaligns.
- **Nav is transparent at all scroll positions** — the `.scrolled` class still toggles past 50px but its style block is a no-op. Do not reintroduce `mix-blend-mode: difference` (conflicts with `backdrop-filter`).
- **Page-level scroll-snap is removed.** `html { scroll-snap-type: y proximity }` and `section { scroll-snap-align: start }` were removed because they bounced when scrolling through the absolute-positioned solution slots inside `.solution-stage`. Don't reintroduce them; viewport-snap is JS-driven now.
- **Solution slot 0 reveal uses `rect.top <= window.innerHeight * 0.5`, not `<= 0`** — the older zero-threshold meant the 300ms `DELAY_MS` only started after snap landed, leaving ~600ms of blank slot that read as "scroll once more". Triggering when the stack is half a viewport away lets the reveal animation finish just as the slot pins. Don't reintroduce a strict-zero threshold.
- **The marquee between About and the solution-stack was removed.** The `id="unstoppable"` lives on the bottom marquee (after Contact). Keep the hero `<a href="#unstoppable">` pointing there; do not re-add the middle marquee.
- **Contact form is taller than 100vh and ALWAYS free-scroll.** `#contact` overrides the base `section { justify-content: center; min-height: 100vh }` with `justify-content: flex-start; min-height: auto; padding: 140px 0 120px` so the form doesn't overflow above the section. `inFreeScrollZone()` returns true whenever the probe is inside `#contact` regardless of viewport width, and `getAnchors()` adds only a single entry anchor (`contact.offsetTop`) — no per-vh inner anchors. Don't re-add per-vh contact anchors; the long form + expandable items are unusable when paginated.
- **About-grid is flex space-between with fit-content + max-width caps (≥1700px).** `.about-grid { display: flex; justify-content: space-between; gap: 80px }` with `.left { width: fit-content; max-width: 960px }` and `.about-media { width: 640px; align-self: center }`. Leftover space sits in the middle on wide displays. About-media uses `aspect-ratio: 3/4` portrait on desktop. ≤1199px the grid switches to a single-column stack and the media flips to `aspect-ratio: 4/3` landscape full-width — see Responsive breakpoints.
- **About video media-placeholder uses `aspect-ratio: 3/4` portrait on desktop / `4/3` landscape on tablet/mobile.** Container width drives height via the aspect-ratio CSS property. The actual `about-video.mp4` is portrait 1080×1280, so `object-fit: cover` center-crops to fill whichever container shape is active. The desktop placeholder is NOT a flex container (`display: block`) — earlier `display: flex + align-self: stretch` was dragging the placeholder taller than its aspect ratio.
- **`.section-bg` and `.solution-ascii` are hidden by a TEMP CSS block** in the style block — the BG-video stack at z:-2 + the hero canvas overlay replace their roles. The original markup is preserved in case the design reverts. The TEMP block is labelled `/* TEMP: ... */` and is the single place to remove if you want the slot-internal video, gradient overlay, and burst-canvas behavior back.
- **Solution BG videos must NOT live inside `.solution-stage`.** Sticky position creates a stacking context that traps internal z-indexes — videos at z:-2 inside the stage still paint above the hero canvas at viewport z:-1. The fixed `.solution-bg-stack` outside the stage is the workaround.
- **`updateBgState` is the single source of truth** for syncing BG video, hero rain, hero wall alpha, and wall-sweep trigger to the active slot. Both `onScroll` (passes scroll-direction `dir`) and `skipToSlot` (passes idx-comparison `dir`) call it — the click path explicitly because `skipMode` suppresses `onScroll`.
- **OG image is generated via Chrome headless** from a temporary `og-template.html` (deleted after render). Render at `--window-size=1200,720` (Chrome headless reserves ~90px vertically — `1200,630` silently clips the bottom of the layout) into `og-image-raw.png`, then top-left crop to 1200x630 with a tiny Swift script using `CGImage.cropping(to:)` (`sips -c` and `--cropOffset` only do center crops; no PIL/ImageMagick on this machine).

## Workflow
- Design iterates rapidly via small commits. User reviews in a local Live Server (auto-reload) — do NOT call `open <file>` after each edit.
- Commits should be meaningful; each push auto-deploys to production.
- Spec + implementation plan live under `docs/superpowers/`.

## Dev shortcuts
- Syntax-check the inline `<script>` without running it: `node -e "const fs=require('fs');const h=fs.readFileSync('index.html','utf8');const m=h.match(/<script>([\s\S]*)<\/script>/);try{new Function(m[1]);console.log('OK')}catch(e){console.log('ERR:',e.message)}"`
- Check CSS brace balance: `python3 -c "import re; s=open('index.html').read(); css=re.search(r'<style>([\s\S]*?)</style>', s).group(1); print(css.count(chr(123)), 'vs', css.count(chr(125)))"`
- No `ffmpeg`, `opencv`, or PyObjC on this machine. For `.mov` references: `qlmanage -t -s 1200 -o /tmp file.mov` (one thumbnail), or Swift + `AVAssetImageGenerator` + `NSBitmapImageRep` for multiple frames.
- Read about-video metadata: `mdls -name kMDItemPixelHeight -name kMDItemPixelWidth about-video.mp4`.
