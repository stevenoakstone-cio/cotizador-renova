---
phase: 02-cocina-bajos
plan: "01"
subsystem: ui
tags: [javascript, es5, despiece, calculateParts, MODS, HW_DEFAULTS, cocina, bajos]

# Dependency graph
requires:
  - phase: 01-foundation
    provides: calculateParts() generic engine and assignHardware() auto-assignment function
provides:
  - 5 new MODS entries for cocina bajo module types (bajoEstandar, fregadero, cajonera, esquineroCiego, hornoBase)
  - 4 new HW_DEFAULTS keys (pata, zoclo_clip, corr_tandem, bisagra_165) with DEMO and REAL prices
  - 5 new calculateParts() else-if blocks generating accurate despiece per INSTRUCTIVO_TECNICO.md
affects:
  - 02-cocina-bajos (plans 02+): assignHardware for bajo types, UI for bajo module selection
  - future calc engine integration for cocina mode

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "ES5 var-only: no let/const, no arrow functions, no template literals in the single HTML monolith"
    - "AUDIT comment convention: every formula constant subtraction gets // AUDIT: inline explanation"
    - "Fondo HDF: t:3 for 3mm HDF backs, t:TK for all structural melamina pieces"
    - "Zone flag: z='i' interior/casco pieces, z='f' frente/visible pieces (chapa cost differentiation)"
    - "Bajo module amarre pattern: 2 x 80mm tiras replace techo where full panel omitted (fregadero/cajonera)"

key-files:
  created: []
  modified:
    - cotizador-template/cotizador-template.html

key-decisions:
  - "bajoEstandar includes techo + 2 amarres + entrepaño per INSTRUCTIVO Module Base con Puertas spec"
  - "fregadero omits techo and entrepaño entirely — replaced by amarre frontal + amarre trasero (2x80mm)"
  - "cajonera omits techo — only amarre superior; Blum Tandem dimensions: -80mm depth, -26mm width for correderas"
  - "esquineroCiego uses w2ec second-leg dimension; Base/Techo footprint = (w-TK) x (w2ec-TK)"
  - "hornoBase entrepaño positioned at 595mm from top; cajH guard (>50mm) prevents degenerate cajón inferior"
  - "HW_DEFAULTS keys added as guarded conditionals (if !HW_DEFAULTS.pata) to preserve override capability"

patterns-established:
  - "Cocina bajo casco height: 720mm standard, passed in as h parameter"
  - "Cocina bajo depth: 580mm standard, passed in as d parameter"
  - "Inner width formula: w - 2*TK (at 15mm TK = w-30); used consistently across all bajo types"
  - "Amarre height: 80mm fixed per INSTRUCTIVO para todos los modulos bajos"
  - "HDF fondo: always t:3, inner width (w-2*TK), full casco height"

requirements-completed: [CBAJ-01, CBAJ-02, CBAJ-03, CBAJ-04, CBAJ-05]

# Metrics
duration: 12min
completed: 2026-02-19
---

# Phase 2 Plan 01: Cocina Bajos Data Layer and calculateParts() Summary

**5 cocina bajo module types (bajoEstandar, fregadero, cajonera, esquineroCiego, hornoBase) added to MODS + HW_DEFAULTS + calculateParts() with INSTRUCTIVO_TECNICO.md-accurate despiece formulas**

## Performance

- **Duration:** 12 min
- **Started:** 2026-02-19T~20:50Z
- **Completed:** 2026-02-19T~21:02Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments
- 5 new MODS entries with correct standard-width arrays per INSTRUCTIVO section 3.2
- 4 new HW_DEFAULTS keys (pata, zoclo_clip, corr_tandem, bisagra_165) with DEMO prices and REAL commented-block prices
- 5 calculateParts() else-if blocks returning accurate piece arrays matching INSTRUCTIVO 3.3 formulas
- Every dimension constant annotated with AUDIT comment explaining the deduction
- Fondo pieces correctly use t:3 (3mm HDF) per INSTRUCTIVO spec; structural pieces use t:TK (15mm)

## Task Commits

Each task was committed atomically:

1. **Task 1: Add MODS entries and HW_DEFAULTS for cocina bajos** - `5f993ae` (feat)
2. **Task 2: Implement calculateParts() blocks for all 5 cocina bajo module types** - `bb6026c` (feat)

## Files Created/Modified
- `cotizador-template/cotizador-template.html` - Added 5 MODS entries, 4 HW_DEFAULTS keys, 5 calculateParts blocks (135 lines net addition)

## Decisions Made
- `bajoEstandar` follows INSTRUCTIVO Module Base con Puertas: costados + base + techo + fondo HDF + 2 amarres (sup/inf 80mm) + entrepaño
- `fregadero` omits techo entirely — 2 amarres (frontal + trasero) provide structural rigidity; no entrepaño for plumbing clearance
- `cajonera` omits techo, uses single amarre sup; Blum Tandem corredera dimensions used (-80mm depth, -26mm width = 13mm per side for Tandem clip system)
- `esquineroCiego` uses `dimensions.w2` as second leg; Base/Techo footprint is `(w-TK) x (w2ec-TK)` to account for one costado on each leg
- `hornoBase` places entrepaño at 595mm from top per spec; cajH guard (`> 50mm`) prevents generating degenerate cajón inferior
- HW_DEFAULTS new keys added as guarded `if (!HW_DEFAULTS.key)` conditionals after the existing fallback block, matching Phase 1 pattern

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
- `cotizador-template` is a git submodule under the parent repo. All commits were made inside the submodule at `D:\claude-projects\project-renova\cotizador-template\`. This is consistent with Phase 1 commits.

## Next Phase Readiness
- 5 bajo module types now available for calculateParts() calls from any calc engine
- assignHardware() (Phase 01-02) does not yet have entries for bajo types — this will be addressed in Phase 02-02
- UI module selector buttons for bajo types (estAddMod calls) are not yet wired — expected in a later Phase 02 plan
- No regressions: all existing closet/TV/cocina module types remain untouched

---
*Phase: 02-cocina-bajos*
*Completed: 2026-02-19*
