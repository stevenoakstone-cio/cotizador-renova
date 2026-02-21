---
phase: 01-foundation
plan: "01"
subsystem: calculation
tags: [javascript, es5, piece-generation, closet, modulo, refactor]

requires: []
provides:
  - "calculateParts(moduleType, dimensions, materials) generic function returning { pieces, hwHints }"
  - "AUDIT comments on all closet module dimension formulas (subtraction constants documented)"
  - "calc() delegates all piece generation to calculateParts() via { pieces, hwHints } return"
affects: [02-cocina, 03-bano, 04-tv, 05-puerta, any plan adding new module types]

tech-stack:
  added: []
  patterns:
    - "calculateParts() as single piece-generation gateway — all module types go through this function"
    - "{ pieces, hwHints } return pattern — separates structural pieces from hardware hints"
    - "// AUDIT: prefix for dimension formula explanations — used on every formula constant"

key-files:
  created: []
  modified:
    - "cotizador-template/cotizador-template.html"

key-decisions:
  - "calculateParts() returns { pieces, hwHints } dual-return so calc() can push hardware hints into autoHW without calculateParts() needing access to the autoHW array"
  - "bisagra/base_clip for puerta type kept in calc() because they require getBisagraCount() which reads CONFIG — not passed into calculateParts()"
  - "esquineroGiratorio uses local w2g variable (not shared w2) to avoid variable shadowing with esquinero block"
  - "doors flag passed into materials param so modulo can suppress jaladera hwHint when doors are present"

patterns-established:
  - "AUDIT comments: every dimension formula subtraction constant is documented inline with // AUDIT: prefix"
  - "hwHints pattern: calculateParts returns hardware hints as { key, q } objects; caller stamps modId and pushes to autoHW"

requirements-completed: [FOUND-01, FOUND-03]

duration: 4min
completed: 2026-02-19
---

# Phase 1 Plan 1: calculateParts() Generic Piece-Generation Engine Summary

**ES5 calculateParts() function extracting all 16 module types from calc() inline chain, with AUDIT comments on every closet formula, returning { pieces, hwHints } for clean delegation**

## Performance

- **Duration:** 4 min
- **Started:** 2026-02-19T03:41:23Z
- **Completed:** 2026-02-19T03:45:20Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments
- Created `calculateParts(moduleType, dimensions, materials)` at line 4618 (before calc()) covering all 16 module types: modulo, colgador, zapatero, esquinero, entrepa, torreLateral, panelTV, moduloCentral, lateralAbierto, alacenaAlta, gabineteBase, cajonBase, despensa, espacioCampana, esquineroGiratorio, puerta
- Added 42 `// AUDIT:` inline comments documenting every dimension subtraction constant (e.g., `w - 2*TK = w - 32 at 16mm`, 6mm drawer gap, 50mm slide clearance, 100mm deduction for slides+structure)
- Wired calc() to delegate via `var cpResult = calculateParts(...)` replacing ~470 lines of inline if/else chain with ~35 lines of delegation + context stamping
- Kept door piece generation and bisagra auto-assignment in calc() as specified (cross-cutting concerns)

## Task Commits

Both tasks committed together as one atomic change (inseparable — function creation and wiring are the same file operation):

1. **Tasks 1+2: calculateParts() + calc() delegation** - `56b7fe1` (feat)

## Files Created/Modified
- `cotizador-template/cotizador-template.html` - Added calculateParts() function (199 lines) before calc(); refactored calc() inline piece chain into 35-line delegation block

## Decisions Made
- `calculateParts()` returns `{ pieces, hwHints }` dual-return so hardware hints flow back to caller without coupling to the autoHW array
- `bisagra`/`base_clip` for `puerta` module type remain in `calc()` because they require `getBisagraCount()` which reads CONFIG and needs height/materialType in the module scope context
- `doors` property passed via materials param so `modulo` can omit `jaladera` hwHint when doors are present (jaladera would go on door, not drawer)
- `esquineroGiratorio` uses separate `w2g` local variable to avoid shadowing the shared `w2` from `esquinero` in the same function

## Deviations from Plan

None - plan executed exactly as written. The dual-return `{ pieces, hwHints }` pattern was specified in Task 2 and implemented as specified.

## Issues Encountered
- `cotizador-template` directory is an independent git repo (gitlinked at mode 160000 in the parent repo). Committed to the inner repo directly. This is the correct workflow for this project.

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- `calculateParts()` is ready at line 4618 — any new module type (cocina variants, bano, puerta de closet) can be added by inserting a new `else if (moduleType === '...')` block
- `calc()` is ~400 lines shorter; piece-generation is now a single delegation call
- AUDIT comments create the documented dimension pattern library that Plans 02+ can reference
- Plan 01-02 (assignHardware generic function) can now use hwHints from calculateParts() as its starting point

---
*Phase: 01-foundation*
*Completed: 2026-02-19*
