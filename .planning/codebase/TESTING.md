# Testing Patterns

**Analysis Date:** 2026-02-19

## Test Framework

**Runner:** None
**Assertion Library:** None
**Config:** No test configuration files exist

There is **no automated testing infrastructure** in this project. No test files, no test framework, no test runner, no CI pipeline.

## Why No Tests Exist

The application is a single-file HTML app (`cotizador-template/cotizador-template.html`, ~12,584 lines) with all JavaScript embedded in an IIFE. There is:
- No `package.json` (no npm/node dependency management)
- No build system (no Webpack, Vite, Rollup, etc.)
- No test files (`*.test.*`, `*.spec.*`) anywhere in the project
- No test configuration (`jest.config.*`, `vitest.config.*`, etc.)
- No CI/CD configuration (`.github/workflows/*`, etc.)

## Current Quality Assurance Approach

**Manual testing only.** The app is tested by opening `cotizador-template.html` directly in a browser and exercising the UI manually.

**Implicit validation patterns in the code:**
- Guard clauses protect against null/undefined: `if (!m) return;`
- Fallback values prevent NaN: `m.height || P.defaultHeight || 200`
- Input constraints via HTML attributes: `min="0"`, `max="100"`, `step="5"`
- File upload validation in `estUploadPlano()`:
  ```javascript
  if (!file.type.match(/image\/(jpeg|png|jpg)/)) {
      alert('Solo se aceptan imagenes JPG o PNG');
      return;
  }
  if (file.size > 10 * 1024 * 1024) {
      alert('La imagen es muy grande (max 10MB)');
      return;
  }
  ```
- User confirmation before destructive actions: `if (!confirm('Eliminar proyecto?')) return;`

## Test Coverage Gaps

**Critical untested areas:**

**Pricing Calculations (HIGH RISK):**
- `calc()` at ~line 4610 — custom mode pricing engine
- `calcEstandarizado()` at ~line 5677 — standardized mode pricing engine
- Material cost calculations, sheet counting with merma factor
- Hardware auto-assignment logic (bisagra count rules, threshold-based pricing)
- These are the core business logic functions where bugs directly impact quoted prices

**PDF Generation (MEDIUM RISK):**
- `genClientPDF()` at ~line 8016 — client-facing PDF
- `genInternalPDF()` at ~line 8412 — internal cost breakdown PDF
- `genEstInternalPDF()` at ~line 8639 — estandarizado internal PDF
- PDF layout, formatting, and content accuracy

**State Management (MEDIUM RISK):**
- The global `S` object mutations in `window.APP` methods
- State consistency when switching between custom and estandarizado modes
- Project save/load serialization (JSON round-trip of `estProject`)

**LED BOM Calculator (LOW-MEDIUM RISK):**
- `calcLedBOM()` at ~line 4475
- `calcModuleLedMeters()` at ~line 4437
- Component quantity calculations, power supply sizing

**Cloud Sync (LOW RISK):**
- `cloudFetchProjects()`, `cloudSaveProject()`, `cloudDeleteProject()`
- Supabase error handling, offline fallback to localStorage

## If Adding Tests

**Recommended approach:** Extract pure calculation functions into a separate `.js` file that can be tested independently.

**Extractable pure functions (no DOM dependencies):**
- `calc()` — depends only on `S.project` state and constants
- `calcEstandarizado()` — depends only on `S.estProject` state and constants
- `calcLedBOM()` — depends on project state and LED_CATALOG
- `calcEstFixedHeight(m)` — pure function of module data
- `calcEstColgadorHeight(m, z)` — pure function
- `getBisagraCount(heightCm, materialType)` — pure function of CONFIG
- `getMatPrice(mat, thickness)` — pure function
- `fmt(n)`, `fmtD(n)` — pure formatting functions

**Non-extractable (DOM-dependent):**
- All `render*` functions (build HTML strings with inline event handlers)
- All `gen*PDF` functions (depend on jsPDF library)
- `window.APP` methods (mutate global state + call render)

**Minimal test setup would require:**
1. A `package.json` with a test runner (e.g., Vitest or Jest)
2. Extracting CONFIG, constants, and pure calc functions into importable modules
3. Creating test fixtures with sample project states
4. Snapshot testing for calc output objects

**Example test structure (if implemented):**
```
cotizador-template/
  tests/
    calc.test.js          # calc() pricing engine
    calcEstandarizado.test.js  # standardized mode pricing
    bisagra.test.js       # hardware auto-assignment
    ledBOM.test.js        # LED BOM calculator
    fixtures/
      project-closet.json # sample project state
      project-cocina.json # sample kitchen project
```

**Example test (hypothetical):**
```javascript
// calc.test.js
describe('getBisagraCount', () => {
    test('returns 3 for height <= 100cm', () => {
        expect(getBisagraCount(80, 'melamina')).toBe(3);
    });
    test('returns 4 for height 101-200cm', () => {
        expect(getBisagraCount(150, 'melamina')).toBe(4);
    });
    test('adds 1 for heavy materials', () => {
        expect(getBisagraCount(80, 'vidrio')).toBe(4); // 3 + 1 bonus
    });
});
```

## Validation Patterns in Code

**Input validation style:** Inline in APP methods, not centralized.

```javascript
// Numeric clamping pattern
m.paint.coats = Math.max(1, Math.min(5, v || 2));

// Null guard + early return pattern
var m = S.project.modules.find(function(x) { return x.id === id; });
if (m) {
    m.doors = !m.doors;
    render();
}

// Fallback chain pattern
var height = m.height || P.defaultHeight || 200;
```

**No centralized validation layer.** Each APP method handles its own input checking.

---

*Testing analysis: 2026-02-19*
