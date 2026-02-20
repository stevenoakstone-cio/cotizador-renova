---
phase: 07-mueble-tv
plan: "02"
subsystem: ui-wiring
tags: [tv, mueble-tv, calc, step2, door-logic, variant-selector, noPuertas]

# Dependency graph
requires:
  - phase: 07-01
    provides: MODS entries, calculateParts(), assignHardware(), bastidorCost ml-pricing, isBastidor pattern
provides:
  - calc() door logic for consolaPatas/consolaFlotante (1 or 2 puertas by width threshold)
  - calc() door logic for torreLateralTV (1 puerta, with noPuertas skip guard)
  - step2() repisaFlotante variant selector (hueca/herraje) + herraje min-thickness hint
  - step2() torreLateralTV noPuertas toggle (open nicho configuration)
  - Complete TV quotation flow: all 5 module types quotable end-to-end
affects: [08-mueble-tv-pdf]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "noPuertas flag: togModProp('id','noPuertas') toggles open/closed config for torreLateralTV; checked in calc() skip guard"
    - "TV consola door dims: doorH = h - 10 (10mm juego), doorW uses innerW formula (w - 2*TK) rather than raw w"
    - "variant selector uses existing setModProp pattern (same as cubiertaType selector)"

key-files:
  created: []
  modified:
    - cotizador-template/cotizador-template.html

key-decisions:
  - "TV consola doorH uses h - 10 (10mm juego total) matching bajoEstandar pattern from INSTRUCTIVO; doorW derived from innerW = w - 2*TK"
  - "torreLateralTV noPuertas handled in skip guard via conditional (m.type === 'torreLateralTV' && m.noPuertas) — not a separate list entry"
  - "repisaFlotante variant selector uses setModProp (not togModProp) — value is a string enum not a boolean"
  - "herraje hint uses gold/beige accent (#c49a6c) matching app theme — informational only, not an error"

requirements-completed: [MTV-05]

# Metrics
duration: 3min
completed: 2026-02-20
---

# Phase 7 Plan 02: Mueble TV UI Wiring Summary

**calc() door logic for consolaPatas/consolaFlotante/torreLateralTV + step2() variant selector for repisaFlotante + noPuertas toggle for torreLateralTV — TV quotation flow complete end-to-end**

## Performance

- **Duration:** ~3 min
- **Started:** 2026-02-20T22:05:51Z
- **Completed:** 2026-02-20T22:08:53Z
- **Tasks:** 2 of 2
- **Files modified:** 1

## Accomplishments

- calc() door logic for consolaPatas/consolaFlotante: 1 puerta if width <=50cm, 2 puertas if >50cm; doorH = h - 10, doorW from innerW formula
- calc() door logic for torreLateralTV: always 1 puerta (narrow tower); doorH = h - 10, doorW = innerW - 4
- calc() skip guard extended: `(m.type === 'torreLateralTV' && m.noPuertas)` added for open nicho configuration
- step2() repisaFlotante variant selector: "Caja hueca" / "Herraje oculto" buttons using setModProp
- step2() herraje hint: gold-accent info box "Minimo 38mm de grosor para herraje oculto" when herraje selected
- step2() torreLateralTV toggle: "Sin puerta (nichos abiertos)" via togModProp for noPuertas flag

## Task Commits

1. **Task 1: calc() door logic + skip guard for TV modules** - `3834a34` (feat)
2. **Task 2: step2() variant selectors for repisaFlotante + torreLateralTV** - `8ce12e1` (feat)

## Files Created/Modified

- `cotizador-template/cotizador-template.html` - calc() door block (consolaPatas/consolaFlotante/torreLateralTV), skip guard (torreLateralTV+noPuertas), step2() variant selector (repisaFlotante), step2() toggle (torreLateralTV)

## Decisions Made

- TV consola doorH uses h - 10 (10mm juego total) matching INSTRUCTIVO pattern; doorW derived from innerW = w - 2*TK for correct inner-dimension fit
- torreLateralTV noPuertas checked as `(m.type === 'torreLateralTV' && m.noPuertas)` in skip guard — conditional rather than unconditional skip
- repisaFlotante variant selector uses setModProp (string enum) not togModProp (boolean toggle)
- herraje hint styled with gold accent (#c49a6c) to match app dark theme — informational, not warning

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- TV flow complete: TYPES, MODS, calculateParts(), assignHardware(), calc() door logic, step2() selectors all in place
- Ready for Phase 07-03: TV PDF output sections (genClientPDF / genInternalPDF TV module rendering)

---
*Phase: 07-mueble-tv*
*Completed: 2026-02-20*
