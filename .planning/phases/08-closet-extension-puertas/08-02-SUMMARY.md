---
phase: 08-closet-extension-puertas
plan: "02"
subsystem: ui
tags: [puerta, calculateParts, assignHardware, mods, step2, bastidor, relleno, bisagra-libro]

# Dependency graph
requires:
  - phase: 07-tv-modules
    provides: isBastidor pricing pattern in calc() bastidorCost accumulator
  - phase: 01-foundation
    provides: calculateParts(), assignHardware() generic functions
provides:
  - Puerta calculateParts rewrite: bastidor sandwich despiece (largueros x2, travesanos x2, tapas MDF x2, marco chambrana + tope)
  - Puerta assignHardware rewrite: relleno-driven bisagra_libro count (3 honeycomb/peinazos, 4 solida) + chapa_puerta x1
  - HW_DEFAULTS.bisagra_libro + HW_DEFAULTS.chapa_puerta keys
  - step2() relleno type selector with bisagra hint text
  - Updated marco toggle: 'Con marco chambrana' text + correct default (marco != false)
affects: [any phase using puerta module type, PDF output phases]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Puerta bastidor despiece: isBastidor: true on larguero/travesano pieces — priced per ml via existing bastidorCost accumulator (same pattern as panelFondo Phase 7)"
    - "Relleno-driven bisagra count: relleno === 'solida' ? 4 : 3 in assignHardware(); relleno passed via m.relleno in calc() options"
    - "Marco default behavior: materials.marco !== false (not materials.marco) — marco generated when property is undefined OR true; excluded only when explicitly false"

key-files:
  created: []
  modified:
    - cotizador-template/cotizador-template.html

key-decisions:
  - "Puerta tapas MDF (zone='f', t=3mm) are the visible faces — no separate frente pieces generated in calc() door block; puerta added to door skip guard"
  - "Bastidor pino pieces use isBastidor:true + bastidorCost accumulator from Phase 7 — no new pricing code needed"
  - "Marco toggle default: m.marco !== false (UI) + materials.marco !== false (calculateParts) ensures marco is ON by default without explicit initialization"
  - "Old materialType selector (Melamina/Vidrio/Metal) removed from puerta UI — replaced by relleno selector (honeycomb/peinazos/solida)"
  - "bisagra_libro replaces bisagra + base_clip for puerta type — door hinge type is libro/mariposa not cup/bastidor"

patterns-established:
  - "Puerta relleno prop flow: m.relleno set via setModProp() in step2 → passed as opts.relleno in assignHardware options in calc()"

requirements-completed: [PTED-01, PTED-02, PTED-03]

# Metrics
duration: 15min
completed: 2026-02-20
---

# Phase 8 Plan 02: Puerta Interior Module Rewrite Summary

**Puerta interior rewritten as bastidor-pino sandwich (largueros + travesanos + 2 tapas MDF + marco chambrana) with relleno-type-driven bisagra libro count (3 for honeycomb/peinazos, 4 for solida) selectable in step2() UI**

## Performance

- **Duration:** 15 min
- **Started:** 2026-02-20T22:40:00Z
- **Completed:** 2026-02-20T22:55:00Z
- **Tasks:** 2
- **Files modified:** 1

## Accomplishments
- Replaced single "Panel puerta" calculateParts piece with correct INSTRUCTIVO 7.1 bastidor sandwich despiece: Larguero bastidor x2 (isBastidor), Travesano bastidor x2 (isBastidor), Tapa MDF x2, and optional Jamba chambrana x2 + Cabezal chambrana x1 + Tope x3 when marco is active
- Rewrote assignHardware puerta branch: relleno === 'solida' → 4 bisagra_libro; honeycomb/peinazos → 3 bisagra_libro; always 1 chapa_puerta; old getBisagraCount-based logic removed
- Added HW_DEFAULTS.bisagra_libro and HW_DEFAULTS.chapa_puerta (guarded conditionals + commented real prices in HW_REAL block)
- Added puerta to calc() door skip guard (tapas MDF are the door faces — no separate frente pieces needed from the door block)
- Replaced old materialType selector (Melamina/Vidrio/Metal) with relleno type selector in step2() with 3 options and bisagra count hint; updated marco toggle to default-ON behavior and renamed "Con marco chambrana"

## Task Commits

Each task was committed atomically:

1. **Task 1: Rewrite puerta calculateParts + assignHardware + HW_DEFAULTS** - `1ab04fc` (feat)
2. **Task 2: step2() puerta relleno selector + marco toggle update** - `78b71e5` (feat)

## Files Created/Modified
- `cotizador-template/cotizador-template.html` - Rewrote puerta calculateParts branch (bastidor sandwich), assignHardware branch (relleno-driven bisagra_libro), added 2 HW_DEFAULTS keys, updated step2() puerta UI block

## Decisions Made
- `bisagra_libro` replaces `bisagra + base_clip` for puerta type — door hinge type is libro/mariposa (butterfly hinge), not cup bisagra; separate base_clip not needed
- `m.marco !== false` default behavior: marco is ON by default (undefined evaluates as not false), user must explicitly toggle it off; consistent between UI and calculateParts check
- Old `materialType` property on puerta modules no longer used; new `relleno` property drives bisagra count
- Bastidor pieces for puerta reuse the same `isBastidor: true` + `bastidorCost` accumulator from Phase 7 panelFondo — no new pricing code required

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered
None.

## User Setup Required
None - no external service configuration required.

## Next Phase Readiness
- Puerta module now generates correct bastidor sandwich despiece with relleno-driven bisagra count
- TYPES.puerta + MODS.puerta forType ['puerta'] confirmed working (existed from prior phases)
- Phase 08 complete: both plans (08-01 colgadorCorto/maletero + 08-02 puerta rewrite) executed
- Ready for Phase 09 (next phase in roadmap)

## Self-Check: PASSED

- FOUND: cotizador-template/cotizador-template.html (modified — bastidor despiece, relleno selector, new HW_DEFAULTS keys)
- FOUND: commit 1ab04fc (Task 1: calculateParts + assignHardware + HW_DEFAULTS)
- FOUND: commit 78b71e5 (Task 2: step2() relleno selector + marco toggle)
- FOUND: .planning/phases/08-closet-extension-puertas/08-02-SUMMARY.md

---
*Phase: 08-closet-extension-puertas*
*Completed: 2026-02-20*
