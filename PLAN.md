# Deepen Diamonds — Diamond Inventory & Catalog Platform

Replacement for the existing `deependiamonds.com/Stock` web tool. Adds a customer
mobile app, a storefront kiosk mode, and WhatsApp catalog sharing.

---

## 1. Product Scope

| Piece | Who | Purpose |
|---|---|---|
| **Customer App** — Flutter (PWA + Android + iOS) | Buyers | Login → filter diamonds → view list → **Hold** a stone. Mobile-first. |
| **Kiosk Mode** — same Flutter build | Storefront | Full-screen browse/filter display, no login / no hold. |
| **Admin Panel** — React (web) | Deepen staff | CSV inventory upload, customer approval, hold management, WhatsApp catalog share. |
| **Backend** — Node + PostgreSQL | — | REST/JSON API, auth, media, deep-link resolver. |

**In scope:** browse, filter, hold, notify admin, share catalog.
**Out of scope (deliberate):** payments, orders, invoicing, sales tracking.
Once a customer **Holds** a stone, everything is manual / direct communication.

---

## 2. Decisions (locked)

- **Inventory source:** Excel/CSV upload in admin, with a **column-mapping step**
  (define canonical schema; supplier files mapped at upload time).
- **Hold behavior:** stone flips to **Hold** (hidden/greyed for others) + **admin notified**;
  then manual. Matches current "Hold" column.
- **Stack:** Node + PostgreSQL API, **React** admin, **Flutter** front-end.
- **Customer access:** **invite-only**, admin approves. Prices only for approved buyers.
- **Pricing display:** **full** — Rap, discount %, $/ct, total amount (B2B buyers).
- **Catalog share:** **both** — single-stone deep link AND a saved filter/selection link.

---

## 3. Data Model (Postgres)

### `diamonds` — canonical schema (supplier files map to this)
| Field | Type | Notes |
|---|---|---|
| id | uuid PK | internal |
| stone_id | text | supplier's stock number (unique per batch) |
| status | enum | `available` \| `hold` \| `memo` \| `sold` |
| location | text | India, HK, … |
| lab | text | GIA, IGI, … |
| cert_no | text | certificate number |
| shape | text | Round, Pear, Princess, Marquise, Oval, Radiant, Emerald, Heart, Cushion, Triangle, Asscher |
| carat | numeric | |
| color | text | D–M, `<N`, fancy |
| clarity | text | FL, IF, VVS1/2, VS1/2, SI1/2 |
| cut | text | EX, VG, GD, FR |
| polish | text | EX, VG, GD, FR |
| symmetry | text | EX, VG, GD, FR |
| fluorescence | text | N, F, M, S |
| hearts_arrows | text | EX, VG, NONE |
| tb | text | TB0/1/2 (table black) |
| sb | text | SB0/1/2 (side black) |
| measurements | text | e.g. 4.30x4.32x2.68 |
| depth_pct | numeric | |
| table_pct | numeric | |
| rap | numeric | Rapaport price |
| discount_pct | numeric | Rap% (usually negative) |
| price_per_ct | numeric | $/Ct |
| amount | numeric | total = price_per_ct × carat |
| image_url | text | |
| video_url | text | |
| cert_url | text | |
| batch_id | uuid FK | which upload it came from |
| created_at / updated_at | timestamptz | |

Indexes on the hot filter columns: shape, color, clarity, carat, status, location.

### `customers`
id, name, company, phone, email, **status** (`pending`/`approved`/`blocked`),
price_visibility (`full` for now; enum kept for future), approved_by, created_at.

### `holds`
id, diamond_id FK, customer_id FK, created_at, released_at, note.
Holding sets `diamonds.status = 'hold'`; releasing reverts to `available`.

### `catalogs` (shared links)
id, slug (short), type (`stone` | `filter` | `selection`), payload (jsonb: stone_id or
filter criteria or stone_id list), created_by, expires_at (nullable), created_at.

### `upload_batches`
id, filename, column_mapping (jsonb), row_count, uploaded_by, created_at.

### `admin_users`
id, name, email, password_hash, role, created_at.

---

## 4. Backend API (Node + Postgres)

- **Auth:** admins = email/password (JWT). Customers = phone/email + OTP, invite-gated.
- `GET /diamonds` — filter + paginate + sort (mirrors current filter set).
- `POST /holds` / `DELETE /holds/:id` — customer hold / admin release. Fires admin notify.
- `POST /admin/upload` — CSV/XLSX → preview → column-map → commit batch.
- `GET/POST /admin/customers` — list, approve, block.
- `POST /admin/catalogs` — create share link (stone / filter / selection).
- `GET /c/:slug` — **deep-link resolver** (see §6). Serves smart-banner + app/PWA redirect.
- **Notifications:** admin alert on new hold + new customer request (WhatsApp Cloud API or
  email to start; dashboard badge always).

---

## 5. Customer App (Flutter)

**Screens:** Login (OTP) → Filter/Search → Results list → Stone detail → My Holds.

- **Filter UI** = mobile redesign of the current filter grid: chip rows for Shape / Color /
  Clarity / Cut / Polish / Symm / Fluor / H&A / TB / SB, carat range slider, location, cert search.
- **Results list** = card layout (not the desktop table): image/video thumb, shape·carat·color·clarity
  headline, cut/pol/sym line, and price block (Rap, disc%, $/ct, amount). Sticky filter/sort bar.
- **Hold** = primary action on card + detail. Confirms, greys the stone, notifies admin.
- **Kiosk mode** = launch flag → no login, browse+filter only, auto-reset on idle, big touch targets.
- One codebase → PWA (web), Android, iOS.

**Design priority:** the results list must look premium and be effortless on a phone — this is
the headline deliverable.

---

## 6. Deep Linking (WhatsApp → app or PWA)

Admin shares `https://deependiamonds.com/c/<slug>`.
- **App installed** (Android App Links / iOS Universal Links) → opens Flutter app on that stone/filter.
- **Not installed** → opens the PWA to the same view, with an install prompt.
- Requires `assetlinks.json` (Android) + `apple-app-site-association` (iOS) hosted on the domain.
- The share message includes a preview image + headline (Open Graph tags on `/c/:slug`).

---

## 7. Build Phases

1. **Foundation** — repo scaffold (api / admin / app), Postgres schema + migrations, auth.
2. **Inventory pipeline** — CSV/XLSX upload → column-mapping → batch commit → `/diamonds` API.
3. **Customer app core** — login, filter UI, results list (the hero screen), stone detail.
4. **Hold + notify** — hold flow, status flip, admin notification + dashboard.
5. **Admin panel** — customer approval, inventory management, hold view.
6. **Catalog share + deep links** — link generation, `/c/:slug` resolver, App/Universal Links, OG preview.
7. **Kiosk mode + polish** — kiosk flag, idle reset, PWA install, store builds (Android/iOS).

---

## 7b. Branding

Client logo received (`brand/deepen-group-logo.ai`). Full kit in `brand/BRAND.md`.
- **Wordmark:** "deepen Group"; the **"p" has a negative-space diamond** = signature mark →
  use the diamond glyph as **app icon / favicon / kiosk splash**.
- **Primary brand blue:** `#1267E8` (sampled from the AI). Used for Hold button, active filter
  chips, links. Canvas `#F5F7FB`, ink `#0B1220`.
- Variants available: blue (light bg), white (dark bg / OG card), black (mono).

## 8. Open Items

- Sample supplier CSV to confirm the column-mapping presets.
- WhatsApp: Cloud API (automated) vs. click-to-chat links (manual) for admin notifications.
- Media hosting: are image/video URLs supplied in the feed, or uploaded separately?
- Domain/DNS access for App Links + Universal Links files.
