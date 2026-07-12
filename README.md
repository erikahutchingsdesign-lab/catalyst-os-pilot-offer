# CatalystOS — Overview Page (Vercel-ready static site)

A responsive single-page marketing/proposal site for **CatalystOS: The AI Marketing Platform**,
with three embedded auto-playing product demos (Platform Dashboard, Event Graphics Builder,
Post Builder).

Pure **static site** — no build step, no server code. Runs entirely in the browser and is tuned
to work well on phones as well as desktop.

## Deploy to Vercel

### Option A — Drag & drop (fastest, no GitHub)
1. Go to https://vercel.com/new
2. Drag this whole folder onto the page (or zip it and upload).
3. Framework preset: **Other** (no build command, no output directory).
4. Deploy. Vercel serves `index.html` at `/`.

### Option B — GitHub → Vercel (auto-deploys on every push)
Put this folder's contents in your repo, then "Import" the repo in Vercel with framework
preset **Other**. Every push redeploys automatically.

### Option C — Vercel CLI
```bash
npm i -g vercel
cd catalystos-site
vercel
vercel --prod
```

## Mobile + cross-platform notes (verified)
- No horizontal scrolling at 360px / 390px widths.
- Layout stacks to a single column on phones; arrows rotate; proof cards fit.
- All three demos **auto-play on tap on mobile** as well as desktop — they're pure
  JavaScript "guided tours" (an animated cursor/scroll walking through the real app), so
  they run on any modern browser including iOS Safari and Android Chrome. No video autoplay,
  so nothing is blocked by mobile autoplay policies.
- Demo close button is a 44px tap target; iOS text auto-inflation disabled.
- Post Builder photo slots fall back to real headshots (no broken images).

## How it works
- `index.html` is the overview page. The three demos live under `demos/` and open in a modal.
- All runtime libraries (React, Babel, jsZip, html-to-image) are **vendored locally** in
  `vendor/` — no runtime dependency on any external CDN. Vercel compresses these automatically
  (Brotli/gzip), so the real download is much smaller than the raw file sizes.
- Requires HTTP(S). Opening files by double-clicking (`file://`) shows blank demos because
  browsers block `fetch()` on `file://`. Any static host (Vercel included) fixes this.
- The page is `noindex, nofollow` (confidential preview). Remove those meta tags in
  `index.html` to make it publicly indexable.
- `HANDOFF-NOTES.md` is the original design-handoff documentation.

## Local preview
```bash
cd catalystos-site
python3 -m http.server 8080
# open http://localhost:8080
```
