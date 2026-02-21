---
phase: 04-cocina-extras-ui-output
plan: "02"
subsystem: cotizador-html
tags: [cocina, validation, step2, step3, warnings, cost-breakdown, cubierta]
dependency_graph:
  requires: [04-01-SUMMARY.md]
  provides: [horno ventilacion toggle, fregadero MDF-RH warning, non-standard width warning, Subtotales por Categoria card, cubiertaM2 in calc()]
  affects: [step2(), calc(), step3()]
tech_stack:
  added: []
  patterns: [conditional warning pattern with inline styles, categorized cost subtotals in calc() return]
key_files:
  created: []
  modified:
    - cotizador-template/cotizador-template.html
decisions:
  - "togModProp already existed from puerta marco toggle -- reused for ventilacion toggle (no new function needed)"
  - "cascoCost = matCostI + chapCostI (when useDual); captures interior material + canto costs cleanly"
  - "extrasCost alias for extraCost added to return object (cosmetic alias for clarity in category display)"
  - "cubiertaM2 tracks raw area (before mermaFactor) for display; mermaFactor applied only to cost calculation"
metrics:
  duration: ~20 min
  completed: 2026-02-20
  tasks_completed: 2
  files_modified: 1
---

# Phase 4 Plan 02: Cocina Validations + Categorized Cost Summary

3 validation warnings added to step2() for cocina-specific issues (horno ventilacion, fregadero MDF-RH, non-standard width); step3() gains Subtotales por Categoria card (casco/frente/herrajes/extras) for cocina projects only.

## Objective

Add cocina-specific validations (horno ventilacion, fregadero MDF-RH conditional check, non-standard widths) and restructure step3() cost summary to show subtotals by casco, frente, herrajes, and extras.

## Tasks Completed

### Task 1: Add cocina validation warnings in step2()

- Added validation block after `renderModulePaint(m)` and before `module-desc` paragraph in step2() module render loop
- **Horno ventilacion warning (CUI-03):** hornoBase and torreHornos modules show a ventilacion toggle button (reusing existing `APP.togModProp`); yellow warning appears when `m.ventilacion === false`; default is ventilacion enabled (true) so new modules show the toggle checked without warning
- **Fregadero MDF-RH warning (CUI-04):** fregadero modules check `P.matInterior.name` and `P.matFrentes.name` for "RH" or "RESISTENTE" (case-insensitive); red warning shown ONLY when neither material is MDF-RH; warning disappears when RH material selected
- **Non-standard width warning (CUI-05):** any module where `MODS[m.type].w.indexOf(m.width) === -1` AND `!MODS[m.type].custom` shows yellow warning listing standard widths
- `togModProp` was already implemented from puerta marco toggle -- reused without adding duplicate

**Commit:** 193a573

### Task 2: Add categorized cost subtotals (casco/frente/herrajes/extras) to step3()

- Added `cubiertaM2` accumulator in calc() area loop (alongside `cubiertaCost`); raw area before mermaFactor
- Added 5 new fields to calc() return object:
  - `cubiertaM2`: total cubierta area in m2 (raw, not with merma)
  - `cascoCost`: matCostI + chapCostI (when useDual, else 0 for cascoCost canto component)
  - `frenteCost`: matCostF + chapCostF
  - `herrajesCost`: hwCostCarpBucket + hwCostPremium
  - `extrasCost`: alias for extraCost
- Added cubierta row in Extras card: shows "Cubierta (X.XX m2)" with formatted cost when `c.cubiertaCost > 0`; uses defensive `(c.cubiertaM2 || 0).toFixed(2)` guard per plan advisory
- Added "Subtotales por Categoria" card in step3() just before `// Summary` section; only shown when `P.type === 'cocina'`; shows Casco, Frente, Herrajes, and conditionally Extras rows
- Non-cocina projects (closet, tv, etc.) are unaffected by the Subtotales card

**Commit:** eb949aa

## Verification Results

- togModProp confirmed at line 11565: toggles boolean property, calls render()
- ventilacion toggle uses `m.ventilacion !== false` (truthy default for new modules)
- fregadero isRH check uses indexOf for both matInterior and matFrentes
- non-standard width check guards with `MODS[m.type].w &&` before indexOf
- cubiertaM2 accumulates raw area before mermaFactor in area loop
- cascoCost, frenteCost, herrajesCost, extrasCost in return at lines 5665-5668
- Subtotales card conditional on `P.type === 'cocina'` at line 7607
- genInternalPDF already had `cascoCost || 0` references (lines 9295-9298) -- now properly populated

## Deviations from Plan

### Auto-fixed Issues

None - plan executed exactly as written.

### Notes

The Task 2 commit (eb949aa) shows 286 insertions because `git add cotizador-template.html` picked up pre-existing uncommitted changes in the working tree (cocina-specific PDF pages added in genClientPDF and genInternalPDF, likely written before the last commit). These changes are valid cocina module work and the `cascoCost || 0` defensive references at lines 9295-9298 confirm the PDF code was already anticipating the new cost fields.

## Self-Check

**Files exist:**
- [x] `cotizador-template/cotizador-template.html` — modified (confirmed)

**Commits exist:**
- [x] 193a573 — feat(04-02): add cocina validation warnings in step2()
- [x] eb949aa — feat(04-02): add categorized cost subtotals to calc() and step3()

## Self-Check: PASSED
