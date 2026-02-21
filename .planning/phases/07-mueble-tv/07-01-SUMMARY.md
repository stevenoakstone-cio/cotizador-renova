---
phase: 07-mueble-tv
plan: "01"
subsystem: data-layer
tags: [tv, mueble-tv, mods, calculateParts, assignHardware, hw-defaults, bastidor]

# Dependency graph
requires:
  - phase: 06-tablered-arauco
    provides: cubrecantoCost pattern (isCubrecanto), HW_DEFAULTS guarded conditional pattern, assignHardware chain pattern
  - phase: 05-banos
    provides: flotante module pattern (soporte_bano), variant-based hardware assignment
provides:
  - 5 MODS entries with forType:['tv'] cat:'TV' for consolaPatas, consolaFlotante, torreLateralTV, repisaFlotante, panelFondo
  - calculateParts() despiece branches for all 5 TV module types
  - assignHardware() branches for all 5 TV module types
  - 3 new HW_DEFAULTS keys: soporte_flotante, herraje_repisa, bastidor_ml
  - isBastidor pricing pattern in calc() (parallel to isCubrecanto)
  - bastidorCost in extraCost bucket + calc() return object
affects: [08-mueble-tv-step2, 09-mueble-tv-pdf, any future TV module additions]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "isBastidor flag: bastidor pine frame pieces excluded from sheet area, priced per ml via bastidor_ml"
    - "variant prop flow: m.variant passed from calc() to both calculateParts() and assignHardware()"
    - "TV cat grouping: cat:'TV' on MODS entries enables step2() UI grouping for TV module picker"

key-files:
  created: []
  modified:
    - cotizador-template/cotizador-template.html

key-decisions:
  - "consolaPatas and consolaFlotante share identical casco despiece; mounting differs only in assignHardware (patas vs soporte_flotante)"
  - "repisaFlotante variant='herraje' forces thickness to max(TK, 38) per INSTRUCTIVO — min 38mm for espigon insertion"
  - "panelFondo panel piece uses z:'f' (frente material, premium visible face); bastidor pieces use z:'i' but isBastidor:true skips sheet area"
  - "repisaFlotante and panelFondo added to calc() door skip guard — no puertas for these module types"
  - "torreLateralTV patas x4 same as cocina bajo pattern; bisagras handled by calc() door block when user toggles m.doors"
  - "bastidorCost added to extraCost bucket parallel to cubrecantoCost; returned in calc() result as bastidorCost key"

patterns-established:
  - "isBastidor pattern: mirror of isCubrecanto; w*q/1000 gives linear meters for ml pricing"
  - "TV module variant flow: m.variant from module object drives both despiece and hardware via options.variant"

requirements-completed: [MTV-01, MTV-02, MTV-03, MTV-04]

# Metrics
duration: 4min
completed: 2026-02-20
---

# Phase 7 Plan 01: Mueble TV Data Layer Summary

**5 TV module types (consolaPatas, consolaFlotante, torreLateralTV, repisaFlotante, panelFondo) with MODS entries, calculateParts() despiece, assignHardware() hardware, and bastidorCost ml-pricing in calc()**

## Performance

- **Duration:** ~4 min
- **Started:** 2026-02-20T00:00:00Z
- **Completed:** 2026-02-20T00:04:00Z
- **Tasks:** 2 of 2
- **Files modified:** 1

## Accomplishments

- 5 MODS entries with `forType: ['tv']` and `cat: 'TV'` for the TV module picker in step2()
- calculateParts() despiece branches for all 5 TV types following INSTRUCTIVO Section 5 specifications
- assignHardware() branches for all 5 TV types: consolaPatas (patas+zoclo), consolaFlotante (soporte_flotante), torreLateralTV (patas), repisaFlotante (variant-driven), panelFondo (empty)
- 3 new HW_DEFAULTS keys: soporte_flotante ($45), herraje_repisa ($120), bastidor_ml ($25) with guarded conditionals
- isBastidor pricing pattern in calc() — bastidor pine frame pieces priced per ml, excluded from sheet area, added to extraCost

## Task Commits

1. **Task 1: MODS entries + HW_DEFAULTS + calculateParts() for 5 TV module types** - `b7bc463` (feat)
2. **Task 2: assignHardware() for 5 TV module types + calc() wiring** - `9560b0f` (feat)

## Files Created/Modified

- `cotizador-template/cotizador-template.html` - 5 MODS entries, 3 HW_DEFAULTS keys, 5 calculateParts() branches, 5 assignHardware() branches, bastidorCost in calc()

## Decisions Made

- consolaPatas and consolaFlotante share identical casco (costado, piso, techo, fondo HDF, amarre frontal, entrepaño); only assignHardware differs
- repisaFlotante variant='herraje' forces `t: Math.max(TK, 38)` on the tablero — INSTRUCTIVO requires min 38mm for espigon insertion
- panelFondo panel piece uses `z:'f'` (frente/premium material, the visible face); bastidor pieces use `isBastidor: true` to skip sheet area
- repisaFlotante and panelFondo added to calc() door skip guard — these module types have no door toggle
- bastidorCost parallel to cubrecantoCost pattern from Phase 6 — both in extraCost bucket, both returned in calc() result

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- TV data layer complete: MODS, calculateParts(), assignHardware(), HW_DEFAULTS all in place
- Ready for Phase 07-02: step2() UI — TV module picker, variant selector (repisaFlotante), door toggle for consolaPatas/consolaFlotante/torreLateralTV
- Ready for Phase 07-03: TV PDF output (genClientPDF / genInternalPDF TV sections)

---
*Phase: 07-mueble-tv*
*Completed: 2026-02-20*
