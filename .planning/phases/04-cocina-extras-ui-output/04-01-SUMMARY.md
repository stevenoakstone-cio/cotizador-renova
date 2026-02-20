---
phase: 04-cocina-extras-ui-output
plan: "01"
subsystem: cotizador-html
tags: [cocina, extras, mods, step2, filtering, grouping, cubierta, zoclo]
dependency_graph:
  requires: [03-02-SUMMARY.md]
  provides: [forType field on all MODS, 4 extras module types, step2 type filtering, category grouping]
  affects: [step2(), calculateParts(), assignHardware(), calc(), addMod()]
tech_stack:
  added: []
  patterns: [forType filter pattern, cat grouping pattern, isCubierta m2 pricing flag]
key_files:
  created: []
  modified:
    - cotizador-template/cotizador-template.html
decisions:
  - "cubierta piece uses isCubierta:true flag + t:0 to signal m2-based pricing; excluded from sheet area loop"
  - "forType filter: modules without forType shown only for 'otro' or unset type (backward compat for legacy modules)"
  - "Category header rendered only when groupOrder.length > 1 to avoid lone header for closet/tv types"
  - "cubiertaCost added to extraCost bucket (multiplied by extrasMultiplier), not carpentryBucket"
  - "zoclo auto-width uses existing.quantity || 1 so future multi-quantity support works correctly"
metrics:
  duration: ~45 min
  completed: 2026-02-20
  tasks_completed: 2
  files_modified: 1
---

# Phase 4 Plan 01: Cocina Extras Modules + Step2 Filtering Summary

4 cocina extras module types (zoclo, vistaLateral, panelRelleno, cubierta) added to MODS with calculateParts() blocks and m2 cubierta pricing; forType field added to all MODS entries enabling step2() to filter and group modules by project type.

## Objective

Add 4 cocina extras module types (zoclo, vista lateral, panel relleno, cubierta) to MODS and calculateParts(). Add `forType` category field to all MODS entries. Implement module filtering and category grouping in step2(). Zoclo auto-calculates width from total bajo module widths.

## Tasks Completed

### Task 1: Add forType field to all MODS entries, add 4 extras MODS + calculateParts(), and add cubierta pricing to CONFIG

- Verified TYPES array already contains `{ id: 'cocina', n: 'Cocina', i: '&#x1f373;' }` (confirmed from Phase 2)
- Added `forType` field to all existing MODS entries:
  - closet modules (modulo, colgador, zapatero, esquinero, entrepa): `forType: ['closet'], cat: 'Modulos'`
  - TV modules (torreLateral, panelTV, moduloCentral, lateralAbierto): `forType: ['tv'], cat: 'Modulos'`
  - cocina bajos: `forType: ['cocina'], cat: 'Bajos'`
  - cocina alacenas: `forType: ['cocina'], cat: 'Alacenas'`
  - cocina torres: `forType: ['cocina'], cat: 'Torres'`
  - puerta: `forType: ['puerta']`
  - legacy modules (alacenaAlta, gabineteBase, etc.): no forType (shown for 'otro' only)
- Added 4 new MODS entries with `forType: ['cocina'], cat: 'Extras'`:
  - `zoclo` (custom:1, w: 30-100cm)
  - `vistaLateral` (w: 58, 60cm)
  - `panelRelleno` (custom:1, w: 5-50cm)
  - `cubierta` (custom:1, w: 60-300cm)
- Added `CONFIG.pricing.cubiertaPrices: { postformado: 2500, cuarzo: 8500, granito: 6500 }` (per m2, MXN)
- Added calculateParts() blocks:
  - `zoclo`: single piece, w=module width, h=150mm, z='f'
  - `vistaLateral`: single piece, w=depth, h=height, z='f'
  - `panelRelleno`: single piece, w=module width, h=height, z='f'
  - `cubierta`: single piece with `isCubierta: true, t: 0`, w=width, h=depth
- Added assignHardware() block for all 4 extras: no hardware (empty)
- Added cubierta m2 pricing in calc() area loop: pieces with `isCubierta:true` are excluded from sheet area and priced via `CONFIG.pricing.cubiertaPrices` per m2 with mermaFactor applied
- `cubiertaType` stamped from module onto piece in calc() stamp loop
- `cubiertaCost` accumulated separately and added to `extraCost` (then multiplied by extrasMultiplier)
- Extended door skip guard: `cajonera || hornoBase || torreHornos || zoclo || vistaLateral || panelRelleno || cubierta`
- Added cubierta type selector (postformado/cuarzo/granito) in step2() module UI
- Added zoclo auto-width helper note in step2() showing total bajo width
- Modified `addMod()` to auto-populate zoclo.width from sum of bajo module widths when type === 'zoclo'

**Commit:** bbeb6d7

### Task 2: Filter step2() module buttons by project type and group by category

- Replaced `for (var k in MODS)` loop with filtered + grouped rendering
- Filter logic: modules with `forType` matching `P.type` are shown; modules without `forType` shown only for 'otro' or unset type
- Group logic: filtered modules grouped by `cat` field using `groups{}` + `groupOrder[]`
- Category header rendered as gray uppercase label with top border separator (only when >1 group)
- Extras module types excluded from quoteMode/door toggle UI: `if (m.type !== 'puerta' && m.type !== 'zoclo' && m.type !== 'vistaLateral' && m.type !== 'panelRelleno' && m.type !== 'cubierta')`
- 4 extras types added to door block skip guard in calc()

**Note:** Task 2 changes were committed together with Task 1 in commit bbeb6d7 (same file, same edit session).

## Verification Results

All 25 automated checks passed:
- forType field on all MODS categories (closet, tv, cocina bajos/alacenas/torres, puerta)
- 4 extras MODS entries with correct fields
- cubiertaPrices in CONFIG.pricing
- calculateParts() blocks for all 4 extras
- isCubierta flag on cubierta piece
- assignHardware() extras block
- Door skip guard includes all 4 extras
- cubierta m2 area skip + cost calculation
- cubiertaCost in extraCost and return object
- step2() filter by forType and group by cat
- cubierta type selector in step2()
- zoclo auto-width in addMod()
- extras quoteMode guard in step2()
- TYPES includes cocina

## Deviations from Plan

### Auto-fixed Issues

**1. [Rule 2 - Missing Logic] cubiertaType not stamped from module onto piece**

- **Found during:** Task 1 cubierta m2 pricing implementation
- **Issue:** The calculateParts() function produces a piece with `isCubierta: true` but has no access to `m.cubiertaType`. The calc() area loop needs `piece.cubiertaType` to select the correct price per m2.
- **Fix:** Added `if (p.isCubierta) p.cubiertaType = m.cubiertaType || 'postformado';` in the piece stamp loop in calc() (alongside mid, mn, mw stamping)
- **Files modified:** cotizador-template/cotizador-template.html
- **Commit:** bbeb6d7

**2. [Rule 3 - Blocking] Category header condition**

- **Found during:** Task 2 implementation
- **Issue:** Plan spec renders category header for any non-empty group name, but for closet (single 'Modulos' group) this would show a lone "MODULOS" header which looks odd.
- **Fix:** Added `groupOrder.length > 1` condition so headers only appear when there are multiple groups (cocina with Bajos/Alacenas/Torres/Extras gets headers; closet with single 'Modulos' group does not)
- **Files modified:** cotizador-template/cotizador-template.html
- **Commit:** bbeb6d7

## Self-Check

**Files exist:**
- [x] `cotizador-template/cotizador-template.html` — modified (confirmed)

**Commits exist:**
- [x] bbeb6d7 — feat(04-01): add 4 extras MODS, forType field to all MODS, calculateParts + assignHardware for extras, cubierta m2 pricing, zoclo auto-width
- [x] 6ca44e9 — feat(04-01): update submodule reference (parent repo)

## Self-Check: PASSED
