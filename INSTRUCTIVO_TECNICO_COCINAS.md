# INSTRUCTIVO TÉCNICO — COCINAS
## Cotizador Pro · Referencia completa para Claude Code
### Basado en estándar IKEA SEKTION + estándar mexicano de carpintería

> **Propósito:** Este archivo define TODA la lógica de módulos de cocina para implementar en `cotizador-template.html`. Claude Code debe leer este archivo completo antes de escribir cualquier código de cocinas. Los closets NO se tocan.

---

## TABLA DE CONTENIDOS

1. [Dimensiones Estándar Globales](#1-dimensiones-estándar-globales)
2. [Catálogo de Módulos](#2-catálogo-de-módulos)
3. [Lógica de Despiece por Módulo](#3-lógica-de-despiece-por-módulo)
4. [Herrajes Automáticos por Módulo](#4-herrajes-automáticos-por-módulo)
5. [Configuración de Toggles por Módulo](#5-configuración-de-toggles-por-módulo)
6. [CONFIG — Flags de activación por cliente](#6-config--flags-de-activación-por-cliente)
7. [Reglas de Negocio y Validaciones](#7-reglas-de-negocio-y-validaciones)
8. [Indicador de Ancho Acumulado](#8-indicador-de-ancho-acumulado)
9. [Fórmulas de Precio](#9-fórmulas-de-precio)
10. [Errores Comunes a Evitar](#10-errores-comunes-a-evitar)

---

## 1. DIMENSIONES ESTÁNDAR GLOBALES

Todas las medidas en **milímetros (mm)**. Estas son las dimensiones default que el sistema pre-carga al agregar un módulo. El usuario puede modificar cualquiera libremente.

### 1.1 Módulos Bajos (van al piso)

| Parámetro | Valor default | Rango permitido | Notas |
|---|---|---|---|
| Alto casco | 720 mm | 650–800 mm | Sin patas ni cubierta |
| Profundidad | 600 mm | 400–700 mm | |
| Alto total (casco + patas + cubierta) | 900 mm | — | Referencia ergonómica, no editable |
| Espesor patas niveladoras | 100–150 mm | — | No genera pieza, es herraje |
| Espesor cubierta | 30 mm (postformado) | — | Variable según tipo |
| Anchos estándar sugeridos | 300, 400, 450, 500, 600, 800, 900 mm | 200–1200 mm | El usuario puede escribir cualquier ancho |

### 1.2 Alacenas / Gabinetes Altos (van a la pared o sobre bajos)

| Parámetro | Valor default | Rango permitido | Notas |
|---|---|---|---|
| Alto | 700 mm | 300–1000 mm | |
| Profundidad | 320 mm | 250–400 mm | |
| Anchos estándar sugeridos | 300, 400, 450, 500, 600, 800, 900 mm | 200–1200 mm | |
| Montaje | pared | pared / sobre_bajos | Toggle en UI |

### 1.3 Torres (piso a techo)

| Parámetro | Valor default | Rango permitido | Notas |
|---|---|---|---|
| Alto | 2200 mm | 1800–2600 mm | |
| Profundidad | 600 mm | 500–700 mm | |
| Anchos estándar sugeridos | 450, 500, 600 mm | 300–800 mm | |

### 1.4 Espesor de tablero

| Uso | Espesor | Material |
|---|---|---|
| Casco (costados, piso, techo, amarres, entrepaños) | 15 mm | Melamina blanca/gris |
| Frentes (puertas, frentes de cajón) | 18 mm | Material premium elegido |
| Fondo trasero | 3 mm | HDF |
| Frentes de cajón (interior) | 15 mm | Melamina blanca |

> **Constante en código:** `const T = 15` (espesor casco), `const TF = 18` (espesor frente), `const THDF = 3` (fondo HDF)

---

## 2. CATÁLOGO DE MÓDULOS

### 2.1 Identificadores y nombres

Cada módulo tiene un `type` en snake_case (usado en DB y lógica) y un `label` en español (mostrado en UI).

| type | label (UI) | Categoría | Enabled default |
|---|---|---|---|
| `bajo_estandar` | Gabinete Bajo Estándar | Bajos | true |
| `bajo_cajones` | Bajo con Cajones | Bajos | true |
| `bajo_fregadero` | Bajo para Fregadero | Bajos | true |
| `bajo_horno` | Bajo para Horno | Bajos | true |
| `bajo_cooktop` | Bajo para Parrilla | Bajos | false |
| `esquinero_bajo` | Esquinero Bajo | Bajos | true |
| `alacena_estandar` | Alacena Estándar | Alacenas | true |
| `alacena_aventos` | Alacena Aventos (abatible) | Alacenas | true |
| `alacena_campana` | Alacena sobre Campana | Alacenas | true |
| `alacena_esquinera` | Alacena Esquinera | Alacenas | false |
| `alacena_profunda` | Alacena Profunda (sobre refri) | Alacenas | false |
| `torre_despensa` | Torre Despensa | Torres | true |
| `torre_hornos` | Torre Hornos + Microondas | Torres | true |
| `torre_microondas` | Torre Microondas | Torres | false |
| `torre_refrigerador` | Torre para Refrigerador | Torres | false |
| `torre_limpieza` | Torre Escobero | Torres | false |
| `cubierta` | Cubierta / Encimera | Extras | true |
| `zoclo` | Zoclo | Extras | true |
| `panel_lateral` | Panel Lateral Vista | Extras | true |
| `panel_relleno` | Panel de Relleno | Extras | false |
| `isla` | Isla | Extras | false |
| `gabinete_remate` | Gabinete de Remate (sobre refri) | Extras | false |

### 2.2 Anchos estándar por categoría

Estos son los valores que aparecen en el dropdown de sugerencias de ancho. El usuario puede escribir cualquier otro valor.

```javascript
const KITCHEN_WIDTH_PRESETS = {
  bajos:    [300, 400, 450, 500, 600, 800, 900],
  alacenas: [300, 400, 450, 500, 600, 800, 900],
  torres:   [450, 500, 600],
  extras:   [] // ancho libre, sin sugerencias
};
```

---

## 3. LÓGICA DE DESPIECE POR MÓDULO

### 3.1 Variables base (aplican a todos)

```javascript
// Inputs del módulo:
const W  = module.width;    // ancho total del módulo (mm)
const H  = module.height;   // alto del casco (mm)
const D  = module.depth;    // profundidad (mm)
const T  = 15;              // espesor tablero casco
const TF = 18;              // espesor frente/puerta
const THDF = 3;             // espesor fondo HDF

// Derivados universales:
const innerWidth  = W - (2 * T);   // ancho interior libre
const innerDepth  = D - T;         // profundidad útil (sin fondo trasero)
const innerHeight = H - (2 * T);   // alto interior libre
```

### 3.2 `bajo_estandar` — Gabinete Bajo Estándar

**Descripción:** Caja base con 1 o 2 puertas, 1 entrepaño interior opcional, amarres superiores. Sin techo de tablero (la cubierta lo reemplaza).

```javascript
function calcBajoEstandar(module) {
  const { W, H, D, T, TF, THDF, innerWidth } = base(module);
  const numPuertas = module.puertas; // 1 o 2, definido por toggle
  const numEntrepanos = module.entrepanos; // 0 o 1, toggle

  const parts = [
    // CASCO
    { name: 'Costado',    qty: 2, largo: D,          ancho: H,           mat: 'casco',  nota: 'Veta vertical' },
    { name: 'Piso',       qty: 1, largo: innerWidth, ancho: D,           mat: 'casco'  },
    // NO lleva techo — la cubierta lo reemplaza
    { name: 'Amarre sup', qty: 2, largo: innerWidth, ancho: 80,          mat: 'casco',  nota: 'Horizontal, adelante y atrás' },
    { name: 'Fondo HDF',  qty: 1, largo: innerWidth, ancho: H,           mat: 'hdf',    nota: '3mm HDF' },
  ];

  // Entrepaño interior (si numEntrepanos > 0)
  for (let i = 0; i < numEntrepanos; i++) {
    parts.push({
      name: `Entrepaño ${i + 1}`, qty: 1,
      largo: innerWidth - 2, ancho: D - 30, mat: 'casco',
      nota: 'Ajustable con pija de repisa'
    });
  }

  // FRENTES — puertas
  const doorW = numPuertas === 1
    ? innerWidth - 4
    : Math.floor((innerWidth / 2)) - 2;
  const doorH = H - 10;

  parts.push({
    name: 'Puerta', qty: numPuertas,
    largo: doorW, ancho: doorH, mat: 'frente',
    nota: numPuertas === 2 ? 'Abisagrada al costado' : ''
  });

  return parts;
}
```

**Regla de puertas sugerida:** `numPuertas = W <= 500 ? 1 : 2`
El usuario puede override con el toggle.

---

### 3.3 `bajo_cajones` — Bajo con Cajones

**Descripción:** Caja base donde el interior es 100% cajones. Sin puertas de hoja. Sin entrepaño.

**Alturas estándar de cajones por configuración:**

| Num cajones | Cajón 1 (superior) | Cajón 2 | Cajón 3 | Cajón 4 |
|---|---|---|---|---|
| 2 cajones | 200 mm | 350 mm | — | — |
| 3 cajones | 150 mm | 200 mm | 270 mm | — |
| 4 cajones | 120 mm | 150 mm | 180 mm | 200 mm |

> Estas alturas son default. El usuario puede editar cada una con `[—] valor [+]`.
> La suma de alturas de cajones + (num_cajones × T) debe ser ≤ innerHeight. Validar en UI.

```javascript
function calcBajoCajones(module) {
  const { W, H, D, T, TF, THDF, innerWidth } = base(module);
  const numCajones = module.cajones; // 2, 3 o 4
  const alturasCajones = module.alturasCajones; // array de alturas, una por cajón

  const parts = [
    // CASCO
    { name: 'Costado',    qty: 2, largo: D,          ancho: H,  mat: 'casco' },
    { name: 'Piso',       qty: 1, largo: innerWidth, ancho: D,  mat: 'casco' },
    { name: 'Amarre sup', qty: 2, largo: innerWidth, ancho: 80, mat: 'casco' },
    { name: 'Fondo HDF',  qty: 1, largo: innerWidth, ancho: H,  mat: 'hdf'   },
  ];

  // Piezas por cada cajón
  alturasCajones.forEach((hCajon, i) => {
    const cajonInnerW = innerWidth - 60; // espacio para correderas Blum (30mm cada lado)
    const cajonDepth  = D - 80;          // profundidad cajón (sin frente y holgura trasera)
    const cajonH      = hCajon - 20;     // alto cuerpo cajón (el frente sobresale)

    parts.push(
      { name: `Cajón ${i+1} — Frente ext`, qty: 1, largo: innerWidth - 4, ancho: hCajon - 4, mat: 'frente' },
      { name: `Cajón ${i+1} — Frente int`, qty: 1, largo: cajonInnerW,    ancho: cajonH,     mat: 'casco'  },
      { name: `Cajón ${i+1} — Costado`,    qty: 2, largo: cajonDepth,     ancho: cajonH,     mat: 'casco'  },
      { name: `Cajón ${i+1} — Trasero`,    qty: 1, largo: cajonInnerW,    ancho: cajonH,     mat: 'casco'  },
      { name: `Cajón ${i+1} — Fondo`,      qty: 1, largo: cajonInnerW,    ancho: cajonDepth, mat: 'hdf', nota: '3mm HDF' }
    );
  });

  return parts;
}
```

---

### 3.4 `bajo_fregadero` — Bajo para Fregadero

**Descripción:** Igual al bajo estándar pero **sin entrepaño** (lo impiden los tubos), 2 puertas siempre, y el módulo tiene una nota de "zona húmeda → usar MDF RH".

```javascript
function calcBajoFregadero(module) {
  const { W, H, D, T, TF, THDF, innerWidth } = base(module);
  // Siempre 2 puertas, sin entrepaño
  const doorW = Math.floor((innerWidth / 2)) - 2;
  const doorH = H - 10;

  return [
    { name: 'Costado',    qty: 2, largo: D,          ancho: H,  mat: 'casco', nota: 'MDF RH recomendado' },
    { name: 'Piso',       qty: 1, largo: innerWidth, ancho: D,  mat: 'casco'  },
    { name: 'Amarre sup', qty: 2, largo: innerWidth, ancho: 80, mat: 'casco'  },
    { name: 'Fondo HDF',  qty: 1, largo: innerWidth, ancho: H,  mat: 'hdf'    },
    { name: 'Puerta',     qty: 2, largo: doorW,      ancho: doorH, mat: 'frente' },
  ];
}
```

---

### 3.5 `bajo_horno` — Bajo para Horno Empotrado

**Descripción:** Módulo con espacio central libre para horno. Opción de 1 cajón arriba o abajo del horno (toggle). Sin entrepaño.

**Medidas del hueco para horno:** El usuario ingresa alto y ancho del hueco. Default: 600 × 600 mm.

```javascript
function calcBajoHorno(module) {
  const { W, H, D, T, TF, THDF, innerWidth } = base(module);
  const hHueco = module.altoHueco || 600; // alto del hueco del horno
  const cajonPos = module.cajonHorno; // 'arriba', 'abajo', 'ninguno'

  const parts = [
    { name: 'Costado',    qty: 2, largo: D,          ancho: H,  mat: 'casco' },
    { name: 'Piso',       qty: 1, largo: innerWidth, ancho: D,  mat: 'casco' },
    { name: 'Amarre sup', qty: 2, largo: innerWidth, ancho: 80, mat: 'casco' },
    { name: 'Fondo HDF',  qty: 1, largo: innerWidth, ancho: H,  mat: 'hdf'   },
    // Tablero separador del hueco del horno
    { name: 'Separador horno', qty: 1, largo: innerWidth, ancho: D - 30, mat: 'casco',
      nota: `Posición: ${cajonPos === 'arriba' ? H - hHueco - T : hHueco + T}mm desde el piso` },
  ];

  // Cajón si aplica
  if (cajonPos !== 'ninguno') {
    const hCajon = H - hHueco - T;
    const cajonInnerW = innerWidth - 60;
    const cajonDepth  = D - 80;
    const cajonH      = hCajon - 20;
    parts.push(
      { name: 'Cajón horno — Frente ext', qty: 1, largo: innerWidth - 4, ancho: hCajon - 4, mat: 'frente' },
      { name: 'Cajón horno — Frente int', qty: 1, largo: cajonInnerW,    ancho: cajonH,     mat: 'casco'  },
      { name: 'Cajón horno — Costado',    qty: 2, largo: cajonDepth,     ancho: cajonH,     mat: 'casco'  },
      { name: 'Cajón horno — Trasero',    qty: 1, largo: cajonInnerW,    ancho: cajonH,     mat: 'casco'  },
      { name: 'Cajón horno — Fondo HDF',  qty: 1, largo: cajonInnerW,    ancho: cajonDepth, mat: 'hdf'    }
    );
  }

  return parts;
}
```

---

### 3.6 `bajo_cooktop` — Bajo para Parrilla/Cubierta de Cocción

**Descripción:** Similar a bajo estándar pero el tablero superior lleva un **recorte** para la parrilla empotrada. El casco es idéntico al bajo estándar. El recorte no cambia el despiece — es una nota técnica en el PDF.

```javascript
// Mismo despiece que bajo_estandar
// Agregar nota: `nota: 'Requiere recorte para parrilla — dimensiones según modelo del cliente'`
// en la pieza Amarre sup o como pieza especial:
{ name: 'Nota recorte parrilla', qty: 1, largo: 0, ancho: 0, mat: 'nota',
  nota: 'Verificar dimensiones del recorte con el cliente antes de cortar' }
```

---

### 3.7 `esquinero_bajo` — Esquinero Bajo

**Descripción:** Módulo de esquina. Tiene 3 variantes elegibles por toggle: `ciego`, `carrusel`, `magic_corner`.

**Dimensiones especiales:**
- Ancho: se ingresan **dos anchos** (eje X y eje Y de la esquina). Default: 900 × 900 mm.
- El casco es una caja en L conceptualmente, pero para el despiece se trabaja como un rectángulo con el eje mayor.

```javascript
function calcEsquineroBajo(module) {
  const Wx = module.widthX || 900;  // ancho eje X
  const Wy = module.widthY || 900;  // ancho eje Y
  const H  = module.height;
  const D  = module.depth;
  const T  = 15;

  // El casco principal usa el eje mayor
  const mainW = Math.max(Wx, Wy);
  const innerW = mainW - (2 * T);

  const parts = [
    { name: 'Costado principal', qty: 2, largo: D,      ancho: H,     mat: 'casco' },
    { name: 'Piso',              qty: 1, largo: innerW, ancho: D,     mat: 'casco' },
    { name: 'Amarre sup',        qty: 2, largo: innerW, ancho: 80,    mat: 'casco' },
    { name: 'Fondo HDF',         qty: 1, largo: innerW, ancho: H,     mat: 'hdf'   },
    { name: 'Panel ciego',       qty: 1, largo: Math.min(Wx, Wy) - T, ancho: H, mat: 'casco',
      nota: 'Cierra el lado corto de la esquina' },
  ];

  // Puertas según variante
  if (module.esquineroTipo === 'ciego') {
    // 1 puerta abatible, bisagra 165°
    parts.push({
      name: 'Puerta esquinera', qty: 1,
      largo: innerW - 4, ancho: H - 10, mat: 'frente',
      nota: 'Bisagra 165° obligatoria'
    });
  } else if (module.esquineroTipo === 'carrusel') {
    // 2 puertas pequeñas (ancho = Wy/2 cada una aprox)
    const dW = Math.floor((Math.min(Wx, Wy) - (2*T)) / 2) - 2;
    parts.push({ name: 'Puerta carrusel', qty: 2, largo: dW, ancho: H - 10, mat: 'frente' });
    // Nota: el carrusel (lazy susan) es herraje, no genera piezas adicionales de tablero
  } else if (module.esquineroTipo === 'magic_corner') {
    // 1 puerta + sistema de canastas extraíbles (herraje premium)
    parts.push({
      name: 'Puerta magic corner', qty: 1,
      largo: innerW - 4, ancho: H - 10, mat: 'frente',
      nota: 'Sistema magic corner — herraje Blum premium'
    });
  }

  return parts;
}
```

---

### 3.8 `alacena_estandar` — Alacena Estándar

**Descripción:** Caja que va a la pared (o sobre bajos). **Sí lleva techo** (a diferencia de los bajos). 1 o 2 puertas, 1 o 2 entrepaños.

```javascript
function calcAlacenaEstandar(module) {
  const { W, H, D, T, TF, THDF, innerWidth } = base(module);
  const numPuertas   = module.puertas;    // 1 o 2
  const numEntrepanos = module.entrepanos; // 0, 1 o 2

  const parts = [
    { name: 'Costado',    qty: 2, largo: D,          ancho: H,  mat: 'casco' },
    { name: 'Piso',       qty: 1, largo: innerWidth, ancho: D,  mat: 'casco' },
    { name: 'Techo',      qty: 1, largo: innerWidth, ancho: D,  mat: 'casco' }, // ← alacenas SÍ llevan techo
    { name: 'Amarre',     qty: 1, largo: innerWidth, ancho: 80, mat: 'casco', nota: 'Centro del módulo' },
    { name: 'Fondo HDF',  qty: 1, largo: innerWidth, ancho: H,  mat: 'hdf'   },
  ];

  for (let i = 0; i < numEntrepanos; i++) {
    parts.push({
      name: `Entrepaño ${i + 1}`, qty: 1,
      largo: innerWidth - 2, ancho: D - 30, mat: 'casco'
    });
  }

  const doorW = numPuertas === 1
    ? innerWidth - 4
    : Math.floor((innerWidth / 2)) - 2;
  const doorH = H - 10;

  parts.push({ name: 'Puerta', qty: numPuertas, largo: doorW, ancho: doorH, mat: 'frente' });

  return parts;
}
```

---

### 3.9 `alacena_aventos` — Alacena con Puertas Abatibles (Blum Aventos)

**Descripción:** Igual que alacena estándar en casco. Las puertas son **horizontales** (se abren hacia arriba). Siempre 1 o 2 puertas horizontales. El sistema Aventos es herraje premium.

```javascript
function calcAlacenaAventos(module) {
  const { W, H, D, T, TF, THDF, innerWidth } = base(module);
  // Casco idéntico a alacena estándar
  const parts = [
    { name: 'Costado',   qty: 2, largo: D,          ancho: H, mat: 'casco' },
    { name: 'Piso',      qty: 1, largo: innerWidth, ancho: D, mat: 'casco' },
    { name: 'Techo',     qty: 1, largo: innerWidth, ancho: D, mat: 'casco' },
    { name: 'Fondo HDF', qty: 1, largo: innerWidth, ancho: H, mat: 'hdf'   },
  ];

  // Puerta horizontal (aventos): 1 o 2 hojas que se dividen verticalmente
  const numHojas = module.aventosHojas || 1; // 1 = puerta completa, 2 = dividida en alto
  const hojaH = numHojas === 1 ? H - 10 : Math.floor((H - 10) / 2) - 2;
  parts.push({
    name: 'Puerta Aventos', qty: numHojas,
    largo: innerWidth - 4, ancho: hojaH,
    mat: 'frente', nota: 'Puertas abatibles — sistema Blum Aventos HF/HS'
  });

  return parts;
}
```

---

### 3.10 `alacena_campana` — Alacena sobre Campana

**Descripción:** Módulo con hueco central para la campana extractora. Dos cuerpos laterales con puertas. El ancho del hueco es configurable (default: ancho campana + 20mm).

```javascript
function calcAlacenaCampana(module) {
  const { W, H, D, T, TF, THDF } = base(module);
  const huecoW = module.anchoCampana || 600; // ancho del hueco central
  const lateralW = Math.floor((W - huecoW) / 2);
  const innerLateral = lateralW - (2 * T);

  // Dos cuerpos laterales idénticos
  const parts = [
    { name: 'Costado exterior', qty: 2, largo: D,             ancho: H, mat: 'casco' },
    { name: 'Panel divisor',    qty: 2, largo: D,             ancho: H, mat: 'casco', nota: 'Separa lateral del hueco' },
    { name: 'Piso lateral',     qty: 2, largo: innerLateral,  ancho: D, mat: 'casco' },
    { name: 'Techo',            qty: 1, largo: W - (2 * T),  ancho: D, mat: 'casco', nota: 'Techo corrido' },
    { name: 'Fondo HDF',        qty: 2, largo: innerLateral,  ancho: H, mat: 'hdf'   },
    { name: 'Puerta lateral',   qty: 2, largo: innerLateral - 4, ancho: H - 10, mat: 'frente' },
  ];

  return parts;
}
```

---

### 3.11 `alacena_esquinera` — Alacena Esquinera

**Descripción:** Similar al esquinero bajo pero para pared. Variantes: `carrusel` o `con_puertas`.

```javascript
function calcAlacenaEsquinera(module) {
  const Wx = module.widthX || 600;
  const Wy = module.widthY || 600;
  const H  = module.height;
  const D  = module.depth;
  const T  = 15;
  const mainW = Math.max(Wx, Wy);
  const innerW = mainW - (2 * T);

  return [
    { name: 'Costado',   qty: 2, largo: D,      ancho: H,     mat: 'casco' },
    { name: 'Piso',      qty: 1, largo: innerW, ancho: D,     mat: 'casco' },
    { name: 'Techo',     qty: 1, largo: innerW, ancho: D,     mat: 'casco' },
    { name: 'Fondo HDF', qty: 1, largo: innerW, ancho: H,     mat: 'hdf'   },
    { name: 'Puerta',    qty: 1, largo: innerW - 4, ancho: H - 10, mat: 'frente',
      nota: 'Bisagra especial 45° para esquinera' },
  ];
}
```

---

### 3.12 `alacena_profunda` — Alacena Profunda (sobre refrigerador)

**Descripción:** Alacena de poca altura (300–400mm) y profundidad igual a los bajos (600mm). Va encima del refrigerador o al final de una línea de bajos.

```javascript
// Mismo cálculo que alacena_estandar
// Valores default diferentes:
//   H = 350 mm
//   D = 600 mm (profunda, igual que bajos)
```

---

### 3.13 `torre_despensa` — Torre Despensa

**Descripción:** Módulo vertical del piso al techo. Puerta completa (o 2 puertas si ancho > 600mm) y múltiples entrepaños. A veces se divide en dos sectores: parte baja con puertas y parte alta con puertas diferentes.

```javascript
function calcTorreDespensa(module) {
  const { W, H, D, T, TF, THDF, innerWidth } = base(module);
  const numEntrepanos = module.entrepanos || 4;
  const numPuertas = W > 600 ? 2 : 1;

  const parts = [
    { name: 'Costado',    qty: 2, largo: D,          ancho: H,  mat: 'casco' },
    { name: 'Piso',       qty: 1, largo: innerWidth, ancho: D,  mat: 'casco' },
    { name: 'Techo',      qty: 1, largo: innerWidth, ancho: D,  mat: 'casco' },
    { name: 'Amarre',     qty: 2, largo: innerWidth, ancho: 80, mat: 'casco' },
    { name: 'Fondo HDF',  qty: 1, largo: innerWidth, ancho: H,  mat: 'hdf'   },
  ];

  for (let i = 0; i < numEntrepanos; i++) {
    parts.push({
      name: `Entrepaño ${i + 1}`, qty: 1,
      largo: innerWidth - 2, ancho: D - 30, mat: 'casco'
    });
  }

  const doorW = numPuertas === 1 ? innerWidth - 4 : Math.floor(innerWidth / 2) - 2;
  // Torres altas: 2 puertas en altura (superior e inferior) si H > 1500mm
  const useDualDoor = H > 1500;
  const doorH = useDualDoor ? Math.floor((H - 10) / 2) - 2 : H - 10;
  const doorQty = numPuertas * (useDualDoor ? 2 : 1);

  parts.push({ name: 'Puerta', qty: doorQty, largo: doorW, ancho: doorH, mat: 'frente' });

  return parts;
}
```

---

### 3.14 `torre_hornos` — Torre para Horno + Microondas

**Descripción:** Torre con dos huecos: uno para microondas (arriba, ~450mm alto) y uno para horno (abajo, ~600mm alto). Entre ellos puede haber cajones. La distribución vertical es configurable.

**Distribución default (de arriba a abajo):**
1. Alacena superior con puertas (resto de altura)
2. Hueco microondas (450mm)
3. Cajones intermedios (150–200mm, 1 a 3 cajones, toggle)
4. Hueco horno (600mm)
5. Cajón inferior (resto de altura)

```javascript
function calcTorreHornos(module) {
  const { W, H, D, T, TF, THDF, innerWidth } = base(module);
  const hMicro  = module.altoMicro  || 450;
  const hHorno  = module.altoHorno  || 600;
  const numCajIntermedios = module.cajonesIntermedios || 1;

  const parts = [
    { name: 'Costado',           qty: 2, largo: D,          ancho: H,  mat: 'casco' },
    { name: 'Piso',              qty: 1, largo: innerWidth, ancho: D,  mat: 'casco' },
    { name: 'Techo',             qty: 1, largo: innerWidth, ancho: D,  mat: 'casco' },
    { name: 'Amarre',            qty: 2, largo: innerWidth, ancho: 80, mat: 'casco' },
    { name: 'Fondo HDF',         qty: 1, largo: innerWidth, ancho: H,  mat: 'hdf'   },
    // Separadores de los huecos
    { name: 'Sep. sup microondas', qty: 1, largo: innerWidth, ancho: D - 30, mat: 'casco', nota: 'Techo del hueco microondas' },
    { name: 'Sep. inf microondas', qty: 1, largo: innerWidth, ancho: D - 30, mat: 'casco', nota: 'Piso del hueco microondas / techo de cajones intermedios' },
    { name: 'Sep. sup horno',      qty: 1, largo: innerWidth, ancho: D - 30, mat: 'casco', nota: 'Techo del hueco horno' },
  ];

  // Cajones intermedios entre microondas y horno
  if (numCajIntermedios > 0) {
    const hCajonInt = Math.floor(150 / numCajIntermedios);
    for (let i = 0; i < numCajIntermedios; i++) {
      const cajonInnerW = innerWidth - 60;
      const cajonDepth  = D - 80;
      const cajonH      = hCajonInt - 20;
      parts.push(
        { name: `Caj.int ${i+1} — Frente ext`, qty: 1, largo: innerWidth - 4, ancho: hCajonInt - 4, mat: 'frente' },
        { name: `Caj.int ${i+1} — Frente int`, qty: 1, largo: cajonInnerW,    ancho: cajonH,        mat: 'casco'  },
        { name: `Caj.int ${i+1} — Costado`,    qty: 2, largo: cajonDepth,     ancho: cajonH,        mat: 'casco'  },
        { name: `Caj.int ${i+1} — Trasero`,    qty: 1, largo: cajonInnerW,    ancho: cajonH,        mat: 'casco'  },
        { name: `Caj.int ${i+1} — Fondo HDF`,  qty: 1, largo: cajonInnerW,    ancho: cajonDepth,    mat: 'hdf'    }
      );
    }
  }

  // Puertas sección superior (por encima del microondas)
  const hSup = H - hMicro - hHorno - (numCajIntermedios > 0 ? 150 : 0) - (4 * T);
  if (hSup > 100) {
    parts.push({ name: 'Puerta superior', qty: 1, largo: innerWidth - 4, ancho: hSup - 10, mat: 'frente' });
  }

  return parts;
}
```

---

### 3.15 `torre_microondas` — Torre solo Microondas

**Descripción:** Torre con un solo hueco para microondas + almacenamiento arriba y abajo.

```javascript
// Mismo patrón que torre_hornos pero con un solo hueco
// H default: 2000mm
// hMicro default: 450mm
// El resto son entrepaños o cajones
```

---

### 3.16 `torre_refrigerador` — Torre para Refrigerador Empotrado

**Descripción:** Carcasa que rodea al refrigerador. Solo genera costados y techo (el refri es el contenido). Opción de gabinete superior de remate.

```javascript
function calcTorreRefrigerador(module) {
  const { W, H, D, T, TF, THDF } = base(module);
  // No lleva frentes — el refrigerador es visible
  return [
    { name: 'Costado',        qty: 2, largo: D, ancho: H, mat: 'casco', nota: 'Marco lateral' },
    { name: 'Techo remate',   qty: 1, largo: W - (2*T), ancho: D, mat: 'casco' },
    { name: 'Fondo HDF',      qty: 1, largo: W - (2*T), ancho: H, mat: 'hdf' },
  ];
}
```

---

### 3.17 `torre_limpieza` — Torre Escobero

**Descripción:** Torre alta con espacio interior libre para escoba/trapeador. Puerta completa. Sin entrepaños.

```javascript
// Igual que torre_despensa con:
// numEntrepanos = 0
// numPuertas = 1 (siempre 1 puerta completa)
// H default = 2000mm
// W default = 450mm
```

---

### 3.18 Extras (no generan casco de tablero)

#### `cubierta` — Encimera

No genera piezas de tablero. Genera una línea de costo basada en m²:

```javascript
const areaCubierta = (module.width / 1000) * (module.depth / 1000); // m²
const costoCubierta = areaCubierta * PRECIOS_CUBIERTA[module.tipoCubierta];

// PRECIOS_CUBIERTA (desde CONFIG, configurables por cliente):
const PRECIOS_CUBIERTA_DEFAULT = {
  postformado: 2500,  // MXN/m²
  cuarzo:      8500,
  granito:     6500,
  acero:       9000,
};
```

#### `zoclo` — Zoclo frontal

```javascript
const mlZoclo = module.metrosLineales; // el usuario ingresa ml
const costoZoclo = mlZoclo * CONFIG.pricing.precioZoclo; // MXN/ml
// Genera 1 pieza de tablero: alto zoclo × ml (para despiece)
const altoZoclo = module.altoZoclo || 120; // mm default
```

#### `panel_lateral` — Panel Lateral Vista

```javascript
// 1 pieza:
{ name: 'Panel lateral', qty: 1, largo: module.depth, ancho: module.height, mat: 'panel_lateral' }
// El material 'panel_lateral' tiene su propio selector (puede ser diferente al frente del proyecto)
```

#### `panel_relleno` — Panel de Relleno

```javascript
// Pieza ciega para ajustar huecos entre módulos y paredes
{ name: 'Panel relleno', qty: 1, largo: module.ancho, ancho: module.height, mat: 'casco' }
```

#### `isla` — Isla

```javascript
// Caja cerrada (sí lleva techo) con acceso desde ambos lados
// Profundidad mínima recomendada: 900mm (cabinetes bajos espalda con espalda)
// Despiece: 2× bajo_estandar espalda con espalda, más panel costado en ambos lados
// El usuario define W, H (720mm default), D (900–1200mm)
```

---

## 4. HERRAJES AUTOMÁTICOS POR MÓDULO

### 4.1 Tabla de herrajes automáticos

La función `assignKitchenHardware(module)` retorna un array de herrajes. Cada herraje tiene:
- `id` — clave del catálogo
- `qty` — cantidad calculada
- `unitCost` — costo unitario (desde catálogo)
- `isPremium` — true si costo > CONFIG.pricing.hardwareThreshold ($500 MXN)

### 4.2 Bisagras por puerta

```javascript
function bisagrasPerDoor(doorHeight, frontMaterial) {
  let count;
  if      (doorHeight <= 1000) count = 2;
  else if (doorHeight <= 2000) count = 3;
  else                         count = 4;

  // +1 si material pesado (vidrio o metal)
  const heavy = ['vidrio', 'metal', 'aluminio'].includes(frontMaterial);
  if (heavy) count += 1;

  return count;
}
// Para cada puerta del módulo, calcular bisagrasPerDoor y acumular
```

### 4.3 Correderas de cajón

```javascript
function correderasCajon(cajonWidth, tipoCorredera) {
  // tipoCorredera: 'blum_tandem' | 'economica' — elegido por usuario en toggle
  if (tipoCorredera === 'blum_tandem') {
    return {
      id: cajonWidth > 500 ? 'blum_tandem_plus' : 'blum_tandem',
      qty: 1, // par de correderas
      isPremium: true
    };
  } else {
    return { id: 'corredera_economica', qty: 1, isPremium: false };
  }
}
```

### 4.4 Patas niveladoras

```javascript
// Solo módulos bajos y torres
// 4 patas por módulo siempre
// Si ancho > 900mm: 6 patas (fila central)
function patasPorModulo(moduleWidth) {
  return moduleWidth > 900 ? 6 : 4;
}
```

### 4.5 Clips de zoclo

```javascript
// 2 clips por cada 300mm de ancho de módulo bajo
function clipsZoclo(moduleWidth) {
  return Math.ceil(moduleWidth / 300) * 2;
}
```

### 4.6 Soportes de alacena (para montaje en pared)

```javascript
// Solo alacenas con montaje === 'pared'
// 2 soportes estándar por módulo
// Si peso estimado > 30kg (alacena profunda o grande): 4 soportes
```

### 4.7 Tabla resumen de herrajes por tipo de módulo

| Módulo | Bisagras | Correderas | Patas | Clips zoclo | Soportes pared | Aventos |
|---|---|---|---|---|---|---|
| bajo_estandar | por puerta y altura | — | 4–6 | sí | — | — |
| bajo_cajones | — | por cajón | 4–6 | sí | — | — |
| bajo_fregadero | por puerta y altura | — | 4–6 | sí | — | — |
| bajo_horno | si lleva cajón | si lleva cajón | 4–6 | sí | — | — |
| bajo_cooktop | por puerta y altura | si lleva cajón | 4–6 | sí | — | — |
| esquinero_bajo | por puerta (165°) | — | 4 | sí | — | — |
| alacena_estandar | por puerta y altura | — | — | — | 2–4 | — |
| alacena_aventos | — | — | — | — | 2 | sí |
| alacena_campana | por puerta y altura | — | — | — | 4 | — |
| alacena_esquinera | 1 bisagra 45° | — | — | — | 2 | — |
| alacena_profunda | por puerta y altura | — | — | — | 4 | — |
| torre_despensa | por puerta y altura | — | 4 | sí | — | — |
| torre_hornos | sup + inf | intermedios | 4 | sí | — | — |
| torre_microondas | sup + inf | si lleva cajón | 4 | sí | — | — |
| torre_refrigerador | — | — | 4 | sí | — | — |
| torre_limpieza | por puerta | — | 4 | sí | — | — |

### 4.8 Herraje especial: Aventos (Blum)

```javascript
// Solo alacena_aventos
// Fuerza del pistón depende del peso de la puerta:
//   puerta ≤ 3kg (hasta W600 × H400) → Aventos HF
//   puerta > 3kg o H > 600mm → Aventos HS (más fuerza)
// isPremium: true siempre
// qty: 1 par por puerta
```

### 4.9 Herraje especial: Carrusel / Magic Corner

```javascript
// Solo esquinero_bajo con tipo 'carrusel' o 'magic_corner'
// Ambos isPremium: true
// qty: 1 por módulo esquinero
```

---

## 5. CONFIGURACIÓN DE TOGGLES POR MÓDULO

Cada tipo de módulo define qué controles muestra en su tarjeta en el Paso 2.

```javascript
const KITCHEN_MODULE_CONTROLS = {

  bajo_estandar: {
    dims:       ['width', 'height', 'depth'],
    widthPreset: 'bajos',
    puertas:    { type: 'button_group', options: [1, 2], auto: (w) => w <= 500 ? 1 : 2 },
    entrepanos: { type: 'stepper', min: 0, max: 3, default: 1 },
    corredera:  false,
    cajones:    false,
  },

  bajo_cajones: {
    dims:       ['width', 'height', 'depth'],
    widthPreset: 'bajos',
    puertas:    false,
    cajones:    { type: 'button_group', options: [2, 3, 4], default: 3 },
    corredera:  { type: 'button_group', options: ['blum_tandem', 'economica'], default: 'blum_tandem',
                  labels: ['Blum Tandem', 'Económica'] },
    entrepanos: false,
  },

  bajo_fregadero: {
    dims:       ['width', 'height', 'depth'],
    widthPreset: 'bajos',
    puertas:    false, // siempre 2, no editable
    entrepanos: false,
    corredera:  false,
    nota:       'Zona húmeda — se recomienda MDF RH en casco',
  },

  bajo_horno: {
    dims:         ['width', 'height', 'depth'],
    widthPreset:  'bajos',
    altoHueco:    { type: 'number', default: 600, label: 'Alto hueco horno (mm)' },
    cajonHorno:   { type: 'button_group', options: ['arriba', 'abajo', 'ninguno'],
                    labels: ['Cajón arriba', 'Cajón abajo', 'Sin cajón'], default: 'abajo' },
    corredera:    { type: 'button_group', options: ['blum_tandem', 'economica'], default: 'blum_tandem',
                    labels: ['Blum Tandem', 'Económica'], showIf: (m) => m.cajonHorno !== 'ninguno' },
  },

  bajo_cooktop: {
    dims:       ['width', 'height', 'depth'],
    widthPreset: 'bajos',
    puertas:    { type: 'button_group', options: [1, 2], auto: (w) => w <= 500 ? 1 : 2 },
    cajones:    { type: 'stepper', min: 0, max: 3, default: 1 },
    corredera:  { type: 'button_group', options: ['blum_tandem', 'economica'], default: 'blum_tandem',
                  labels: ['Blum Tandem', 'Económica'], showIf: (m) => m.cajones > 0 },
  },

  esquinero_bajo: {
    dims:           ['height', 'depth'], // ancho X y Y son campos separados
    widthX:         { type: 'number', default: 900, label: 'Ancho eje X (mm)' },
    widthY:         { type: 'number', default: 900, label: 'Ancho eje Y (mm)' },
    esquineroTipo:  { type: 'button_group', options: ['ciego', 'carrusel', 'magic_corner'],
                      labels: ['Ciego', 'Carrusel', 'Magic Corner'], default: 'ciego' },
  },

  alacena_estandar: {
    dims:       ['width', 'height', 'depth'],
    widthPreset: 'alacenas',
    montaje:    { type: 'button_group', options: ['pared', 'sobre_bajos'],
                  labels: ['A la pared', 'Sobre bajos'], default: 'pared' },
    puertas:    { type: 'button_group', options: [1, 2], auto: (w) => w <= 500 ? 1 : 2 },
    entrepanos: { type: 'stepper', min: 0, max: 4, default: 2 },
  },

  alacena_aventos: {
    dims:         ['width', 'height', 'depth'],
    widthPreset:  'alacenas',
    montaje:      { type: 'button_group', options: ['pared', 'sobre_bajos'], default: 'pared' },
    aventosHojas: { type: 'button_group', options: [1, 2],
                    labels: ['1 hoja completa', '2 hojas divididas'], default: 1 },
    entrepanos:   { type: 'stepper', min: 0, max: 3, default: 1 },
  },

  alacena_campana: {
    dims:          ['width', 'height', 'depth'],
    anchoCampana:  { type: 'number', default: 600, label: 'Ancho hueco campana (mm)' },
    entrepanos:    false,
  },

  alacena_esquinera: {
    dims:          ['height', 'depth'],
    widthX:        { type: 'number', default: 600, label: 'Ancho eje X (mm)' },
    widthY:        { type: 'number', default: 600, label: 'Ancho eje Y (mm)' },
    esquineroTipo: { type: 'button_group', options: ['con_puertas', 'carrusel'], default: 'con_puertas' },
  },

  alacena_profunda: {
    dims:       ['width', 'height', 'depth'],
    widthPreset: 'alacenas',
    montaje:    { type: 'button_group', options: ['pared', 'sobre_bajos'], default: 'pared' },
    puertas:    { type: 'button_group', options: [1, 2], auto: (w) => w <= 500 ? 1 : 2 },
    entrepanos: { type: 'stepper', min: 0, max: 3, default: 1 },
  },

  torre_despensa: {
    dims:       ['width', 'height', 'depth'],
    widthPreset: 'torres',
    entrepanos: { type: 'stepper', min: 2, max: 8, default: 4 },
  },

  torre_hornos: {
    dims:                ['width', 'height', 'depth'],
    widthPreset:         'torres',
    altoMicro:           { type: 'number', default: 450, label: 'Alto hueco microondas (mm)' },
    altoHorno:           { type: 'number', default: 600, label: 'Alto hueco horno (mm)' },
    cajonesIntermedios:  { type: 'stepper', min: 0, max: 3, default: 1 },
    corredera:           { type: 'button_group', options: ['blum_tandem', 'economica'], default: 'blum_tandem',
                           showIf: (m) => m.cajonesIntermedios > 0 },
  },

  torre_microondas: {
    dims:       ['width', 'height', 'depth'],
    widthPreset: 'torres',
    altoMicro:  { type: 'number', default: 450, label: 'Alto hueco microondas (mm)' },
    entrepanos: { type: 'stepper', min: 1, max: 6, default: 3 },
  },

  torre_refrigerador: {
    dims:  ['width', 'height', 'depth'],
    nota:  'Solo genera el marco/carcasa. El refrigerador no se cotiza.',
  },

  torre_limpieza: {
    dims:       ['width', 'height', 'depth'],
    widthPreset: 'torres',
  },

  cubierta: {
    dims:         ['width', 'depth'], // ancho y largo de la cubierta
    tipoCubierta: { type: 'button_group',
                    options: ['postformado', 'cuarzo', 'granito', 'acero'],
                    labels:  ['Postformado', 'Cuarzo', 'Granito', 'Acero inox'],
                    default: 'postformado' },
  },

  zoclo: {
    metrosLineales: { type: 'number', default: 1, label: 'Metros lineales' },
    altoZoclo:      { type: 'number', default: 120, label: 'Alto zoclo (mm)' },
  },

  panel_lateral: {
    dims:     ['height', 'depth'],
    matPanel: { type: 'material_selector', label: 'Material panel' }, // selector independiente
  },

  panel_relleno: {
    dims: ['width', 'height'],
  },

  isla: {
    dims:       ['width', 'height', 'depth'],
    entrepanos: { type: 'stepper', min: 0, max: 4, default: 1 },
    cajones:    { type: 'stepper', min: 0, max: 4, default: 2 },
    corredera:  { type: 'button_group', options: ['blum_tandem', 'economica'], default: 'blum_tandem' },
  },

  gabinete_remate: {
    dims: ['width', 'height', 'depth'],
    nota: 'Gabinete de poca altura. Va sobre refrigerador o al remate de línea.',
  },
};
```

---

## 6. CONFIG — FLAGS DE ACTIVACIÓN POR CLIENTE

En el `CONFIG` de cada cliente (en `cotizador-template.html`):

```javascript
kitchen: {
  // Zoclo: si se incluye automáticamente en bajos o se cotiza aparte
  zocloMode: 'configurable', // 'auto' | 'extra' | 'configurable'
  // Si zocloMode === 'auto': se suma 1ml por módulo bajo automáticamente
  // Si zocloMode === 'extra': el usuario agrega módulo zoclo manualmente
  // Si zocloMode === 'configurable': toggle en cada módulo bajo ('¿incluir zoclo?')

  // Cubierta: si la carpintería la cotiza o el cliente la compra
  cubiertas: true, // false = el módulo cubierta no aparece

  // Precios de cubierta (MXN/m²)
  preciosCubierta: {
    postformado: 2500,
    cuarzo:      8500,
    granito:     6500,
    acero:       9000,
  },

  // Precio de zoclo (MXN/ml)
  precioZoclo: 350,

  // Módulos activos/inactivos
  modules: {
    bajo_estandar:       true,
    bajo_cajones:        true,
    bajo_fregadero:      true,
    bajo_horno:          true,
    bajo_cooktop:        false,
    esquinero_bajo:      true,
    alacena_estandar:    true,
    alacena_aventos:     true,
    alacena_campana:     true,
    alacena_esquinera:   false,
    alacena_profunda:    false,
    torre_despensa:      true,
    torre_hornos:        true,
    torre_microondas:    false,
    torre_refrigerador:  false,
    torre_limpieza:      false,
    cubierta:            true,
    zoclo:               true,
    panel_lateral:       true,
    panel_relleno:       false,
    isla:                false,
    gabinete_remate:     false,
  }
}
```

---

## 7. REGLAS DE NEGOCIO Y VALIDACIONES

### 7.1 Validaciones que el UI debe hacer en tiempo real

| Validación | Cuándo | Mensaje |
|---|---|---|
| Ancho mínimo de módulo | Al cambiar width | "Ancho mínimo: 200mm" |
| Cajones: suma de alturas | Al cambiar alturas | "Las alturas de cajones superan el alto interior (Xmm)" |
| Torre: alto mínimo | Al cambiar height | "Torres mínimo 1800mm" |
| Hueco horno + cajón | En torre_hornos | "La suma de huecos y cajones supera el alto total" |
| Módulo de esquina: ambos anchos requeridos | En esquineros | "Ingresa el ancho en ambos ejes" |

### 7.2 Reglas de material

- Los módulos en zona húmeda (`bajo_fregadero`, `bajo_horno` si está junto al fregadero) muestran una advertencia si el material casco no es MDF RH.
- El `panel_lateral` tiene su propio selector de material, independiente del frente del proyecto. Default: igual al frente.
- La cubierta no usa material de tablero — tiene su propio tipo y precio por m².

### 7.3 Frente del proyecto

- Todo el proyecto de cocina comparte **un solo material de frente** por default.
- Se selecciona en la configuración del Paso 2 (encabezado del proyecto, no en cada módulo).
- El usuario puede hacer override por módulo individual si necesita mezclar (checkbox "diferente al proyecto" en la tarjeta del módulo).

### 7.4 Nota de ventilación en torres con horno

```
Si moduleType === 'torre_hornos' || 'torre_microondas':
  Mostrar nota: "⚠️ Verificar saque trasero de ventilación (mín. 100mm) antes de instalar"
```

---

## 8. INDICADOR DE ANCHO ACUMULADO

En el Paso 2, mostrar una barra/indicador debajo de la lista de módulos:

```
Ancho total de bajos:    [████████░░] 2,340 mm / 3,000 mm  (destino ingresado)
Ancho total de alacenas: [██████░░░░] 1,800 mm / 3,000 mm
```

**Implementación:**
- El usuario ingresa el "ancho disponible del espacio" al inicio del Paso 2 (campo opcional: "Ancho del muro (mm)").
- Si no lo ingresa, mostrar solo el acumulado sin barra de progreso.
- Los bajos y las alacenas se acumulan por separado (pueden tener diferente distribución).
- Las torres NO se suman al acumulado de bajos (van en posición fija).
- Si el acumulado supera el ancho del muro: mostrar indicador en rojo con mensaje "⚠️ Excede el ancho disponible por Xmm".
- Si el acumulado está entre 95% y 100%: mostrar en amarillo.
- Si es ≤ 95%: mostrar en verde.

---

## 9. FÓRMULAS DE PRECIO

### 9.1 Cálculo de láminas necesarias

```javascript
const SHEET_W = 2440; // mm
const SHEET_H = 1220; // mm
const WASTE   = 1.20; // 20% desperdicio

function sheetsNeeded(parts, mat) {
  const totalArea = parts
    .filter(p => p.mat === mat)
    .reduce((sum, p) => sum + (p.largo * p.ancho * p.qty), 0); // mm²

  const sheetArea = SHEET_W * SHEET_H; // mm²
  return Math.ceil((totalArea * WASTE) / sheetArea);
}
```

### 9.2 Metros lineales de chapacanto

```javascript
function mlChapacanto(parts, mat) {
  // El chapacanto va en los cantos visibles de cada pieza
  // Regla simplificada: perímetro de piezas que son frente/casco visible
  return parts
    .filter(p => p.mat === mat && p.mat !== 'hdf')
    .reduce((sum, p) => {
      const perimeter = 2 * ((p.largo + p.ancho) / 1000); // metros
      return sum + (perimeter * p.qty);
    }, 0);
}
```

### 9.3 Precio carpintería

Igual que closets. Sin cambios.

```
carpintería = (costLáminas + costCorte + costChapacanto + costManoObra + costFlete) × 3
```

### 9.4 Precio extras de cocina

```
cubierta  = m² × precioPorTipo × 4
zoclo     = ml × precioZoclo × 4 (si se cotiza aparte)
LED       = si aplica, igual que closets
```

### 9.5 Precio herrajes

Igual que closets:
- Herraje unitario ≤ $500 MXN → absorbido en carpintería (ya incluido en el ×3)
- Herraje unitario > $500 MXN → `precio = costo / 0.70`

---

## 10. ERRORES COMUNES A EVITAR EN IMPLEMENTACIÓN

| Error | Consecuencia | Solución |
|---|---|---|
| Usar `H - 10` para altura de puerta en alacenas que llevan techo | Puerta demasiado pequeña si techo suma al cálculo | Verificar que `H` ya es el alto del casco (sin techo contado dos veces) |
| Calcular `innerWidth` sin restar ambos costados | Piezas interiores quedan 30mm grandes | `innerWidth = W - (2 * T)` siempre |
| Poner techo en módulos bajos | Los bajos no llevan techo — la cubierta lo reemplaza | Solo alacenas y torres llevan techo de tablero |
| No separar frente externo de frente interno en cajones | Despiece incorrecto | Cajón tiene: frente externo (frente premium, se ve), frente interno (melamina, estructura del cajón) |
| Bisagra estándar en esquinero | La puerta choca | Usar bisagra 165° en todos los esquineros tipo `ciego` |
| No agregar Aventos como herraje premium | Precio incorrecto | `alacena_aventos` siempre tiene Blum Aventos como herraje isPremium:true |
| Torre de hornos sin validar suma de huecos | La torre no cierra matemáticamente | `H >= hMicro + hHorno + (cajones×alturaCajon) + (separadores×T) + 2T` |
| Calcular cubierta con piezas de tablero | La cubierta no es tablero — es precio por m² | Cubierta va a `extras`, no a `parts` |
| Acumular torres en el indicador de ancho de bajos | El indicador muestra ancho erróneo | Torres se acumulan separado o no se acumulan |

---

*INSTRUCTIVO_TECNICO_COCINAS.md — Cotizador Pro*
*Basado en IKEA SEKTION + estándar mexicano de carpintería*
*Todas las medidas en mm · Todos los precios en MXN*
