# Proyecto Renova — Cotizador Template: Implementation Instructions

## 1. Fix Hardware Pricing Logic

Replace the current flat hardware margin with a cost-threshold rule:

- If total hardware item cost is **≤ $500 MXN** → absorb into carpentry, apply **3× multiplier** (same as materials)
- If total hardware item cost is **≥ $501 MXN** → treat as Blum/premium hardware, apply **30% margin** (price = cost / 0.70)

Add to CONFIG:
```javascript
pricing: {
  carpentryMultiplier: 3,
  hardwareThreshold: 500,        // items at or below this fold into carpentry
  premiumHardwareMargin: 0.30,   // items above threshold: price = cost / 0.70
  ivaRate: 0.16
}
```

The pricing engine should evaluate each hardware line item individually against the threshold before deciding which margin to apply.

---

## 2. Add 4 New Furniture Types to CONFIG.modules

Add these alongside the existing closet modules. Each entry needs: type name, display label (Spanish), and its default zone/component list.

**Cocinas**
- Zones: alacena alta, gabinete base, cajón base, despensa, espacio campana, esquinero giratorio

**Muebles de TV**
- Zones: módulo central, lateral abierto, cajones, entrepaños, nicho central

**Lavanderas**
- Zones: módulo base, espacio lavadora, espacio secadora, alacena superior, entrepaños

**Puertas**
- Structurally different — no interior zones. Just: panel, marco (optional toggle), material type flag (standard / vidrio / metal). No zone picker needed.

---

## 3. Add Door Bisagra Auto-Assignment Rules

Add to CONFIG under `hardware.autoAssignRules.puertas`:

```javascript
bisagras: [
  { maxHeight: 100, count: 3 },
  { maxHeight: 200, count: 4 },
  { maxHeight: Infinity, count: 5 }
],
heavyMaterialBonus: 1   // add 1 bisagra if material is vidrio or metal
```

The auto-assignment engine should read these rules from CONFIG rather than having the numbers hardcoded. When the module type is `puerta`, it reads door height, looks up the rule, then checks the material flag for the bonus bisagra.

---

## 4. Verify Client PDF is Minimal

The client-facing PDF should match the format of a real cotización: one narrative description, total price, legal terms. No module-by-module breakdown.

Check the current Client PDF output and if it's showing per-module detail, suppress it. That detail stays in the Internal PDF only. The client PDF structure should be:

1. Header (brand logo, contact info, date, quote number)
2. Client name + project name
3. Single description block (narrative text, editable per project)
4. Price summary: subtotal → IVA → total
5. Legal terms block (from `CONFIG.brand.legalTerms`)

---

## 5. Move Legal Terms to CONFIG

The payment terms, delivery timeline, cancellation policy, installation scope, and materials warranty disclaimer should all live in `CONFIG.brand.legalTerms` as a template string — not hardcoded in the PDF generation function.

Default value should include:
- 15-day quote validity
- 6–8 week delivery from design approval and 60% deposit
- 60% anticipo to start, 40% before installation
- No cancellation after deposit
- Installation included within ZMM, Monday–Friday 9am–5pm
- No electrical/hydraulic/gas work included
- UV and humidity damage not covered under warranty

Each new client gets their own version in their CONFIG without touching app logic.

---

## 6. Add `// CUSTOMIZE:` Comments

Anywhere a future client would need to change something not covered by CONFIG, add a `// CUSTOMIZE:` comment. Priority spots:

- Where module types are rendered in the UI (in case a client doesn't use all 5 types)
- The bisagra threshold rules (some clients may use different bisagra brands or counts)
- The hardware threshold value ($500) — some clients may want a different cutoff
- The PDF header layout if a client has a horizontal vs square logo

---

## What Not to Touch

- IIFE architecture and S state object
- `window.APP` handler structure
- jsPDF and Supabase integration patterns
- PWA service worker
- Dark theme and mobile-first layout
- Existing closet module logic — extend, don't replace
- Settings panel margin controls

---

## Output Language Note

All code, variable names, comments, and CONFIG keys in **English**. All UI strings, labels, PDF content, and user-facing text remain in **Spanish**.
