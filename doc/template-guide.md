# Storefront template — usage guide

This repo is a portable storefront template (Astro 7 + Tailwind v4, static build, no runtime backend).
Porting it to a new restaurant/project = **copy the code, declare the data objects, rebrand, verify**.
The code has no hardcoded business content: everything the site shows comes from the fixture objects in `src/data/api/`.

## Quick path

1. Copy the template files into the new project; `pnpm install`.
2. Declare the 9 data objects in `src/data/api/` with the new restaurant's content.
3. Rebrand: `site` in `astro.config.mjs`, tokens in `src/storefront/styles/global.css` and `src/styles/tokens.css`, assets in `public/`.
4. Verify: `pnpm build && node scripts/smoke.mjs` (update smoke expectations to the new data — see below).

## What to copy

Every path belongs to an update zone (next section): **Core** = template-managed, **App** = yours forever, **Composition** = shared — conflicts only where you edit.

| Path | Role | Zone |
| --- | --- | --- |
| `src/storefront/` | The whole domain — the portable core: `services/` (data access + fallbacks), `types/` (contracts), `menu/` (blocks, cards, product detail), `commerce/` (cart), `layout/`, `ui/` (modals), `home/`, `enlaces/`, `icons/`, `scripts/`, `styles/`, `utils/` | Core |
| `src/pages/` | The four routes: `index.astro` (landing), `menu.astro`, `galeria.astro`, `enlaces.astro` | Composition |
| `src/data/api/` | Fixture objects — copy as placeholders, then replace with your declared data | App |
| `src/styles/tokens.css` | Brand: fonts (`@font-face`) + Tailwind v4 `@theme` tokens | App |
| `public/` | Static assets (logos, images, favicon) | App |
| `astro.config.mjs`, `package.json`, `tsconfig.json`, `pnpm-lock.yaml`, `.gitattributes` | Tooling — Astro, Tailwind v4 via `@tailwindcss/vite`, sitemap integration, update-zone rules | Core |
| `scripts/smoke.mjs` | End-to-end smoke over CDP (headless Chrome), no test framework needed | Core |
| `scripts/smoke-data.json` | Smoke expectations pinned to your data (CTA count, headings, min items, labels) | App |

Not portable by design: `doc/` (project history), `.env*` (local), `dist/` (build output).

## Update zones

A template is only updatable where the owner did NOT edit — the same reason `node_modules` exists. Each copied file belongs to exactly one zone:

| Zone | Paths | Rule |
| --- | --- | --- |
| **Core** | `src/storefront/`, `src/pages/`, `scripts/smoke.mjs`, tooling | Don't edit — extend through the seams instead: design slots in `menu.astro`, fixtures, tokens. Template updates apply cleanly here. |
| **App** | `src/data/api/*.json`, `src/styles/tokens.css`, `public/`, `scripts/smoke-data.json` | Yours forever. `.gitattributes` marks these `merge=ours`, so an update merge never overwrites your data, brand, or assets. |
| **Composition** | The parts of `src/pages/*.astro` / `menu.astro` you customize | Conflicts appear only where you edited; resolve by hand (keep yours or take the template's). |

## Applying an update

The scaffolder wires the `upstream` remote and the copied version for you:

```bash
git fetch upstream && git merge upstream/vX.Y.Z
```

- Untouched core files merge clean.
- App files are protected by `merge=ours` — your data and brand win.
- Conflicts are confined to composition files you actually edited.

Releases stay small and frequent with a CHANGELOG; breaking changes ship with a migration note in this guide — small merges are what keep updates practical.

## Rule of thumb

**Edit the fixture/token, not the component.** If a change truly needs a core edit, that file leaves the update channel from then on — a decision, not a failure.

## Objects to declare

All 9 live in `src/data/api/`. Every loader in `loadStorefront()` (`src/storefront/services/index.js`) has a per-resource fallback, but a missing object means a missing section — declare them all.

| Object | Shape | Feeds |
| --- | --- | --- |
| `settings.json` | `{ data: {...} }` — the business object | Header, footer, WhatsApp ordering, hours, SEO, geo, i18n |
| `menu.json` | `{ data: { blocks: [ { id, categories: [ { id, name, description, block_id, products: [...] } ] } ] } }` | The menu page and product dialogs |
| `combos.json` | `{ data: [ { id, name, description, image, price, price_before } ] }` | Combo cards (`price_before` renders the struck-through price) |
| `promotions.json` | `{ data: [ { id, name, description, image, price, price_before, valid_until } ] }` | Promotion cards + dialog hosts |
| `gallery.json` | `{ data: [ { id, title, category, type, url, alt, description, aspect_ratio?, embed_html? } ] }` | `/galeria` masonry; `type` = `image` \| `video` \| `embed` |
| `landing.json` | `{ meta, blocks: [ { type, ... } ] }` | Declarative landing blocks — block types render via `src/storefront/home/` |
| `enlaces.json` | `{ meta, blocks: [ { type, enabled, ... } ] }` | Linktree page; blocks with `enabled: false` are filtered out; types: `link`, `banner`, `product-carousel`, `video`, `gallery-grid` |
| `flyers.json` | `{ enabled, delay_seconds, show_once_per_session, items }` | Flyer modal popup |
| `theme.json` | `{ data: { page, brand, share, header, blocks } }` | Visual decoration overlays |

### Product shape (menu / combos / promotions)

Products carry their own price range (`price_range`) with embedded `entries` — no join needed. `price_before` on combos/promotions renders the struck-through original price.

## Data validation

Every fixture is validated against its Zod schema at build time (`src/storefront/schemas/`). Schemas are strict: unknown keys fail the build. A missing fixture falls back to a safe empty shape; a schema violation **throws and fails the build** — it is never silently converted into a fallback.

## Smoke test

`scripts/smoke.mjs` runs an end-to-end check over headless Chrome (CTA count, headings, min items, labels). Expectations are pinned in `scripts/smoke-data.json` — update them to match your data.

Run it after scaffolding and after each data change:

```bash
pnpm build && node scripts/smoke.mjs
```

## What not to edit

Do not edit Core zone files (`src/storefront/`, `src/pages/`, `scripts/smoke.mjs`, tooling). If a change truly needs a core edit, that file leaves the update channel from then on. Extend through the seams instead: design slots in `menu.astro`, fixtures, tokens.

## Per-page docs

See `doc/pages/index.md`, `doc/pages/menu.md` (not yet written), `doc/pages/galeria.md`, `doc/pages/enlaces.md` for route-specific notes.
