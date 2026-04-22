# YDM Technology Service — Landing Page Deploy Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ship the existing `ydm-tech-v3.html` landing page to Vercel via a GitHub-backed auto-deploy pipeline, with polish-layer metadata (SEO, favicon, OG image, README).

**Architecture:** Static HTML hosted on Vercel. Single `index.html` at repo root. GitHub repo connected to Vercel for auto-deploy on push to `main`. No framework, no build step, no backend.

**Tech Stack:** HTML5, CSS (inline in page), Google Fonts (Prompt), git, GitHub CLI (`gh`), Vercel, Chrome headless (for OG image generation).

**Context:** Working directory is `/Users/charintip.b/Library/CloudStorage/GoogleDrive-charintip@adyim.com/My Drive/Claude/YDM-Techservice/`. Run all commands from this directory. Spec: `docs/superpowers/specs/2026-04-22-deploy-landing-page-design.md`.

**"Testing" note:** Static HTML has no unit-test harness. Each task's verification step is a concrete command (`ls`, `grep`, `curl`, visual check in browser) with explicit expected output. Treat these as the test equivalents.

---

## Task 1: Initialize git repository

**Files:**
- Create: `.gitignore`

- [ ] **Step 1: Verify not already a git repo**

Run:
```bash
git rev-parse --is-inside-work-tree 2>&1 || echo "not a repo (expected)"
```
Expected: `not a repo (expected)` (or similar "not a git repository" message).

- [ ] **Step 2: Initialize git with `main` as default branch**

Run:
```bash
git init -b main
```
Expected: `Initialized empty Git repository in ...`

- [ ] **Step 3: Create `.gitignore`**

Write `.gitignore` with:
```
.DS_Store
node_modules/
.vercel/
*.log
.env
.env.local
.omc/
```

- [ ] **Step 4: Verify the file exists**

Run:
```bash
cat .gitignore
```
Expected: the seven lines above.

---

## Task 2: Rename landing page to `index.html`

**Files:**
- Rename: `ydm-tech-v3.html` → `index.html`

- [ ] **Step 1: Rename the file**

Run:
```bash
mv ydm-tech-v3.html index.html
```

- [ ] **Step 2: Verify**

Run:
```bash
ls -la index.html ydm-tech-v3.html 2>&1 | head -3
```
Expected: `index.html` listed with size ~57KB; `ydm-tech-v3.html` shows "No such file or directory".

---

## Task 3: Create `favicon.svg`

**Files:**
- Create: `favicon.svg`

- [ ] **Step 1: Look up the accent color from `index.html`**

Run:
```bash
grep -E "^\s*--accent\s*:" index.html | head -1
```
Expected: a line like `--accent: #<hex>;` — note the hex value for use in the favicon.

- [ ] **Step 2: Write `favicon.svg`**

Create `favicon.svg` containing a square dark background with the letters "YDM" in the accent color. Use this content (replace `ACCENT_HEX` with the value from Step 1, e.g. `#E4FF00` — use the exact hex observed):

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 64 64">
  <rect width="64" height="64" fill="#0A0A0A"/>
  <text x="32" y="42" text-anchor="middle"
        font-family="Prompt, Arial, sans-serif"
        font-size="22" font-weight="700"
        fill="ACCENT_HEX">YDM</text>
</svg>
```

- [ ] **Step 3: Verify the file renders**

Run:
```bash
open favicon.svg
```
Expected: the file opens in the default SVG viewer (Preview / browser) showing a black square with yellow "YDM" text. Close after confirming.

---

## Task 4: Inject polish metadata into `index.html` `<head>`

**Files:**
- Modify: `index.html` (lines 1–10, the `<head>` block)

- [ ] **Step 1: Read the current `<head>`**

Run:
```bash
sed -n '1,12p' index.html
```
Capture the current state. Existing lines are:
```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>YDM Technology Services — Transformation at the Front Line of Technology</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Prompt:..." rel="stylesheet">
<style>
```

- [ ] **Step 2: Replace the `<title>` line with title + meta description + favicon + OG + Twitter tags**

Use Edit tool to replace the exact block:

old_string:
```html
<title>YDM Technology Services — Transformation at the Front Line of Technology</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
```

new_string:
```html
<title>YDM Technology Services — Transformation at the Front Line of Technology</title>
<meta name="description" content="YDM Technology Service — transformation partner at the front line of technology. Enterprise platforms, data, and AI engineering.">
<meta name="theme-color" content="#0A0A0A">
<link rel="icon" type="image/svg+xml" href="/favicon.svg">
<link rel="canonical" href="https://ydm-technology-service.vercel.app/">

<meta property="og:type" content="website">
<meta property="og:title" content="YDM Technology Services">
<meta property="og:description" content="Transformation at the front line of technology.">
<meta property="og:url" content="https://ydm-technology-service.vercel.app/">
<meta property="og:image" content="https://ydm-technology-service.vercel.app/og-image.png">
<meta property="og:image:width" content="1200">
<meta property="og:image:height" content="630">

<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="YDM Technology Services">
<meta name="twitter:description" content="Transformation at the front line of technology.">
<meta name="twitter:image" content="https://ydm-technology-service.vercel.app/og-image.png">

<link rel="preconnect" href="https://fonts.googleapis.com">
```

Note: `canonical` and `og:url` hardcode the expected Vercel URL. If the actual deploy URL differs, Task 11 covers the post-deploy fixup.

- [ ] **Step 3: Verify metadata present**

Run:
```bash
grep -c 'og:\|twitter:\|canonical\|description\|theme-color\|favicon' index.html
```
Expected: `>= 14` (all new meta/link tags plus existing ones).

- [ ] **Step 4: Open locally to confirm no layout regression**

Run:
```bash
open index.html
```
Expected: page renders identically to before (visually). Favicon tab shows the YDM mark. Close browser when confirmed.

---

## Task 5: Generate `og-image.png` via Chrome headless

**Files:**
- Create: `og-template.html` (temporary, deleted at end of task)
- Create: `og-image.png`

- [ ] **Step 1: Read the accent color again for consistency**

Run:
```bash
grep -E "^\s*--accent\s*:" index.html | head -1
```
Note the hex value (same as Task 3).

- [ ] **Step 2: Create `og-template.html`**

Write a 1200×630 template using the accent color. Replace `ACCENT_HEX` with the hex from Step 1:

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<link href="https://fonts.googleapis.com/css2?family=Prompt:wght@400;700&display=swap" rel="stylesheet">
<style>
  html, body { margin: 0; padding: 0; width: 1200px; height: 630px; }
  body {
    background: #0A0A0A;
    color: #F5F5F0;
    font-family: 'Prompt', sans-serif;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    padding: 72px 88px;
    box-sizing: border-box;
  }
  .mark { font-size: 16px; letter-spacing: 0.3em; color: ACCENT_HEX; text-transform: uppercase; font-weight: 500; }
  h1 {
    font-size: 96px; line-height: 1.02; font-weight: 700;
    letter-spacing: 0.005em; text-transform: uppercase; margin: 0;
  }
  h1 em { font-style: normal; color: ACCENT_HEX; }
  .foot { font-size: 14px; letter-spacing: 0.25em; color: #7A7A75; text-transform: uppercase; }
</style>
</head>
<body>
  <div class="mark">YDM / Technology Service</div>
  <h1>Transformation at the<br><em>Front Line</em> of Technology</h1>
  <div class="foot">ydm-technology-service.vercel.app</div>
</body>
</html>
```

- [ ] **Step 3: Render to PNG using Chrome headless**

Run (one command):
```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless \
  --disable-gpu \
  --hide-scrollbars \
  --window-size=1200,630 \
  --screenshot="$PWD/og-image.png" \
  "file://$PWD/og-template.html"
```
Expected: prints `[...INFO:headless_shell.cc(...)] Written to file .../og-image.png`.

- [ ] **Step 4: Verify dimensions**

Run:
```bash
file og-image.png
```
Expected: `og-image.png: PNG image data, 1200 x 630, ...`

- [ ] **Step 5: Visual check**

Run:
```bash
open og-image.png
```
Expected: sees black card with "YDM / TECHNOLOGY SERVICE" small label top-left, large headline with "Front Line" in accent color, URL footer. Close preview when confirmed.

- [ ] **Step 6: Delete the template file**

Run:
```bash
rm og-template.html
```

---

## Task 6: Add `vercel.json` (minimal static config)

**Files:**
- Create: `vercel.json`

- [ ] **Step 1: Create `vercel.json`**

Write `vercel.json`:
```json
{
  "cleanUrls": true,
  "trailingSlash": false
}
```

- [ ] **Step 2: Validate JSON**

Run:
```bash
python3 -c "import json; json.load(open('vercel.json')); print('ok')"
```
Expected: `ok`.

---

## Task 7: Write `README.md`

**Files:**
- Create: `README.md`

- [ ] **Step 1: Create `README.md`**

Write `README.md`:
```markdown
# YDM Technology Service — Landing Page

Static landing page for YDM Technology Service.

## Live

https://ydm-technology-service.vercel.app/

(URL updates automatically when pushed to `main`.)

## Local preview

```bash
open index.html
```

No build step. No dependencies. Single-file HTML with inline CSS and Google Fonts (Prompt).

## Deploy workflow

- Push to `main` → Vercel deploys to production.
- Push to any other branch → Vercel creates a preview URL (commented on the PR if one exists).

## Structure

- `index.html` — the landing page
- `favicon.svg` — browser tab icon
- `og-image.png` — social-share preview (1200×630)
- `vercel.json` — hosting config
- `docs/superpowers/` — design spec and implementation plan
```

- [ ] **Step 2: Verify**

Run:
```bash
head -20 README.md
```
Expected: first 20 lines match the content above.

---

## Task 8: Initial commit

**Files:** all of the above, staged together.

- [ ] **Step 1: Stage files**

Run:
```bash
git add .gitignore index.html favicon.svg og-image.png vercel.json README.md docs/
```

- [ ] **Step 2: Confirm staged contents**

Run:
```bash
git status
```
Expected: `new file:` entries for each file listed above under "Changes to be committed". No unstaged changes other than possibly `YDM-Tech-Landing-Content.docx` (which is intentionally untracked).

- [ ] **Step 3: Verify `.docx` is NOT staged**

Run:
```bash
git status --porcelain | grep -E '\.docx$' || echo "docx not tracked (good)"
```
Expected: `docx not tracked (good)`.

- [ ] **Step 4: Commit**

Run:
```bash
git commit -m "feat: initial landing page with deploy pipeline

Static HTML landing page for YDM Technology Service with
polish-layer metadata (SEO, OG, Twitter), favicon, and
OG share image. Configured for Vercel deploy."
```
Expected: `[main (root-commit) <sha>] ...` with 7+ files changed.

---

## Task 9: Create GitHub repo and push

**Files:** none (remote operation).

- [ ] **Step 1: Verify `gh` auth**

Run:
```bash
gh auth status 2>&1 | head -5
```
Expected: includes `✓ Logged in to github.com account bcharintip`.

- [ ] **Step 2: Create the repo and push in one step**

Run:
```bash
gh repo create ydm-technology-service \
  --public \
  --source=. \
  --remote=origin \
  --push \
  --description "YDM Technology Service — static landing page (AI-assisted end-to-end deploy demo)"
```
Expected:
- `✓ Created repository bcharintip/ydm-technology-service on GitHub`
- `✓ Added remote https://github.com/bcharintip/ydm-technology-service.git`
- `✓ Pushed commits to https://github.com/bcharintip/ydm-technology-service.git`

- [ ] **Step 3: Verify remote is set**

Run:
```bash
git remote -v
```
Expected: `origin https://github.com/bcharintip/ydm-technology-service.git (fetch/push)`.

- [ ] **Step 4: Open the repo in browser to confirm**

Run:
```bash
gh repo view --web
```
Expected: browser opens to the repo page; README is rendered, files are visible.

---

## Task 10: Connect Vercel and deploy (manual one-time step)

**Files:** none (done in Vercel dashboard).

- [ ] **Step 1: Open Vercel new-project page**

Run:
```bash
open "https://vercel.com/new"
```
Sign in if needed (use GitHub if prompted).

- [ ] **Step 2: Import the GitHub repo**

In the Vercel UI:
- Under "Import Git Repository", find `bcharintip/ydm-technology-service` and click **Import**.
- If the repo isn't listed, click **Adjust GitHub App Permissions** and grant access.
- **Project Name:** leave as `ydm-technology-service`
- **Framework Preset:** `Other`
- **Root Directory:** `./` (default)
- **Build Command:** leave empty / default
- **Output Directory:** leave empty / default (Vercel serves the root)
- **Install Command:** leave empty
- Click **Deploy**.

- [ ] **Step 3: Wait for first deploy**

The deploy should complete in under 30 seconds (no build step). When done, Vercel shows a confetti screen and a "Visit" button.

- [ ] **Step 4: Capture the production URL**

From the dashboard, note the URL — expected to be one of:
- `https://ydm-technology-service.vercel.app` (if the name was available)
- `https://ydm-technology-service-<hash>.vercel.app` (if the simple name collided)

- [ ] **Step 5: Open the URL and confirm the page loads**

Run:
```bash
open "<production-url-from-step-4>"
```
Expected: landing page loads, looks identical to local preview, favicon visible in tab.

---

## Task 11: Post-deploy URL fixup (only if the live URL differs from the assumed one)

**Files:**
- Modify: `index.html` (canonical/og:url/og:image/twitter:image), `README.md` (live URL line)

- [ ] **Step 1: Check whether fixup is needed**

If the production URL from Task 10 Step 4 is exactly `https://ydm-technology-service.vercel.app/`, **skip this entire task** — the hardcoded values are already correct.

If it differs, continue.

- [ ] **Step 2: Replace the URL in `index.html`**

Use Edit tool with `replace_all: true` to swap:
- old_string: `https://ydm-technology-service.vercel.app`
- new_string: `<actual-production-url-without-trailing-slash>`

- [ ] **Step 3: Replace the URL in `README.md`**

Use Edit tool to swap the same string in `README.md`.

- [ ] **Step 4: Commit and push**

Run:
```bash
git add index.html README.md
git commit -m "fix: correct canonical and OG URLs to match deployed host"
git push
```
Expected: push triggers a new Vercel deploy automatically.

---

## Task 12: Verification

**Files:** none (observational).

- [ ] **Step 1: HTTP status check**

Run (substitute `<prod-url>` with the actual URL):
```bash
curl -sI "<prod-url>" | head -5
```
Expected: `HTTP/2 200` and reasonable headers (content-type `text/html`, a `cache-control` header).

- [ ] **Step 2: Favicon reachable**

Run:
```bash
curl -sI "<prod-url>/favicon.svg" | head -3
```
Expected: `HTTP/2 200` and `content-type: image/svg+xml`.

- [ ] **Step 3: OG image reachable**

Run:
```bash
curl -sI "<prod-url>/og-image.png" | head -3
```
Expected: `HTTP/2 200` and `content-type: image/png`.

- [ ] **Step 4: Social preview check**

Open the social preview tool in a browser:
```bash
open "https://www.opengraph.xyz/"
```
Paste `<prod-url>` into the input field and submit. Expected: the preview renders the OG image and "YDM Technology Services" title correctly across the Facebook / Twitter / LinkedIn / Discord preview tabs. If the image is missing or shows 404, re-check Task 11 (URL fixup) and re-push.

- [ ] **Step 5: Auto-deploy smoke test**

Run:
```bash
echo "" >> README.md
git add README.md
git commit -m "chore: trigger auto-deploy smoke test"
git push
```
Expected: within ~30 seconds, a new deployment appears in the Vercel dashboard with status "Ready". `curl -sI "<prod-url>"` still returns 200.

- [ ] **Step 6: Preview-URL smoke test**

Run:
```bash
git checkout -b preview-smoke-test
echo "<!-- preview smoke test -->" >> index.html
git add index.html
git commit -m "test: preview deploy smoke test"
git push -u origin preview-smoke-test
```
Expected: within ~30 seconds, Vercel creates a preview deployment at a URL distinct from production (e.g. `ydm-technology-service-git-preview-smoke-test-bcharintip.vercel.app`). Check the Vercel dashboard. Then clean up:
```bash
git checkout main
git branch -D preview-smoke-test
git push origin --delete preview-smoke-test
```

- [ ] **Step 7: Final report to user**

Summarize in chat:
- Production URL
- GitHub URL
- Verification checklist: all green
- Next actionable: "design edits can now be made to `index.html`; each push to `main` auto-deploys."

---

## Done criteria

All tasks checked. Live URL returns 200. Pushing a commit deploys automatically. Non-main branch produces a preview URL. OG image renders in social-preview tools. Favicon visible in browser tab.
