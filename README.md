# Cotizador Template

A customizable furniture quoting app (cotizador) for carpentry businesses. Single-file PWA with PDF generation, cloud storage, and a complete pricing engine.

## Quick Start

1. Open `cotizador-template.html` in a text editor
2. Find the `CONFIG` object at the top of the `<script>` block
3. Fill in your business details:

```js
var CONFIG = {
    brand: {
        name: 'MI EMPRESA',      // Your business name (uppercase)
        mono: 'ME',              // 2-3 letter abbreviation
        tagline: 'Muebles a la Medida',
        altBrand: null           // Optional second brand (see below)
    },
    contact: {
        name: 'Tu Nombre',
        office: '00 0000-0000',
        cell: '00 0000-0000',
        email: 'contacto@miempresa.com',
        city: 'Tu Ciudad'
    },
    ...
};
```

4. Open the file in a browser — it works offline immediately

## CONFIG Reference

### `brand`
| Field | Description |
|-------|-------------|
| `name` | Business name shown in PDFs and quotes |
| `mono` | Short abbreviation for icons, logos |
| `tagline` | Subtitle shown in quote headers |
| `altBrand` | Optional: `{ id:'alt', mono:'ALT', name:'OTHER', tagline:'...' }` |

### `contact`
Contact info used in PDFs, quote previews, and text exports.

### `quoting`
| Field | Description |
|-------|-------------|
| `prefix` | Quote number prefix (e.g., `'COT'` → `COT-2026-0001`) |
| `defaultDelivery` | Default delivery time text |
| `defaultAnticipo` | Default advance payment percentage |
| `terms[]` | PDF terms (use `{anticipo}` and `{deliveryTime}` as placeholders) |
| `htmlTerms[]` | Quote preview terms (same placeholders) |

### `storage`
| Field | Description |
|-------|-------------|
| `prefix` | localStorage key prefix (keep unique per deployment) |

### `pwa`
| Field | Description |
|-------|-------------|
| `cacheName` | Service worker cache name (change on major updates to bust cache) |

### `pricing`
| Field | Default | Description |
|-------|---------|-------------|
| `iva` | `0.16` | Tax rate (16% Mexico) |
| `marginCarp` | `66` | Default carpentry margin % |
| `marginHwStd` | `66` | Default standard hardware margin % |
| `marginBlum` | `0.30` | Blum hardware margin (decimal) |
| `extrasMultiplier` | `4` | Extras sale = cost × this |
| `cuttingCostPerSheet` | `75` | Sheet cutting cost |
| `glassCostPerM2` | `800` | Glass cost per m² |
| `defaultLaborCost` | `5000` | Default labor cost per project |
| `defaultFreightCost` | `2000` | Default freight cost |
| `defaultChapPrice` | `13.10` | Default edge banding price per meter |
| `defaultEnchapPrice` | `13` | Default enchap price |
| `defaultChapSpec` | `'1mm x 19mm ABS...'` | Default edge banding specification text |
| `saleMultiplierPresets` | `[1.5, 2, 2.5, 3]` | Estandarizado mode multiplier buttons |
| `defaultSaleMultiplier` | `2` | Default sale multiplier |

### `supabase`
| Field | Description |
|-------|-------------|
| `url` | Supabase project URL (blank = localStorage only) |
| `key` | Supabase anon key (blank = localStorage only) |

### `bundledProjects`
Array of demo projects to include. Leave empty `[]` for a clean start.

## Customization Points

The code includes `// CUSTOMIZE:` comments at key extension points:

- **Module types** (`MODS`) — add new module types with different widths
- **Project types** (`TYPES`) — add/remove project categories
- **Zone types** (`EST_ZONE_TYPES`) — zones for estandarizado mode
- **Templates** (`TEMPLATES`) — quick-quote templates with base pricing
- **Hardware auto-rules** — which hardware is auto-added per module
- **Quick quote ratio logic** — pricing formula for template-based quotes
- **Paint calc** — coverage rates and painting costs
- **Estandarizado module defaults** — default widths/zones per module type

Search for `// CUSTOMIZE:` to find all extension points.

## Material Database

Materials come from two sources:

1. **FINSA** — `COLORS` and `FINSA_PRICES` arrays in the main HTML file
2. **ARAUCO + Hardware** — `supplier-catalog-data.js` external file

To update material prices, edit the `FINSA_PRICES` object or the supplier catalog JS file.

## Adding a Second Brand

Set `CONFIG.brand.altBrand`:
```js
altBrand: {
    id: 'secondary',
    mono: 'SEC',
    name: 'SECOND BRAND',
    tagline: 'Different Tagline'
}
```
Users can switch brands in Settings, and PDFs will use the selected brand.

## Supabase Setup (Cloud Storage)

1. Create a Supabase project at [supabase.com](https://supabase.com)
2. Create a `projects` table with columns: `id` (text, primary key), `data` (jsonb), `updated_at` (timestamp)
3. Enable Row Level Security or set appropriate policies
4. Add your project URL and anon key to `CONFIG.supabase`

Without Supabase, the app works fully offline using localStorage.

## Deployment

The app is a single HTML file with two JS dependencies:
- `jspdf.umd.min.js` — PDF generation
- `jspdf.plugin.autotable.min.js` — PDF tables
- `supplier-catalog-data.js` — material/hardware catalog

Deploy all four files to any static host (Vercel, Netlify, GitHub Pages, S3, etc.).

## Files

```
cotizador-template/
├── cotizador-template.html          # Main app (edit CONFIG here)
├── supplier-catalog-data.js         # Material & hardware catalog
├── jspdf.umd.min.js               # PDF generation library
├── jspdf.plugin.autotable.min.js   # PDF table plugin
└── README.md                       # This file
```
