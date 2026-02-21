---
phase: 02-cocina-bajos
plan: "02"
subsystem: ui
tags: [javascript, es5, assignHardware, calc, cocina, bajos, bisagra_165, pata, zoclo_clip, corr_tandem]

# Dependency graph
requires:
  - phase: 02-cocina-bajos
    plan: "01"
    provides: 5 bajo module types in MODS + HW_DEFAULTS + calculateParts() blocks
  - phase: 01-foundation
    provides: assignHardware() generic engine and calc() door block

provides:
  - assignHardware() blocks for all 5 cocina bajo types (bajoEstandar, fregadero, cajonera, esquineroCiego, hornoBase)
  - calc() door count auto-selection by width for bajoEstandar (<=50cm=1, >50cm=2)
  - calc() fregadero always-2-puertas rule
  - calc() cajonera/hornoBase door-skip guard
  - calc() bisagra_165 key for esquineroCiego enL type
affects:
  - 02-cocina-bajos (plan 03+): UI wiring for bajo module selector
  - future cost calculation integration for cocina mode

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "ES5 var-only: no let/const, no arrow functions, no template literals"
    - "AUDIT comment convention: every domain rule constant documented inline"
    - "Door skip guard pattern: if (cajonera||hornoBase) {} else { ... } inside door block"
    - "bisKey variable pattern: conditional key selection before autoHW.push calls"

key-files:
  created: []
  modified:
    - cotizador-template/cotizador-template.html

key-decisions:
  - "cajonera door guard: m.type==='cajonera' skips door block entirely — frentes in calculateParts()"
  - "hornoBase door guard: m.type==='hornoBase' skips door block — oven has its own door not in despiece"
  - "bajoEstandar width threshold: m.width<=50 (cm) = 1 puerta; >50 = 2 puertas per INSTRUCTIVO 3.3"
  - "fregadero: hardcoded doorCnt=2 regardless of width per INSTRUCTIVO (always 2 puertas)"
  - "esquineroCiego bisagra: m.esquineroTipo==='enL' uses bisagra_165 key; default ciego uses standard bisagra"
  - "cajonera jaladeras: assigned in assignHardware() per cajón count, NOT in door block"
  - "hornoBase jaladera: 1 jaladera for cajón inferior assigned in assignHardware()"

patterns-established:
  - "Door guard pattern: check m.type before doorCnt calculation to skip non-door modules"
  - "bisKey pattern: set var bisKey before autoHW.push to support different bisagra types"
  - "Cocina bajo hardware split: patas+zoclo_clip in assignHardware(); bisagras in calc() door block"

requirements-completed: [CBAJ-01, CBAJ-02, CBAJ-03, CBAJ-04, CBAJ-05, CBAJ-06]

# Metrics
duration: 8min
completed: 2026-02-20
---

# Phase 2 Plan 02: Cocina Bajos Hardware Auto-Assignment and Door Logic Summary

**assignHardware() now returns pata x4 + zoclo_clip x4 for all 5 bajo types; calc() auto-selects 1 or 2 puertas by width for bajoEstandar, hardcodes 2 for fregadero, skips doors for cajonera/hornoBase, and uses bisagra_165 for esquineroCiego enL**

## Performance

- **Duration:** 8 min
- **Started:** 2026-02-20T02:54:25Z
- **Completed:** 2026-02-20T03:02:25Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments
- 5 new else-if blocks in assignHardware() covering all cocina bajo module types with patas, zoclo clips, correderas Tandem, and jaladeras per INSTRUCTIVO_TECNICO.md table 3.8
- calc() door block extended: fregadero always-2, bajoEstandar width-threshold logic (<=50cm = 1 door, >50cm = 2 doors)
- cajonera and hornoBase protected by door-skip guard so no phantom puerta pieces are generated
- bisKey conditional enables bisagra_165 for esquineroCiego enL type without changing existing door hardware flow

## Task Commits

Each task was committed atomically:

1. **Task 1: Implement assignHardware() blocks for all 5 cocina bajo types** - `999eb24` (feat)
2. **Task 2: Update calc() door block for cocina bajo door count and 165-degree bisagras** - `954a5b8` (feat)

## Files Created/Modified
- `cotizador-template/cotizador-template.html` - Added 5 assignHardware blocks (42 lines) + extended calc() door block with door-count logic and bisagra_165 support (25 lines net addition)

## Decisions Made
- `cajonera` and `hornoBase` skip the door block via an early guard (`if m.type === 'cajonera' || m.type === 'hornoBase'`) — these types have no exterior puertas; their front surfaces are cajón frentes from calculateParts()
- `bajoEstandar` threshold is `m.width <= 50` because `m.width` is stored in cm in the calc() loop (50cm = 500mm)
- `bisKey` variable introduced before the two bisagra `autoHW.push` calls to cleanly support the `bisagra_165` key for esquineroCiego enL without duplicating the push block
- Jaladeras for cajonera drawers assigned in `assignHardware()` (per cajón), not in the door block, to match the pattern from `modulo` (closet drawers)

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- Hardware auto-assignment is complete for all 5 cocina bajo types — patas, zoclo clips, correderas Tandem, bisagras, and jaladeras all auto-populate from module type
- calc() door count rules are live: 1 vs 2 puertas auto-selects by width for bajoEstandar, fregadero always 2, cajonera/hornoBase skip entirely
- Next: UI wiring — bajo module selector buttons (estAddMod calls) and any UI for esquineroTipo selection (ciego vs enL)
- No regressions: all existing closet/TV module door logic unchanged (the new doorCnt code falls through to the `else` branch for non-cocina types)

## Self-Check: PASSED

All claims verified:
- FOUND: bajoEstandar in assignHardware (2 occurrences - calculateParts + assignHardware)
- FOUND: fregadero in assignHardware (2 occurrences)
- FOUND: pata q:4 pushed (5 times - all 5 bajo types)
- FOUND: zoclo_clip q:4 pushed (5 times - all 5 bajo types)
- FOUND: corr_tandem q:caj in cajonera
- FOUND: bisKey=bisagra_165 conditional in door block
- FOUND: cajonera/hornoBase guard in door block
- FOUND: doorCnt=2 for fregadero (2 occurrences)
- FOUND: m.width<=50 threshold for bajoEstandar
- FOUND: commits 999eb24 and 954a5b8

---
*Phase: 02-cocina-bajos*
*Completed: 2026-02-20*
