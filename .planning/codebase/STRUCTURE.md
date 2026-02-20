# Codebase Structure

**Analysis Date:** 2026-02-19

## Directory Layout

```
project-renova/
├── .claude/                          # Claude AI settings
│   ├── settings.json
│   └── settings.local.json
├── .planning/                        # GSD planning documents
│   └── codebase/                     # Codebase analysis docs
├── cotizador-template/               # Main application directory
│   ├── cotizador-template.html       # THE app (12,584 lines — HTML + CSS + JS)
│   ├── supplier-catalog-data.js      # Demo supplier catalog (30 lines)
│   ├── supplier-catalog-data-REAL.js # Real supplier catalog (production data)
│   ├── jspdf.umd.min.js             # jsPDF library (PDF generation)
│   ├── jspdf.plugin.autotable.min.js # jsPDF autoTable plugin
│   ├── logo_renova.webp              # Company logo image
│   ├── README.md                     # Template documentation
│   └── renova-instructions.md        # Business-specific instructions
├── HOW_TO/                           # Reference PDFs for furniture design
│   ├── 17_16486_pdf_sch_foll-web_muebleria_como_disenar_cocina_colombia_11may_16-pdf_376_so1.pdf
│   ├── credenza-1.pdf
│   ├── librero-1.pdf
│   └── mesa-1.pdf
├── CONTEXT.md                        # Project context documentation
├── INSTRUCTIVO_TECNICO.md            # Technical instructional doc (48KB)
└── TASKS.md                          # Task tracking
```

## Directory Purposes

**`cotizador-template/`:**
- Purpose: Contains the entire application and its dependencies
- Contains: Single HTML app file, vendor JS libraries, external data files, assets
- Key files: `cotizador-template.html` (the application), `supplier-catalog-data.js` (product catalog)

**`HOW_TO/`:**
- Purpose: Reference materials for furniture design (PDFs)
- Contains: Design guides for kitchens, credenzas, bookshelves, tables
- Generated: No
- Committed: Yes

**`.planning/codebase/`:**
- Purpose: Codebase analysis documents for GSD workflow
- Contains: Architecture, structure, conventions, concerns docs
- Generated: Yes (by codebase mapping)
- Committed: Yes

## Key File Locations

**Entry Point:**
- `cotizador-template/cotizador-template.html`: The entire application — open this file in a browser to run

**Configuration:**
- `cotizador-template/cotizador-template.html` lines 2951-3050: `CONFIG` object (brand, contact, quoting, pricing, hardware rules, supabase)
- `cotizador-template/cotizador-template.html` lines 3072-3117: `COLORS` and `FINSA_PRICES` (material catalog)
- `cotizador-template/cotizador-template.html` lines 3119-3233: `TYPES` and `MODS` (project/module type definitions)
- `cotizador-template/cotizador-template.html` lines 3240-3295: `HW_DEFAULTS` (hardware fallback prices)
- `cotizador-template/cotizador-template.html` lines 3297-3370: Estandarizado constants (`EST_HEIGHT`, `EST_ZONE_TYPES`, etc.)
- `cotizador-template/cotizador-template.html` lines 3788-3878: `LED_CATALOG` (Hafele LOOX5 LED system)
- `cotizador-template/cotizador-template.html` lines 3881-4056: `TEMPLATES` (quick-quote templates)

**State:**
- `cotizador-template/cotizador-template.html` lines 4176-4246: `var S = { ... }` (global application state)

**Core Logic (Calculation):**
- `cotizador-template/cotizador-template.html` line 4610: `calc()` — custom mode pricing engine
- `cotizador-template/cotizador-template.html` line 5677: `calcEstandarizado()` — estandarizado mode pricing engine
- `cotizador-template/cotizador-template.html` line 4475: `calcLedBOM()` — LED bill of materials
- `cotizador-template/cotizador-template.html` line 5550: `calcEstLedBOM()` — estandarizado LED BOM

**Rendering (UI):**
- `cotizador-template/cotizador-template.html` line 10350: `render()` — main render dispatcher
- `cotizador-template/cotizador-template.html` line 6602: `renderTemplates()` — home/template selection screen
- `cotizador-template/cotizador-template.html` line 6873: `step1()` — project type & material selection
- `cotizador-template/cotizador-template.html` line 6938: `step2()` — module builder
- `cotizador-template/cotizador-template.html` line 7084: `step3(c)` — cost breakdown display
- `cotizador-template/cotizador-template.html` line 7238: `step4(c)` — hardware management
- `cotizador-template/cotizador-template.html` line 7492: `step5(c)` — final summary & PDF actions
- `cotizador-template/cotizador-template.html` line 9240: `renderEstBuilder()` — estandarizado module builder
- `cotizador-template/cotizador-template.html` line 9329: `renderEstEditor()` — estandarizado module editor
- `cotizador-template/cotizador-template.html` line 10090: `renderEstSummary()` — estandarizado cost summary

**PDF Generation:**
- `cotizador-template/cotizador-template.html` line 8016: `genClientPDF()` — dark-themed client PDF
- `cotizador-template/cotizador-template.html` line 8412: `genInternalPDF()` — detailed internal PDF with autoTable
- `cotizador-template/cotizador-template.html` line 8639: `genEstInternalPDF()` — estandarizado internal PDF
- `cotizador-template/cotizador-template.html` line 8897: `genDespiece()` — cut list (despiece) PDF

**Event Handlers:**
- `cotizador-template/cotizador-template.html` line 10401: `window.APP = { ... }` — all user interaction handlers

**Persistence:**
- `cotizador-template/cotizador-template.html` line 4286: `loadSavedProjects()` — localStorage + Supabase merge
- `cotizador-template/cotizador-template.html` line 4320: `cloudFetchProjects()` — Supabase fetch
- `cotizador-template/cotizador-template.html` line 4348: `cloudSaveProject()` — Supabase upsert

**External Data:**
- `cotizador-template/supplier-catalog-data.js`: `var CATALOG = [...]` — demo supplier catalog (correderas, bisagras, tip-on, jaladeras, niveladores)
- `cotizador-template/supplier-catalog-data-REAL.js`: Production catalog with real supplier prices

**Vendor Libraries:**
- `cotizador-template/jspdf.umd.min.js`: jsPDF library for PDF generation
- `cotizador-template/jspdf.plugin.autotable.min.js`: autoTable plugin for jsPDF tables

**CSS:**
- `cotizador-template/cotizador-template.html` lines 17-2941: All CSS in `<style>` block — dark theme (#0d0d0d bg, #c49a6c gold accent)

## HTML Structure Within Main File

```
cotizador-template.html
├── <head>
│   ├── Meta tags (viewport, PWA, theme-color)
│   ├── <script> vendor libs (jspdf, autotable, supabase CDN)
│   └── <style> (~2,920 lines of CSS)
├── <body>
│   ├── <div id="app"></div>  (single mount point)
│   ├── <script src="supplier-catalog-data.js">
│   └── <script> IIFE (~9,635 lines)
│       ├── CONFIG block (lines 2951-3050)
│       ├── Data constants (COLORS, TYPES, MODS, HW_DEFAULTS, etc.) (lines 3072-4056)
│       ├── Catalog helpers (lines 4059-4143)
│       ├── State initialization (lines 4157-4246)
│       ├── Utility functions (lines 4249-4421)
│       ├── Calculation engines (calc, calcEstandarizado) (lines 4424-6600)
│       ├── Render functions (lines 6602-10398)
│       ├── APP controller methods (lines 10400-12536)
│       ├── Initial render() call (line 12538)
│       └── PWA setup (lines 12540-12581)
```

## Naming Conventions

**Files:**
- `kebab-case.ext`: `cotizador-template.html`, `supplier-catalog-data.js`
- Vendor files keep original names: `jspdf.umd.min.js`

**JavaScript Functions:**
- `camelCase`: `calcEstandarizado()`, `renderEstBuilder()`, `genClientPDF()`
- Render functions prefixed with `render` or `step`: `renderTemplates()`, `step1()`, `step3(c)`
- Calc functions prefixed with `calc`: `calc()`, `calcEstLedBOM()`
- PDF functions prefixed with `gen`: `genClientPDF()`, `genInternalPDF()`
- Estandarizado functions contain `Est`: `calcEstFixedHeight()`, `renderEstEditor()`

**Variables:**
- Constants: `UPPER_CASE` or `PascalCase`: `CONFIG`, `COLORS`, `TYPES`, `MODS`, `HW_DEFAULTS`, `EST_HEIGHT`
- State: single `S` object
- Project references: `P` (custom project `S.project`) or `EP` (estandarizado `S.estProject`)
- Short aliases inside calc: `TK` (thickness), `W` (width in mm), `H` (height in mm), `D` (depth in mm)

**CSS Classes:**
- Short abbreviated names: `.hdr`, `.nav-btn`, `.tpl-card`, `.qq-body`
- BEM-like but informal: `.step-dot.active`, `.nav-btn.next`, `.type-btn.sel`
- Estandarizado prefixed with `est-`: `.est-bar`, `.est-builder`, `.est-save-btn`

## Where to Add New Code

**New Module Type:**
1. Add to `MODS` object: `cotizador-template/cotizador-template.html` ~line 3146
2. Add `defaultEstModule()` case: ~line 3373
3. Add piece generation in `calc()`: ~line 4610 (custom mode)
4. Add piece generation in `calcEstandarizado()`: ~line 5677 (estandarizado mode)
5. Add zone type to `EST_ZONE_TYPES` if applicable: ~line 3304

**New Configuration Option:**
- Add to `CONFIG` object: lines 2951-3050
- Reference via `CONFIG.yourSection.yourKey` throughout code

**New UI Screen or Step:**
- Add render function following pattern: `function renderNewScreen() { var h = '...'; return h; }`
- Wire into `render()` function at line 10350
- Add APP methods for user interaction at line 10401

**New PDF Output:**
- Add function following pattern of `genClientPDF()` (line 8016) or `genInternalPDF()` (line 8412)
- Use `jspdf` and `autoTable` — see existing functions for dark theme constants

**New Hardware Catalog Items:**
- Add to `cotizador-template/supplier-catalog-data.js` following existing `{ codigo, descripcion, unidad, precio, marca, categoria, tipo }` pattern
- Add to `HW_DEFAULTS` if it should be auto-assigned: ~line 3240

**New Supplier/Material Brand:**
- Add to `COLORS` array for FINSA-style boards: ~line 3072
- Add catalog lookup in `renderMaterialSelect()`: ~line 6694
- Add pricing logic in `getMatPrice()`: ~line 4424

**New APP Event Handler:**
- Add method to `window.APP` object: ~line 10401
- Reference in rendered HTML: `onclick="APP.yourMethod()"`

## Special Directories

**`cotizador-template/`:**
- Purpose: Self-contained application — can be deployed by copying this folder to any static host
- Generated: No
- Committed: Yes
- Note: The app works offline as a PWA (service worker generated inline at line 12574)

**`HOW_TO/`:**
- Purpose: Reference PDFs, not consumed by the application
- Generated: No
- Committed: Yes

## Demo vs Production Data

The codebase ships with demo data and comments indicating where to swap for real data:

- **Materials:** Demo `COLORS` array at line 3072, real data commented out at line 3088
- **Hardware:** Demo `HW_DEFAULTS` at line 3240, real data commented out at line 3280
- **Catalog:** Demo `supplier-catalog-data.js` (30 items), real `supplier-catalog-data-REAL.js` (swap filename to activate)

To switch to production: uncomment real data blocks and rename `supplier-catalog-data-REAL.js` to `supplier-catalog-data.js`.

---

*Structure analysis: 2026-02-19*
