---
phase: 04-cocina-extras-ui-output
plan: "03"
subsystem: cotizador-html
tags: [cocina, pdf, genClientPDF, genInternalPDF, autoTable, despiece, herrajes]
dependency_graph:
  requires: [04-01-SUMMARY.md, 04-02-SUMMARY.md]
  provides: [cocina client PDF sections, cocina internal PDF sections]
  affects: [genClientPDF(), genInternalPDF()]
tech_stack:
  added: []
  patterns: [autoTable grid theme, P.type cocina gate, ES5 function expressions]
key_files:
  created: []
  modified:
    - cotizador-template/cotizador-template.html
decisions:
  - "Pieces use p.q field (not p.qty) — matches calc() output format throughout codebase"
  - "genInternalPDF() cocina sections use intP=S.project alias to avoid var shadowing risk"
  - "c.cascoCost/frenteCost/herrajesCost/extrasCost used with || 0 guards (defensive, fields exist from 04-02)"
  - "perModule cost fallback: shows 0 if c.perModule is absent — simplest safe fallback per plan spec"
metrics:
  duration: ~5 min
  completed: 2026-02-20
  tasks_completed: 2
  files_modified: 1
---

# Phase 4 Plan 03: Cocina PDF Sections Summary

Cocina-specific PDF sections added to both genClientPDF() and genInternalPDF(): module index table, full despiece, hardware summary, extras list (conditional), and technical assembly note for client PDF; module index with costs, hardware detail with bucket classification (Carpinteria/Premium), and category cost summary for internal PDF.

## Objective

Extend genClientPDF() and genInternalPDF() with cocina-specific sections so that cocina quotations produce professional PDF output: a module table, despiece per module, hardware summary, extras list, and technical notes. Non-cocina PDFs are completely unaffected.

## Tasks Completed

### Task 1: Add cocina module table and despiece sections to genClientPDF()

All cocina-specific sections inserted in genClientPDF() after the HERRAJES section and before addPlanoPage(), gated by `if (P.type === 'cocina')`:

1. **Module Index Table** — new page, lists all modules with #, name, width, height, depth, quantity
2. **Despiece Completo** — all pieces from c.pieces with module name, piece name, quantity, w/h/esp, material type (Frente/Interior based on p.z === 'f')
3. **Resumen de Herrajes** — allHW items with description, marca, quantity, unit price (fmtD), total
4. **Extras y Acabados** — conditional (only when zoclo/vistaLateral/panelRelleno/cubierta modules exist), lists extra module names, widths, quantities
5. **Nota Tecnica** — fixed text block covering melamina specs, Blum hardware warranty, installation steps, cubierta note, and ventilacion requirement

Key fix applied: used `p.q` (not `p.qty`) for piece quantity field, matching the actual calc() piece array format.

**Commit:** eb949aa (included in 04-02 agent scope expansion)

### Task 2: Add cocina-specific sections to genInternalPDF()

All cocina-specific sections inserted in genInternalPDF() after the main cost breakdown table and before the Summary box, gated by `if (intP.type === 'cocina')`:

1. **Detalle de Modulos (Interno)** — module index with cost column (from c.perModule[m.id].total if available, else 0) and margin column (cost * CONFIG.pricing.margin)
2. **Herrajes Detalle (Interno)** — full hardware table with unit price, total, and bucket classification (Carpinteria for unitPrice <= hwPremiumThreshold, Premium for above). Includes hardware total row.
3. **Resumen por Categoria (Interno)** — compact 2-column table: Casco, Frente, Herrajes, Extras costs using c.cascoCost/frenteCost/herrajesCost/extrasCost with || 0 guards

Used `intP = S.project` alias to avoid var name conflicts within the function scope.

**Commit:** eb949aa (included in 04-02 agent scope expansion)

## Verification Results

Code verified against plan requirements:
- [x] genClientPDF() has `if (P.type === 'cocina')` gate at correct insertion point (after HERRAJES, before addPlanoPage)
- [x] Module index table: 6 columns (#, Modulo, Ancho, Alto, Prof., Cant.)
- [x] Despiece table: 7 columns using p.q (not p.qty), p.z === 'f' check for material type
- [x] Hardware summary renders when hwRows.length > 0
- [x] Extras conditional: checks for zoclo/vistaLateral/panelRelleno/cubierta module types
- [x] Technical note: splitTextToSize for proper wrapping
- [x] genInternalPDF() has `if (intP.type === 'cocina')` gate
- [x] Internal module table: 7 columns with Costo and Margen columns
- [x] Hardware detail: Carpinteria/Premium bucket classification with hwPremiumThreshold
- [x] Category summary: 4 rows using cascoCost/frenteCost/herrajesCost/extrasCost || 0
- [x] Non-cocina PDFs: both blocks gated, closet/tv/other projects unaffected

## Deviations from Plan

### Scope Pre-completion

The 04-02 agent (previous execution) went beyond its assigned scope and included the 04-03 PDF sections in commit eb949aa. The code was verified to match the 04-03 plan exactly, with one intentional fix:

**[Rule 1 - Bug] Used p.q instead of p.qty for piece quantity**
- **Found during:** Task 1 implementation review
- **Issue:** Plan instructions reference `p.qty` but the actual calc() piece array uses `p.q` throughout the codebase (confirmed at lines 5464, 5471, 5477, 9118, etc.)
- **Fix:** Used `p.q + ''` in despRows array instead of `p.qty + ''`
- **Files modified:** cotizador-template/cotizador-template.html
- **Commit:** eb949aa

## Self-Check

**Files exist:**
- [x] `cotizador-template/cotizador-template.html` — modified (confirmed)
- [x] `COCINA-SPECIFIC PAGES (client PDF)` section present at line 8834
- [x] `COCINA-SPECIFIC PAGES (internal PDF)` section present at line 9196

**Commits exist:**
- [x] eb949aa — feat(04-02): add categorized cost subtotals to calc() and step3() [contains 04-03 PDF sections]

## Self-Check: PASSED
