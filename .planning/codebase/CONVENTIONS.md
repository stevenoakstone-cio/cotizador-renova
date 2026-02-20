# Coding Conventions

**Analysis Date:** 2026-02-19

## Language & Runtime

**Language:** ES5 JavaScript (no `let`, `const`, arrow functions, template literals, or ES6+ features)
- The entire codebase uses `var` exclusively (~1005 occurrences, zero `let`/`const`)
- Use `function(){}` syntax, never arrow functions
- Use string concatenation with `+`, never template literals
- Use `.forEach(function(x){})`, never `.map`/`.filter` with arrows

**Why ES5:** The app is a single-file HTML application designed for maximum browser compatibility. All code lives inside one IIFE.

## Naming Patterns

**Files:**
- `cotizador-template.html` — kebab-case for the main HTML file
- `supplier-catalog-data.js` — kebab-case for external JS data files
- `logo_renova.webp` — snake_case for assets

**Functions:**
- camelCase throughout: `calcEstandarizado()`, `renderEstBuilder()`, `genClientPDF()`
- Prefixes indicate purpose:
  - `calc*` — calculation/pricing logic: `calc()`, `calcEstandarizado()`, `calcLedBOM()`, `calcEstFixedHeight()`
  - `render*` — HTML string generation: `renderTemplates()`, `renderEstBuilder()`, `renderModuleLed()`
  - `gen*` — PDF generation: `genClientPDF()`, `genInternalPDF()`, `genEstInternalPDF()`
  - `step*` — wizard step renderers: `step1()`, `step2()`, `step3()`, `step4()`, `step5()`
  - `get*` — data accessors: `getMatPrice()`, `getCatalog()`, `getHWCats()`
  - `est*` — estandarizado-mode specific: `estModHeight()`, `estEspacioUtil()`
  - `default*` — factory functions: `defaultMat()`, `defaultEstModule()`, `defaultEstProject()`
  - `cloud*` — Supabase operations: `cloudFetchProjects()`, `cloudSaveProject()`, `cloudDeleteProject()`
  - `tog*` — toggle booleans: `togDoor()`, `togTipon()`, `togModProp()`
  - `set*` — state setters: `setClient()`, `setType()`, `setThick()`
- Short utility names: `fmt()`, `fmtD()`, `uid()`, `$()`, `svg()`

**Variables:**
- Single-letter or very short names for local scope: `h` (HTML string), `m` (module), `c` (calculation result), `P` (project shorthand), `S` (global state), `w`/`h`/`d` (width/height/depth)
- `EP` = estandarizado project reference (shorthand for `S.estProject`)
- `TK` = thickness value
- UPPER_SNAKE_CASE for constants: `CONFIG`, `COLORS`, `FINSA_PRICES`, `TYPES`, `MODS`, `HW_DEFAULTS`, `EST_HEIGHT`, `EST_ZONE_TYPES`, `LED_CATALOG`, `TEMPLATES`, `BRANDS`, `CATALOG`

**Data Object Properties:**
- Single-letter abbreviations in data structures for compactness:
  - `{ c: 'D01', n: 'Roble Natural', f: 'Demo' }` — code, name, finish
  - `{ id, n, w, d }` — name, widths, description (in MODS)
  - `{ mid, mn, mw, n, q, w, h, t, c, z }` — piece properties (module id, module name, module width, name, quantity, width, height, thickness, chap edges, zone)
- Full names for user-facing or configuration properties: `enabled`, `height`, `width`, `doors`, `doorMode`, `doorType`

**CSS Classes:**
- Abbreviated kebab-case: `hdr`, `hdr-txt`, `hdr-btn`, `hdr-logo`
- Descriptive kebab-case: `step-dot`, `nav-btn`, `type-btn`, `mat-select`
- State classes: `.active`, `.done`, `.future`, `.sel`, `.on`
- Prefixed for sections: `est-*` (estandarizado), `tpl-*` (template), `sp-*` (saved project), `led-*` (LED)

## Bilingual Naming

The codebase uses a **Spanish/English mix** consistently:
- **Spanish** for domain terms: `cajones`, `bisagra`, `colgador`, `zapatero`, `entrepa`, `maletero`, `zoclo`, `chapacanto`, `enchapado`, `puerta`, `jaladera`, `melamina`, `cocina`, `closet`, `mueble`
- **English** for programming constructs: `render`, `calc`, `filter`, `toggle`, `push`, `splice`, `forEach`, `enabled`, `doors`, `width`, `height`
- Mixed in function names: `calcEstandarizado()`, `renderEstBuilder()`, `genClientPDF()`
- CONFIG keys in English: `pricing`, `hardware`, `quoting`, `brand`, `storage`
- UI text (user-facing strings) in **Spanish**: `'Nuevo Proyecto'`, `'Informacion basica'`, `'Nombre del Cliente'`

**Rule:** Use Spanish for furniture/carpentry domain terms and English for code structure. When adding new features, follow this pattern.

## Code Style

**Formatting:**
- No formatter tool detected (no Prettier, ESLint, or Biome config)
- 4-space indentation in the `<style>` block
- 8-space indentation inside the IIFE `<script>` block
- Semicolons used consistently
- Opening braces on same line as statement
- Single quotes for strings

**Variable Declaration:**
- Always use `var` (ES5 constraint)
- Multiple variables on one line with commas: `var P = S.project, pieces = [], autoHW = [];`
- Guard clauses with early return: `if (!m) return;`

**HTML Generation:**
- Build HTML strings by concatenation into a `var h = '';` variable
- Inline event handlers via string: `onclick="APP.methodName(args)"`
- Return the completed HTML string from render functions
- Use Unicode escapes for special characters: `'\u00f1'` for n-tilde, `'\u00e9'` for accented e
- HTML entities for emoji/icons: `'&#x1f454;'`, `'&#x1f6aa;'`

**Example — standard render function pattern:**
```javascript
function renderSomething() {
    var P = S.project;
    var h = '<div class="title"><h2>Title</h2></div>';
    h += '<div class="field"><label>Label</label>';
    h += '<input type="text" value="' + P.someValue + '" onchange="APP.setSomeValue(this.value)">';
    h += '</div>';
    return h;
}
```

## State Management

**Pattern:** Single mutable state object `S` + `render()` on every change.

- Global state lives in `var S = { ... }` (~line 4176)
- All mutations happen in `window.APP` methods
- Every APP method mutates `S` directly, then calls `render()`
- `render()` rebuilds the entire DOM via `$('app').innerHTML = h`
- Scroll position preserved manually via `scrollY` variable

**Example — standard APP method pattern:**
```javascript
window.APP = {
    setClient: function(v) {
        S.project.client = v;
    },
    togDoor: function(id) {
        var m = S.project.modules.find(function(x) { return x.id === id; });
        if (m) {
            m.doors = !m.doors;
            S.editedPieces = {};
            render();
        }
    }
};
```

**Key rule:** Always call `render()` after state mutations (except for text inputs where `oninput` just sets the value without re-render to avoid cursor issues).

## Import Organization

**Script loading order in HTML:**
1. `jspdf.umd.min.js` — PDF generation library (local)
2. `jspdf.plugin.autotable.min.js` — PDF table plugin (local)
3. `https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/dist/umd/supabase.min.js` — Supabase CDN
4. `supplier-catalog-data.js` — hardware catalog data (local, loaded in `<body>`)
5. Inline `<script>` — main app IIFE

**No module system.** All scripts use global variables (`CATALOG`, `jspdf`, `supabase`).

## Error Handling

**Pattern:** Silent `try/catch` with empty catch blocks for non-critical operations.

```javascript
try {
    savedLogo = localStorage.getItem(CONFIG.storage.prefix + 'logo_base64');
} catch (e) {}
```

- **localStorage operations:** Always wrapped in `try/catch` with empty catch — graceful degradation if storage unavailable
- **Supabase operations:** Use `.then()` with error checking via `if (res.error) console.error(...)`
- **Image loading:** `try/catch` around `doc.addImage()` in PDF generation
- **Null guards:** Check `if (!m) return;` before accessing module properties
- **Fallback values:** Use `||` for defaults: `m.height || P.defaultHeight || 200`
- **No thrown exceptions** — errors are silently absorbed or logged to console

**Rule:** Wrap localStorage and external API calls in `try/catch`. Use `|| defaultValue` for optional properties. Never throw exceptions.

## Comments

**Section Headers — Box-drawing characters:**
```javascript
// ╔══════════════════════════════════════════════════════════════╗
// ║  CONFIG — Edit this section to customize for your business  ║
// ╚══════════════════════════════════════════════════════════════╝
```
```javascript
// ┌─────────────────────────────────────────────────────────────┐
// │  DEMO MATERIALS — Replace with real COLORS array when ready  │
// └─────────────────────────────────────────────────────────────┘
```

**Section dividers:**
```javascript
// === CALC: Cost-based pricing engine ===
// === V2.3: Estandarizado calc ===
// === RENDER FUNCTIONS ===
// === APP METHODS ===
```

**CUSTOMIZE markers** — indicate user-editable sections:
```javascript
// CUSTOMIZE: Legal terms for client PDF — edit these to match your business terms
// CUSTOMIZE: Hardware pricing — items with total cost <= threshold go into carpentry bucket
// CUSTOMIZE: Bisagra auto-assignment rules — adjust count by door height and material
```

**Version markers:**
```javascript
// V2.3: Closet Estandarizado constants
// V2.3: Estandarizado state
```

**Inline comments:** Brief, explain "why" not "what":
```javascript
var count = 3; // default
// Legacy brand-based accumulators (kept for backward compat in return object)
// colgador is flexible — not counted here
```

**No JSDoc/TSDoc.** Functions have no documentation blocks. Parameter types are inferred from usage.

**Rule:** Use `// === SECTION NAME ===` for major sections. Use `// CUSTOMIZE:` prefix for user-editable sections. Keep inline comments brief and purposeful.

## Function Design

**Size:** Functions range from 3 lines (utilities) to 500+ lines (calc functions). Render functions for complex UI sections commonly exceed 100 lines.

**Parameters:** Positional, no destructuring (ES5). Functions typically receive 0-3 parameters.
- Calculation functions: `calc()` (no params, reads from `S`), `calcEstandarizado()` (same)
- Render functions: `renderModuleLed(m)` receives module, `step3(c)` receives calc result
- APP methods: receive primitive values from inline handlers: `APP.setModW(id, w)`

**Return Values:**
- Render functions return HTML strings
- Calc functions return plain objects with computed values
- APP methods return nothing (mutate `S` + call `render()`)
- Utility functions return formatted values

## Module/Code Organization (within the IIFE)

**Order inside the IIFE (~line 2947-12581):**
1. **CONFIG** (~2951): Business configuration object
2. **Data constants** (~3072-3878): COLORS, FINSA_PRICES, TYPES, MODS, HW_DEFAULTS, EST_* constants, LED_CATALOG, TEMPLATES
3. **Catalog helpers** (~4058): getCatalog(), findCatItem(), adjPrice(), filterHW()
4. **Derived constants** (~4145): BRANDS, CONTACT
5. **Persistence** (~4157): localStorage loading
6. **State** (~4176): `var S = { ... }`
7. **Utilities** (~4249): $(), fmt(), fmtD(), uid(), date()
8. **Cloud storage** (~4275): Supabase CRUD functions
9. **SVG icons** (~4388): svg() function with inline SVG paths
10. **Calculation engines** (~4436-5900): calc(), calcEstandarizado(), helper calcs
11. **Render functions** (~6601-10349): UI builders, step functions, PDF generators
12. **APP methods** (~10400-12536): window.APP object with all event handlers
13. **Initialization** (~12538): render() call, PWA setup

**Rule:** New features follow this order. Add new constants near existing ones, new calc functions in the calc section, new render functions in the render section, new APP methods inside the APP object.

## Data Flow Pattern

1. User interaction triggers `onclick="APP.methodName(args)"`
2. APP method mutates `S` (state object)
3. APP method calls `render()`
4. `render()` calls calc functions (which read from `S`)
5. `render()` calls step/render functions, passing calc results
6. Render functions build HTML strings referencing `APP.methodName` in event handlers
7. `$('app').innerHTML = h` replaces entire DOM

## Formatting Functions

- `fmt(n)` — currency without decimals: `$1,234`
- `fmtD(n)` — currency with 2 decimals: `$1,234.56`
- Both use `'es-MX'` locale

**Rule:** Use `fmt()` for display prices (rounded), `fmtD()` for cost breakdowns (precise).

---

*Convention analysis: 2026-02-19*
