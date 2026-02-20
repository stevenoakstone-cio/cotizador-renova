# Roadmap: Cotizador Pro — Renova

## Overview

Starting from a complete closet quotation engine (~12,584 lines), this roadmap extends the single-file HTML app to cover every furniture type a carpentry shop quotes: cocinas (the revenue driver), baños, muebles TV, Tablered Arauco products, closet extensions, and puertas. After all product types are working, multi-tenant SaaS features unlock resale to other carpinterías, and a final quality phase hardens the product for the first real customer.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Foundation** - Generic calculateParts() and assignHardware() functions; closet patterns documented
- [x] **Phase 2: Cocina Bajos** - All base-cabinet module types with auto despiece and hardware
- [x] **Phase 3: Cocina Altos + Torres** - Wall cabinets (alacenas) and full-height tower columns (completed 2026-02-20)
- [x] **Phase 4: Cocina Extras + UI + Output** - Finishing pieces, cocina wizard UI, validations, and PDF
- [ ] **Phase 5: Baños** - Bathroom module types with MDF RH enforcement and filtered UI
- [ ] **Phase 6: Tablered Arauco** - Exact-piece despiece for Librero, Credenza TV, and Mesa de Centro
- [ ] **Phase 7: Mueble TV** - TV unit module types (consola, torre, repisa, fondo panel)
- [ ] **Phase 8: Closet Extension + Puertas** - Missing closet modules and interior door quotation
- [ ] **Phase 9: Multi-Tenant SaaS** - First-run config, branded PDFs, connection indicator, license screen
- [ ] **Phase 10: Quality + Polish** - End-to-end cocina test, professional PDF, tooltips, sheet optimizer

## Phase Details

### Phase 1: Foundation
**Goal**: Generic piece-calculation and hardware-assignment functions exist and are validated against existing closet logic, making all future module types fast to implement
**Depends on**: Nothing (first phase)
**Requirements**: FOUND-01, FOUND-02, FOUND-03
**Success Criteria** (what must be TRUE):
  1. Calling `calculateParts(moduleType, dimensions, materials)` returns a correct despiece array for a known closet module
  2. Calling `assignHardware(moduleType, dimensions, parts)` returns the correct hardware list matching current closet output
  3. Closet audit document (or inline comments) records all dimension formulas and hardware rules reused across module types
**Plans:** 2 plans
Plans:
- [x] 01-01-PLAN.md — Extract calculateParts() with audited closet formulas
- [x] 01-02-PLAN.md — Create assignHardware() and wire both into calc()

### Phase 2: Cocina Bajos
**Goal**: Users can quote all five base-cabinet module types for a cocina with automatic despiece and hardware assignment
**Depends on**: Phase 1
**Requirements**: CBAJ-01, CBAJ-02, CBAJ-03, CBAJ-04, CBAJ-05, CBAJ-06
**Success Criteria** (what must be TRUE):
  1. User can add a base module and the app generates the correct despiece (fondo, lados, techo, base, entrepaño) with 1 or 2 puertas auto-selected by width
  2. User can add a fregadero module and the despiece shows no techo, includes 2 amarres, and flags the saque trasero for tuberías
  3. User can add a cajonera and the despiece shows the correct number of cajón pieces with Blum Tandem correderas assigned
  4. User can add an esquinero ciego or en L and hardware shows 165° bisagras
  5. User can add an horno base and the despiece shows the entrepaño at 595mm with a cajón inferior and ventilación note
  6. Hardware totals for any bajo module show patas, zoclo clip count, and correct bisagra/corredera assignments
**Plans:** 3/3 plans complete
Plans:
- [x] 02-01-PLAN.md — MODS + HW_DEFAULTS + calculateParts() for 5 bajo module types
- [x] 02-02-PLAN.md — assignHardware() + calc() door logic for cocina bajos
- [x] 02-03-PLAN.md — Gap closure: cajonera cajones selector in step2() UI

### Phase 3: Cocina Altos + Torres
**Goal**: Users can quote wall cabinets and full-height tower columns, completing the vertical range of a cocina layout
**Depends on**: Phase 2
**Requirements**: CALT-01, CALT-02, CALT-03, CTOR-01, CTOR-02
**Success Criteria** (what must be TRUE):
  1. User can add an alacena estándar and the despiece shows correct wall-mount pieces with 1-2 puertas and 1-2 entrepaños
  2. User can add an alacena aventos and hardware shows the selected Aventos model (HK/HL/HS) with the single elevating door
  3. User can add an alacena sobre campana or sobre refri with reduced depth correctly reflected in the despiece
  4. User can add a torre de hornos with microondas and horno niches, cajón inferior, and ventilación note
  5. User can add a torre despensa and the despiece shows entrepaños on 32mm system with 2 puertas (superior + inferior)
**Plans:** 2/2 plans complete
Plans:
- [x] 03-01-PLAN.md — MODS + HW_DEFAULTS + calculateParts() for 5 alto + torre module types
- [x] 03-02-PLAN.md — assignHardware() + calc() door logic + step2() UI selectors

### Phase 4: Cocina Extras + UI + Output
**Goal**: A complete cocina quotation flows from project creation through extras/finishing pieces to a professional PDF, with validations catching common mistakes
**Depends on**: Phase 3
**Requirements**: CEXT-01, CEXT-02, CEXT-03, CEXT-04, CUI-01, CUI-02, CUI-03, CUI-04, CUI-05, COUT-01, COUT-02
**Success Criteria** (what must be TRUE):
  1. User can select "Cocina" as project type in Step 1 and the module picker shows only Bajos, Alacenas, Torres, and Extras categories
  2. User can add zoclo, vista lateral, panel de relleno, and cubierta (with m²-based pricing for postformado/cuarzo/granito)
  3. App shows a red warning if a fregadero or baño module has non-MDF-RH material selected
  4. App shows a warning if an horno module lacks a ventilación note, and warns if any module width is non-standard
  5. Cost summary breaks down subtotals by casco, frente, herrajes, and extras
  6. Exported PDF includes a module table, despiece per module, hardware summary, extras list, and technical note
**Plans:** 3 plans
Plans:
- [x] 04-01-PLAN.md — Extras MODS + calculateParts() + forType filtering + category grouping in step2()
- [x] 04-02-PLAN.md — Cocina validations (horno/fregadero/width) + categorized cost subtotals in step3()
- [x] 04-03-PLAN.md — Cocina PDF sections (module table, despiece, hardware, extras, technical note)

### Phase 5: Baños
**Goal**: Users can quote a complete bathroom vanity project with MDF RH enforced and all bathroom-specific module constraints applied
**Depends on**: Phase 4
**Requirements**: BANO-01, BANO-02, BANO-03, BANO-04, BANO-05
**Success Criteria** (what must be TRUE):
  1. User can select "Baño" as project type and the module picker shows only bathroom modules
  2. User can add a bajo lavabo and the despiece shows the cajón with a 220×180mm saque en U, all pieces in MDF RH
  3. User can add a cajonera baño within the 500-600mm height range with MDF RH material enforced
  4. User can add a botiquín/espejo at 100-150mm depth with wall-mount hardware
  5. Selecting any non-MDF-RH material for any baño module shows a red blocking warning
**Plans:** 2 plans
Plans:
- [ ] 05-01-PLAN.md — MODS + HW_DEFAULTS + calculateParts() + assignHardware() for 3 bano module types
- [ ] 05-02-PLAN.md — TYPES entry + calc() door logic + MDF-RH validation + cajones selector

### Phase 6: Tablered Arauco
**Goal**: Users can generate exact-specification despiece for the three Tablered Arauco catalog products (Librero KIRA, Credenza NERO, Mesa de Centro) with piece labels matching the official catalog
**Depends on**: Phase 1
**Requirements**: TABL-01, TABL-02, TABL-03, TABL-04
**Success Criteria** (what must be TRUE):
  1. User can select Librero and the despiece lists exactly 9 labeled pieces (A–I), 59.2ml cubrecanto, 8 bisagras, and 6 regatones
  2. User can select Credenza TV and the despiece lists exactly 7 labeled pieces (A–G), 49ml cubrecanto, 8 bisagras, and 4 patas
  3. User can select Mesa de Centro and the despiece shows 2 blocks (KIRA 5 pcs + NERO 4 pcs), 1 pistón 150N, 2 bisagras, and 4 patas
  4. User can select librero, credenza, or mesa_centro as distinct project types with the module picker filtered to only that type
**Plans**: TBD

### Phase 7: Mueble TV
**Goal**: Users can quote TV unit projects across all four module types with auto despiece and hardware
**Depends on**: Phase 4
**Requirements**: MTV-01, MTV-02, MTV-03, MTV-04, MTV-05
**Success Criteria** (what must be TRUE):
  1. User can select "TV" as project type and the module picker shows only TV unit modules
  2. User can add a consola con patas or consola flotante with correct despiece for each mounting method
  3. User can add a torre lateral TV (300-500mm wide) with an open or glass door option
  4. User can add a repisa flotante choosing between hollow-box construction or hidden bracket, and the despiece reflects the chosen method
  5. User can add a panel de fondo (lambrín) and the despiece includes the hidden bastidor pieces
**Plans**: TBD

### Phase 8: Closet Extension + Puertas
**Goal**: Closet module coverage is complete and users can quote interior doors as a standalone project type
**Depends on**: Phase 4
**Requirements**: CLOS-01, CLOS-02, PTED-01, PTED-02, PTED-03
**Success Criteria** (what must be TRUE):
  1. All closet module types from INSTRUCTIVO_TECNICO.md are available: colgador largo, corto doble, zapatero, maletero, and torre entrepaños
  2. Adding any closet module taller than 1,500mm shows an anti-vuelco warning
  3. User can select "Puerta" as project type and add a puerta interior with bastidor pino, 2 tapas MDF, marco chambrana, and tope in the despiece
  4. Bisagra count auto-calculates as 3 for honey-comb or peinazos relleno and 4 for sólida relleno
**Plans**: TBD

### Phase 9: Multi-Tenant SaaS
**Goal**: The app can be deployed for any carpintería with their own branding, Supabase instance, and license key — without code changes
**Depends on**: Phase 4
**Requirements**: MTNT-01, MTNT-02, MTNT-03, MTNT-04, MTNT-05
**Success Criteria** (what must be TRUE):
  1. On first launch (no APP_CONFIG stored), the app shows a setup screen where the user enters Supabase URL, anon key, company name, and uploads a logo
  2. PDFs generated after setup show the configured company name and logo instead of hardcoded "Renova"
  3. The app header shows a green indicator when connected to Supabase and a red indicator when offline
  4. A license screen shows the app version, client name, expiration date, and support contact
  5. An invalid or expired licenseKey shows a lock screen preventing app use
**Plans**: TBD

### Phase 10: Quality + Polish
**Goal**: The cocina workflow is verified end-to-end, the PDF is professional-grade, and usability aids (tooltips, sheet calculator) are in place
**Depends on**: Phase 9
**Requirements**: QUAL-01, QUAL-02, QUAL-03, QUAL-04
**Success Criteria** (what must be TRUE):
  1. A complete cocina project (add modules → view despiece → view costs → save to Supabase → export PDF) completes without errors
  2. The cocina PDF has a cover page, module index, per-module despiece, and material + hardware summary sections
  3. Hovering over any module type button shows a tooltip with standard dimensions and a description
  4. The sheet optimizer calculates the number of 1220×2440mm sheets needed from the full despiece piece list
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9 → 10

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Foundation | 2/2 | Complete | 2026-02-19 |
| 2. Cocina Bajos | 3/3 | Complete   | 2026-02-20 |
| 3. Cocina Altos + Torres | 2/2 | Complete   | 2026-02-20 |
| 4. Cocina Extras + UI + Output | 3/3 | Complete | 2026-02-20 |
| 5. Baños | 0/2 | Planning | - |
| 6. Tablered Arauco | 0/TBD | Not started | - |
| 7. Mueble TV | 0/TBD | Not started | - |
| 8. Closet Extension + Puertas | 0/TBD | Not started | - |
| 9. Multi-Tenant SaaS | 0/TBD | Not started | - |
| 10. Quality + Polish | 0/TBD | Not started | - |
