# True "Export to Canva" — Backend Spec

The in-app button currently renders the PNGs and opens Canva for manual upload, because a
static page can't authenticate to Canva. To make graphics land in the user's Canva account
automatically (one click, no manual upload), you need a small backend that talks to the
**Canva Connect API**. Here's the full plan.

---

## 1. What Canva requires

- A **Canva Developer account** + a registered **Integration** (gives you a `client_id` /
  `client_secret`). https://www.canva.com/developers/
- The integration must request these **scopes**: `design:content:write`, `asset:write`
  (and `profile:read` for the connect handshake).
- Each end user authorizes your integration once via **OAuth 2.0 (PKCE)**. You store their
  refresh token so future exports don't re-prompt.
- Relevant endpoints:
  - `POST /v1/asset-uploads` — upload an image, returns an `asset.id`.
  - `POST /v1/designs` — create a design; can place uploaded assets.
  - (Import API / `POST /v1/imports` if you'd rather push a whole multi-page file.)

> Canva has no public client-side "import these images" URL — that's why this must be a
> server. Your `client_secret` and users' tokens can never live in the shipped HTML.

---

## 2. Architecture

```
Browser (the Builder app)
   │  1. user clicks "Export as Canva"
   │  2. app renders the 7 PNGs (already works) → POSTs them to your backend
   ▼
Your backend  (Node/Express, Cloudflare Worker, or a serverless function)
   │  3. has the user's Canva token (from a one-time OAuth connect)
   │  4. uploads each PNG via /v1/asset-uploads
   │  5. creates a design (or multi-page design) from the assets
   ▼
Canva  → returns design URLs → backend returns them to the app → app shows links
```

## 3. Backend endpoints to build

| Route | Purpose |
|---|---|
| `GET  /auth/canva/start` | Begin OAuth (PKCE); redirect user to Canva consent. |
| `GET  /auth/canva/callback` | Exchange code → access+refresh tokens; store per user. |
| `POST /export/canva` | Body: the rendered PNGs (multipart or base64). Uploads each as an asset, creates design(s), returns the Canva links. |
| `POST /auth/refresh` (internal) | Use refresh token to get a fresh access token when expired. |

## 4. Token storage

- Minimum: a small DB (Postgres, DynamoDB, or even SQLite) keyed by your app's user id,
  holding `{ canva_refresh_token, canva_user_id }`.
- Encrypt refresh tokens at rest. Never return them to the browser.

## 5. Front-end change (small)

Replace the current "download + open Canva" handler with:
```js
const imgs = await renderAll(btn);                 // already implemented
const form = new FormData();
imgs.forEach(im => form.append('files', im.blob, im.name));
const res = await fetch('https://your-backend.com/export/canva', {
  method: 'POST', credentials: 'include', body: form
});
const { designs } = await res.json();              // [{name, url}, ...]
// show the links / open the first one
```
First-time users get bounced through `/auth/canva/start` once; after that it's one click.

## 6. Hosting options (pick one)

- **Cloudflare Workers / Pages Functions** — cheapest, scales to zero, easy secrets.
- **Vercel / Netlify serverless functions** — same repo as the static app.
- **A tiny Node/Express app** on Render/Railway/Fly.io.

## 7. Effort estimate

- OAuth connect + token storage: ~0.5–1 day.
- Asset upload + design creation + wiring the button: ~0.5–1 day.
- Testing across the 7 formats + error handling: ~0.5 day.
- **Total: ~2–3 days** for a developer familiar with OAuth.

## 8. Security checklist

- `client_secret` only on the server, in env vars / secrets manager.
- Restrict CORS on `/export/canva` to your app's domain.
- Validate file count/size server-side before forwarding to Canva.
- Rotate/refresh tokens; handle revoked-access (re-prompt) gracefully.

---

### Hand-off note
The static app is already doing step 2 (it renders all 7 PNGs as blobs in `renderAll()`).
A developer only needs to (a) stand up the backend above and (b) swap the button handler
in `Webinar Graphics Builder.html` to POST those blobs instead of downloading them.
