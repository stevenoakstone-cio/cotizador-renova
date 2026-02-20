---
phase: 06-tablered-arauco
plan: "01"
subsystem: tablered-products
tags: [tablered, arauco, librero, credenzaTV, mesaCentro, calculateParts, assignHardware, despiece-fijo]
dependency_graph:
  requires: [05-01, 05-02]
  provides: [TABL-01, TABL-02, TABL-03, TABL-04]
  affects: [calc, calculateParts, assignHardware, TYPES, MODS, HW_DEFAULTS]
tech_stack:
  added: []
  patterns:
    - isCubrecanto flag on PVC edge-band pieces (parallel to isCubierta for m2 pricing)
    - Fixed despiece pattern: hardcoded piece arrays with label A-I matching INSTRUCTIVO catalog
    - isPuerta flag on door/lid pieces in calculateParts (prevents calc() door block duplication)
    - Tablered bisagras in assignHardware (not calc() door block) since puertas come from calculateParts
key_files:
  created: []
  modified:
    - cotizador-template/cotizador-template.html
decisions:
  - "Tablered bisagras go in assignHardware (not calc door block) because puertas are fixed pieces in calculateParts"
  - "cubrecantoCost added to extraCost bucket (parallel to cubiertaCost) and exposed in calc() return object"
  - "isCubrecanto pieces have w=mlTotal*1000, h=0 so they don't contribute to sheet area calculation"
  - "credenzaTV uses forType:['credenza'] mapping to the existing credenza TYPES entry"
metrics:
  duration: 12 min
  completed: 2026-02-20
  tasks_completed: 2
  tasks_total: 2
  files_modified: 1
---

# Phase 6 Plan 01: Tablered Arauco Fixed-Despiece Products Summary

Fixed-despiece implementation for 3 Tablered Arauco catalog products (Librero KIRA, Credenza TV NERO, Mesa de Centro bicolor) with TYPES entries, MODS entries, calculateParts() blocks, assignHardware() blocks, and full calc() integration including cubrecanto pricing per ml.

## What Was Built

### Task 1: MODS + TYPES + calculateParts()

**TYPES entries added:**
- `librero` — Librero (book: &#x1f4da;)
- `mesa_centro` — Mesa de Centro (coffee: &#x2615;)
- `credenza` already existed — no change needed

**MODS entries added (3 Tablered products):**
- `librero` — Librero KIRA, w:[138.5], forType:['librero'], cat:'Tablered'
- `credenzaTV` — Credenza TV NERO, w:[160], forType:['credenza'], cat:'Tablered'
- `mesaCentro` — Mesa de Centro, w:[100], forType:['mesa_centro'], cat:'Tablered'

**calculateParts() blocks (3 fixed despiece arrays):**

| Product | Pieces (A-I labels) | Total qty | Cubrecanto |
|---------|---------------------|-----------|------------|
| librero | A-I (9 types) | 18 pzs | 59.2 ml KIRA |
| credenzaTV | A-G (7 types) | 16 pzs | 49 ml NERO |
| mesaCentro | A-I (9 types, 2 bloques) | 11 pzs | 12 ml KIRA + 9 ml NERO |

All piece dimensions match INSTRUCTIVO Sections 8, 9, 10 exactly. Piece labels (A, B, C...) match official catalog nomenclature for traceability.

### Task 2: assignHardware() + calc() Integration

**assignHardware() blocks:**

| Product | Hardware |
|---------|----------|
| librero | bisagra x8 (4 puertas x 2), regaton x6 |
| credenzaTV | bisagra x8 (4 puertas x 2), pata_metalica x4 |
| mesaCentro | bisagra x2 (tapa abatible), piston_150n x1, pata_niveladora x4 |

**HW_DEFAULTS new keys (5 guarded conditionals):**
- `regaton` — $8 DEMO (nivelador tachuela)
- `pata_metalica` — $85 DEMO (pata metalica 10cm)
- `piston_150n` — $350 DEMO (piston gas 150N)
- `pata_niveladora` — $25 DEMO (pata niveladora 5cm)
- `cubrecanto_ml` — $15/ml DEMO (PVC 0.45mm por ml)

**calc() integration:**
- Door skip guard extended: `librero || credenzaTV || mesaCentro` added to skip list (puertas in calculateParts)
- `isCubrecanto` pieces handled in area loop: priced per ml, excluded from sheet area, added to extraCost
- `cubrecantoCost` accumulated and returned in calc() result object

## Piece Dimensions Cross-Reference

### Librero KIRA (INSTRUCTIVO S.9) — 1385x400x1800mm

| Label | Name | Qty | Largo | Ancho | Cubrecanto |
|-------|------|-----|-------|-------|------------|
| A | Piso/Techo zona izq | 2 | 1000 | 385 | 3 lados |
| B | Lateral exterior | 2 | 1770 | 385 | 2 lados |
| C | Fondo trasero | 1 | 1770 | 970 | no |
| D | Puerta media | 2 | 745 | 335 | 4 lados |
| E | Puerta baja | 2 | 385 | 335 | 4 lados |
| F | Repisa larga horizontal | 2 | 970 | 370 | 1 lado |
| G | Paral vertical interno | 4 | 699 | 370 | 1 lado |
| H | Repisa corta zona der | 2 | 310 | 370 | 1 lado |
| I | Repisa media zona der | 1 | 645 | 370 | 1 lado |

### Credenza TV NERO (INSTRUCTIVO S.8) — 1600x450x440mm

| Label | Name | Qty | Largo | Ancho | Cubrecanto |
|-------|------|-----|-------|-------|------------|
| A | Piso/Techo | 2 | 1600 | 450 | 3 lados |
| B | Fondo trasero | 1 | 1570 | 435 | no |
| C | Costado exterior | 2 | 410 | 450 | 2 lados |
| D | Paral interno vertical | 3 | 410 | 435 | 1 lado |
| E | Repisa horizontal interna | 4 | 380 | 435 | 1 lado |
| F | Puerta central | 2 | 400 | 390 | 4 lados |
| G | Puerta lateral | 2 | 320 | 405 | 4 lados |

### Mesa de Centro (INSTRUCTIVO S.10) — 1000x600x400mm — NERO+KIRA bicolor

**BLOQUE 1 — KIRA:**

| Label | Name | Qty | Largo | Ancho |
|-------|------|-----|-------|-------|
| A | Tapa abatible | 1 | 600 | 645 |
| B | Costado | 2 | 385 | 650 |
| C | Frente | 1 | 360 | 570 |
| D | Fondo interno | 1 | 385 | 570 |
| E | Piso interno | 1 | 570 | 635 |

**BLOQUE 2 — NERO:**

| Label | Name | Qty | Largo | Ancho |
|-------|------|-----|-------|-------|
| F | Piso | 1 | 570 | 350 |
| G | Techo/Cubierta | 1 | 600 | 350 |
| H | Costado | 2 | 385 | 350 |
| I | Fondo interno | 1 | 370 | 570 |

## Deviations from Plan

None — plan executed exactly as written.

## Self-Check

**Files exist:**
- [x] `cotizador-template/cotizador-template.html` — modified (verified)

**Commits exist:**
- [x] `eae7f37` — feat(06-01): add TYPES + MODS + calculateParts for 3 Tablered products
- [x] `9e5dcf1` — feat(06-01): assignHardware + calc() integration for Tablered products

**Functional verification (code search):**
- [x] `moduleType === 'librero'` found in calculateParts() and assignHardware() (lines 5281, 5492)
- [x] `moduleType === 'credenzaTV'` found in calculateParts() and assignHardware() (lines 5296, 5498)
- [x] `moduleType === 'mesaCentro'` found in calculateParts() and assignHardware() (lines 5309, 5503)
- [x] MODS entries: librero (3398), credenzaTV (3405), mesaCentro (3412)
- [x] TYPES entries: librero (3150), mesa_centro (3154), credenza (3130)
- [x] HW_DEFAULTS: regaton, pata_metalica, piston_150n, pata_niveladora, cubrecanto_ml (lines 3477-3481)
- [x] calc() door skip guard includes librero, credenzaTV, mesaCentro (line 5576)
- [x] isCubrecanto handling in calc() area loop (lines 5689-5694)
- [x] cubrecantoCost added to extraCost (line 5822)
- [x] cubrecantoCost in calc() return object (line 5892)

## Self-Check: PASSED
