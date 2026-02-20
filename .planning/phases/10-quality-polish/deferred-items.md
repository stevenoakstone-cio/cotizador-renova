# Deferred Items — Phase 10 Quality Polish

## Out-of-scope issues discovered during 10-03 review

### 1. `iW` undefined in calculateParts() for bano module types

**Found during:** Task 1 code review (10-03)
**Affects:** bajoLavabo, cajoneraBano, botiquin module types
**Issue:** Variable `iW` is used in calculateParts() for these bano modules but is never declared in that function's scope. Should be `var iW = w - 2 * TK`. This will produce NaN in piece dimensions for bano projects.
**Files:** cotizador-template/cotizador-template.html (~lines 5856-5907)
**Scope:** Out of scope for QUAL-01 (cocina workflow). Bano workflow is a separate phase concern.
**Severity:** High — will cause all bano module costs to be NaN

### 2. Missing `esquineroTipo` UI selector in step2()

**Found during:** Task 1 code review (10-03)
**Affects:** esquineroCiego module type
**Issue:** The calc() function checks `m.esquineroTipo === 'enL'` to assign 165-degree bisagras and 2 doors, but step2() has no UI toggle to set this property. Users cannot switch between 'ciego' (1 door) and 'enL' (2 doors with 165° bisagras) variants.
**Files:** cotizador-template/cotizador-template.html (step2 function)
**Scope:** Out of scope for QUAL-01. Default 'ciego' behavior works correctly for common case.
**Severity:** Low — default works; enhancement needed to unlock enL variant
