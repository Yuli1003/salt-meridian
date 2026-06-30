# Salt Meridian

> Coastal Basque cooking, fired over driftwood — a 30-seat driftwood asador above the harbor in San Sebastián.

Built with **Wix Headless** + **Astro 5** + **React 19 islands** + **Tailwind v4**, per the HEADLESS DAY brief (`spec-0101-salt-meridian.md`).

The site renders fully **before any Wix setup** because every CMS read falls back to local seed data (`src/data/content.ts`). Link your Wix project, create the collections, and it lights up with live, owner-editable content — no code changes.

---

## Quick start

```bash
npm install
cp .env.local.example .env.local      # then paste your WIX_CLIENT_ID
npm run dev
```

`WIX_CLIENT_ID` comes from **Dashboard → Settings → Headless Settings → OAuth apps**.
The `@wix/astro` integration reads it (via `astro:env`) and wires the Wix SDK auth
context on both server and client automatically — so `items.query()` / `items.insert()`
just work, no manual `createClient`.

> The dev server runs the Wix auth middleware on every request, so it needs a **real**
> `WIX_CLIENT_ID` (plus the server vars below) to render. To preview the UI without any
> Wix credentials, build the static, CMS-free variant: `npm run build:static && npx astro preview --config astro.config.verify.mjs`.

Server-side env vars the integration also expects once linked (in `.env.local`):
`WIX_CLIENT_INSTANCE_ID`, `WIX_CLIENT_PUBLIC_KEY`, `WIX_CLIENT_SECRET`.

---

## Link a Wix Headless project

If this folder isn't linked yet:

```bash
npm create @wix/new@latest -- headless link
```

(or scaffold fresh elsewhere with `npm create @wix/new@latest` and copy `src/` in).

---

## CMS collections

Create these in **Dashboard → CMS → Collections**. Field ids must match exactly
(they map 1:1 to `src/data/content.ts`). Then seed them: `npm run seed` (below).

### `BoardDish`  — the menu (≥ 8 rows)
| Field id | Type | Notes |
|---|---|---|
| `name` | Text | Dish name |
| `section` | Text | One of: From the Sea, From the Fire, From the Garden, To Drink, To Finish |
| `price` | Text | e.g. `€58 / kg` |
| `catchTag` | Text | `Today's Catch`, `Signature`, or empty |
| `tastingDescription` | Text (long) | Short description |
| `image` | Text | An image key (`hero`, `txuleta`, `grillBars`, …) — see `src/lib/images.ts` |
| `order` | Number | Sort order within section |

### `Review` — homepage quotes (3 rows)
| Field id | Type |
|---|---|
| `name` | Text |
| `quote` | Text (long) |
| `detail` | Text |

### `StoryBlock` — The Fire page (1 row)
| Field id | Type |
|---|---|
| `heading` | Text |
| `body` | Text (long) |

### `TodaysCatch` — the live banner (1 row) — **bonus challenge**
| Field id | Type |
|---|---|
| `heading` | Text |
| `body` | Text (long) |
| `updatedLabel` | Text |

The owner edits this single row in the CMS; the `TodaysCatchBanner` island reads it
**client-side** (`client:visible`), so changes appear without a redeploy.

### `Reservations` — inquiry collection (booking form target)
| Field id | Type |
|---|---|
| `name` | Text |
| `phone` | Text |
| `date` | Text/Date |
| `partySize` | Number |
| `seatingPreference` | Text |
| `note` | Text (long) |
| `status` | Text |

**Permissions:** set `Reservations` so **Anyone** can *add* items (form submits as a
site visitor). Keep the content collections admin-managed.

---

## Seed content

```bash
WIX_API_KEY=...  WIX_SITE_ID=...  npm run seed          # add --truncate to reset
```

`WIX_API_KEY`: **Dashboard → Settings → API Keys** (grant *Wix Data*).
`WIX_SITE_ID`: **Dashboard → Settings → Headless Settings**.

---

## Imagery

Ships with verified Unsplash CDN URLs (fast, always-on). To generate **on-brand**
imagery and host it on Wix Media (permanent `static.wixstatic.com`, AVIF/WebP via
`enc_auto`):

```bash
WIX_SITE_ID=...  npm run generate:images          # or: ... npm run generate:images hero room
```

This calls Wix's Runware image API (model `google:4@2`, Nano Banana Pro) with
art-directed prompts, imports each result into your Media library, and patches the
media ids into `src/lib/images.ts`. The resolver (`img()` / `srcset()`) then serves
Wix-hosted images automatically. Confirm `RUNWARE_PATH` in the script matches your
project's image-inference route.

---

## Deploy

```bash
npx wix build
npx wix release      # prints the live *.wix-site-host.com URL
```

`npx wix build` supplies the production server adapter (the committed
`astro.config.mjs` deliberately omits one). Submit the released URL.

---

## Project map

```
src/
  pages/            6 routes: / /menu /about /reservations /gallery /contact
  layouts/          BaseLayout — head/SEO, nav, footer, mobile sticky bar
  components/       Astro UI + islands/ (React)
    islands/        TodaysCatchBanner (client:visible), ReservationForm (client:idle),
                    NavBookButton (client:load), MobileBookStickyBar (client:load),
                    MapFacade (client:visible)
  lib/              wix auth is auto-wired; cms.ts (queries + seed fallback),
                    images.ts (resolver), site.ts (business facts), schema.ts (JSON-LD)
  data/             content.ts — seed content (mirrors the CMS collections)
  styles/           global.css (theme, fonts, ember-glow device) + components.css
scripts/            seed-cms.mjs, generate-images.mjs
```

### Brief coverage
- **Requirements:** 10-dish menu w/ prices + descriptions · 8-image gallery · contact w/ map facade, hours, tap-to-call · mobile-first responsive.
- **Bonus:** owner-editable **Today's Catch** banner (live CMS read).
- **SEO:** Restaurant + Menu + FAQPage JSON-LD, OG/Twitter, canonical, semantic landmarks.
- **Perf/a11y:** LCP hero preloaded w/ `fetchpriority=high` + responsive `srcset`; self-hosted woff2 (`font-display:swap`); ember-glow via CSS scroll-driven animation (no JS); facades for map; honors `prefers-reduced-motion`; 44px targets; visible focus; skip link.
- **Hydration plan** matches the brief (see `islands/`).
```
