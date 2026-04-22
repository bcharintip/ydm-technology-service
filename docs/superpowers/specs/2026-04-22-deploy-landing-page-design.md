# YDM Technology Service — Landing Page Deploy Pipeline

**Date:** 2026-04-22
**Status:** Approved (design phase)
**Owner:** bcharintip

## Goal

Set up a production-style deploy pipeline for the existing `ydm-tech-v3.html` landing page so that:

1. The page is live at a public URL (Vercel-hosted).
2. Future design/content edits push through a real dev workflow: `git commit → git push → Vercel auto-deploy`.
3. The repository demonstrates end-to-end AI-assisted development work (commit history, polished metadata, live preview deployments) — this is a capability demo for stakeholders, not a customer-facing production launch.

The *design of the page itself* is expected to keep evolving after deploy; this spec covers the pipeline and polish layer only, not a content freeze.

## Non-Goals

- Custom domain registration or DNS configuration
- Framework migration (Next.js / Astro / etc.)
- Contact form or any backend / serverless functions
- Analytics integration
- Multi-page structure or i18n switcher
- Refactoring the single-file HTML into components
- Locking content before deploy (content can change freely after pipeline exists)

## Stack

| Concern | Choice | Reason |
|---|---|---|
| Page tech | Static HTML (unchanged) | Fastest path; no framework rewrite |
| Source control | Git + GitHub (`bcharintip/ydm-technology-service`, public) | Demonstrates real dev workflow; public repo lets stakeholders inspect history |
| Hosting | Vercel (GitHub integration) | Free tier, automatic preview deployments per branch, trivial static-site support |
| CLI tooling | `gh` (already installed, authenticated as `bcharintip`) | Automates repo creation + push |

## Repo Layout

```
ydm-technology-service/
├── index.html            # renamed from ydm-tech-v3.html
├── favicon.svg           # inline SVG, "YDM" mark on dark background
├── og-image.png          # 1200x630, social share card
├── README.md             # project summary + live deploy URL
├── .gitignore            # standard (.DS_Store, node_modules, etc.)
├── vercel.json           # optional: static config / clean URLs
└── docs/superpowers/specs/
    └── 2026-04-22-deploy-landing-page-design.md   # this file
```

The reference content file `YDM-Tech-Landing-Content.docx` stays outside the repo — it's source material, not a web asset.

## Deploy Flow

```
1. init git repo in working directory
2. rename ydm-tech-v3.html → index.html
3. add .gitignore, README.md, favicon.svg, og-image.png, vercel.json
4. inject polish <meta> tags into index.html <head>
5. initial commit (includes spec doc in docs/)
6. gh repo create bcharintip/ydm-technology-service --public --source=.
7. git push -u origin main
8. On vercel.com: Import Project → select GitHub repo
   - Framework Preset: Other
   - Build Command: (none)
   - Output Directory: .
9. Vercel deploys → URL like ydm-technology-service.vercel.app
10. From here, any `git push` auto-triggers a new production deploy;
    branches get preview URLs automatically.
```

Step 8 is manual in the Vercel dashboard (one-time). Everything else is scripted via `gh` and `git`.

## Polish Layer (content-safe)

These items can be added now without locking page content, because they describe the site as a whole, not section-specific copy.

### `<head>` metadata

Inject into `index.html`:

- `<meta name="description" content="...">` — one-line summary derived from the existing tagline block already in the page
- `<meta property="og:title">`, `og:description`, `og:url`, `og:image`, `og:type="website"`
- `<meta name="twitter:card" content="summary_large_image">` + twitter:title/description/image
- `<link rel="icon" type="image/svg+xml" href="/favicon.svg">`
- `<link rel="canonical" href="https://ydm-technology-service.vercel.app/">` (updated once the real URL is known)

Keep `<html lang="en">` as-is for now — if the final copy shifts primarily to Thai, update in a follow-up commit.

### Favicon

Inline SVG, text-mark style: uppercase "YDM" in the site's accent yellow on the site's black background. No external asset dependency; ships in the repo as `favicon.svg`.

### OG Image

1200×630 PNG generated once. Shows "YDM Technology Service" wordmark on the black background with the accent yellow used elsewhere on the page. Stored at `og-image.png` in repo root.

### README

Purpose: give anyone (including the stakeholder) immediate context when they hit the GitHub page.

Sections:
- What it is (one paragraph)
- Live URL (filled in after first Vercel deploy)
- How to run locally (`open index.html` — that's it)
- Deploy workflow (one-liner: "push to `main` → auto-deploy on Vercel")

### `vercel.json`

Minimal config:
```json
{
  "cleanUrls": true,
  "trailingSlash": false
}
```

Only added if it measurably helps; otherwise omit and let Vercel's defaults handle it.

## Verification

After deploy, confirm:

1. Live URL loads, shows the landing page identical to opening `ydm-tech-v3.html` locally
2. Favicon shows in browser tab
3. `curl -I <url>` returns 200 + reasonable cache headers
4. Pasting the URL into a social-preview tool (e.g. `opengraph.xyz`) renders the OG image and title correctly
5. Pushing a trivial commit (e.g. whitespace tweak) triggers a new Vercel deployment within ~30s
6. Non-main branch push produces a preview URL distinct from production

## Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Google Fonts (Prompt family) fails in certain regions | Already using `preconnect`; acceptable for demo. Self-host as follow-up if needed. |
| Vercel free tier build limits | Static site = effectively unlimited; not a concern at this scale. |
| `og:url` / `canonical` hardcoded before Vercel URL is known | Deploy first to get the actual URL, then update `<head>` in a follow-up commit. |
| Content in HTML diverges from `.docx` source | Explicitly out of scope for this spec; content reconciliation happens later. |

## Out-of-Scope Follow-ups (noted, not in this plan)

- Custom domain + DNS (e.g., ydm.co.th / ydmtech.com)
- Plausible or GA4 analytics
- Self-host Google Fonts for offline / privacy compliance
- Convert to Next.js if multi-page features become needed
- Thai localization and `lang="th"` switchover
