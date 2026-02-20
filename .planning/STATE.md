# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-02-19)

**Core value:** Given module dimensions and material selection, instantly generate a complete, accurate despiece with pricing.
**Current focus:** Phase 3 — Cocina Altos + Torres (IN PROGRESS)

## Current Position

Phase: 3 of 10 (Cocina Altos + Torres)
Plan: 1 of 3 in current phase (COMPLETE)
Status: In progress
Last activity: 2026-02-20 — Plan 03-01 completed: MODS entries, HW_DEFAULTS, and calculateParts() blocks for 5 alacena/torre types

Progress: [████░░░░░░] 22%

## Performance Metrics

**Velocity:**
- Total plans completed: 2
- Average duration: 5 min
- Total execution time: 0.18 hours

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 01-foundation | 2 | 10 min | 5 min |
| 02-cocina-bajos | 3 | 23 min | ~8 min |

**Recent Trend:**
- Last 5 plans: 01-01 (4 min), 01-02 (6 min), 02-01 (12 min), 02-02 (8 min), 02-03 (3 min)
- Trend: Stable

*Updated after each plan completion*

## Accumulated Context

### Decisions

Decisions are logged in PROJECT.md Key Decisions table.
Recent decisions affecting current work:

- Architecture: Single HTML monolith — keep all code in one file, no build step
- Hardware pricing: Threshold-based ($500 split) — items at/below go into carpentry bucket (3x), above get premium margin (30%)
- Block execution order: Cocinas first (Phase 2-4) because it is the product for the first sale
- Generic functions (FOUND-01, FOUND-02): Build these in Phase 1 before any new module type to avoid duplication
- calculateParts() return pattern changed: now returns plain pieces[] only (hwHints pattern retired in Plan 01-02)
- assignHardware() handles all module-specific hardware; puerta bisagra/base_clip now in assignHardware() (not calc())
- AUDIT comments: every dimension formula subtraction constant documented inline with // AUDIT: prefix
- esquinero gets tubo + soporte (same as colgador) — corner module has a hanging bar per plan spec
- Cocina bajo casco: h=720mm, d=580mm standard; inner width = w - 2*TK (at 15mm = w-30)
- fregadero omits techo and entrepaño entirely — 2 amarres (frontal+trasero 80mm) for structural rigidity + plumbing space
- Blum Tandem corredera dimensions: -80mm depth, -26mm width (13mm per side for clip system)
- esquineroCiego Base/Techo footprint: (w-TK) x (w2-TK) to account for costado on each leg
- hornoBase entrepaño at 595mm from top per INSTRUCTIVO; cajH guard >50mm prevents degenerate cajón
- HW_DEFAULTS new keys: guarded conditionals (if !HW_DEFAULTS.key) matching Phase 1 pattern
- assignHardware() bajo types: pata x4 + zoclo_clip x4 for all bajos; corr_tandem for cajonera (per cajón) + hornoBase (x1)
- calc() door guard: cajonera + hornoBase skip door block entirely (no puertas)
- calc() bajoEstandar: 1 puerta if m.width<=50cm, 2 if >50cm per INSTRUCTIVO 3.3
- calc() fregadero: always doorCnt=2 regardless of width
- calc() bisKey pattern: esquineroCiego enL uses bisagra_165; all others use bisagra
- Phase 2 complete: all 3 plans (02-01 + 02-02 + 02-03) executed
- [Phase 02-cocina-bajos]: Cajonera cajones selector offers [2,3,4] only — matching MODS '2-4 cajones Blum Tandem'
- alacenaAventos calculateParts(): no entrepaño block — single open cavity for Aventos lift door sweep arc; door frente in Plan 02 calc()
- alacenaSobreCampana: structurally identical to alacenaEstandar; depth reduction is caller-driven (smaller d parameter)
- torreHornos microH defaults 400mm via materials.microHeight || 400; cajH formula: h - microH - 595 - 3*TK - 10
- torreDespensa: 4 entrepaños default on 32mm adjustable pin system (overrideable via materials.shelves)
- soporte_alacena HW_DEFAULTS key: $25 DEMO for wall-mount alacena hardware cost

### Pending Todos

None yet.

### Blockers/Concerns

None yet.

## Session Continuity

Last session: 2026-02-20
Stopped at: Completed 03-01-PLAN.md — MODS entries, HW_DEFAULTS, and calculateParts() for 5 cocina altos + torres types
Resume file: None
