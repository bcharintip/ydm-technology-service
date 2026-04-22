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
