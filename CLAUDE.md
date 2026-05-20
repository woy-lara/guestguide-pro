# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**GuestGuide Pro** — multi-tenant SaaS hotel concierge prototype for Boquete, Panamá. Single-file HTML SPA, no build system, no backend, no package manager. The entire app lives in `index.html` (~10,700 lines, ~700 KB).

## Running / Inspecting

```bash
# Serve locally (any static server works)
python3 -m http.server 3000     # then open http://localhost:3000

# Sanity checks after edits — the file is 10k+ lines and one missing brace
# breaks the whole SPA silently. Run these after any non-trivial edit:
wc -l index.html
ls -lh index.html
python3 -c "s=open('index.html').read(); import re; print('{}:', s.count('{'), s.count('}')); print('():', s.count('('), s.count(')'))"
```

There are **no tests, no linter, no build step**. Open the file in a browser. The "test" is: switch role in the top banner, click through tabs, watch the console.

## Architecture

### Everything is in one file

CDN-only deps loaded in `<head>`: Tailwind, Lucide icons, Leaflet (maps), `qrcode-generator@1.4.4`, Google Fonts (Manrope + DM Mono). The `<style>` block defines design tokens (sage/olive palette — `--sidebar-bg`, `--accent`, etc.) and Tailwind overrides. Everything else is one giant `<script>`.

### The data model: `DATABASE` (line ~1185)

In-memory mutable global. Mutations happen in-place; the UI re-renders by calling `renderApp()`. No persistence layer except `localStorage` for language prefs and the JSON export/import feature in the Backup view.

Key collections: `roles`, `hotels` (with nested `team`, `services`, `experiences`, `pmsGuests`), `potentialHotels` (CRM-style leads), `providers`, `potentialProviders`, `vouchers`, `guestRequests`, `qrCodes`, `analyticsEvents`, `auditLog`, `invoices`, `notifications`. Many seed generators run on boot: `ensureQrCodes`, `ensureAnalytics`, `ensureAuditLog`, `ensureInvoices`, `ensureTourMetrics`.

### Role-based routing

Two pieces of global state drive the entire UI:

```js
let currentRole = "saas_admin";   // 'saas_admin' | 'hotel_1' | 'hotel_2' | 'hotel_3' | 'prov_1'
let activeTab   = "overview";
```

The role selector in the banner (`#role-simulator`) calls `switchRole()`. The sidebar and `renderApp()` (line ~5465) branch on `currentRole.startsWith('hotel_')` / `'prov_'` / `=== 'saas_admin'` to render the right view for the right tab.

**When adding a new view**: add the tab in `renderSidebar()`, add the branch in `renderApp()`, and write a `renderXxx(container, ...)` function that writes HTML into the passed element.

### Rendering pattern

There's no framework. Views are functions that return or set `innerHTML` using template literals. State change → mutate `DATABASE` or a global (`activeTab`, `tempHotel`, etc.) → call `renderApp()`. Re-renders are full subtree replacements. After every render, `lucide.createIcons()` runs to swap `<i data-lucide="...">` into real SVGs.

Two helpers worth knowing:
- `esc(str)` / `escAttr(str)` — escape user data inside template literals. **Use these** whenever you interpolate `DATABASE` strings into HTML.
- `_renderPreservingFocus()` — re-renders while keeping the currently focused input alive (used by search boxes that mutate state on every keystroke).

### Builders use `tempHotel` / `tempProvider`

`openHotelBuilder(id)` clones the target hotel into `tempHotel`, switches `activeTab` to `'hotel_builder'`, and the builder edits the temp. `saveHotelBuilderData()` writes back to `DATABASE.hotels`. Same shape for providers (`tempProvider`, `openProviderBuilder`, `saveProviderData`). The builder has nested tab state (`activeEditorTab` / `activeProviderBuilderTab`) and re-renders only itself via `renderHotelBuilderEditorTabs()` for snappier feedback.

The **autosave indicator** (`markDirty()` / `markSaved()`) is fed by every `updateTempXxxField` call.

## i18n — two independent systems

This is the dominant pattern in the current codebase and the area most edits land in. **Read this carefully before touching strings.**

### Dashboard i18n (`dashLang` / `tD` / `DASH_I18N`)

```js
let dashLang = (() => localStorage.getItem('gg_dashLang') || 'es')();
const DASH_I18N = { es: { /* ~620 keys */ }, en: { /* same keys */ } };

function tD(key, ...args) {
    const dict = DASH_I18N[dashLang] || DASH_I18N.es;
    const val = dict[key] !== undefined ? dict[key] : DASH_I18N.es[key];
    return typeof val === 'function' ? val(...args) : (val ?? key);
}
```

- Function values support interpolation: `tD('reservas_this_month', count)` → `` `${count} reservas · este mes` ``.
- The ES/EN toggle in the banner is wired via `data-dash-lang-btn` attributes; `setDashLang()` updates state, persists to `localStorage`, and calls `applyStaticI18n()` + `renderApp()`.
- `applyStaticI18n()` patches elements rendered **outside** `renderApp()` (the banner, role selector option labels, the toggle buttons themselves).

### Mobile preview i18n (`mobileLang` / `tM` / `MOBILE_I18N`)

Separate dictionary for the phone mockup inside the dashboard. Same shape as `tD`. Don't merge them — the mobile preview simulates the guest-facing app and has its own copy register.

### Conventions for new strings

1. **Stored data stays in ES.** When ES strings are written to `DATABASE` (request status, request type, invoice status), translate at **display** time using small mapping objects:
   ```js
   const statusKey = { 'Pendiente': 'req_status_pending', 'En Proceso': 'req_status_processing', 'Completado': 'req_status_completed' };
   const label = statusKey[r.status] ? tD(statusKey[r.status]) : r.status;
   ```
   Do not migrate stored values to keys.
2. **Add to both `es` and `en` simultaneously.** Untranslated keys fall back to ES, which silently hides missing English. Keep parity.
3. **Naming**: prefix by area — `hl_*` (hotel list), `pl_*` (provider list), `inv_*` (invoice), `qe_drawer_*` (quick-edit), `bulk_*`, `confirm_*`, `ck_*` (command palette), `hbt_*` / `hba_*` (hotel builder tabs / amenities), `pbi_*` / `pbav_*` / `pbg_*` / `pbm_*` (provider builder tabs), `ai_*`, `notif_*`, `toast_*`. Audit actions use the namespaced `audit.<action>` key resolved by `auditLabel(action)`.
4. **Inline ternaries for one-offs.** Short generic words ("Updated", "Delete") that don't merit a dict entry can use `${dashLang==='en'?'X':'Y'}` inline. Don't proliferate — anything reused 3+ times should become a key.
5. **Toasts** use `showToastT(titleKey, msgVal, type)`. `titleKey` is always a dict key; `msgVal` can be a key or a literal string (it auto-detects).

## Conventions to follow

- **Mutate, then `renderApp()`** — don't try to re-render only a piece unless you're inside a builder (which has its own `renderHotelBuilderEditorTabs()` / `renderProviderBuilder()`).
- **Always escape interpolated user data** with `esc()` / `escAttr()`. Hotel names, notes, request items all come from `DATABASE` and can contain quotes/HTML.
- **Audit every meaningful change** with `logAudit(action, target, details, actorOverride)`. Actions are defined in `AUDIT_ACTIONS` (line ~3970) with severity (`info` / `warning` / `success`). Add new ones there.
- **Map status/type values via key dicts** rather than storing translation keys in the DB.
- **Test by switching role** — many bugs only show in `hotel_*` or `prov_*` views.
- **`lucide.createIcons()`** runs automatically after `renderApp()`. If you inject HTML out-of-band (modals, drawers), call it yourself.
