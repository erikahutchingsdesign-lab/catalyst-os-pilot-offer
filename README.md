# CatalystOS — Overview Page + Demos

Static site: the CatalystOS overview page plus three linked interactive demos.

## Contents

```
index.html                     ← entry point (the overview page)
Demo Access.dc.html            ← same page, original filename
support.js                     ← runtime (required, must sit next to the pages)
brand/                         ← logos
demos/
  CatalystOS Interactive Laptop.dc.html   ← Platform Dashboard demo
  Post Builder Laptop.dc.html             ← Post Builder demo
  event-graphics/
    Event Graphics Laptop.dc.html         ← Event Graphics demo
    app/ ...                               ← inner builder + assets
  postbuilder/ ...                         ← inner builder + assets
  support.js                               ← runtime for the demos
```

## IMPORTANT: must be served over HTTP

The page and demos load their content and nested builders with `fetch()`.
`fetch()` is **blocked on the `file://` protocol**, so **do not open the HTML
files by double-clicking them** — the demos will appear blank.

Always serve over HTTP.

### Test locally

From this folder:

```bash
python3 -m http.server 8000
# then open http://localhost:8000/
```

or

```bash
npx serve .
```

## Deploy to GitHub Pages

1. Create a new repository and push the **contents of this folder** to the root
   (so `index.html` is at the repo root).
2. In the repo: **Settings → Pages → Build and deployment → Source: Deploy from a branch**,
   pick your branch and `/ (root)`, save.
3. Open the published URL. The demos load in iframes and work automatically.

The included `.nojekyll` file tells GitHub Pages to serve every folder as-is
(needed because some asset folders would otherwise be skipped).

## Deploy anywhere else

Any static host works (Netlify, Vercel, Cloudflare Pages, S3 + CloudFront, nginx).
Upload the whole folder; no build step is required.
