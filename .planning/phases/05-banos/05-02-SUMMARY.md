---
phase: 05-banos
plan: "02"
subsystem: ui-wiring
tags: [bano, TYPES, step2, MDF-RH, cajones, calc, door-logic]

# Dependency graph
requires:
  - phase: 05-banos
    plan: "01"
    provides: bajoLavabo/cajoneraBano/botiquin MODS entries with forType:['bano']
provides:
  - bano selectable as project type in Step 1 wizard (TYPES array entry)
  - calc() door skip for bajoLavabo/cajoneraBano (cajón frentes from calculateParts)
  - calc() botiquin door logic (1 or 2 puertas by width threshold)
  - MDF-RH validation extended to all 3 bano module types in step2()
  - cajones selector for bajoLavabo (1-2, default 2) in step2()
  - cajones selector for cajoneraBano (2-3, default 2) in step2()
affects: 05-banos (PDF plan 03), any bano workflow end-to-end

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Bano TYPES entry follows same { id, n, i } pattern as cocina/closet"
    - "door skip guard extended with || m.type === 'bajoLavabo' || m.type === 'cajoneraBano'"
    - "botiquin door count: w<=50cm = 1 puerta, w>50cm = 2 puertas (matches alacenaEstandar pattern)"
    - "MDF-RH validation: single condition covers fregadero + all 3 bano types"
    - "cajones selector uses APP.setModC (same as cocina cajonera) — m.cajones flows to calculateParts via materials.cajones"

key-files:
  created: []
  modified:
    - cotizador-template/cotizador-template.html

key-decisions:
  - "cajones selector uses APP.setModC (not setModShelves) — plan spec had wrong function; setModC correctly sets m.cajones which flows to calculateParts"
  - "MDF-RH condition extended to single || check covering fregadero + bajoLavabo + cajoneraBano + botiquin — DRY, reuses exact same isRH logic"
  - "botiquin is NOT in door skip guard — it correctly generates puertas via the door block"

patterns-established:
  - "Bano UI pattern: MDF-RH warning + cajones selector for lavabo/cajonera types using existing App.setModC mechanism"

requirements-completed: [BANO-04, BANO-05]

# Metrics
duration: 10min
completed: 2026-02-20
---

# Phase 5 Plan 02: Bano UI Wiring Summary

**Bano selectable as project type, MDF-RH validation for all bano modules, cajones selectors for bajoLavabo/cajoneraBano, and correct calc() door logic for botiquin and cajón types**

## Performance

- **Duration:** ~10 min
- **Started:** 2026-02-20T21:00:00Z
- **Completed:** 2026-02-20T21:10:00Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments

- Added `{ id: 'bano', n: 'Bano', i: '&#x1f6bf;' }` to TYPES array after cocina entry
- Extended calc() door skip guard to include `bajoLavabo` and `cajoneraBano` (their frentes come from calculateParts, same as cocina cajonera)
- Added `botiquin` to calc() door block with 1/2 puerta threshold at 500mm (same as alacenaEstandar pattern)
- Extended MDF-RH validation condition from `fregadero`-only to cover all 3 bano types (bajoLavabo, cajoneraBano, botiquin)
- Added cajones selector for bajoLavabo (1-2 options, default 2) in step2() using APP.setModC
- Added cajones selector for cajoneraBano (2-3 options, default 2) in step2() using APP.setModC
- Confirmed cajones value flows through existing m.cajones -> materials.cajones -> calculateParts() path (no extra plumbing needed)

## Task Commits

1. **Task 1: TYPES + calc() door logic** - `e003969` (feat)
2. **Task 2: MDF-RH validation + cajones selector** - `fafbc07` (feat)

## Files Created/Modified

- `cotizador-template/cotizador-template.html` - TYPES entry for bano, door skip guard update, botiquin door logic, MDF-RH validation extension, cajones selectors for bajoLavabo and cajoneraBano

## Decisions Made

- Used `APP.setModC` (not `setModShelves` as plan spec suggested) for cajones selectors — setModC correctly sets `m.cajones` which flows through to calculateParts via `materials.cajones`. The plan referenced setModShelves but that function only updates `m.shelves`, not `m.cajones`.
- MDF-RH condition merged into single `||` check rather than separate `if` blocks — DRY, single validation path for all wet-zone modules.
- botiquin correctly excluded from door skip guard — it has puertas (bisagras generated via calc() door block), unlike bajoLavabo/cajoneraBano.

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 1 - Bug] Used APP.setModC instead of APP.setModShelves for cajones selectors**
- **Found during:** Task 2
- **Issue:** Plan spec said to use `APP.setModShelves(i, n)` with index `i`, but the correct function for cajones is `APP.setModC(id, n)` with module `id`. setModShelves updates `m.shelves` (not `m.cajones`), and the existing cajonera pattern uses setModC with id.
- **Fix:** Used `APP.setModC` with `m.id` (matching the cajonera pattern), and defaulted cajones to 2 (not the plan's suggested index-based approach)
- **Files modified:** cotizador-template/cotizador-template.html

## Issues Encountered

- sed command (bash) emptied the file — recovered via `git restore`. All subsequent edits used the Edit tool exclusively.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- Bano UI wiring complete: TYPES, step2() module cards with cajones selectors and MDF-RH warnings, calc() door logic
- Ready for Phase 5 Plan 03: Bano PDF sections (genClientPDF/genInternalPDF bano-specific output)

## Self-Check: PASSED

- `id: 'bano'` confirmed at line 3142 in TYPES array
- `bajoLavabo || cajoneraBano` in door skip guard confirmed at line 5476
- `botiquin` door block confirmed at line 5504
- MDF-RH condition includes all 4 wet-zone types at line 7592
- bajoLavabo cajones selector [1,2] confirmed at line 7491
- cajoneraBano cajones selector [2,3] confirmed at line 7500
- Commits e003969 and fafbc07 exist in submodule git log

---
*Phase: 05-banos*
*Completed: 2026-02-20*
