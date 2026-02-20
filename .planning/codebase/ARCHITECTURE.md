# Architecture

**Analysis Date:** 2026-02-19

## Pattern Overview

**Overall:** Single-file IIFE (Immediately Invoked Function Expression) with imperative DOM rendering

**Key Characteristics:**
- Entire application lives in one HTML file (`cotizador-template/cotizador-template.html`, ~12,584 lines)
- All JavaScript is wrapped in a single `(function() { ... })()` starting at line 2947
- No framework — pure vanilla JS with string-concatenation-based HTML rendering
- Global mutable state object `S` drives all UI via a `render()` function
- Two parallel modes: "Custom" (5-step wizard) and "Estandarizado" (module builder)
- External supplier catalog loaded from separate JS file via `<script>` tag

## Layers

**Configuration Layer:**
- Purpose: Define all business rules, pricing, branding, and product constants
- Location: `cotizador-template/cotizador-template.html` lines 2951-4056
- Contains: `CONFIG`, `COLORS`, `FINSA_PRICES`, `TYPES`, `MODS`, `HW_DEFAULTS`, `EST_ZONE_TYPES`, `LED_CATALOG`, `TEMPLATES`
- Depends on: Nothing
- Used by: All other layers (calc, render, PDF generation)

**State Layer:**
- Purpose: Central mutable state object that drives the entire UI
- Location: `cotizador-template/cotizador-template.html` lines 4176-4246
- Contains: Single `var S = { ... }` object with all application state
- Depends on: CONFIG for defaults
- Used by: Every render function, every APP method

**Data/Catalog Layer:**
- Purpose: External supplier product catalog (hardware, materials)
- Location: `cotizador-template/supplier-catalog-data.js` (demo), `cotizador-template/supplier-catalog-data-REAL.js` (production)
- Contains: `var CATALOG = [...]` array of product objects with `{ codigo, descripcion, unidad, precio, marca, categoria, tipo }`
- Depends on: Nothing (global var)
- Used by: Catalog helper functions (`getCatalog()`, `findCatItem()`, `filterHW()`)

**Persistence Layer:**
- Purpose: Save/load projects via localStorage and optional Supabase cloud
- Location: `cotizador-template/cotizador-template.html` lines 4157-4380
- Contains: `loadSavedProjects()`, `saveSavedProjects()`, `cloudFetchProjects()`, `cloudSaveProject()`, `cloudDeleteProject()`
- Depends on: `CONFIG.storage.prefix`, `CONFIG.supabase`, Supabase JS SDK
- Used by: APP methods (save, load, delete)

**Calculation Engine:**
- Purpose: Cost-based pricing — generate cut pieces, compute material/hardware costs, apply margins
- Location: `cotizador-template/cotizador-template.html`
  - Custom mode: `calc()` at line 4610
  - Estandarizado mode: `calcEstandarizado()` at line 5677
  - LED BOM: `calcLedBOM()` at line 4475, `calcEstLedBOM()` at line 5550
  - Paint: `calcPaintM2()` at line 5496, `calcPaintLiters()` at line 5492
  - Chapacanto: `calcEstChapML()` at line 5471
- Contains: Piece generation from modules, area/canto calculations, material cost, hardware cost (threshold-based), margin application
- Depends on: State (`S.project` or `S.estProject`), CONFIG pricing constants, HW_DEFAULTS, CATALOG
- Used by: Render functions (step3-step5), PDF generators

**Rendering Layer:**
- Purpose: Generate HTML strings and inject into DOM via `innerHTML`
- Location: `cotizador-template/cotizador-template.html` lines 6602-10398
- Contains: `render()` (main entry), `renderTemplates()`, `step1()`-`step5()`, `renderEstandarizado()`, `renderEstBuilder()`, `renderEstEditor()`, `renderEstSummary()`
- Depends on: State `S`, calculation results from `calc()`/`calcEstandarizado()`
- Used by: Called after every state mutation via `render()`

**PDF Generation Layer:**
- Purpose: Generate downloadable PDF documents (client-facing and internal)
- Location: `cotizador-template/cotizador-template.html`
  - Client PDF: `genClientPDF()` at line 8016
  - Internal PDF: `genInternalPDF()` at line 8412
  - Estandarizado Internal: `genEstInternalPDF()` at line 8639
  - Despiece (cut list): `genDespiece()` at line 8897
- Contains: jsPDF document construction with dark theme, autoTable plugin for tables
- Depends on: jsPDF (`jspdf.umd.min.js`), autoTable plugin (`jspdf.plugin.autotable.min.js`), calc results, state, CONFIG

**Event/Controller Layer (APP):**
- Purpose: Handle all user interactions — exposed as `window.APP` methods
- Location: `cotizador-template/cotizador-template.html` lines 10400-12536
- Contains: `window.APP = { ... }` object with ~100+ methods for navigation, state mutation, PDF triggers
- Depends on: State `S`, render(), calc functions, PDF generators
- Used by: `onclick` handlers in rendered HTML strings

## Data Flow

**Custom Mode Quotation Flow:**

1. User lands on `renderTemplates()` screen (`S.screen = 'templates'`)
2. User clicks "Nuevo proyecto" or selects template → `APP.newProject()` / `APP.selectTpl(id)`
3. **Step 1** (`step1()`): Select project type (`TYPES`), client name, material (FINSA/ARAUCO/Custom), thickness (16mm/25mm)
4. **Step 2** (`step2()`): Add modules from `MODS` catalog — each module has type, width, height, drawer count, doors, LED, paint
5. **Step 3** (`step3(c)`): View cost breakdown — `calc()` generates pieces list, computes material area, hardware BOM, applies margins
6. **Step 4** (`step4(c)`): Hardware picker — browse CATALOG, add manual hardware items, review auto-assigned hardware
7. **Step 5** (`step5(c)`): Final summary — view totals, generate PDF (client/internal/despiece)

**Estandarizado Mode Flow:**

1. User clicks "Closet Estandarizado" → `APP.newEstandarizado()`
2. **Builder** (`renderEstBuilder()`): Add modules with visual silhouette, select project type, set depth
3. **Editor** (`renderEstEditor()`): Configure each module — zones (cajones, colgador, zapatero, entrepaños), doors, chapacanto edges, LED, paint, maletero, zoclo
4. **Material** (`renderEstMaterial()`): Select front/interior materials, thickness, chapacanto pricing
5. **Summary** (`renderEstSummary()`): View calculated costs via `calcEstandarizado()`, generate PDFs

**State Management:**

- Single mutable `var S` object at module scope
- Every user action mutates `S` directly, then calls `render()`
- `render()` reads `S.screen` and `S.step` to decide which HTML to generate
- `render()` injects full HTML into `$('app').innerHTML` (full re-render on every change)
- Scroll position preserved via `scrollY` variable captured before re-render

**Calculation Pipeline (both modes):**

1. Iterate `modules[]` → generate `pieces[]` array (each piece has width, height, quantity, thickness, zone flag `f`/`i`)
2. Iterate modules → generate `autoHW[]` array (hardware auto-assignments based on module type)
3. Sum piece areas → compute sheet count (with 20% merma factor)
4. Sum canto (edge banding) linear meters
5. Material cost = sheets x price/sheet + canto x price/meter + cutting + enchapado
6. Carpentry cost = material cost + labor + freight
7. Hardware: threshold-based split ($500) — low items → carpentry bucket (3x), high items → premium (30% margin)
8. Extras = LED BOM + glass + paint
9. Apply margins → subtotal → IVA (16%) → total

## Key Abstractions

**Module:**
- Purpose: A single furniture unit (cabinet, closet section, shelf unit, TV panel, etc.)
- Examples: `S.project.modules[]` (custom), `S.estProject.modules[]` (estandarizado)
- Pattern: Plain object with `{ id, type, width, height, zones[], doors, doorType, doorCount, led, paint, chap, zoclo, maletero, quoteMode }`

**Zone (Estandarizado only):**
- Purpose: A vertical section within a module (drawers, hanging bar, shelves, shoe rack)
- Examples: `module.zones[]` array
- Pattern: `{ id, type, count, cajonHeight, tubos, repisas, repisaHeight, doors, doorType, doorMaterial }`

**Piece:**
- Purpose: A single cut piece of material (generated by calc)
- Examples: Output of `calc()` and `calcEstandarizado()` in `pieces[]`
- Pattern: `{ mid, mn, mw, n, q, w, h, t, c, z }` where `w`/`h` are in mm, `z` is zone ('f'=front, 'i'=interior), `c` is canto edge count

**Hardware Item:**
- Purpose: A hardware component (slide, hinge, handle, etc.)
- Examples: `HW_DEFAULTS` keys, `CATALOG[]` items, `allHW[]` in calc result
- Pattern: Auto-assigned from `HW_DEFAULTS` with catalog price lookup via `findCatItem()`, or manually added from CATALOG

**Material:**
- Purpose: Board material selection (melamina brand/color/price)
- Examples: `S.project.matFrentes`, `S.project.matInterior`
- Pattern: `{ brand, code, name, finish, precio, thickness, catItem?, customPrice? }`

## Entry Points

**HTML Entry:**
- Location: `cotizador-template/cotizador-template.html` line 2944
- Triggers: Browser loads file
- Responsibilities: `<div id="app"></div>` is the single mount point

**JavaScript Entry:**
- Location: `cotizador-template/cotizador-template.html` line 2947
- Triggers: IIFE auto-executes on script parse
- Responsibilities: Define all constants, state, functions; call `render()` at line 12538; set up PWA manifest and service worker

**User Interaction Entry:**
- Location: `window.APP` object at line 10401
- Triggers: `onclick` attributes in rendered HTML (e.g., `onclick="APP.next()"`)
- Responsibilities: Mutate state `S`, call `render()` to update UI

## Error Handling

**Strategy:** Minimal — mostly `try/catch` around localStorage and Supabase operations

**Patterns:**
- localStorage reads wrapped in `try/catch` with silent fallback (lines 4159-4173)
- Supabase errors logged to `console.error` but do not surface to user (lines 4326-4328)
- `findCatItem()` returns `null` if no match, callers fall back to `def.fallback` price
- No user-facing error messages or validation feedback
- PWA registration wrapped in `try/catch` with empty catch

## Cross-Cutting Concerns

**Logging:** `console.warn` and `console.error` for Supabase issues only. No structured logging.

**Validation:** Minimal — step navigation gated by `S.project.type !== ''` and `S.project.modules.length > 0`. No input validation on numeric fields.

**Authentication:** None. Supabase client uses anon key if configured. No user auth.

**Internationalization:** Hardcoded Spanish (es-MX locale for number formatting, all UI strings in Spanish).

**Formatting:** `fmt(n)` for currency display (`$1,234`), `fmtD(n)` for decimals, `date()` for localized date strings.

---

*Architecture analysis: 2026-02-19*
