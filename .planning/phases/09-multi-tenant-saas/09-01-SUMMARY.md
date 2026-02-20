---
phase: 09-multi-tenant-saas
plan: "01"
subsystem: ui
tags: [localStorage, multi-tenant, saas, setup-wizard, app-config]

# Dependency graph
requires:
  - phase: 01-foundation
    provides: CONFIG object structure, storage prefix, brand/contact/supabase fields
provides:
  - APP_CONFIG localStorage persistence layer (loadAppConfig, saveAppConfig, hasAppConfig)
  - First-run setup screen with company name, monogram, tagline, logo upload, Supabase credentials, contact info
  - CONFIG.brand/contact/supabase override at boot from localStorage
  - Boot guard that blocks main app until setup is complete
affects:
  - 09-02 (Supabase cloud sync — will use APP_CONFIG supabase credentials)
  - All future phases (CONFIG overrides affect all brand-dependent rendering)

# Tech tracking
tech-stack:
  added: []
  patterns:
    - APP_CONFIG localStorage layer: tenant config stored under cot_app_config key, loaded at boot before BRANDS init
    - CONFIG override pattern: loadAppConfig() modifies CONFIG.brand/contact/supabase in place
    - Boot guard pattern: !hasAppConfig() check before render() call, showSetupScreen() + return early

key-files:
  created: []
  modified:
    - cotizador-template/cotizador-template.html

key-decisions:
  - "APP_CONFIG key: CONFIG.storage.prefix + 'app_config' = 'cot_app_config' (consistent with existing storage prefix pattern)"
  - "loadAppConfig() called at line ~3369 — immediately after declaration, before BRANDS initialization at ~4467"
  - "Logo stored in both APP_CONFIG object (logoBase64 field) and the standard logo_base64 key so existing S.logoBase64 pickup works unchanged"
  - "Boot guard uses return inside IIFE — valid since the IIFE wraps all init code; render() and PWA setup skipped cleanly"
  - "Setup screen reloads page after save (location.reload()) so all CONFIG overrides take effect via clean boot"

patterns-established:
  - "APP_CONFIG layer: loadAppConfig() called before BRANDS — ensures CONFIG is patched before any derived data is computed"
  - "Exposed on APP object: APP.hasAppConfig, APP.saveAppConfig, APP.loadAppConfig for use by future setup/settings UI"

requirements-completed: [MTNT-01]

# Metrics
duration: 2min
completed: 2026-02-20
---

# Phase 9 Plan 1: APP_CONFIG System and First-Run Setup Screen Summary

**Runtime tenant branding via localStorage APP_CONFIG layer with full-screen setup wizard that blocks main app until company name and monogram are configured**

## Performance

- **Duration:** 2 min
- **Started:** 2026-02-20T22:54:29Z
- **Completed:** 2026-02-20T22:56:39Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments
- APP_CONFIG system: `loadAppConfig()`, `saveAppConfig()`, `hasAppConfig()` functions that read/write `cot_app_config` in localStorage
- CONFIG override: on every boot, APP_CONFIG values replace CONFIG.brand (name, mono, tagline, legalTerms), CONFIG.contact (name, city, office, cell, email), and CONFIG.supabase (url, key)
- First-run setup screen: full-screen dark-theme wizard with required fields (empresa, monograma) and optional fields (lema, logo, Supabase URL/key, contact info)
- Boot guard at main render(): `if (!hasAppConfig()) { showSetupScreen(); return; }` prevents main app from loading before setup

## Task Commits

1. **Task 1: APP_CONFIG localStorage layer and CONFIG override at boot** - `b0c14fa` (feat)
2. **Task 2: First-run setup screen UI** - `3cbc571` (feat)

## Files Created/Modified
- `cotizador-template/cotizador-template.html` - APP_CONFIG functions (~50 lines), showSetupScreen() (~140 lines), setup CSS (~120 lines), boot guard (5 lines)

## Decisions Made
- Used `CONFIG.storage.prefix + 'app_config'` as the key (consistent with existing storage keys)
- `loadAppConfig()` called immediately after its declaration (before BRANDS init) so CONFIG is fully patched before any derived objects are computed
- Logo stored redundantly: in APP_CONFIG JSON and re-written to standard `cot_logo_base64` key so existing `S.logoBase64` code works without modification
- Boot guard uses `return` inside the IIFE — cleanly skips `render()`, title/favicon/PWA/ServiceWorker setup on first run; everything runs fresh after reload
- Setup screen forces `window.location.reload()` after save rather than re-initializing in-place — ensures clean boot with all CONFIG overrides applied from the start

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
None.

## User Setup Required
None - the setup screen IS the user setup. Users open the app and are guided through configuration automatically on first launch.

## Next Phase Readiness
- APP_CONFIG system is in place; Phase 09-02 (Supabase cloud sync) can read `APP_CONFIG.supabaseUrl` and `APP_CONFIG.supabaseKey` via `CONFIG.supabase.url` and `CONFIG.supabase.key` after override
- Future Settings screen (if planned) can call `APP.saveAppConfig()` and `APP.loadAppConfig()` to update tenant config in-place

## Self-Check

- `b0c14fa` exists: `feat(09-01): APP_CONFIG localStorage layer and CONFIG override at boot`
- `3cbc571` exists: `feat(09-01): first-run setup screen UI with boot guard`
- File exists: `cotizador-template/cotizador-template.html` - confirmed, 14164 lines

## Self-Check: PASSED

---
*Phase: 09-multi-tenant-saas*
*Completed: 2026-02-20*
