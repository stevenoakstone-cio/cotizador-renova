# CONTEXT.md — Cotizador Pro (Renova)
> Este archivo es para Claude Code. Léelo completo antes de tocar cualquier código.

---

## ¿Qué es esta app?

Un cotizador de carpintería para la empresa Renova (Monterrey, México). Genera cotizaciones con despiece de materiales, calcula precios automáticamente y exporta PDF profesional. Se vende como producto SaaS: cada cliente (carpintería) tiene su propia instancia en Vercel + Supabase.

**Stack:**
- Single HTML monolítico (`cotizador-template.html`, ~12,500 líneas, HTML + CSS + JS en un solo IIFE)
- Sin framework. Sin bundler. Sin backend. Vanilla JS puro.
- Supabase para sincronización cloud (opcional — funciona 100% offline con localStorage)
- Vercel para deploy como archivos estáticos
- jsPDF + AutoTable para generación de PDF

---

## Estructura de archivos

```
cotizador-template/
├── cotizador-template.html         ← TODO el código está aquí
├── supplier-catalog-data.js        ← Catálogo de materiales (precios de proveedor)
├── supplier-catalog-data-REAL.js   ← Versión real con precios de Arauco/Herraxa
├── jspdf.umd.min.js                ← PDF (local, offline)
├── jspdf.plugin.autotable.min.js
├── logo_renova.webp
├── CONTEXT.md                      ← Este archivo
├── TASKS.md                        ← Plan de desarrollo activo
├── INSTRUCTIVO_TECNICO.md          ← Dimensiones, materiales y lógica de armado
└── README.md
```

---

## Schema de Supabase

### Tabla: `projects`

| Columna | Tipo | Descripción |
|---|---|---|
| `id` | text (PK) | ID único del proyecto |
| `name` | text | Nombre del proyecto |
| `client` | text | Nombre del cliente |
| `type` | text | Tipo de mueble: `"closet"`, `"cocina"`, `"bano"`, `"tv"`, `"librero"`, `"credenza"`, `"mesa_centro"`, `"puerta"` |
| `mode` | text | `"custom"` o `"estandarizado"` |
| `data` | jsonb | Objeto completo (módulos, materiales, hardware, precios, despiece, todo) |
| `updated_at` | timestamp | Última modificación |

### Funciones cloud ya implementadas en el HTML

```javascript
cloudFetchProjects()           // GET todos los proyectos ordenados por updated_at
cloudSaveProject(entry)        // UPSERT por id
cloudDeleteProject(id)         // DELETE por id
```

La app funciona igual con o sin Supabase. Si no hay credenciales configuradas, todo va a localStorage.

---

## Lógica central del cotizador (ya funciona para closets)

### Cómo se estructura un `project.data`

```javascript
{
  projectInfo: {
    name: string,
    client: string,
    type: "closet" | "cocina" | "bano" | ...,
    date: string,
    notes: string
  },
  modules: [
    {
      id: string,           // uuid
      type: string,         // tipo de módulo (ej: "colgador_largo", "cajonera", etc.)
      name: string,         // nombre display
      width: number,        // mm
      height: number,       // mm
      depth: number,        // mm
      quantity: number,
      material: string,     // id del material del casco
      frontMaterial: string,// id del material del frente (sistema dual)
      hardware: [],         // herrajes asignados
      parts: [],            // despiece calculado automáticamente
      price: number         // precio calculado
    }
  ],
  materials: { ... },       // materiales seleccionados del catálogo
  totals: {
    subtotal: number,
    tax: number,
    total: number
  }
}
```

### Sistema Dual Material (Casco + Frente)

La lógica más importante del cotizador. Ya implementada para closets, se debe extender a todos los tipos:
- **Casco**: material económico (melamina blanca/gris) — el interior del mueble
- **Frente**: material premium (el diseño visible que el cliente elige)
- El cotizador calcula y cotiza ambos por separado

### Cálculo de piezas (despiece automático)

Para cada módulo, el sistema calcula automáticamente todas las piezas a partir de las dimensiones del módulo:

```javascript
// Fórmula base que aplica a todos los módulos:
const THICKNESS = 15; // mm, espesor estándar de todos los tableros

// Ancho interior del módulo:
const innerWidth = width - (2 * THICKNESS); // = width - 30mm

// Piezas del casco (siempre iguales para módulo base):
parts = [
  { name: 'Costado', qty: 2, w: depth, h: height },
  { name: 'Piso',    qty: 1, w: innerWidth, h: depth },
  { name: 'Techo',   qty: 1, w: innerWidth, h: depth },
  { name: 'Fondo',   qty: 1, w: innerWidth, h: height, thickness: 3 }, // HDF
]

// Puertas (frente):
const doorQty = width <= 500 ? 1 : 2;
const doorWidth = doorQty === 1 
  ? innerWidth - 4 
  : (innerWidth / 2) - 2;
const doorHeight = height - 10;
parts.push({ name: 'Puerta', qty: doorQty, w: doorWidth, h: doorHeight, material: 'frente' })
```

Las fórmulas específicas por tipo de módulo están en `INSTRUCTIVO_TECNICO.md`.

### Catálogo de materiales (`supplier-catalog-data-REAL.js`)

```javascript
// Estructura del catálogo:
{
  id: "melam_blanca_15",
  name: "Melamina Blanca 15mm",
  thickness: 15,          // mm
  sheetSize: { w: 2440, h: 1220 }, // mm
  pricePerSheet: 450,     // MXN
  category: "casco",      // "casco" | "frente" | "hardware"
  supplier: "Arauco"
}
```

---

## Reglas de negocio importantes

### Precios de herrajes
- Herraje ≤ $500 MXN: se absorbe en el costo de carpintería (multiplicado por factor ×3)
- Herraje ≥ $501 MXN (ej: Blum): precio de lista ÷ 0.70 (margen del 30%)

### Bisagras por puerta según altura
- Puerta ≤ 1,000 mm: 2 bisagras
- Puerta 1,000–2,000 mm: 3 bisagras  
- Puerta > 2,000 mm: 4 bisagras
- +1 bisagra si el material es pesado (>12 kg/m²)

### Correderas de cajón (Blum Tandem)
- Ancho cajón ≤ 500 mm: corredera estándar
- Ancho cajón > 500 mm: Tandem Plus con freno

### Zoclo en cocinas
- 1 ml lineal de zoclo por cada módulo bajo
- Siempre cotizar como pieza separada

---

## UI actual (closets)

La app tiene un flujo de trabajo por pasos (wizard):
1. **Paso 1 — Tipo de proyecto**: seleccionar tipo de mueble
2. **Paso 2 — Configuración**: agregar módulos, seleccionar materiales
3. **Paso 3 — Resumen**: ver despiece, materiales y precios
4. **Paso 4 — PDF**: previsualizar y exportar cotización

El UI filtra automáticamente los módulos disponibles según el tipo de proyecto seleccionado en el Paso 1. Cuando el usuario selecciona "Cocina", **solo ve módulos de cocina**. Cuando selecciona "Clóset", **solo ve módulos de clóset**. Esto ya funciona para closets — se extiende al resto.

---

## Tipos de muebles soportados (objetivo)

| type (en DB) | Display | Estado |
|---|---|---|
| `closet` | Clóset / Vestidor | ✅ Funciona completo |
| `cocina` | Cocina | 🔲 Por construir |
| `bano` | Baño / Lavabo | 🔲 Por construir |
| `tv` | Mueble de TV | 🔲 Por construir |
| `librero` | Librero | 🔲 Por construir |
| `credenza` | Credenza para TV | 🔲 Por construir |
| `mesa_centro` | Mesa de Centro | 🔲 Por construir |
| `puerta` | Puerta de Intercomunicación | 🔲 Por construir |

---

## Referencia técnica de dimensiones y armado

Toda la información técnica de cada tipo de mueble (dimensiones estándar, despiece, herrajes, pasos de armado) está en `INSTRUCTIVO_TECNICO.md`. Claude Code debe leerlo antes de implementar cualquier módulo nuevo.

---

## Convenciones de código

- Todas las medidas en **milímetros** (mm)
- Todos los precios en **pesos mexicanos** (MXN)
- IDs de módulos: UUID generado con `crypto.randomUUID()`
- Los módulos nuevos siguen el mismo patrón que los de closet ya implementados
- No cambiar la estructura de `project.data` — solo extender con nuevos tipos
- No romper la funcionalidad de closets existente
- Mantener todo en el mismo HTML monolítico (no separar archivos sin pedirlo explícitamente)
