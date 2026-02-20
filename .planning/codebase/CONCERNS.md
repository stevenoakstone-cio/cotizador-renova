# Codebase Concerns

**Analysis Date:** 2026-02-19

## Tech Debt

**Single-File Architecture (12,584 lines):**
- Issue: The entire application — CSS (~2,940 lines), HTML, and JavaScript (~9,600 lines) — lives in one file `cotizador-template/cotizador-template.html`. This makes navigation, editing, and merge conflicts extremely difficult.
- Files: `cotizador-template/cotizador-template.html`
- Impact: Any edit risks unintended changes to unrelated sections. IDE features like code folding struggle at this scale. Multiple developers cannot work simultaneously without conflicts.
- Fix approach: Extract CSS into a separate `.css` file. Split JS into modules (config, calc engine, renderers, PDF generators, APP methods). Use a bundler or simple concatenation script.

**Duplicated Calculation Logic (calc vs calcEstandarizado):**
- Issue: `calc()` (~751 lines, line 4610) and `calcEstandarizado()` (~925 lines, line 5677) share nearly identical cost calculation logic — material area computation, sheet counting, chapacanto calculation, hardware consolidation, margin application, and LED BOM generation. The same formulas appear in both with minor structural differences.
- Files: `cotizador-template/cotizador-template.html` lines 4610-5360 and 5677-6600
- Impact: Any pricing formula change must be applied in two places. Bugs fixed in one function may not be fixed in the other. This is the highest-risk tech debt in the codebase.
- Fix approach: Extract shared costing logic into a unified `calcCosts(pieces, autoHW, projectConfig)` function. Both `calc()` and `calcEstandarizado()` would generate their respective piece lists, then call the shared costing function.

**Duplicated LED BOM Logic:**
- Issue: `calcLedBOM()` (line 4475) and `calcEstLedBOM()` (line 5550) duplicate LED bill-of-materials calculation. Similarly `calcModuleLedMeters()` (line 4437) and `calcEstModuleLedMeters()` (line 9859) are near-duplicates.
- Files: `cotizador-template/cotizador-template.html` lines 4437, 4475, 5550, 9859
- Impact: LED pricing changes require updating two code paths.
- Fix approach: Unify into a single parameterized function.

**Duplicated PDF Generation:**
- Issue: Three PDF functions — `genClientPDF()` (line 8016, ~396 lines), `genInternalPDF()` (line 8412, ~227 lines), and `genEstInternalPDF()` (line 8639, ~258 lines) — share similar page setup, header rendering, and styling logic.
- Files: `cotizador-template/cotizador-template.html` lines 8016-8897
- Impact: PDF layout changes need to be applied across multiple functions.
- Fix approach: Extract shared PDF utilities (header, footer, page setup, color schemes) into helper functions.

**Monolithic APP Object (~2,136 lines):**
- Issue: `window.APP` (line 10401) is a single massive object containing ~100+ methods for all user interactions across both custom and estandarizado modes. No separation of concerns.
- Files: `cotizador-template/cotizador-template.html` lines 10400-12536
- Impact: Difficult to find specific handlers. Easy to accidentally break unrelated functionality when editing.
- Fix approach: Group APP methods by feature domain (navigation, custom-mode, estandarizado-mode, PDF, settings, saved-projects).

**Demo Data Blocking Production Use:**
- Issue: The codebase ships with demo data (COLORS, HW_DEFAULTS, CATALOG) instead of real supplier data. Real data exists in commented-out blocks and as `supplier-catalog-data-REAL.js`. Switching requires manual uncommenting and file renaming.
- Files: `cotizador-template/cotizador-template.html` lines 3072-3117 (COLORS), 3240-3295 (HW_DEFAULTS), `cotizador-template/supplier-catalog-data.js`, `cotizador-template/supplier-catalog-data-REAL.js`
- Impact: Production deployment requires manual code edits. Risk of accidentally deploying demo data.
- Fix approach: Use a single CONFIG flag (e.g., `CONFIG.demoMode`) to toggle between demo and real data at runtime, or use a build step to select the correct data files.

**Deep Cloning via JSON.parse(JSON.stringify()):**
- Issue: Used 14+ times throughout the codebase for deep cloning objects (material objects, project state, modules). This is fragile — it silently drops `undefined` values, functions, and circular references.
- Files: `cotizador-template/cotizador-template.html` lines 10431, 10578, 10587, 10594, 10635, 11372, 11579, 12028, 12038, 12045, 12086, 12462, 12500
- Impact: Could cause subtle data loss if objects ever contain non-JSON-serializable values. Performance concern for large project objects.
- Fix approach: Implement a proper `deepClone()` utility function or use `structuredClone()` (available in modern browsers).

**Legacy Brand-Based Hardware Accumulators:**
- Issue: After migrating to threshold-based hardware pricing, the code still maintains legacy brand-based accumulators (`hwCostHerraxa`, `hwCostHafele`, `hwCostBlum`) with a comment "kept for backward compat."
- Files: `cotizador-template/cotizador-template.html` lines 5234-5236, 5252-5254
- Impact: Dead code that adds confusion and maintenance burden.
- Fix approach: Remove legacy accumulators if no downstream code references them.

## Security Considerations

**Unused Sanitize Function (XSS Risk):**
- Risk: A `sanitize()` function exists at line 7572 that strips `<>&"'` characters, but it is never called anywhere in the codebase. All user-provided text (client names, project names, module names) is interpolated directly into HTML strings via string concatenation, then injected via `.innerHTML`.
- Files: `cotizador-template/cotizador-template.html` line 7572 (definition), lines 10356/10367/10394 (innerHTML assignments)
- Current mitigation: The app runs locally/offline as a PWA; user input is self-generated. Low risk in practice but poor practice.
- Recommendations: Call `sanitize()` on all user-provided strings before HTML interpolation — especially client name, project name, and module names. There are ~198 `onclick=` handlers and ~63 `onchange=` handlers using inline event attributes with string interpolation, all potential injection vectors.

**Supabase Credentials in Client-Side Code:**
- Risk: Supabase URL and anon key are configured in `CONFIG.supabase` (lines 3045-3048) and used directly in client-side code (line 4280). Currently empty strings, but when populated, the anon key will be visible in source.
- Files: `cotizador-template/cotizador-template.html` lines 3045-3048, 4276-4280
- Current mitigation: Keys are empty; Supabase not active. Anon keys are designed to be public, so this is acceptable if Row Level Security (RLS) is properly configured on the Supabase side.
- Recommendations: Ensure RLS policies are configured before deploying with real Supabase credentials. Document that the key must be the anon/public key, never the service_role key.

**Silent Error Swallowing:**
- Risk: Nearly all `catch` blocks are empty (`catch (e) {}`) — 15+ instances throughout the codebase. Failed localStorage operations, failed image loading, failed PDF generation, and failed Supabase operations all fail silently.
- Files: `cotizador-template/cotizador-template.html` lines 4161, 4167, 4173, 4316, 8041, 8409, 11186, 11213, 11226, 11233, 12545, 12569, 12578
- Current mitigation: None.
- Recommendations: Add at minimum `console.warn()` to catch blocks. For critical operations (save, PDF generation), show a user-facing error notification.

## Performance Bottlenecks

**Full DOM Re-render on Every State Change:**
- Problem: The `render()` function (line 10350) rebuilds the entire DOM via string concatenation and sets `$('app').innerHTML`. Every user interaction triggers a full re-render of all HTML content.
- Files: `cotizador-template/cotizador-template.html` line 10394
- Cause: No virtual DOM or diffing. The entire UI is reconstructed as a string and injected. Scroll position is manually saved/restored (line 10353, 10397).
- Improvement path: For the current architecture, this is acceptable at current complexity. If performance degrades, consider targeted DOM updates for frequent operations (e.g., quantity changes) or adopt a lightweight reactive library.

**calc() Called on Every Render:**
- Problem: In custom mode, `calc()` (line 10370) is called on every render, recalculating all piece dimensions, hardware consolidation, material costs, and LED BOM — even when the user is just scrolling or viewing results.
- Files: `cotizador-template/cotizador-template.html` line 10370
- Cause: No memoization or dirty-checking.
- Improvement path: Add a dirty flag to skip recalculation when project state has not changed, or cache the calc result and invalidate on project mutations.

**String Concatenation for HTML:**
- Problem: All UI rendering functions build HTML via string concatenation (`h += '<div...'`). In step3 alone, this produces hundreds of concatenation operations per render.
- Files: `cotizador-template/cotizador-template.html` lines 7084-7237 (step3), 9240-9698 (est builder/editor), 10090-10312 (est summary)
- Cause: No template engine or tagged template literals.
- Improvement path: Use template literals (backtick strings) for readability. Consider a lightweight template engine for complex sections.

## Fragile Areas

**Magic Numbers in Piece Calculations:**
- Files: `cotizador-template/cotizador-template.html` lines 4619-4985 (calc), 5677-6200 (calcEstandarizado)
- Why fragile: Furniture piece dimensions use hardcoded offsets without named constants:
  - `w - 32` (two 16mm panel thicknesses, appears 15+ times)
  - `w - 6` (3mm gap per side for drawer fronts)
  - `h - 100` (100mm deducted for drawer height calc)
  - `d - 50` (50mm clearance for drawer sides)
  - `ch - 80` (80mm clearance for drawer interior)
  - `w - 100` (100mm clearance for drawer internals)
  - `zapFrH = 50` (line 5969, zapatero front height)
  - `zocD = 550` (line 6152, zoclo depth)
- Safe modification: Extract all magic numbers into named constants at the top of the IIFE (e.g., `var GAP_DRAWER_FRONT = 6`, `var CLEARANCE_DRAWER_SIDE = 50`). Then search-and-replace usage. Always test with a known project to verify piece list output.
- Test coverage: None. No automated tests exist.

**Thickness Assumption (w - 32 = w - 2*16):**
- Files: `cotizador-template/cotizador-template.html` lines 4643, 4730, 4754, 4765, 4809, 4820, 4928, 4939, 4974, 4985, 5020, 5029, 5030, 5035, 5036, 5042, 5056, 5057, 5062, 5073, 5074, 5080, 5081
- Why fragile: The value `32` assumes 16mm thickness (`2 * TK`), but `TK` is a variable (supporting 16mm and 25mm). If thickness is 25mm, `w - 32` should be `w - 50`. However, some usages correctly use `w - 2 * TK` while many use the hardcoded `w - 32`.
- Safe modification: Audit every `w - 32` and replace with `w - 2 * TK`. Some may already be correct for the specific context; each needs individual verification.
- Test coverage: None.

**Inline Event Handler String Interpolation:**
- Files: `cotizador-template/cotizador-template.html` (throughout all render functions)
- Why fragile: Event handlers are built as inline HTML strings like `onclick="APP.setLabor(' + p.v + ')"`. If any interpolated value contains special characters, the handler breaks silently. There are 198 `onclick=` and 63 `onchange=` instances.
- Safe modification: Continue the current pattern but ensure all interpolated values are numeric or pre-validated. For string values, always sanitize.
- Test coverage: None.

**Scroll Position Restoration:**
- Files: `cotizador-template/cotizador-template.html` lines 10351-10353, 10395-10397
- Why fragile: `scrollY` is captured before render and restored after, but only for `#content`. If rendering changes the DOM structure significantly (e.g., adding/removing cards), the restored scroll position may be incorrect.
- Safe modification: Consider saving scroll position per screen/step rather than globally.
- Test coverage: None.

## Scaling Limits

**localStorage Size Limit:**
- Current capacity: Browsers typically allow 5-10MB per origin.
- Limit: Saving many projects with embedded plano images (base64) could exceed localStorage limits. Each base64 image can be 1-5MB.
- Scaling path: Supabase integration (partially implemented) would offload storage. Consider compressing images before storing, or store only Supabase URLs for images.

**Single-Thread Calculation:**
- Current capacity: Works fine for projects with ~10-20 modules.
- Limit: Very large projects (50+ modules) would cause noticeable lag during `calc()` / `calcEstandarizado()` since they run on the main thread and trigger full re-renders.
- Scaling path: Unlikely to be a real issue given typical furniture project sizes (1-15 modules).

## Dependencies at Risk

**CDN-Loaded Supabase (No Pinned Version):**
- Risk: `https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/dist/umd/supabase.min.js` loads the latest v2.x. A breaking change in a minor/patch release could break the app.
- Files: `cotizador-template/cotizador-template.html` line 16
- Impact: App could fail to initialize Supabase after a CDN update.
- Migration plan: Pin to a specific version (e.g., `@supabase/supabase-js@2.39.0`) or bundle the library locally like jsPDF.

**Locally Bundled jsPDF (No Version Tracking):**
- Risk: `jspdf.umd.min.js` and `jspdf.plugin.autotable.min.js` are bundled locally with no version identifier. Upgrading requires manually replacing files with no way to know the current version.
- Files: `cotizador-template/jspdf.umd.min.js`, `cotizador-template/jspdf.plugin.autotable.min.js`
- Impact: Cannot easily check for security patches or new features.
- Migration plan: Add version comments to the top of bundled files, or use a package.json to track versions.

## Missing Critical Features

**No Automated Tests:**
- Problem: Zero test files exist in the project. No test framework configured. No unit tests for `calc()`, `calcEstandarizado()`, or any pricing logic.
- Blocks: Safe refactoring of any calculation logic. Cannot verify correctness after changes. Cannot catch regressions.

**No Input Validation on Dimensions:**
- Problem: Module width, height, and depth values from user input are used directly in calculations with no bounds checking. Negative values, zero values, or extremely large values could produce nonsensical piece lists and costs.
- Files: `cotizador-template/cotizador-template.html` lines 4619-4621 (calc dimension setup)
- Blocks: Reliable quoting — a typo (e.g., 9000 instead of 900mm) would produce an incorrect quote with no warning.

**No Undo/Redo:**
- Problem: All state mutations in the APP object are immediate and irreversible. Deleting a module, changing material, or resetting a project cannot be undone.
- Blocks: User confidence — accidental deletions require re-entering data manually.

## Test Coverage Gaps

**All Calculation Logic (Critical):**
- What's not tested: `calc()`, `calcEstandarizado()`, `calcLedBOM()`, `calcEstLedBOM()`, `calcPaintLiters()`, `calcPaintM2()`, `getBisagraCount()`, piece generation, hardware consolidation, margin application, sheet counting with merma factor.
- Files: `cotizador-template/cotizador-template.html` lines 3053-3066, 4437-5360, 5492-6600
- Risk: Pricing errors go undetected. A single wrong offset (e.g., `w - 32` instead of `w - 2 * TK`) could produce incorrect quotes for all 25mm-thickness projects.
- Priority: High — this is a financial tool; incorrect calculations directly impact business revenue.

**PDF Output:**
- What's not tested: PDF generation functions produce correct layouts, correct totals, correct legal terms.
- Files: `cotizador-template/cotizador-template.html` lines 8016-9199
- Risk: PDF totals could diverge from on-screen totals if PDF-specific formatting introduces rounding differences.
- Priority: Medium.

**State Management:**
- What's not tested: localStorage save/load round-trips, Supabase sync, project cloning, state mutations via APP methods.
- Files: `cotizador-template/cotizador-template.html` lines 4286-4380, 10400-12536
- Risk: Data loss during save/load cycles.
- Priority: Medium.

---

*Concerns audit: 2026-02-19*
