# External Integrations

**Analysis Date:** 2026-02-19

## APIs & External Services

**Supabase (Cloud Database):**
- Purpose: Cloud storage and sync for quotation projects
- SDK: `@supabase/supabase-js@2` loaded via CDN (`https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/dist/umd/supabase.min.js`)
- Configuration: `CONFIG.supabase.url` and `CONFIG.supabase.key` in `cotizador-template/cotizador-template.html` line 3045
- Status: **Currently disabled** (URL and key are empty strings)
- Fallback: App operates fully on localStorage when Supabase is unavailable (line 4282)

**Supabase Operations (lines 4319-4380):**
- `cloudFetchProjects()` - Reads from `projects` table, ordered by `updated_at` descending
- `cloudSaveProject(entry)` - Upserts to `projects` table with fields: `id`, `name`, `client`, `type`, `mode`, `data`, `updated_at`
- `cloudDeleteProject(id)` - Deletes from `projects` table by `id`
- Table schema expected: `projects` with columns `id`, `name`, `client`, `type`, `mode`, `data` (JSONB), `updated_at`

## Data Storage

**Primary (Local):**
- `localStorage` with prefix `cot_`
- Keys used:
  - `cot_saved_projects` - JSON array of saved quotation projects
  - `cot_logo_base64` - Base64-encoded company logo for PDFs
  - `cot_quote_counter` - Sequential quote number counter
  - `cot_brand` - Selected brand identifier

**Secondary (Cloud):**
- Supabase PostgreSQL (when configured)
- Cloud data is cached to localStorage after fetch (line 4341-4342)
- Projects merge cloud + bundled projects with deduplication by `id`

**File Storage:**
- No external file storage service
- Logo stored as base64 in localStorage after Canvas resize (line 11209)

**Caching:**
- Service Worker with cache-first-on-failure strategy (line 12574)
- Cache name: `cotizador-v5` (configurable via `CONFIG.pwa.cacheName`)
- Old caches are automatically purged on service worker activation

## Authentication & Identity

**Auth Provider:**
- None. No user authentication system.
- Supabase connection uses anonymous/public key (no auth flow)
- All data is accessible without login

## CDN Dependencies

**jsDelivr:**
- `https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/dist/umd/supabase.min.js`
- Only external CDN dependency
- App functions without it (Supabase features degrade gracefully)

## Local File Dependencies

**Required at load time:**
- `jspdf.umd.min.js` - Must be co-located with HTML file
- `jspdf.plugin.autotable.min.js` - Must be co-located with HTML file
- `supplier-catalog-data.js` - Hardware catalog data; must define global `CATALOG` array

**Optional assets:**
- `logo_renova.webp` - Default logo image (can be overridden via settings upload)
- `manifest.json` - Referenced in HTML but overridden by inline manifest at runtime
- `icon-192.png` - Referenced as apple-touch-icon

## Monitoring & Observability

**Error Tracking:**
- None. No Sentry, Datadog, or similar service.

**Logs:**
- `console.warn` and `console.error` for Supabase failures only
- No structured logging or remote log shipping

## CI/CD & Deployment

**Hosting:**
- Static file hosting (no specific platform detected)
- No deployment configuration files present

**CI Pipeline:**
- None. No GitHub Actions, no CI config files.

## Environment Configuration

**Required for basic operation:**
- None. App works out of the box with localStorage only.

**Required for cloud sync:**
- `CONFIG.supabase.url` - Supabase project URL
- `CONFIG.supabase.key` - Supabase anon/public API key

**All configuration is inline** in the `CONFIG` object at `cotizador-template/cotizador-template.html` line 2951. There are no `.env` files or external configuration sources.

## Webhooks & Callbacks

**Incoming:**
- None

**Outgoing:**
- None

## Third-Party Data

**Supplier Catalog:**
- `cotizador-template/supplier-catalog-data.js` - Demo hardware catalog (30 items)
- `cotizador-template/supplier-catalog-data-REAL.js` - Real supplier data (large file, same schema)
- Schema: `{ codigo, descripcion, unidad, precio, marca, categoria, tipo }`
- Loaded as global `CATALOG` variable, consumed by hardware selection UI
- Swapping catalogs requires renaming files (no runtime toggle)

---

*Integration audit: 2026-02-19*
