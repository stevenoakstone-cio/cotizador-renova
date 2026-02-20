# Technology Stack

**Analysis Date:** 2026-02-19

## Languages

**Primary:**
- HTML5 - Single-file application shell (`cotizador-template/cotizador-template.html`, 12,584 lines)
- JavaScript (ES5) - All application logic embedded in a single IIFE starting at line 2947
- CSS3 - Embedded in `<style>` block (lines 17-2941), no external stylesheets

**Secondary:**
- SVG - Inline dynamic favicon and PWA icons generated via JavaScript

## Runtime

**Environment:**
- Browser-only (no server, no Node.js)
- Targets modern mobile and desktop browsers
- PWA-capable with inline service worker and manifest (lines 12547-12580)

**Package Manager:**
- None. No `package.json`, `node_modules`, or build tooling exists.
- All dependencies are either local files or CDN-loaded.

## Frameworks

**Core:**
- None. Pure vanilla JavaScript with manual DOM manipulation via `innerHTML` string construction.
- No React, Vue, Angular, or any UI framework.

**PDF Generation:**
- jsPDF v2.5.1 - Local file `cotizador-template/jspdf.umd.min.js`
- jsPDF-AutoTable v3.8.2 - Local file `cotizador-template/jspdf.plugin.autotable.min.js`

**Cloud/Backend:**
- Supabase JS SDK v2 - CDN loaded from `https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/dist/umd/supabase.min.js`

**Testing:**
- None. No test framework, no test files.

**Build/Dev:**
- None. No bundler (no Webpack, Vite, Rollup, esbuild). No transpilation. No minification pipeline.
- The app is edited and served as-is.

## Key Dependencies

**Critical (application will not function without these):**
- `jspdf.umd.min.js` (v2.5.1) - PDF document generation for client and internal quotes
- `jspdf.plugin.autotable.min.js` (v3.8.2) - Table rendering in generated PDFs
- `supplier-catalog-data.js` - Hardware catalog data loaded as global `CATALOG` variable

**Optional (app degrades gracefully without):**
- `@supabase/supabase-js@2` (CDN) - Cloud project storage; falls back to localStorage when unavailable
- `logo_renova.webp` - Company logo for PDF headers; stored as base64 in localStorage after upload

## Configuration

**Environment:**
- All configuration is embedded in the `CONFIG` object at line 2951 of `cotizador-template/cotizador-template.html`
- Supabase credentials configured via `CONFIG.supabase.url` and `CONFIG.supabase.key` (currently empty strings)
- No `.env` files exist; no environment variable loading

**Build:**
- No build configuration. The HTML file is the distributable artifact.

## Browser APIs Used

**Storage:**
- `localStorage` - Primary persistence for projects, settings, logo, quote counter (prefix: `cot_`)
- Service Worker Cache API - Offline caching via inline service worker (`cotizador-v5` cache name)

**File/Media:**
- `FileReader` API - Logo upload and file import (lines 11198, 12447)
- `Canvas` API - Image resizing for logo uploads (line 11202)
- `Blob` / `URL.createObjectURL` - Service worker registration and manifest injection
- `navigator.clipboard.writeText` - Copy-to-clipboard functionality (line 11509)

**PWA:**
- Inline Web App Manifest (generated dynamically at line 12547)
- Inline Service Worker (generated from string at line 12574)
- Apple mobile web app meta tags (lines 8-10)

## Application Architecture

**Pattern:** Single-file monolith with IIFE encapsulation
- All HTML, CSS, and JS in one file
- One global IIFE wrapping all application state and functions
- State managed via closure variables (e.g., `S` state object)
- UI rendered by functions that set `innerHTML` on `#app`
- External data via global `CATALOG` array from `supplier-catalog-data.js`

**Code Organization (approximate line ranges in `cotizador-template.html`):**
- CSS: lines 17-2941
- CONFIG object: lines 2951-3049
- Material/color data: lines 3068+
- Core calculation functions: `calc()` ~line 4425, `calcEstandarizado()` ~line 5096
- UI rendering: `step3()` ~line 6763, `renderEstSummary()` ~line 9535
- PDF generation: `genClientPDF()` and `genInternalPDF()` ~lines 8017-9107
- Settings panel: ~line 11185
- PWA setup: lines 12547-12580

## Platform Requirements

**Development:**
- Any text editor
- A web browser (Chrome/Safari recommended for PWA features)
- A local HTTP server (file:// protocol blocks service worker registration)

**Production:**
- Static file hosting (no server-side processing required)
- HTTPS recommended for PWA and service worker functionality

---

*Stack analysis: 2026-02-19*
