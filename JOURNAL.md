# Production Journal

## 2026-05-21

### Re-anchored Project State

- Confirmed the workspace has both `production/` and `production-luxury/`.
- Treated `production-luxury/` as the preferred luxury visual direction.
- Treated `production/` as the deployable production folder because it already
  had the multi-page funnel, package lock, Vite config, and installed
  dependencies.

### Luxury Homepage Port

- Copied the luxury homepage into `production/index.html`.
- Copied `production-luxury/src/main.js` into `production/src/main.js`.
- Copied `production-luxury/src/styles.css` into `production/src/styles.css`.
- Copied `production-luxury/public/assets/` into `production/public/assets/`.
- Generated responsive AVIF/WebP variants with `npm run optimize:images`.

### Preserved Production Funnel

- Kept these pages in place:
  - `collections.html`
  - `product.html`
  - `bespoke.html`
  - `heritage.html`
  - `contact.html`
- Kept existing funnel scripts:
  - `js/data.js`
  - `js/ui.js`
  - `js/whatsapp.js`
  - `js/analytics.js`
- Added a Vite build hook to copy legacy `js/`, `images/`, and `videos/` folders
  into `dist/`.

### Homepage Integration Fixes

- Replaced stale `/product-detail.html` links with production product routes.
- Added the floating WhatsApp CTA to the luxury homepage.
- Loaded `js/analytics.js` and `js/whatsapp.js` on the homepage.
- Added WhatsApp button and modal styles to `src/styles.css`.
- Fixed the CTA image source set to use the generated `diamond-macro-1080`
  variant instead of a missing `diamond-macro-1280` variant.
- Added static hero fallback sizing so the hero image fills the viewport when
  the video is disabled for mobile, reduced-data, or reduced-motion users.

### Broken Path Cleanup

- Replaced fragile parent-directory media paths in `bespoke.html` and
  `heritage.html`.
- `bespoke.html` now references `/assets/videos/atelier.mp4` for the hero video.
- `bespoke.html` and `heritage.html` now use `/assets/images/diamond-macro.jpeg`
  instead of `../website/images/product_image1.jpeg`.

### Size Pass

- Initial luxury merge build was about 107 MB.
- After legacy path cleanup it was about 97 MB.
- Temporarily removed `hero2.mp4` and `atelier.mp4`, reducing the build to about
  18.7 MB.
- User requested the hero and atelier videos back.
- Restored:
  - `public/assets/videos/hero2.mp4`
  - `public/assets/videos/atelier.mp4`
- Restored homepage hero video and atelier video references.
- Restored `public/assets/asset-manifest.json` video entries.

### Verification

- `cmd /c npm run build` completed successfully.
- Local dev server was started at `http://localhost:5173/`.
- Confirmed route health for:
  - `/`
  - `/collections.html`
  - `/product.html?id=maharani-viraasat`
  - `/product.html?id=raj-tilak-emerald`
  - `/bespoke.html`
  - `/heritage.html`
  - `/contact.html`
- Confirmed local `href` and `src` references were reachable after cleanup.
- Confirmed core scripts and assets returned `200`.

### Current Caveats

- `hero2.mp4` is about 61 MB and `atelier.mp4` is about 21 MB. Final deployment
  should compress these videos if performance is a priority.
- Vite warnings about non-module legacy scripts are expected because those files
  are copied as static compatibility assets instead of bundled.
- Browser automation was not available in this environment, so checks were done
  through build, route, asset, and markup inspection.

### Header Check Against Approved `website/`

- Compared `production/index.html` header against approved `website/index.html`.
- Confirmed the production header preserves the approved essentials:
  - logo link
  - Collections, Bespoke, Heritage, Contact links
  - Search Archive control
  - language selector
  - theme toggle
  - The Inquiry CTA
  - mobile menu with Home and page links
- Added approved compatibility details to the production luxury header:
  - `data-i18n` keys on header links and CTA text
  - `id="lang-selector"` and `id="lang-selector-mobile"`
  - `id="theme-icon"` and `id="theme-icon-mobile"`
  - styled `archive-search-text` to match the approved search control intent
- Updated `src/main.js` so theme icons sync between light and dark modes and
  header text updates through the same `data-i18n` mechanism.

### Product Management System

- Created a CSV-driven product management workflow.
- **Folder structure:** `products/{category}/{product-id}/` — 7 categories,
  21 products, each with a dedicated folder for images and videos.
- **Master catalog:** `product-inventory.csv` — single source of truth with
  columns: id, category, subcategory, name, shortDesc, description, priceRange,
  stone, metal, weight, cert, featured, imagePath, videoPath.
- **Sync script:** `scripts/sync-products.cjs` — reads the CSV and regenerates
  `src/data.js` automatically.
- **Workflow:** Edit CSV in Excel → drop real product images into product
  folders → run `node scripts/sync-products.cjs` → `npm run build`.
- Verified build passes after CSV-to-JS sync.

### Decap CMS — Client Product Management UI

- Installed Decap CMS at `/admin` for client-facing product editing.
- **`public/admin/index.html`** — CMS entry point (loads decap-cms-app).
- **`public/admin/config.yml`** — Product model with all fields (id, name, category,
  subcategory, price, specs, featured, images, description via markdown editor).
  Includes category filters, editorial workflow, and sort options.
- **`api/oauth.js`** — Vercel serverless function for GitHub OAuth authentication.
- Products are now stored as individual markdown files in `products/{category}/{id}.md`
  with YAML frontmatter. The CMS reads/writes these files directly on GitHub.
- **`scripts/csv-to-md.cjs`** — One-time migration script (CSV → markdown files).
- **Updated `scripts/sync-products.cjs`** — Now reads from `products/*.md` files
  instead of CSV, generates `src/data.js` for the Vite build.
- CSV is preserved as a human-readable reference / bulk-import format.
- Updated `vercel.json` with `api/oauth.js` function config.

**Setup required for production (manual, one-time):**
1. Go to GitHub → Settings → Developer settings → OAuth Apps → New OAuth App
2. Set Homepage URL to `https://houseofgiriraj.vercel.app`
3. Set Callback URL to `https://houseofgiriraj.vercel.app/api/oauth?provider=github`
4. Copy Client ID and generate Client Secret
5. Add to Vercel env vars: `GITHUB_CLIENT_ID` and `GITHUB_CLIENT_SECRET`
6. Redeploy

## 2026-05-21 (later)

### CMS "No Entry" Bug — Depth Fix

- Products were stored in `products/{category}/{id}.md` (2 levels deep).
- The CMS backend proxy (`entriesByFolder`) sent `depth: 1`, which only returned
  category directories, not the .md files inside them.
- **Fix:** Flattened files to `products/{id}.md` and removed `{{category}}/` from
  the CMS slug config. This let the CMS find all files at depth 1.

### Image Management in CMS

- Added `image` widget (widget: `image`) to the CMS config for uploading product images.
- `media_folder: public/assets/images/products` — CMS saves uploads here.
- `public_folder: /assets/images/products` — public URL path for images.
- **Issue:** The CMS stores image paths as absolute URLs (`/assets/images/products/...`)
  rather than relative paths. Fixed `sync-products.cjs` to handle both formats.
- **Fallback order:** `image` (CMS Main Image) > first gallery image > legacy `imagePath`.

### Gallery and Video Support

- Added **Image Gallery** (list of image+caption) and **Videos** (list of video+poster)
  to the CMS config.
- Updated `product.html` with a thumbnail strip below the main image. Clicking a
  thumbnail swaps the main display (image or video).
- Gallery uses `js-yaml` for proper YAML frontmatter parsing (supports lists/objects).

### File Watcher for Auto-Sync

- Added `npm run dev:sync` — uses nodemon to watch `products/*.md` and auto-run
  `sync-products.cjs` on every CMS save.
- Keeps `src/data.js` in sync without manual script runs.

### Per-Category CMS Collections

- Restructured from a single flat "Products" collection into 7 category-specific
  collections: Chokers, Necklaces, Chandeliers, Bracelets, Bangles, Rings, Studs.
- Each collection appears as its own section in the CMS sidebar.
- Files moved back to `products/{category}/{id}.md` (subdirectories).
- Category is now inferred from the directory path in `sync-products.cjs`, not
  from the frontmatter field.
- The CMS editor no longer shows a category selector — it's implicit from which
  collection the product lives in.

### Current Terminal Workflow

| Terminal | Command |
|---|---|
| 1 | `npx decap-server` |
| 2 | `npm run dev` |
| 3 | `npm run dev:sync` |

### Cleanup: Remove Legacy imagePath/videoPath

- Removed `imagePath` and `videoPath` columns from CSV (all image management is
  now CMS-based).
- Removed `imagePath`/`videoPath` from `csv-to-md.cjs` fields.
- Removed `imagePath` fallback from `sync-products.cjs` (relying only on `image`
  and gallery fields).
- **Important caveat:** Running `csv-to-md.cjs` after CMS edits will overwrite
  CMS-managed data (images, gallery, videos). The CSV is now a seed/reference
  file only.

### SEO: JSON-LD + Sitemap

- Added `Organization` schema JSON-LD in `<head>` of `product.html`.
- Added dynamic `Product` schema JSON-LD (name, description, images, category,
  offers/price) to each product page.
- Integrated sitemap generation into `sync-products.cjs` — produces
  `public/sitemap.xml` with 34 URLs (6 static pages, 7 category filters,
  21 product pages). Auto-updated on every sync.

### Known Caveats

- `main` variable name conflicted with page-scoped `main` in product.html gallery
  JS — renamed to `galleryMain`.
- CSV re-import overwrites CMS data — only use `csv-to-md.cjs` for initial bulk import.

### Product Image Folders — Category-Nested Structure

- **Created** 21 product image folders at `public/assets/images/products/{category}/{product-id}/`
  (7 categories × 3 products each).
- **Restructured** 3 existing image sets into the nested convention:
  - `ekta-lineage/` → `bracelets/ekta-lineage/`
  - `maharani-viraasat/` → `necklaces/maharani-viraasat/`
  - `raj-tilak/` → `necklaces/raj-tilak-emerald/` (also renamed to match product ID)
- **Added** `image: "{category}/{product-id}/hero.jpg"` field to all 21 product `.md` frontmatter files.
- **Regenerated** `src/data.js` via `sync-products.cjs` — all 21 products now have populated image paths.
- **Git note**: git auto-detected the moves as renames (100% similarity), preserving history.

### Master Inventory Excel

- Created `product details/master-inventory.xlsx` with 3 sheets:
  - **Sheet 1 — All Inventory**: 119 consolidated items from 5 supplier XLS files with Status/Mapped Product columns.
  - **Sheet 2 — Product Mapping**: 9 website products mapped to inventory tags, showing found/missing status and specs.
  - **Sheet 3 — Missing Tags**: 5 unmatched tags needing sourcing (12725, 12325, 12536, 12316, 12294).
- Extracted real specs for 3 matched pieces from inventory (dynasty-bloom/12695, royal-lace/12509, ruby-aurora/12479).

### House Collection Copy — Inventory-Populated Descriptions

- **Dynasty Bloom**: Updated category to "Emerald, Pearl & Diamond"; description now reads *"33 carats of brilliant diamonds, 44 carats of vivid emeralds, and 26 carats of lustrous pearls converge in an opulent floral garland..."*
- **Royal Lace**: Description now reads *"65 carats of precisely calibrated diamonds set in an intricate openwork lattice..."*
- **Ruby Aurora**: Description now reads *"49 carats of diamonds frame 79 carats of vivid rubies in a composition of ceremonial grandeur..."*
- **Regenerated** `src/data/house-collection.js` via `npm run sync:house`.

### Hero Visual Refinement

- **Reduced title font-size**: `clamp(3.6rem, 9vw, 10.5rem)` → `clamp(2.8rem, 6.5vw, 7.5rem)` (~28% smaller max).
- **Pushed text to bottom**: Changed `.hero` from `place-items: center` → `place-items: end center`;
  `.hero-copy` padding reduced from `7rem 0 5rem` → `0 0 2.5rem`.
- **Merged headline**: "Where Value / Takes Form." → single-line "Where Value Takes Form." (merged two `.split-line` spans).

## 2026-05-29

### CTA Label Consolidation Fixes

- **Home page house collection**: Changed from "WhatsApp Consultation" (button with `data-wa-btn`) to
  `View Full Collection` (`<a>` linking to `/collections.html`).
- **Product page secondary CTA**: Label changed from "WhatsApp Consultation" to "Enquire on WhatsApp",
  keeps the `data-wa-btn` WhatsApp funnel trigger.

### Maharani Cascade Necklace — Hero Product

- Added as a **full-width hero row** at the top of The House Collection on the homepage.
- 3 images at `public/assets/images/collection/celestial-cascade/` (celestial-cascade-1.png, -2.jpg, -3.png).
- **Data source**: Created `src/data/house-collection-entries/celestial-cascade.md` with YAML frontmatter
  (multi-line description using `|` literal block, `isHero: true`).
- **Rendering**: `src/main.js` — extracted `createHouseCard()` from `renderHouseCollection()`;
  hero piece (`isHero: true`) rendered first as `.row-hero` before the regular grid.
- **CSS**: Added `.row-hero` (full-width via `grid-template-columns: 1fr`, 3/2 aspect ratio image,
  larger title, full description with `white-space: pre-line` for paragraph breaks).
- **Build pipeline fix**: `sync-house-collection.cjs` now includes `isHero` field in output.
- **Bug fix**: Sync script treated `data.row = 0` and `data.homepageOrder = 0` as falsy,
  producing `null` and `1` instead. Changed `||` to `!= null` checks.
- **Count updated**: "Eight curated high jewellery masterpieces" → "Nine" in `index.html`.
- **Image fix**: Second image changed from `.png` to `.jpg` — updated markdown entry to match.
- **ID rename**: Directory renamed from `celestial-cascade` to `maharani-cascade` — updated markdown id and image paths to match; renamed markdown file to `maharani-cascade.md`.

### House Collection Layout Rebalance

- **Row layout**: Changed from 1+3+3+2 to **1+2+2+2+2** (hero row + 4 rows of 2 pieces each).
- **Reassigned rows** across 5 markdown files:
  - `imperial-cascade.md`: row 1→2, order 3→1
  - `dynasty-bloom.md`: order 1→2
  - `ruby-aurora.md`: row 2→3, order 2→1
  - `celestial-rain.md`: row 2→3, order 3→2
  - `emerald-reverie.md`: row 3→4, order 2→2
  - `royal-lace.md`: row 3→4, order 1→1

### Card Aspect Ratio

- **Changed** `.house-card-image` from `aspect-ratio: 4/5` to `3/4` (taller display).
- **Max-height** bumped from `600px` to `700px` to avoid cropping on the taller ratio.

### Hero Subtitle Redesign

- **Translucent background**: Added `background: rgba(0,0,0,0.35)` + `backdrop-filter: blur(2px)` on `.hero-subtitle`.
- **Font**: Switched from Manrope 300 → **Italianno** (flowing cursive script) for decorative elegance.
- **Styling**: `letter-spacing: 0.04em` for elongated appearance, white text at 0.88 opacity, inline-block display so the background wraps only the text content.
- **Intermediate font tried**: Alex Brush (too uniformly thick for the "thin and long" request), Italianno selected as thinner naturally.
- **Google Fonts**: Added `Italianno` to the `index.html` font import.

### Hero Title Font-Size Reduction

- **Reduced desktop font-size**: `clamp(3.6rem, 9vw, 10.5rem)` → `clamp(2.8rem, 6.5vw, 7.5rem)`.
- **Pushed text to bottom**: `.hero` from `place-items: center` → `place-items: end center`;
  `.hero-copy` padding `7rem 0 5rem` → `0 0 2.5rem`.
- **Merged headline**: Two `.split-line` spans combined into one —
  "Where Value Takes Form." single line.

### Mobile Button Overlap Fix

- Hero button on mobile overlapped with WhatsApp FAB due to full-width sizing.
- **Fix**: `.hero-actions .button` on ≤640px: `min-height: 2.5rem; padding: 0.5rem 1.25rem;
  width: auto; align-self: flex-start`.

### Hero Title Mobile 3-Line Break Fix

- At smaller widths the title wrapped to 3 lines.
- **Fix**: Mobile font-size reduced from `clamp(3.15rem, 16vw, 4.9rem)` →
  `clamp(2.4rem, 12vw, 3.8rem)`; `max-width: none` on mobile override.

### Desktop Hero Title Max-Width Removal

- `max-width: 12ch` removed from `.hero-title` so desktop displays on one line
  without forced wrapping.

### Video Playback Fallback

- Non-Chrome mobile browsers (Firefox, Samsung Internet) failed to play H.264
  videos due to `moov` atom at end of file (requires full download before playback).
- **JS poster fallback**: `initVideoFallbacks()` in `src/main.js` catches `play()`
  promise rejection and swaps `<video>` to `<img>` with the poster image.
- Also adds a `stalled` event listener as additional trigger.

### FFmpeg Faststart — All Videos Optimized

- `ffmpeg` installed via `winget install Gyan.FFmpeg` (found at
  `%LOCALAPPDATA%\Microsoft\WinGet\Packages\...\ffmpeg.exe`).
- All 6 site videos processed with `-movflags +faststart -codec copy`:
  - `collections-hero.mp4` (20.5s, 17.7 Mb/s) — hero background
  - `stone-dictates-form.mp4` (6s, 17.1 Mb/s) — editorial section
  - `atelier.mp4` (9.6s, 17.6 Mb/s) — atelier section
  - `Maharani_Cascade-film.mp4` (10s, 17.6 Mb/s) — hero product trailer
  - `hero2.mp4` — already had faststart (secondary hero)
  - `curation.mp4` — already had faststart (curation section)
- **Result**: All 6 verified moov-at-start via binary scan. Firefox, Safari, and
  Samsung Internet can now stream without full download.
