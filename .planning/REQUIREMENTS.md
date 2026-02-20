# Requirements: Cotizador Pro

**Defined:** 2026-02-19
**Core Value:** Given module dimensions and material selection, instantly generate a complete, accurate despiece with pricing.

## v1 Requirements

### Foundation

- [x] **FOUND-01**: Generic `calculateParts()` function that generates despiece for any module type from dimensions
- [x] **FOUND-02**: Generic `assignHardware()` function that auto-assigns hardware by module type and dimensions
- [x] **FOUND-03**: Closet code audited and patterns documented for reuse

### Cocina Bajos

- [x] **CBAJ-01**: User can add base module with 1 or 2 doors (auto-calculated from width)
- [x] **CBAJ-02**: User can add fregadero module (no techo, 2 amarres, saque trasero for tuberías)
- [x] **CBAJ-03**: User can add cajonera module (2, 3, or 4 cajones with Blum Tandem correderas)
- [x] **CBAJ-04**: User can add esquinero ciego and esquinero en L (165° bisagras)
- [x] **CBAJ-05**: User can add horno base module (entrepaño at 595mm, cajón inferior, ventilación note)
- [x] **CBAJ-06**: Hardware auto-assigned for all bajos (patas, zoclo, bisagras/correderas per type)

### Cocina Altos

- [x] **CALT-01**: User can add alacena estándar (1-2 puertas, wall-mounted, 1-2 entrepaños)
- [x] **CALT-02**: User can add alacena aventos (single elevating door, model selection HK/HL/HS)
- [x] **CALT-03**: User can add alacena sobre campana/refri (shorter, reduced depth)

### Cocina Torres

- [x] **CTOR-01**: User can add torre de hornos (microondas + horno nichos, cajón inferior, ventilación)
- [x] **CTOR-02**: User can add torre despensa (entrepaños regulables 32mm, 2 puertas sup/inf)

### Cocina Extras

- [x] **CEXT-01**: User can add zoclo (calculated as linear meters from total bajo module widths)
- [x] **CEXT-02**: User can add vista lateral panel (premium material, costado size)
- [x] **CEXT-03**: User can add panel de relleno (5-100mm between module and wall)
- [x] **CEXT-04**: User can add cubierta (priced per m² — postformado, cuarzo, granito)

### Cocina UI

- [x] **CUI-01**: User can select "Cocina" as project type in Step 1 wizard
- [x] **CUI-02**: When type is cocina, UI shows only cocina modules grouped by category (Bajos, Alacenas, Torres, Extras)
- [ ] **CUI-03**: Validation warns if horno module lacks ventilación note
- [ ] **CUI-04**: Validation warns if fregadero uses non-MDF-RH material
- [ ] **CUI-05**: Validation warns if module width is non-standard

### Cocina Output

- [ ] **COUT-01**: Summary shows subtotals by casco, frente, herrajes, and extras
- [ ] **COUT-02**: PDF includes module table, despiece table, hardware summary, extras list, technical note

### Baño

- [ ] **BANO-01**: User can add bajo lavabo (MDF RH enforced, cajón with saque en U 220×180mm)
- [ ] **BANO-02**: User can add cajonera baño (MDF RH, 500-600mm height)
- [ ] **BANO-03**: User can add botiquín/espejo (thin depth 100-150mm, wall-mounted)
- [ ] **BANO-04**: User can select "Baño" as project type with filtered UI
- [ ] **BANO-05**: Validation shows red warning if non-MDF-RH material selected for baño

### Mueble TV

- [ ] **MTV-01**: User can add consola con patas and consola flotante
- [ ] **MTV-02**: User can add torre lateral TV (300-500mm wide, open or glass door)
- [ ] **MTV-03**: User can add repisa flotante (hollow box or hidden bracket, user chooses method)
- [ ] **MTV-04**: User can add panel de fondo (lambrín with hidden bastidor)
- [ ] **MTV-05**: User can select "TV" as project type with filtered UI

### Tablered Arauco

- [ ] **TABL-01**: Librero with exact 9-piece despiece KIRA (A-I), 59.2ml cubrecanto, 8 bisagras, 6 regatones
- [ ] **TABL-02**: Credenza TV with exact 7-piece despiece NERO (A-G), 49ml cubrecanto, 8 bisagras, 4 patas
- [ ] **TABL-03**: Mesa de Centro with 2-block despiece (KIRA 5 pcs + NERO 4 pcs), 1 pistón 150N, 2 bisagras, 4 patas
- [ ] **TABL-04**: User can select librero/credenza/mesa_centro as project types with filtered UI

### Closet Extension

- [ ] **CLOS-01**: All closet modules from INSTRUCTIVO_TECNICO.md available (colgador largo, corto doble, zapatero, maletero, torre entrepaños)
- [ ] **CLOS-02**: Anti-vuelco warning for modules >1,500mm height

### Puertas

- [ ] **PTED-01**: User can add puerta interior (bastidor pino + 2 tapas MDF + marco chambrana + tope)
- [ ] **PTED-02**: Bisagra count auto-calculated by relleno type (honey-comb/peinazos → 3, sólida → 4)
- [ ] **PTED-03**: User can select "Puerta" as project type with filtered UI

### Multi-Tenant

- [ ] **MTNT-01**: First-run config screen (Supabase URL, anon key, company name, logo upload)
- [ ] **MTNT-02**: PDF uses company name and logo from APP_CONFIG instead of hardcoded Renova
- [ ] **MTNT-03**: Connection status indicator in header (green=cloud, red=offline)
- [ ] **MTNT-04**: License screen with version, client name, expiration date, support contact
- [ ] **MTNT-05**: Basic license protection (licenseKey in APP_CONFIG, hash validation, lock screen)

### Quality

- [ ] **QUAL-01**: End-to-end manual test of cocina quotation (despiece, herrajes, PDF, Supabase save)
- [ ] **QUAL-02**: Cocina PDF with cover page, module index, despiece per module, material/hardware summaries
- [ ] **QUAL-03**: Tooltips on module types showing standard dimensions and description
- [ ] **QUAL-04**: Sheet optimization calculator (pieces → number of 1220×2440mm sheets needed)

## v2 Requirements

### Advanced Features

- **ADV-01**: Esquinero carousel / Le Mans system (premium rotating hardware)
- **ADV-02**: Alacena esquinera alta (corner wall cabinet)
- **ADV-03**: Moldura de luz / gola with LED integration under alacenas
- **ADV-04**: Cornisa / ceja decorative molding
- **ADV-05**: Glass door option for torre TV (aluminum frame + tempered glass)
- **ADV-06**: Automated nesting/optimization of pieces on standard sheets

## Out of Scope

| Feature | Reason |
|---------|--------|
| Framework migration (React, Vue) | Monolithic HTML is deliberate architecture |
| Backend/API server | Supabase handles all cloud needs |
| User authentication | Each instance is single-tenant |
| Automated testing | Manual testing sufficient for current scale |
| File separation | Unless explicitly requested |
| Mobile native app | PWA covers mobile needs |
| CNC export / G-code | Different product entirely |
| 3D visualization | Complexity not justified for MVP |

## Traceability

| Requirement | Phase | Phase Name | Status |
|-------------|-------|------------|--------|
| FOUND-01 | Phase 1 | Foundation | Pending |
| FOUND-02 | Phase 1 | Foundation | Pending |
| FOUND-03 | Phase 1 | Foundation | Pending |
| CBAJ-01 | Phase 2 | Cocina Bajos | Pending |
| CBAJ-02 | Phase 2 | Cocina Bajos | Pending |
| CBAJ-03 | Phase 2 | Cocina Bajos | Pending |
| CBAJ-04 | Phase 2 | Cocina Bajos | Pending |
| CBAJ-05 | Phase 2 | Cocina Bajos | Pending |
| CBAJ-06 | Phase 2 | Cocina Bajos | Pending |
| CALT-01 | Phase 3 | Cocina Altos + Torres | Pending |
| CALT-02 | Phase 3 | Cocina Altos + Torres | Pending |
| CALT-03 | Phase 3 | Cocina Altos + Torres | Pending |
| CTOR-01 | Phase 3 | Cocina Altos + Torres | Pending |
| CTOR-02 | Phase 3 | Cocina Altos + Torres | Pending |
| CEXT-01 | Phase 4 | Cocina Extras + UI + Output | Complete (04-01) |
| CEXT-02 | Phase 4 | Cocina Extras + UI + Output | Complete (04-01) |
| CEXT-03 | Phase 4 | Cocina Extras + UI + Output | Complete (04-01) |
| CEXT-04 | Phase 4 | Cocina Extras + UI + Output | Complete (04-01) |
| CUI-01 | Phase 4 | Cocina Extras + UI + Output | Complete (04-01) |
| CUI-02 | Phase 4 | Cocina Extras + UI + Output | Complete (04-01) |
| CUI-03 | Phase 4 | Cocina Extras + UI + Output | Pending |
| CUI-04 | Phase 4 | Cocina Extras + UI + Output | Pending |
| CUI-05 | Phase 4 | Cocina Extras + UI + Output | Pending |
| COUT-01 | Phase 4 | Cocina Extras + UI + Output | Pending |
| COUT-02 | Phase 4 | Cocina Extras + UI + Output | Pending |
| BANO-01 | Phase 5 | Baños | Pending |
| BANO-02 | Phase 5 | Baños | Pending |
| BANO-03 | Phase 5 | Baños | Pending |
| BANO-04 | Phase 5 | Baños | Pending |
| BANO-05 | Phase 5 | Baños | Pending |
| TABL-01 | Phase 6 | Tablered Arauco | Pending |
| TABL-02 | Phase 6 | Tablered Arauco | Pending |
| TABL-03 | Phase 6 | Tablered Arauco | Pending |
| TABL-04 | Phase 6 | Tablered Arauco | Pending |
| MTV-01 | Phase 7 | Mueble TV | Pending |
| MTV-02 | Phase 7 | Mueble TV | Pending |
| MTV-03 | Phase 7 | Mueble TV | Pending |
| MTV-04 | Phase 7 | Mueble TV | Pending |
| MTV-05 | Phase 7 | Mueble TV | Pending |
| CLOS-01 | Phase 8 | Closet Extension + Puertas | Pending |
| CLOS-02 | Phase 8 | Closet Extension + Puertas | Pending |
| PTED-01 | Phase 8 | Closet Extension + Puertas | Pending |
| PTED-02 | Phase 8 | Closet Extension + Puertas | Pending |
| PTED-03 | Phase 8 | Closet Extension + Puertas | Pending |
| MTNT-01 | Phase 9 | Multi-Tenant SaaS | Pending |
| MTNT-02 | Phase 9 | Multi-Tenant SaaS | Pending |
| MTNT-03 | Phase 9 | Multi-Tenant SaaS | Pending |
| MTNT-04 | Phase 9 | Multi-Tenant SaaS | Pending |
| MTNT-05 | Phase 9 | Multi-Tenant SaaS | Pending |
| QUAL-01 | Phase 10 | Quality + Polish | Pending |
| QUAL-02 | Phase 10 | Quality + Polish | Pending |
| QUAL-03 | Phase 10 | Quality + Polish | Pending |
| QUAL-04 | Phase 10 | Quality + Polish | Pending |

**Coverage:**
- v1 requirements: 52 total
- Mapped to phases: 52
- Unmapped: 0

---
*Requirements defined: 2026-02-19*
*Last updated: 2026-02-19 — Roadmap created, phase names added to traceability*
