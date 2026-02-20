# Cotizador Pro — Renova

## What This Is

A SaaS carpentry quotation tool for Renova (Monterrey, México). Generates automatic piece breakdowns (despiece), calculates material and hardware costs, and exports professional PDF quotes. Each client (carpentry shop) gets their own branded instance deployed on Vercel + Supabase.

## Core Value

Given module dimensions and material selection, instantly generate a complete, accurate despiece with pricing — eliminating manual calculation errors and saving hours per quote.

## Requirements

### Validated

- ✓ Closet quotation with automatic despiece — existing
- ✓ Dual material system (casco + frente) — existing
- ✓ Automatic hardware assignment (bisagras, correderas) — existing
- ✓ Threshold-based hardware pricing ($500 split) — existing
- ✓ Client PDF generation (dark theme, minimal) — existing
- ✓ Internal PDF generation (detailed, autoTable) — existing
- ✓ Despiece PDF export — existing
- ✓ Supabase cloud sync (optional, falls back to localStorage) — existing
- ✓ PWA offline support — existing
- ✓ Estandarizado mode with zone-based modules — existing
- ✓ Template system for quick project starts — existing
- ✓ Supplier catalog integration (Arauco/Herraxa) — existing
- ✓ 20% merma factor on material calculations — existing
- ✓ LED BOM calculation — existing

### Active

- [ ] Generic `calculateParts()` function for all module types
- [ ] Generic `assignHardware()` function for all module types
- [ ] Cocina: módulos bajos (base, fregadero, cajonera, esquinero, horno)
- [ ] Cocina: módulos altos / alacenas (estándar, aventos, sobre campana)
- [ ] Cocina: torres / columnas (hornos, despensa)
- [ ] Cocina: piezas extra (zoclo, vista lateral, panel relleno, cubierta)
- [ ] Cocina: UI filtrado por tipo + agrupación visual
- [ ] Cocina: validaciones (ventilación horno, MDF RH fregadero, anchos estándar)
- [ ] Cocina: totales desglosados + PDF profesional
- [ ] Baño: módulos (bajo lavabo, cajonera, botiquín)
- [ ] Baño: MDF RH obligatorio + saque en U + montaje flotante
- [ ] Mueble TV: módulos (consola, torre lateral, repisa flotante, panel fondo)
- [ ] Librero: despiece exacto Tablered Arauco (9 piezas KIRA)
- [ ] Credenza TV: despiece exacto Tablered Arauco (7 piezas NERO)
- [ ] Mesa de Centro: despiece 2 bloques Tablered (KIRA + NERO)
- [ ] Closet: extensión con módulos faltantes (zapatero, maletero, torre entrepaños)
- [ ] Puertas de intercomunicación: bastidor + tapas + marco + herrajes
- [ ] Multi-tenant: pantalla configuración inicial (Supabase URL, branding)
- [ ] Multi-tenant: PDF con nombre/logo del cliente
- [ ] Multi-tenant: indicador estado conexión Supabase
- [ ] Multi-tenant: pantalla licencia y protección básica

### Out of Scope

- Framework migration (React, Vue, etc.) — monolithic HTML is the deliberate architecture
- Backend/API server — Supabase handles all cloud needs
- User authentication — each instance is single-tenant
- Automated testing framework — manual testing is sufficient for current scale
- File separation (splitting HTML into modules) — unless explicitly requested
- Mobile native app — PWA covers mobile needs

## Context

- **Market:** Carpinterías in Monterrey, México. Small shops (1-5 people) that quote manually today.
- **Existing codebase:** ~12,584 lines single HTML file with closet functionality complete.
- **Architecture:** Single IIFE, global state `S`, full DOM re-render via `render()`, string-based HTML construction.
- **Dual calc engines:** `calc()` for custom mode, `calcEstandarizado()` for standardized mode — significant code duplication (~1,676 combined lines).
- **Supplier catalog:** `supplier-catalog-data.js` (demo) / `supplier-catalog-data-REAL.js` (production with Arauco/Herraxa prices).
- **Technical reference:** All dimensions, formulas, hardware rules, and assembly steps documented in `INSTRUCTIVO_TECNICO.md`.
- **Development plan:** Block-by-block execution order documented in `TASKS.md`.

## Constraints

- **Architecture**: Keep everything in single HTML file — deliberate choice for simplicity and deployment
- **Measurements**: All dimensions in millimeters (mm), all prices in MXN
- **Backwards compatibility**: Never break existing closet functionality
- **Data structure**: Don't change `project.data` structure — only extend with new module types
- **Materials**: Always use MDF RH in baño/fregadero zones — business rule, not optional
- **Thickness**: Standard 15mm for all structural boards
- **Formulas**: Inner width = width - 30mm (2 × 15mm thickness) — universal formula
- **Priority**: Cocinas first — it's the product for the first sale

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Single HTML monolith | Simple deploy, no build step, each client gets one file | ✓ Good |
| Dual material system (casco + frente) | Reflects real carpentry pricing — cheap interior, premium exterior | ✓ Good |
| Threshold-based hardware pricing ($500) | Simplifies hardware margin calculation vs. per-brand rules | ✓ Good |
| Supabase for cloud (optional) | Zero-config for offline, easy setup for cloud | ✓ Good |
| Generic calculateParts/assignHardware functions | Avoid duplicating calc logic for each new furniture type | — Pending |
| Block execution order (0→1→2→4→3→5→6→7→8) | Cocinas first (revenue), then easier types, then SaaS features | — Pending |

---
*Last updated: 2026-02-19 after initialization*
