# Changelog

All notable changes to MobiNest (backend, database and project setup). Newest first.
Format: date · what changed · files/areas touched.

## 2026-11-13 (≈14:40): live demo deployment (Vercel + Railway)
- **Live:** storefront https://namma-mobiles.vercel.app · API https://mobinest-api-production.up.railway.app (Swagger at `/swagger-ui.html`). Full steps in **[DEPLOYMENT.md](DEPLOYMENT.md)**.
- **Backend → Railway** (project `namma-mobiles`, workspace classtapacademy): service `mobinest-api` built by Railpack (no Docker) + Railway MySQL. Flyway V1–V7 and the seeder ran on first start (3 brands, 18 phones).
- **Frontend → Vercel** (team class-thetapacadems-projects, project `namma-mobiles`): `vercel.json` proxies `/api/*` to Railway and falls back to `index.html` for client routes. Env: `VITE_CLERK_PUBLISHABLE_KEY`.
- **Code changes:** Java target 25 → **21** (Railway's JDK; 31/31 tests pass); `server.port: ${PORT:8080}`; `railway.json`, `.railwayignore`.
- **Bug fixed:** the ignore rule `config/` (for the local secrets folder) also hid the Java package `com.mobinest.config` (`AppProperties`, `SecurityConfig`, `OpenApiConfig`), so it was missing from GitHub and a clone could not compile. Rule anchored to `/config/`; files added (repo commit `74d4aea`). `mvnw` marked executable in git.
- **GitHub:** all pushed to https://github.com/classtapacademy/NAMMA-MOBILES (`2af5382`). The GitHub login is saved in Windows Credential Manager, so later pushes need no device code.

## 2026-11-12 (≈16:20): admin back office (Clerk + ADMIN role)
- **Access control:** `ClerkRoleJwtConverter` grants `ROLE_ADMIN` from `customer.role` (looked up by the JWT `sub`, cached 30 s, fail-closed; token role claims ignored). Bootstrap admins via `ADMIN_CLERK_USER_IDS` (`mobinest.admin.clerk-user-ids`, empty by default). `/api/admin/**`: 401 without/invalid token, 403 for non-admins.
- **Flyway V7 `admin_back_office`:** `order_status_history`, `admin_audit_log`, indexes on `orders(created_at)`, `orders(status, created_at)`, `sku(stock)`. Applied to the live DB.
- **Admin API (`com.mobinest.admin.*`):** dashboard KPIs; orders (filters, detail, timeline, status changes with restock on cancel, payment status); transactions + totals; brands/phones visibility, SKU and accessory price/stock (with stale-stock protection); coupons CRUD (soft delete); customers with order stats and role management; audit log.
- **Admin UI (`mobinest/src/pages/admin/**`, lazy `/admin/*` route with its own layout):** sign-in redirect for signed-out users, "No admin access" page for others; dashboard, orders, order detail, transactions, coupons, brands, phones, SKUs, accessories, customers & admins, audit log. Admin button on the Account page (admins only). Dev-only `?preview=1` with fake data (excluded from production builds).
- **Tests:** 31 pass (+23: role service/converter, admin rules, MockMvc security and endpoint tests with rolled-back writes).
- Docs: **[ADMIN.md](ADMIN.md)** (how to become admin, every endpoint, screenshots in `screenshots/admin/`).

## 2026-11-12 (≈18:00): admin side done, GitHub repo, final checks
- **Admin side (subagent), complete:**
  - Clerk + DB role (`ClerkRoleJwtConverter`, `CustomerRoleService`, bootstrap via `ADMIN_CLERK_USER_IDS`).
  - Flyway **V7 `admin_back_office`** (`order_status_history`, `admin_audit_log`).
  - `/api/admin/**`: dashboard, orders + status workflow, payments, catalogue, coupons, customers/roles, audit.
  - React `/admin/*` back office. 23 new tests; **31/31 pass**. See `ADMIN.md`.
- **Home polish (subagent):**
  - Trending as a segmented control with up to 12 phones per tab.
  - Budget cards with 3 product photos, no signal icon, tier typography.
  - Gallery: muted autoplay while visible, plus "Watch with sound".
  - Card text padding on grey bands (plus a site-wide `.section--tint .product-card__body` fix).
- **Fix:** malformed JSON on store endpoints now returns **400 `MALFORMED_REQUEST`** instead of 500.
- **GitHub:** monorepo prepared at `C:\Users\admin\NAMMA-MOBILES` for https://github.com/classtapacademy/NAMMA-MOBILES, containing `mobinest/`, `mobinest-api/`, a root README with screenshots and a root `.gitignore`.
  - The old saved GitHub login (`harshit-tapacademy`) was removed from Windows Credential Manager at the owner's request.
  - The first push must be done by the owner in a normal PowerShell window (`git push -u origin main`), signing in as the class account, because Claude Code's non-interactive shell can't show GitHub's sign-in.

## 2026-11-12 (≈17:00): brand hover effect + shareable guide
- **"Pick your favourite":** Aceternity **Card Hover Effect** (`components/BrandHoverRow.jsx`, `motion`). A soft blue highlight glides between brand cards, just outside the card edge; it follows keyboard focus and respects reduced motion. Chosen by the main session at the owner's request; the 5-option preview remains at `/design/brand-cards`.
- **`context/NAMMA-MOBILES-COMPLETE-GUIDE.md`:** the single shareable guide (setup from zero incl. your own Clerk keys, architecture, frontend, backend, DB, API, admin, payments, testing, recipes, troubleshooting, decisions, timeline, legal).
- **`context/setup/make-share-zip.ps1`:** builds a secret-free zip of both projects plus the guide (fails closed if a key or password is found).
- **`context/tools/e2e/`:** the end-to-end test packaged (README, package.json, configurable base URL and browser).
- **`mobinest/.env.example`:** template for students' Clerk keys.

## 2026-11-12 (≈16:30): owner feedback round
- **Google Pixel brand mark:** the full **Google** wordmark (Wikimedia Commons `Google_2015_logo.svg`, PD-textlogo, single colour) replaces the "Pixel" wordmark. All brand logos are left-aligned with the card titles.
- **Brand-card hover:** the blurred gradient layer is removed. It's now a very subtle *outer* halo (accent border plus a soft low-opacity shadow outside the card), with no inner tint and no lift.
- **Header search:** Aceternity **Placeholders and Vanish Input** (adds the `motion` package), adapted in `components/ui/placeholders-and-vanish-input.jsx`:
  - theme tokens, rotating example searches, particle "vanish" on submit, then navigation to results;
  - keeps the live suggestions panel, Escape and the ARIA attributes;
  - reduced-motion aware.
- **Fix:** suggestion thumbnails in the search dropdown were full-width; they're now 40px (the generic `.product-image` rule was overriding them).
- **In progress (subagents):** home polish (trending tabs, more phones, budget-card images, no price-meter icon, tier typography, autoplaying gallery, left-padding fixes in Certified Refurbished and Accessories); the five brand-card design options; the admin side.

## 2026-11-12 (≈15:45): Razorpay draft quarantined
- The payments subagent, briefed to write **only** `context/PAYMENTS.md`, started a Razorpay implementation. It said it was following a direct owner message asking for full integration; this is pending the owner's confirmation. Its migration `V6__razorpay_payments.sql` clashed with the existing V6 (`add_iphone_18_pro_max`), and its `CustomerOrder` changes referenced columns that don't exist.
- Action:
  - The draft classes, the migration and copies of the modified files were moved to `context/quarantine/razorpay-draft/` (restorable).
  - `AppProperties`, `application.yml`, `CustomerOrder`, `OrderEnums` and `OrderRepository` were restored to their previous versions.
  - `SkuRepository`/`AccessoryRepository` keep a harmless `incrementStock` (used to release stock on cancellation).
  - Nothing was applied to the database.
  - Unit tests pass, and Flyway validates V1–V6.
- If the owner wants the full integration, the migration must be renumbered (V8+; V7 is reserved for the admin work).

## 2026-11-12 (afternoon): redesign, rebrand, new phones, end-to-end tests

The frontend (`mobinest/`) is now a **git repository**. Baseline commit `4e8db47`, then a commit per milestone. Backup zip: `C:\Users\admin\mobinest-backups\mobinest-before-shadcn-*.zip`.

### Frontend
- **Tailwind v4 + shadcn/ui, safe install:** no preflight, so the 8 key pages stayed pixel-identical. App tokens that clashed were renamed (`--muted` → `--text-muted`, `--accent` → `--brand-orange`); shadcn colours map onto the app tokens. Also added the shadcn MCP config (`.mcp.json`) and the Aceternity **Spotlight** (hero glare) and Magic UI **Animated Theme Toggler** components.
- **Dark mode:** `html.dark` token set, plus role tokens `--card`, `--contrast-bg/fg`, `--chrome-bg` and `--header-bg`. It follows the OS by default, the choice is saved, and there's no flash (`src/lib/theme-init.js`). Clerk cards follow the theme via `src/lib/ThemedClerkProvider.jsx` (`@clerk/themes`).
- **Rebrand to Namma Mobiles** (subagent): badge logo, favicons, web manifest and og tags in `public/brand/`; all visible text renamed. localStorage keys are deliberately unchanged.
- **Brand colour:** logo blue `#0b5bd3` (dark mode `#3d8bff`) replaces purple `#5b3df5`; navy hero with an azure gradient.
- **Typography:** Plus Jakarta Sans (self-hosted `@fontsource-variable`) for the whole UI; Google Fonts link removed.
- **Home:**
  - Hero caption and stats removed; Spotlight glare added; hero phone is now the iPhone 18 Pro Max in Burgundy.
  - Brand cards: Apple, Samsung and Pixel logos (Simple Icons CC0, plus the public-domain Pixel wordmark), no left stripe, bright logos, a blue gradient glow on hover.
  - All small labels above section titles ("eyebrows") removed site-wide.
  - New sections (subagent): New & trending tabs, Shop by price (4 ranges), a Gallery of official brand films, and comparisons reordered.
- **Clerk:** the "Development mode" banner is hidden with Clerk's official `appearance.options.unsafe_disableDevelopmentModeWarnings`. "Secured by Clerk" stays; removing it is a Clerk dashboard setting (paid plan for production).
- **Fixes:**
  - Mobile menu drawer is rendered through a portal; it was clipped to ~195px by the header's `backdrop-filter`.
  - The Apple brand page showed a duplicate film (React key warning).
  - The 404 page now has an `h1`.
  - Rating shows "New · No reviews yet" instead of an invented score.

### Backend / database
- **V4** `add_iphone_duo`: Apple's foldable, ₹2,99,900–₹4,49,900, Star White / Night Sky, 8 SKUs.
- **V5** `rebrand_namma_mobiles`: warranty texts and the accessory house brand renamed.
- **V6** `add_iphone_18_pro_max`: ₹1,79,900–₹3,29,900, Burgundy / Glacier / Silver / Black, 16 SKUs.
- The V4–V6 migrations are fresh-install safe: on an empty DB they insert nothing and the seeder loads the same data from `seed/*.json`. The seed was re-exported (34 phones).
- Validation error `fields` now use the form field names (`phone`, not `address.phone`).
- `context/setup/start-mobinest.ps1` starts MySQL, the API and the storefront in their own windows (they survive Claude sessions).

### Verified
- End-to-end suite: Playwright driving the installed Edge, 23 checks covering browse, search, filters, product, cart and coupon, real guest checkout, compare, wishlist, sell, brand pages, 404, Clerk modal and sign-up, the account guard, theme toggle, and the mobile menu. All flows pass, with 0 console errors and 0 failed requests. The one "fail" was a test-script encoding bug, not the app.
- The test orders it created are listed in DATABASE.md.

## 2026-11-12

### Frontend: infinite offers ticker in the header (owner request)
- Replaced the static top strip ("Free delivery over ₹499 · 12-month warranty · 7-day replacement") with an infinite, seamless scrolling ticker: Diwali Sale (highlighted, links to /phones), 25% HDFC credit-card discount, up to 12-month warranty, free delivery over ₹499, no-cost EMI, 7-day replacement, exchange your old phone.
- Files (in `mobinest/`): NEW `src/components/layout/AnnouncementBar.jsx`; NEW `src/data/promotions.js` (edit the offers here); `src/components/layout/Header.jsx` (topbar → `<AnnouncementBar />`); `src/styles/global.css` (`.ticker*` styles; `.topbar__inner` removed).
- Behaviour: pure CSS (no new dependency), pauses on hover/focus, ~6 s per offer, edge fade, the duplicate copy is hidden from screen readers. With "reduce motion" it becomes a static scrollable row.
- Verified: eslint clean, `vite build` OK, headless-browser screenshot shows it scrolling. The frontend session was notified before and after editing.

### Verified + fixed: frontend ↔ backend integration
- Both servers had stopped (they ran as background tasks of earlier Claude sessions). Added `context/setup/start-mobinest.ps1`, which starts MySQL (if needed), the API and the storefront in their own windows so they survive session ends. Used it to start both.
- Verified through the frontend's Vite proxy (`:5173/api`): catalogue (3 brands, 16 phones, 17 accessories); a COD order with the exact `Checkout.jsx` payload → `MNMWDCH4O7KZS7` (₹34,298, PENDING); `/api/orders/me` → 401 without a token.
- **Fix (backend):** validation `fields` keys for the address now use the form names (`phone`, not `address.phone`), so the checkout highlights the right inputs. File: `common/error/GlobalExceptionHandler.java`. No frontend change needed.

### Ops: backend restarted
- The API had stopped when the previous Claude session ended (it runs as a background process of the session; no crash in the log). Restarted with `mvnw.cmd spring-boot:run`: Flyway validated 3 migrations (schema up to date), data intact (3 brands, 16 phones, 17 accessories). Swagger UI opened for the owner's review.

## 2026-11-11

### Integrated: frontend connected to the API (by frontend session `admin-73`)
- The React app loads `/api/catalog` via a Vite proxy, places orders with `POST /api/orders` (Clerk Bearer token when signed in), and lists `/api/orders/me`. Browser end-to-end test order `MNMWBZJEATG1IG` (₹44,999) was created.
- Backend and frontend sessions are linked (`admin-b4` ↔ `admin-73`) for coordination.
- Frontend confirmed V3 on the live site; it fixed blank brand names in accessory "compatible with" lists. No further API requests (image URLs parked).

### Added: Flyway V3 brand visibility (16:06 IST)
- `V3__brand_visibility.sql`: `brand.active`, `hero_phone_slug`, `hero_color`; only apple, samsung, google active; new taglines; google → "Google Pixel".
- Code: `Brand` entity fields; `BrandDto` gains `hero` + `active`; `BrandRepository.findAllByActiveTrueOrderBySortOrderAsc`; `PhoneRepository.findAllVisible()` / `findVisibleBySlug()`; `PhoneSpecifications.visible()`; `OrderService` rejects hidden-brand phones; `CatalogSeeder` reads `active`/`hero` from `seed/brands.json`.
- `seed/brands.json` re-exported from the frontend (12 brands, 3 active). `phones.json` unchanged (32).
- Export tool now exports the frontend's full `ALL_BRANDS`/`ALL_PHONES` (via a Node load hook).
- Verified: API catalogue equals the frontend's active catalogue; hidden phones → 404 for detail, search and order; a fresh install (throwaway DB, removed afterwards) equals the upgraded DB apart from test-order stock; 7/7 unit tests pass.

### Detected: frontend catalogue drift (resolved by V3)
- The React data now has 3 active brands / 16 phones (was 12 / 32), brand `hero` + taglines, "Google Pixel" rename, and `imageManifest.json` (photos). Phones and accessories that remain are unchanged. Logged as open item 0 in CONTEXT.md.
- Fixed `tools/export-seed-from-frontend.mjs` to resolve the frontend's extension-less imports (`from './brands'`); now needs Node 22.15+.

### Added: project documentation folder `context/`
- `README.md` (start here + run guide), `CONTEXT.md` (conversation log, decisions, status), `PROJECT.md` (architecture, patterns, rules, roadmap), `DATABASE.md` (schema, ERD, live stats), `API.md` (endpoints + examples), `CHANGELOG.md`.
- `setup/setup-database.ps1`: one-command DB + app user + local config (tested on a throwaway DB, then cleaned up).
- `setup/create-database.template.sql`, `setup/application.properties.example`.
- `tools/export-seed-from-frontend.mjs`: regenerates seed JSON from the React data.
- Removed `config/create-database.sql` (it contained the real password; replaced by the script and template).
- Root `README.md` now points to `context/`.

### Verified: end-to-end checks
- Health UP; catalogue counts correct; 144/144 server prices equal the frontend formula.
- Test guest order `MNMWBYYOZCWMIP` (₹89,497, WELCOME10 capped at ₹2,000, stock 75 → 73).
- COD > ₹50,000 → 422; overselling → 409 with rollback; invalid form → 400 with field messages; protected endpoints → 401 without a valid token.

### Added: database
- Created MySQL database `mobinest` (utf8mb4) and least-privilege user `mobinest_app`. Password lives in git-ignored `config/application.properties`.
- Flyway `V1__catalog.sql` (brand, condition_grade, phone, phone_variant, phone_color, sku, accessory_category, accessory).
- Flyway `V2__customers_coupons_orders.sql` (customer, coupon + 3 seed coupons, orders, order_item).
- Seeded on first start: 12 brands, 4 conditions, 32 phones, 67 variants, 101 colours, 470 SKUs, 8 categories, 17 accessories.

### Added: backend `mobinest-api` (Spring Boot 4.1.2, Java 25)
- Generated from start.spring.io (web, data-jpa, mysql, flyway, validation, security, oauth2-resource-server, actuator, lombok, devtools) + springdoc-openapi 3.2.0.
- `config/`: SecurityConfig (stateless, Clerk JWT via JWKS + issuer + `azp` validation, CORS for :5173), OpenApiConfig, AppProperties.
- `common/`: RFC 9457 error handling, JSON column converters, PageResponse.
- `catalog/`: entities, repositories (atomic stock decrement), PricingPolicy, PhoneSpecifications, CatalogService/Mapper, CatalogController, CatalogSeeder.
- `customer/`: Customer (Clerk-linked), JIT provisioning, `/api/me`.
- `order/`: CustomerOrder/OrderItem/ShippingAddress, coupons (DiscountType strategy), payment gateways (COD + demo online), OrderService facade, OrderPlacedEvent + after-commit listener, OrderController.
- Unit tests: PricingPolicyTest (4), CouponTest (3), all passing.

### Installed: tooling
- Eclipse Temurin **JDK 25.0.5** (winget). MySQL 8.0.45, Node 24 and Git were already present. Maven via wrapper; Tomcat embedded.

### Decided
- Backend-only focus; the owner handles the frontend separately (no frontend files changed).
- Auth = Clerk (from the frontend context) → the API verifies Clerk JWTs instead of issuing its own.
- Architecture and design patterns: see PROJECT.md.
