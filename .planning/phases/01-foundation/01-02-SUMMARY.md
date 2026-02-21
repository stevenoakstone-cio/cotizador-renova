---
phase: 01-foundation
plan: "02"
subsystem: calculation
tags: [javascript, es5, hardware-assignment, closet, cocina, refactor]

requires:
  - phase: 01-foundation
    plan: "01"
    provides: "calculateParts() returning { pieces, hwHints } + calc() delegation pattern"

provides:
  - "assignHardware(moduleType, dimensions, options) generic function returning array of { key, q }"
  - "calculateParts() cleaned to return plain pieces[] array (no hwHints)"
  - "calc() delegates piece generation to calculateParts() and hardware to assignHardware()"
  - "AUDIT comments on all 6 hardware rules in assignHardware()"

affects: [02-cocina, 03-bano, 04-tv, 05-puerta, any plan adding new module types]

tech-stack:
  added: []
  patterns:
    - "assignHardware() as single hardware-assignment gateway — all module-specific hardware goes through this function"
    - "Clean separation: calculateParts() = pieces only, assignHardware() = hardware only, calc() door block = cross-cutting door hardware"
    - "// AUDIT: prefix on all hardware assignment rules — corrTier, jaladera suppression, hinge count logic"

key-files:
  created: []
  modified:
    - "cotizador-template/cotizador-template.html"

key-decisions:
  - "assignHardware() returns plain array of { key, q } objects — caller stamps modId and pushes to autoHW in calc()"
  - "calculateParts() changed from returning { pieces, hwHints } to returning pieces[] only — hwHints pattern retired"
  - "puerta module hardware (bisagra, base_clip) moved from calc() inline block into assignHardware() — heightCm and materialType passed via options"
  - "esquinero gets tubo + soporte (same as colgador) per plan spec — corner module has a hanging bar"
  - "Tasks 1+2 committed atomically (inseparable — function creation and wiring are the same file operation)"

patterns-established:
  - "assignHardware() gateway: add new module hardware by inserting else if (moduleType === '...') block"
  - "AUDIT comments on every hardware rule constant (corrTier default, jaladera suppression, hinge count source)"

requirements-completed: [FOUND-02]

duration: 6min
completed: 2026-02-19
---

# Phase 1 Plan 2: assignHardware() Generic Hardware Auto-Assignment Engine Summary

**ES5 assignHardware() function with 6 module hardware rules extracted from calc()/hwHints pattern, clean separation from calculateParts() which now returns pieces[] only**

## Performance

- **Duration:** 6 min
- **Started:** 2026-02-19T03:50:00Z
- **Completed:** 2026-02-19T03:56:00Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments
- Created `assignHardware(moduleType, dimensions, options)` at line 4813 (after calculateParts(), before calc()) covering 6 module types: modulo, colgador, zapatero, esquinero, cajonBase, puerta — plus empty-return comment for all others
- Added AUDIT comments on every hardware rule: corrTier default, jaladera suppression logic, tube+bracket pattern, shoe tray count, concealed slide type, hinge count via getBisagraCount()
- Refactored `calculateParts()` to return plain `pieces[]` array — removed hwHints variable and all hwHints.push() calls from modulo, colgador, zapatero, cajonBase blocks
- Refactored `calc()` to call `assignHardware()` after `calculateParts()`, replacing the hwHints delegation block and the inline puerta bisagra block
- Kept the door hardware block in calc() unchanged — cross-cutting concern (doors on any module type)

## Task Commits

Both tasks committed together as one atomic change (inseparable — function creation and wiring are the same file operation):

1. **Tasks 1+2: assignHardware() + calc() refactor + calculateParts() cleanup** - `e59d8ce` (feat)

## Files Created/Modified
- `cotizador-template/cotizador-template.html` - Added assignHardware() function (57 lines) between calculateParts() and calc(); refactored calc() delegation block; cleaned calculateParts() of hwHints

## Decisions Made
- `assignHardware()` returns plain `{ key, q }[]` — caller stamps `modId` and pushes to `autoHW` (consistent with how pieces are stamped before pushing)
- `calculateParts()` now returns plain `pieces[]` — hwHints pattern retired; clean separation between piece generation and hardware assignment
- `puerta` module type's bisagra/base_clip moved from calc() inline block into assignHardware() — `heightCm` and `materialType` are passed via options, so getBisagraCount() works correctly
- Committed Tasks 1 and 2 atomically (same single-file operation — can't logically separate function creation from its wiring)

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
- None.

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- `assignHardware()` is ready at line 4813 — any new module type adds an `else if (moduleType === '...')` block
- Foundation layer is complete: `calculateParts()` for pieces, `assignHardware()` for hardware — both generic, both extensible
- Phase 2 (cocina) can add `alacenaAlta`, `gabineteBase` hardware rules to `assignHardware()` without touching calc()
- calc() is cleaner: piece delegation (calculateParts) + hardware delegation (assignHardware) + door block (cross-cutting)

## Self-Check: PASSED

- FOUND: `.planning/phases/01-foundation/01-02-SUMMARY.md`
- FOUND: `cotizador-template/cotizador-template.html`
- FOUND: commit `e59d8ce` in cotizador-template repo
- FOUND: `function assignHardware` at line 4813
- FOUND: `function calculateParts` returns `pieces` array only (no hwHints)
- FOUND: `function calc` calls `assignHardware()` + `calculateParts()` with clean separation
- CONFIRMED: zero `hwHints` references remaining in codebase

---
*Phase: 01-foundation*
*Completed: 2026-02-19*
