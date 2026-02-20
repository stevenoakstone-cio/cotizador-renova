---
phase: 05-banos
plan: "01"
subsystem: data-layer
tags: [mdf-rh, bano, calculateParts, assignHardware, HW_DEFAULTS, MODS]

# Dependency graph
requires:
  - phase: 04-cocina-extras-ui-output
    provides: calculateParts/assignHardware patterns, HW_DEFAULTS guarded conditional pattern
provides:
  - bajoLavabo MODS entry with forType:['bano'] and calculateParts/assignHardware branches
  - cajoneraBano MODS entry with forType:['bano'] and calculateParts/assignHardware branches
  - botiquin MODS entry with forType:['bano'] and calculateParts/assignHardware branches
  - HW_DEFAULTS.soporte_bano wall-mount bracket key
affects: 05-banos (UI/output plans), any phase using assignHardware or calculateParts for bano types

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Bano modules follow flotante (wall-mount) pattern: no patas, no zoclo_clip, soporte_bano x2"
    - "bajoLavabo top cajon uses saqueU:true flag on frente interno piece for cespol documentation"
    - "botiquin door count: w<=500mm = 1 puerta, w>500mm = 2 puertas (same pattern as alacenaEstandar)"
    - "cajH formula for bajoLavabo 2-cajón: (h - 2*TK) / 2; for 1-cajón: h - TK"
    - "cajH formula for cajoneraBano: (h - TK) / cajCount"

key-files:
  created: []
  modified:
    - cotizador-template/cotizador-template.html

key-decisions:
  - "Bano modules are all flotante (wall-mounted): no patas, no zoclo_clip for any bano type"
  - "bajoLavabo cajH formula split: 2-cajón uses (h-2*TK)/2 to account for top amarre; single-cajón uses (h-TK)"
  - "saqueU:true flag on top cajon frente interno piece documents the cespol saque (not a separate cut piece)"
  - "botiquin assignHardware only assigns soporte_bano — bisagras handled by calc() door block (same as alacenaEstandar)"
  - "cajoneraBano uses loop for per-cajon pieces (same pattern as cocina cajonera)"

patterns-established:
  - "Flotante bano pattern: soporte_bano x2 on all bano modules; no patas or zoclo_clip"
  - "saqueU flag pattern: boolean flag on piece object documents special fabrication notes without adding new piece"

requirements-completed: [BANO-01, BANO-02, BANO-03]

# Metrics
duration: 8min
completed: 2026-02-20
---

# Phase 5 Plan 01: Bano Modules Data Layer Summary

**3 bathroom module types (bajoLavabo, cajoneraBano, botiquin) in MODS with calculateParts despiece and assignHardware wall-mount hardware per INSTRUCTIVO Section 4**

## Performance

- **Duration:** ~8 min
- **Started:** 2026-02-20T20:44:00Z
- **Completed:** 2026-02-20T20:52:52Z
- **Tasks:** 2 (implemented together in 1 commit)
- **Files modified:** 1

## Accomplishments
- Added 3 MODS entries (bajoLavabo, cajoneraBano, botiquin) with `forType: ['bano']` for step2() filter
- Added HW_DEFAULTS.soporte_bano guarded conditional for wall-mount bracket hardware
- Implemented calculateParts() branches for all 3 types with correct casco/cajon/frente pieces and AUDIT comments
- Implemented assignHardware() branches: Blum Tandem per cajón (bajo/cajonera), soporte_bano x2 (all 3)
- bajoLavabo top cajon has `saqueU: true` flag documenting the 220x180mm cespol notch
- botiquin: 1 or 2 puertas based on w<=500mm threshold (matching alacenaEstandar pattern)
- No patas or zoclo_clip on any bano module (all flotante per INSTRUCTIVO 4.3)

## Task Commits

1. **Task 1+2: Bano MODS + HW_DEFAULTS + calculateParts + assignHardware** - `e23d354` (feat)

**Plan metadata:** (in final commit)

## Files Created/Modified
- `cotizador-template/cotizador-template.html` - 3 MODS entries, HW_DEFAULTS.soporte_bano, calculateParts blocks for bajoLavabo/cajoneraBano/botiquin, assignHardware blocks for all 3

## Decisions Made
- bajoLavabo cajH formula differs by cajCount: `(h - 2*TK) / 2` for 2-cajón (accounts for techo amarre), `h - TK` for 1-cajón
- `saqueU: true` is a flag on the piece object (documentation marker) — the actual fabrication cut is noted, no extra piece needed
- botiquin bisagras delegated to calc() door block (same pattern as alacenaEstandar/alacenaSobreCampana) — assignHardware only handles non-door hardware

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
- `cotizador-template/` is a git submodule — had to commit inside the submodule directory (not the root). This matches established project pattern from prior phases.

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- Bano data layer complete: MODS, calculateParts, assignHardware all in place
- Ready for Phase 5 Plan 02: Bano UI (step1 type selector, step2 module cards, step3 display)
- Ready for Phase 5 Plan 03: Bano PDF sections

## Self-Check: PASSED

- SUMMARY.md exists at `.planning/phases/05-banos/05-01-SUMMARY.md`
- Commit e23d354 exists in submodule git log
- 3 MODS entries with `forType: ['bano']` confirmed (grep count = 3)
- 6 `moduleType ===` branches confirmed (3 calculateParts + 3 assignHardware)
- `soporte_bano` HW_DEFAULTS entry confirmed
- `saqueU: true` flag on bajoLavabo top cajon confirmed

---
*Phase: 05-banos*
*Completed: 2026-02-20*
