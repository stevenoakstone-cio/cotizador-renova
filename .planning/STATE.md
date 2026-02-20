# Project State

## Project Reference

See: .planning/PROJECT.md (updated 2026-02-19)

**Core value:** Given module dimensions and material selection, instantly generate a complete, accurate despiece with pricing.
**Current focus:** Phase 5 — Banos (IN PROGRESS)

## Current Position

Phase: 5 of 10 (Banos)
Plan: 1 of 3 in current phase (COMPLETE)
Status: Phase 5 plan 1 complete — data layer done, ready for UI plan
Last activity: 2026-02-20 — Plan 05-01 completed: 3 bano MODS entries (bajoLavabo, cajoneraBano, botiquin) with calculateParts despiece and assignHardware wall-mount hardware per INSTRUCTIVO Section 4

Progress: [████████░░] 43%

## Performance Metrics

**Velocity:**
- Total plans completed: 3 (in phase 03)
- Average duration: ~4 min
- Total execution time: 0.23 hours

**By Phase:**

| Phase | Plans | Total | Avg/Plan |
|-------|-------|-------|----------|
| 01-foundation | 2 | 10 min | 5 min |
| 02-cocina-bajos | 3 | 23 min | ~8 min |
| 03-cocina-altos-torres | 2 (of 3) | 5 min | ~2.5 min |

**Recent Trend:**
- Last 5 plans: 02-01 (12 min), 02-02 (8 min), 02-03 (3 min), 03-01 (2 min), 03-02 (3 min)
- Trend: Accelerating (data layer plans very fast)

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
- Phase 3 plans 03-01 + 03-02 complete (03-03 is PDF/output for phase 3, may be phase 4 scope)
- forType filter pattern: MODS entries use forType array + cat for step2() filtering + grouping
- cubierta pricing: isCubierta flag on piece + cubiertaType from module; m2-based pricing separate from sheet area
- Extras modules (zoclo/vistaLateral/panelRelleno/cubierta): no hardware, no door toggles, no quoteMode
- zoclo auto-width: addMod('zoclo') sums all bajo module widths to pre-populate width field
- [Phase 02-cocina-bajos]: Cajonera cajones selector offers [2,3,4] only — matching MODS '2-4 cajones Blum Tandem'
- alacenaAventos calculateParts(): no entrepaño block — single open cavity for Aventos lift door sweep arc; door frente in Plan 02 calc()
- alacenaSobreCampana: structurally identical to alacenaEstandar; depth reduction is caller-driven (smaller d parameter)
- torreHornos microH defaults 400mm via materials.microHeight || 400; cajH formula: h - microH - 595 - 3*TK - 10
- torreDespensa: 4 entrepaños default on 32mm adjustable pin system (overrideable via materials.shelves)
- soporte_alacena HW_DEFAULTS key: $25 DEMO for wall-mount alacena hardware cost
- aventosModel must be passed to assignHardware() call — added to calc() options object in Plan 02
- alacenaAventos: no bisagra, no jaladera — aventos lift mechanism replaces both
- torreDespensa: doorH = round(h/2) - 3, doorCnt=2 (2 stacked half-height puertas)
- APP.setModShelves() added for cocina module shelf count (separate from closet setModS())
- togModProp reused for ventilacion toggle on horno modules (already existed from puerta marco toggle)
- cubiertaM2 tracks raw area before mermaFactor for display; mermaFactor applied only to cost calculation
- cascoCost = matCostI + chapCostI (when useDual) in calc() return; frenteCost = matCostF + chapCostF
- Subtotales por Categoria card conditional on P.type === 'cocina' only; closet/tv unaffected
- PDF pieces use p.q field (not p.qty) — plan spec had wrong field name, corrected during implementation
- genClientPDF() cocina sections: module index, despiece, hardware summary, extras (conditional), technical note
- genInternalPDF() cocina sections: module index with cost/margin columns, hardware detail with Carpinteria/Premium bucket, category cost summary
- Phase 4 complete: all 3 plans executed (04-01, 04-02, 04-03)
- Bano modules are all flotante (wall-mounted): no patas, no zoclo_clip for any bano type
- bajoLavabo cajH formula: (h-2*TK)/2 for 2-cajón, (h-TK) for 1-cajón (techo amarre difference)
- saqueU:true flag on bajoLavabo top cajon frente interno piece documents cespol notch (not separate piece)
- botiquin bisagras delegated to calc() door block (same as alacenaEstandar) — assignHardware only handles non-door hardware
- Phase 5 plan 05-01 complete: bajoLavabo/cajoneraBano/botiquin MODS + calculateParts + assignHardware + HW_DEFAULTS.soporte_bano

### Pending Todos

None yet.

### Blockers/Concerns

None yet.

## Session Continuity

Last session: 2026-02-20
Stopped at: Completed 05-01-PLAN.md — Bano data layer: 3 MODS entries + HW_DEFAULTS.soporte_bano + calculateParts + assignHardware for bajoLavabo, cajoneraBano, botiquin.
Resume file: None
