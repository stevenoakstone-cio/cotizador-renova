---
phase: 09-multi-tenant-saas
plan: "03"
subsystem: ui
tags: [license, validation, lock-screen, multi-tenant, saas, app-config]

# Dependency graph
requires:
  - phase: 09-01
    provides: APP_CONFIG localStorage layer, saveAppConfig/loadAppConfig/hasAppConfig, setup screen
  - phase: 09-02
    provides: connection status indicator pattern, APP object exposure pattern
provides:
  - validateLicense() function with hash-based key verification
  - showLockScreen() full-screen overlay for invalid/expired/missing license
  - generateLicenseKey(client, expDate) for admin key generation
  - showLicenseInfo() modal with version, client, expiration, status, support contact
  - License info button in all three app headers
affects:
  - Boot sequence (license check added between hasAppConfig and render)
  - Setup screen (licenseKey field added)
  - All headers (license info button added)

# Tech tracking
tech-stack:
  added: []
  patterns:
    - License key format: base64(JSON{client, exp, hash}) where hash = simpleHash(client+exp+SALT)
    - Boot gate pattern: validateLicense() called after hasAppConfig(); showLockScreen() + return if invalid
    - Lock screen unlock: saves new key to APP_CONFIG.licenseKey, re-validates, reloads on success

key-files:
  created: []
  modified:
    - cotizador-template/cotizador-template.html

key-decisions:
  - "License key is base64(JSON) for easy paste/transfer; hash is a simple deterrent (not crypto-grade)"
  - "SECRET_SALT hardcoded as 'renova-cotizador-2024' — obfuscation, not cryptographic security"
  - "Boot order: hasAppConfig() check first, then validateLicense() — setup screen takes priority over lock screen"
  - "Lock screen includes inline key entry + Validar button so users can unlock without console access"
  - "generateLicenseKey exposed on APP object for admin console key generation"
  - "showLicenseInfo reads validateLicense() result at call time — always reflects current license state"
  - "License info button uses circled-i emoji (&#x2139;&#xFE0F;) — no additional SVG needed"

patterns-established:
  - "License validation gate: validateLicense() returns {valid, reason, client, exp} — structured for easy UI mapping"
  - "APP.generateLicenseKey('Client Name', 'YYYY-MM-DD') pattern for admin key generation from browser console"

requirements-completed: [MTNT-04, MTNT-05]

# Metrics
duration: 2min
completed: 2026-02-20
---

# Phase 9 Plan 3: License Key Validation and License Info Screen Summary

**Hash-based license validation with lock screen for invalid/expired/missing keys and a license info modal accessible from all app headers**

## Performance

- **Duration:** 2 min
- **Started:** 2026-02-20T23:04:41Z
- **Completed:** 2026-02-20T23:07:13Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments

### Task 1: License key validation and lock screen
- `LICENSE_SALT` constant: `'renova-cotizador-2024'` (deterrent hash seed)
- `simpleHash(str)` function: djb2-style hash returning base-36 string
- `validateLicense()` function: reads APP_CONFIG.licenseKey, decodes base64 JSON, verifies hash, checks expiration
  - Returns `{valid: false, reason: 'no_key'}` when missing
  - Returns `{valid: false, reason: 'invalid'}` when hash mismatch
  - Returns `{valid: false, reason: 'expired', client, exp}` when expired
  - Returns `{valid: true, client, exp}` when valid
- `generateLicenseKey(client, expDate)` function: creates base64(JSON{client, exp, hash}) — exposed as `APP.generateLicenseKey`
- `showLockScreen(reason, details)` function: full-screen overlay (#0d0d0d background) with reason-specific message, key input field, Validar button
  - Validar button saves entered key to APP_CONFIG and calls validateLicense(); reloads on success
- Boot sequence updated: `validateLicense()` called after `hasAppConfig()` check; `showLockScreen()` + early return if invalid
- `licenseKey` field added to setup screen form under new "Licencia" section

### Task 2: License info screen
- `CONFIG.version = '1.0'` added to CONFIG object
- `showLicenseInfo()` function: modal overlay with table showing Version, Cliente, Expiracion, Estado, Soporte
  - Status: 'Activa' (green #4caf50), 'Expirada' (red #f44336), 'Sin licencia' (gray #888)
  - Spanish date formatting: "DD de Mes YYYY" (e.g., "31 de diciembre 2027")
  - Closes on Cerrar button or backdrop click
- License info button (circled-i) added to all three headers: templates, quoter, estandarizado
- `APP.showLicenseInfo` exposed on APP object

## Task Commits

1. **Task 1: License key validation, lock screen, setup field** - `b05b28f` (feat)
2. **Task 2: License info screen and header button** - `b6220b0` (feat)

## Files Created/Modified
- `cotizador-template/cotizador-template.html` - 5 new functions (~200 lines), licenseKey setup field, header buttons, CONFIG.version, boot gate (10 lines)

## Decisions Made
- License key format is base64-encoded JSON (not a hex token) — easy to read/paste and self-describing
- `SECRET_SALT` is hardcoded in the single-file app — this is an obfuscation layer, not cryptographic security (acceptable for single-file app distribution model)
- Boot order kept: setup check first, license check second — ensures tenant is always configured before license is validated
- Lock screen (z-index: 10001) renders above everything including the setup overlay (z-index: 10000 range) — prevents any edge case bypass
- Admin generates keys via `APP.generateLicenseKey('Client', 'YYYY-MM-DD')` in browser console and sends base64 string to client
- License info modal reads `validateLicense()` at call time so status is always fresh

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
None.

## User Setup Required
- Admin generates license keys using `APP.generateLicenseKey('Client Name', 'YYYY-MM-DD')` in browser console
- Key is base64 string that gets pasted into setup screen or APP_CONFIG.licenseKey field
- Example: `APP.generateLicenseKey('Mi Carpinteria', '2027-12-31')` → base64 key string

## Next Phase Readiness
- Phase 9 is now complete: APP_CONFIG (09-01) + connection indicator/PDF branding (09-02) + license validation (09-03) all deployed
- Full multi-tenant SaaS foundation: tenant config, Supabase cloud storage, visual feedback, license protection
- No blockers for Phase 10

## Self-Check

- `b05b28f` exists: `feat(09-03): license key validation, lock screen, and setup screen licenseKey field`
- `b6220b0` exists: `feat(09-03): license info screen and header license button`
- File exists: `cotizador-template/cotizador-template.html` - confirmed (14398 lines after changes)
- `validateLicense` function at line 3244: confirmed
- `showLockScreen` function at line 3270: confirmed
- `showLicenseInfo` function at line 3329: confirmed
- `generateLicenseKey` function at line 3264: confirmed
- Boot gate at line ~14338: confirmed
- License button in estandarizado header at line 12077: confirmed
- License button in templates header at line 12124: confirmed
- License button in quoter header at line 12134: confirmed

## Self-Check: PASSED

---
*Phase: 09-multi-tenant-saas*
*Completed: 2026-02-20*
