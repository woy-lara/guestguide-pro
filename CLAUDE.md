# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**GuestGuide Pro** — multi-tenant SaaS hotel concierge prototype for Boquete, Panamá. Single-file HTML SPA, no build system, no backend, no package manager. The entire app lives in `index.html` (~13,300 lines, ~890 KB).

Deployed publicly to GitHub Pages at https://woy-lara.github.io/guestguide-pro/. The first real pilot client (Hacienda Los Molinos, with real menu, coords, contact info) is wired into the demo data.

## Running / Inspecting

```bash
# Serve locally (any static server works)
python3 -m http.server 3000     # then open http://localhost:3000

# Sanity checks after edits — the file is 13k+ lines and one missing brace
# breaks the whole SPA silently. Run these after any non-trivial edit:
wc -l index.html
ls -lh index.html
python3 -c "s=open('index.html').read(); print('{}:', s.count('{'), s.count('}')); print('():', s.count('('), s.count(')')); print('[]:', s.count('['), s.count(']'))"
```

There are **no tests, no linter, no build step**. Open the file in a browser. The "test" is: switch role in the top banner, click through tabs, watch the console. After every change, push to GitHub — Pages rebuilds in ~30-60s.

## Architecture

### Everything is in one file

CDN-only deps loaded in `<head>`: Tailwind, Lucide icons, Leaflet (maps), `qrcode-generator@1.4.4`, Google Fonts (Manrope + DM Mono). The `<style>` block defines design tokens (sage/olive palette — `--sidebar-bg`, `--accent`, etc.) and Tailwind overrides. Everything else is one giant `<script>`.

### The data model: `DATABASE` (line ~1185)

In-memory mutable global. Mutations happen in-place; the UI re-renders by calling `renderApp()`. No persistence layer except `localStorage` for language prefs, theme, auth state, and the JSON export/import in the Backup view.

Key collections: `roles`, `hotels` (with nested `team`, `services`, `experiences`, `pmsGuests`, `enabledModules`, `restaurantMenu`), `potentialHotels`, `providers`, `potentialProviders`, `vouchers`, `guestRequests`, `qrCodes`, `analyticsEvents`, `auditLog`, `invoices`, `notifications`, `moduleRequests`.

Many seed generators run on boot — invoked from `window.onload` and `applyImport`/`resetToDemo`:
- `ensureQrCodes`, `ensureAnalytics`, `ensureAuditLog`, `ensureInvoices`, `ensureTourMetrics`
- `ensureHotelModules` — defaults `enabledModules` to `['room_service','maintenance','amenities']` per hotel
- `ensureRestaurantMenus` — seeds curated menus per hotel by id
- `ensureModuleRequestsSeed` — 4 demo integration requests across hotels

### Role-based routing

Two pieces of global state drive the entire UI:

```js
let currentRole = "saas_admin";   // 'saas_admin' | 'hotel_1'..'hotel_4' | 'prov_1'
let activeTab   = "overview";
```

Banner role selector (`#role-simulator`) calls `switchRole()`. The auth flow's role-select screen calls `selectRoleAndEnter(roleId)` and persists `gg_authenticated_role` to localStorage. `renderApp()` branches on `currentRole.startsWith('hotel_')` / `'prov_'` / `=== 'saas_admin'` and routes `activeTab` to the right render function.

**When adding a new view**: add the tab in `renderSidebar()`, add the branch in `renderApp()`, and write a `renderXxx(container, ...)` function that writes HTML into the passed element.

Current routes per role:
- `saas_admin`: overview, hotels, stay_services, finances, scanner, ai_assistant, hotel_builder, providers, provider_builder, analytics, audit, module_requests, backup
- `hotel_*`: overview, stay_services, modules, restaurant_menu, finances, ai_assistant, analytics, qr_manager, audit
- `prov_*`: scanner, overview, analytics, qr_manager, audit

### Rendering pattern

There's no framework. Views are functions that set `innerHTML` using template literals. State change → mutate `DATABASE` or a global (`activeTab`, `tempHotel`, etc.) → call `renderApp()`. Re-renders are full subtree replacements. After every render, `lucide.createIcons()` runs to swap `<i data-lucide="...">` into real SVGs.

Helpers worth knowing:
- `esc(str)` / `escAttr(str)` — escape user data inside template literals. **Use these** whenever you interpolate `DATABASE` strings into HTML.
- `_renderPreservingFocus()` — re-renders while keeping the currently focused input alive (used by search boxes).
- `showToastT(titleKey, msgVal, type)` — i18n-aware toast.

### Builders use `tempHotel` / `tempProvider`

`openHotelBuilder(id)` clones the target hotel into `tempHotel`, switches `activeTab` to `'hotel_builder'`, and the builder edits the temp. `saveHotelBuilderData()` writes back to `DATABASE.hotels`. Same shape for providers. Builder has nested tab state (`activeEditorTab` / `activeProviderBuilderTab`) and re-renders only itself via `renderHotelBuilderEditorTabs()` / `renderProviderBuilder()` for snappier feedback.

⚠️ **`renderRestaurantMenu` pointed `tempHotel = hotel`** (live reference, not a clone) so its sidebar phone preview reflects edits in real time. `openHotelBuilder` always re-clones, so this doesn't leak.

The **autosave indicator** (`markDirty()` / `markSaved()`) is fed by every `updateTempXxxField` call.

## The Modules system (Hotel marketplace)

Each hotel can enable/disable per-hotel "modules". The marketplace concept and `MODULES_CATALOG` constant (line ~3683) define 9 modules — 4 currently functional (`room_service`, `maintenance`, `amenities`, `restaurant`) and 5 "Solicitar integración" placeholders.

```js
hotel.enabledModules = ['room_service', 'maintenance', 'amenities', /* maybe 'restaurant' */];
hotel.restaurantMenu = [
  { id, category: 'breakfast'|'lunch'|'dinner'|'drinks'|'dessert', name, price, description, photo, available }
];
DATABASE.moduleRequests = [{ id, hotelId, moduleId, notes, timestamp, status: 'pending'|'in_review'|'in_progress'|'done'|'rejected' }];
```

Module configurability flag: `MODULES_CATALOG[i].configurable` enables a "Configurar →" CTA on the active-module card and routes to a sub-page (currently only `restaurant_menu`). To add another configurable module, set `available:true, configurable:true`, create `renderXxxModule()`, wire to `activeTab`, add to sidebar submenu under `nav-group` "Módulos".

## Mobile preview (interactive phone inside the dashboard)

`mobileViewState` drives what's shown inside the simulated phone. Existing states: `home`, `tour_detail`, `menu_list`, `menu_detail`. Each branch in `renderMobilePreview()` sets `phone.innerHTML`. Navigation uses `setMobileView(viewName, idArg)`.

Two phone-preview hosts: `#guest-phone-preview` (Hotel Builder + Restaurant Menu page) and `#scan-phone-preview` (QR scan modal). The phone is hidden on real mobile via `.mobile-preview-aside { display: none !important; }` since the user IS on a phone.

## Auth flow (simulated)

`authState` cycles `login → magic_sent → role_select → authenticated`. `initAuthFlow()` checks localStorage on boot (`gg_authenticated`, `gg_authenticated_role`, `gg_auth_email`). The full-screen `#auth-overlay` covers everything until `selectRoleAndEnter` is called. `logout()` clears localStorage and reverts to login. The role-select screen lists all hotels from `DATABASE.hotels` + first provider — to add a new hotel, also add an entry to `roleColors` in `renderAuthScreen`.

## Dark mode

`html[data-theme="dark"]` flips the design-token vars. `appTheme` is read from localStorage on boot (or `prefers-color-scheme`), applied before any render. Toggle button is in the banner; `toggleTheme()` flips the attribute, persists, and re-renders. Many Tailwind overrides (e.g. `bg-white`, `bg-slate-*`, `text-slate-*`, `border-*`) point to CSS variables, so dark mode mostly Just Works — but inline `style="background:..."` colors are *not* auto-flipped.

## i18n — two independent systems

This is the dominant pattern in the current codebase. **Read this carefully before touching strings.**

### Dashboard i18n (`dashLang` / `tD` / `DASH_I18N`)

```js
let dashLang = (() => localStorage.getItem('gg_dashLang') || 'es')();
const DASH_I18N = { es: { /* ~750+ keys */ }, en: { /* same keys */ } };

function tD(key, ...args) {
    const dict = DASH_I18N[dashLang] || DASH_I18N.es;
    const val = dict[key] !== undefined ? dict[key] : DASH_I18N.es[key];
    return typeof val === 'function' ? val(...args) : (val ?? key);
}
```

- Function values support interpolation: `tD('rm_count_items', n)` → `` `${n} platos` ``.
- The ES/EN toggle exists in the banner AND in the auth overlay (independent). `setDashLang()` updates state, persists to `localStorage`, calls `applyStaticI18n()` + `renderApp()` (or `renderAuthScreen` if overlay visible).
- `applyStaticI18n()` patches elements rendered **outside** `renderApp()` (the banner, role selector option labels, the toggle buttons themselves, theme icon).

### Mobile preview i18n (`mobileLang` / `tM` / `MOBILE_I18N`)

Separate dictionary for the phone mockup. Same shape as `tD`. Don't merge them — the mobile preview simulates the guest-facing app and has its own copy register.

### Conventions for new strings

1. **Stored data stays in ES.** Translate at **display** time via mapping objects (e.g. `statusKey`, `typeKey`).
2. **Add to both `es` and `en` simultaneously.** Untranslated keys fall back to ES silently — keep parity.
3. **Naming prefixes**: `hl_*` (hotel list), `pl_*` (provider list), `inv_*` (invoice), `qe_drawer_*` (quick-edit), `bulk_*`, `confirm_*`, `ck_*` (command palette), `hbt_*`/`hba_*` (hotel builder), `pbi_*`/`pbav_*`/`pbg_*`/`pbm_*` (provider builder), `ai_*`, `notif_*`, `toast_*`, `auth_*` (auth screens), `onboard_*`, `hov_*`/`pov_*` (hotel/provider overview), `mod_*`/`modreq_*`/`modtoggle_*` (modules), `rm_*` (restaurant menu), `mob_*` (mobile preview), `mreq_*` (SaaS admin module requests), `greet_*` (time-of-day greetings). Audit actions use `audit.<action>` resolved by `auditLabel(action)`.
4. **Inline ternaries for one-offs.** Generic words can use `${dashLang==='en'?'X':'Y'}` inline. Don't proliferate.
5. **Toasts** use `showToastT(titleKey, msgVal, type)`.

## Conventions to follow

- **Mutate, then `renderApp()`** — except inside builders, which re-render themselves.
- **Always escape interpolated user data** with `esc()` / `escAttr()`.
- **Audit every meaningful change** with `logAudit(action, target, details, actorOverride)`. New audit actions go in `AUDIT_ACTIONS` (line ~5014). Recent additions: `module.enable`, `module.disable`, `module.request`, `module.request_status`, `restaurant.menu_update`.
- **Map status/type values via key dicts** rather than storing translation keys in the DB.
- **Test by switching role** — many bugs only show in `hotel_*` or `prov_*` views.
- **Test on mobile viewport (375px)** — the app has substantial responsive CSS in a single `@media (max-width: 767px)` block; many fixes live there.
- **`lucide.createIcons()`** runs automatically after `renderApp()`. If you inject HTML out-of-band (modals, drawers, phone-preview screens), call it yourself.
