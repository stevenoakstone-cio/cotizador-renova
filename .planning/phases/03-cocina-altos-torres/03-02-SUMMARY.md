---
phase: 03-cocina-altos-torres
plan: "02"
subsystem: ui
tags: [cotizador, cocina, alacena, aventos, torre, assignHardware, calc, step2, Blum]

# Dependency graph
requires:
  - phase: 03-cocina-altos-torres
    plan: "01"
    provides: "5 MODS entries, 4 HW_DEFAULTS keys (aventos_hk/hl/hs, soporte_alacena), calculateParts() blocks"
provides:
  - 5 assignHardware() blocks for alacena/torre types
  - calc() door logic for all 5 new types (skip guard, aventos single door, alacena 1-2, despensa 2 half-height)
  - step2() UI selectors for aventos model (HK-S/HL/HS) and entrepaño count
  - APP.setModAventos() and APP.setModShelves() methods
affects:
  - 03-cocina-altos-torres (plan 03): rendering/summary for new module types

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "alacenaAventos: aventosKey = 'aventos_' + (opts.aventosModel || 'hk') — key lookup pattern for model-specific HW"
    - "torreHornos in door skip guard: same approach as cajonera/hornoBase — oven tower has no external door"
    - "torreDespensa doorH = round(h/2) - 3 — stacked half-height door pair, not full-height single"
    - "aventos bisagra guard: m.type !== 'alacenaAventos' — adventure lift doors replace bisagra/jaladera entirely"

key-files:
  created: []
  modified:
    - cotizador-template/cotizador-template.html

key-decisions:
  - "aventosModel passed to assignHardware() via calc() options — required for key lookup to work (was not originally included in call)"
  - "alacenaAventos has no bisagra and no jaladera — aventos mechanism handles both opening and handle functions"
  - "torreDespensa doorH = round(h/2) - 3 to represent 2 stacked doors; doorCnt=2 always per INSTRUCTIVO"
  - "APP.setModShelves() is a new dedicated method (not reusing setModS) to match naming convention for cocina module properties"

patterns-established:
  - "UI selector pattern: if (m.type === 'X') { var prop = m.prop || default; h += selector buttons; }"
  - "APP method pattern: function(id, val) { find m; m.prop = val; S.editedPieces = {}; render(); }"

requirements-completed: [CALT-01, CALT-02, CALT-03, CTOR-01, CTOR-02]

# Metrics
duration: 3min
completed: 2026-02-20
---

# Phase 03 Plan 02: Cocina Altos + Torres Hardware + UI Summary

**5 assignHardware() blocks, calc() door rules for altos/torres (aventos single door, alacena 1-2 by width, despensa 2 half-height), and step2() selectors for aventos model (HK-S/HL/HS) and entrepaño count**

## Performance

- **Duration:** 3 min
- **Started:** 2026-02-20T03:42:38Z
- **Completed:** 2026-02-20T03:45:58Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments
- Implemented 5 assignHardware() else-if blocks covering all alacena and torre module types with correct hardware returns
- Extended calc() door block: torreHornos skip guard, alacenaAventos single elevating door (no bisagras/jaladera), alacenaEstandar/alacenaSobreCampana 1-2 doors by width threshold, torreDespensa 2 half-height stacked doors
- Added step2() UI selectors for aventos model (HK-S/HL/HS), alacena entrepaños (0-3), and torre despensa entrepaños (2-6)
- Added APP.setModAventos() and APP.setModShelves() methods with re-render trigger

## Task Commits

Each task was committed atomically:

1. **Task 1: assignHardware() blocks and calc() door logic for altos + torres** - `355696b` (feat)
2. **Task 2: step2() UI selectors and APP methods for aventos/alacena/torre** - `86396cf` (feat)

## Files Created/Modified
- `cotizador-template/cotizador-template.html` - 5 assignHardware blocks, calc() door extensions, step2() selectors, 2 APP methods (114 lines added total)

## Decisions Made
- aventosModel was not originally included in the assignHardware() call in calc() — added it as a Rule 2 auto-fix (missing critical option propagation) so the key lookup `'aventos_' + opts.aventosModel` functions correctly
- alacenaAventos skips both bisagra push AND jaladera/tipon block — aventos lift mechanism includes integrated handle, so neither is appropriate
- torreDespensa doorH = Math.round(h / 2) - 3 correctly represents 2 stacked doors at half the tower height each
- APP.setModShelves() created as a separate method from setModS() to match the consistent cocina naming pattern (setModC, setModW2, etc.)

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 2 - Missing Critical] Added aventosModel to assignHardware() call in calc()**
- **Found during:** Task 1 (assignHardware() implementation)
- **Issue:** Plan specified opts.aventosModel lookup in assignHardware(), but the caller (calc()) did not pass aventosModel in the options object — the lookup would always default to 'hk'
- **Fix:** Added `aventosModel: m.aventosModel` to the assignHardware() options object in calc()
- **Files modified:** cotizador-template/cotizador-template.html
- **Verification:** grep confirms aventosModel is present in the assignHardware call
- **Committed in:** 355696b (Task 1 commit)

---

**Total deviations:** 1 auto-fixed (1 missing critical option propagation)
**Impact on plan:** Auto-fix was essential for aventos model selection to work at all. No scope creep.

## Issues Encountered
None

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- All 5 new module types now have complete hardware assignment and door cost logic
- step2() shows appropriate UI for each new type (aventos model, entrepaño counts)
- Plan 03 can now implement step3() rendering / summary display for alacena and torre modules

---
*Phase: 03-cocina-altos-torres*
*Completed: 2026-02-20*
