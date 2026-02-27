# TASKS.md — Plan de Desarrollo Cotizador Pro
> Para usar con Claude Code + plugin GetShitDone.
> Leer CONTEXT.md, INSTRUCTIVO_TECNICO.md e INSTRUCTIVO_TECNICO_COCINAS.md antes de empezar cualquier tarea.
> No romper closets. No separar archivos. Todo en cotizador-template.html salvo que se indique.

---

## BLOQUE 0 — Fundación (hacer primero, todo lo demás depende de esto)

- [x] **0.1** Leer `CONTEXT.md`, `INSTRUCTIVO_TECNICO.md` e `INSTRUCTIVO_TECNICO_COCINAS.md` completos antes de empezar.

- [x] **0.2** Auditar el código de closets en `cotizador-template.html`:
  - Identificar dónde están definidos los módulos de closet (objeto/array con los tipos)
  - Identificar la función de despiece automático (donde se calculan las `parts`)
  - Identificar el sistema de filtrado de UI por tipo de mueble
  - Identificar la función de asignación automática de herrajes
  - Documentar los hallazgos en comentarios en el código o en un archivo `AUDIT.md`

- [x] **0.3** Crear la función genérica `calculateParts(moduleType, dimensions, material)` que:
  - Recibe tipo de módulo, dimensiones (w, h, d) y material
  - Aplica la fórmula base: `innerWidth = width - 30` (2 × espesor 15mm)
  - Retorna array de piezas `[{ name, qty, largo, ancho, thickness, mat, nota }]`
  - Es el núcleo que todos los módulos nuevos usarán
  - **Referencia:** Sección 3.1 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **0.4** Crear la función `assignHardware(moduleType, dimensions, parts, config)` que:
  - Asigna automáticamente herrajes según tipo y dimensiones
  - Regla bisagras: altura ≤ 1000mm → 2; 1000–2000mm → 3; >2000mm → 4; +1 si vidrio/metal
  - Regla correderas: el usuario elige entre Blum Tandem o económica (toggle)
  - Regla patas: módulos bajos y torres → 4 patas; ancho > 900mm → 6 patas
  - Regla precio: herraje ≤ $500 MXN → absorbido en ×3; >$500 → precio = costo / 0.70
  - **Referencia:** Sección 4 de INSTRUCTIVO_TECNICO_COCINAS.md

---

## BLOQUE 1 — Cocinas (prioridad máxima)

> Leer INSTRUCTIVO_TECNICO_COCINAS.md completo antes de este bloque.
> No tocar nada de closets. Todo código nuevo va en secciones claramente marcadas con `// COCINAS`.

---

### 1A — Catálogo de módulos y CONFIG

- [x] **1A.1** Agregar al CONFIG el objeto `kitchen` con todos los flags de activación por cliente:
  ```javascript
  kitchen: {
    zocloMode: 'configurable', // 'auto' | 'extra' | 'configurable'
    cubiertas: true,
    preciosCubierta: { postformado: 2500, cuarzo: 8500, granito: 6500, acero: 9000 },
    precioZoclo: 350,
    modules: {
      bajo_estandar: true, bajo_cajones: true, bajo_fregadero: true,
      bajo_horno: true, bajo_cooktop: false, esquinero_bajo: true,
      alacena_estandar: true, alacena_aventos: true, alacena_campana: true,
      alacena_esquinera: false, alacena_profunda: false,
      torre_despensa: true, torre_hornos: true, torre_microondas: false,
      torre_refrigerador: false, torre_limpieza: false,
      cubierta: true, zoclo: true, panel_lateral: true,
      panel_relleno: false, isla: false, gabinete_remate: false,
    }
  }
  ```
  **Referencia:** Sección 6 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1A.2** Definir el objeto `KITCHEN_MODS` con los 22 módulos, sus dimensiones default, categoría y label:
  - Cada entrada: `{ type, label, category, defaultW, defaultH, defaultD, enabledKey }`
  - Categorías: `'bajos'`, `'alacenas'`, `'torres'`, `'extras'`
  - El UI solo muestra los módulos donde `CONFIG.kitchen.modules[enabledKey] === true`
  - **Referencia:** Sección 2.1 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1A.3** Definir los anchos estándar sugeridos por categoría:
  ```javascript
  const KITCHEN_WIDTH_PRESETS = {
    bajos:    [300, 400, 450, 500, 600, 800, 900],
    alacenas: [300, 400, 450, 500, 600, 800, 900],
    torres:   [450, 500, 600],
    extras:   []
  };
  ```
  Estos se usan como dropdown de sugerencias en el campo de ancho. El usuario puede escribir cualquier otro valor.

---

### 1B — Despiece de módulos bajos

> Constantes globales para todo el bloque: `T=15`, `TF=18`, `THDF=3`
> Fórmula base: `innerWidth = W - (2 * T)` — aplicar en TODOS los módulos sin excepción.
> Los módulos bajos NO llevan techo de tablero — la cubierta lo reemplaza.

- [x] **1B.1** Implementar `calcBajoEstandar(module)`:
  - Piezas casco: 2× Costado (D×H), 1× Piso (innerWidth×D), 2× Amarre sup (innerWidth×80), 1× Fondo HDF (innerWidth×H)
  - Puertas: si W ≤ 500 → 1 puerta; si W > 500 → 2 puertas (sugerencia, el usuario puede cambiar con toggle)
  - doorW (1 puerta) = innerWidth - 4 | doorW (2 puertas) = floor(innerWidth/2) - 2
  - doorH = H - 10
  - Entrepaños: stepper 0–3, default 1 | medidas: (innerWidth-2) × (D-30)
  - **Referencia:** Sección 3.2 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1B.2** Implementar `calcBajoCajones(module)`:
  - Piezas casco: 2× Costado, 1× Piso, 2× Amarre, 1× Fondo HDF (sin techo)
  - Por cada cajón: 1× Frente ext (frente premium), 1× Frente int (casco), 2× Costado cajón, 1× Trasero, 1× Fondo HDF
  - Medidas cajón: `cajonInnerW = innerWidth - 60` | `cajonDepth = D - 80` | `cajonH = alturaCajon - 20`
  - Alturas default: 2 cajones → [200, 350] | 3 cajones → [150, 200, 270] | 4 cajones → [120, 150, 180, 200]
  - Validación: suma de alturas + (numCajones × T) ≤ H - (2×T)
  - **Referencia:** Sección 3.3 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1B.3** Implementar `calcBajoFregadero(module)`:
  - Idéntico a bajo estándar pero: sin entrepaño, siempre 2 puertas, nota "Zona húmeda — MDF RH recomendado"
  - **Referencia:** Sección 3.4 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1B.4** Implementar `calcBajoHorno(module)`:
  - Casco base + 1× Separador horno (innerWidth × (D-30)) en posición calculada
  - Toggle cajonHorno: 'arriba' | 'abajo' | 'ninguno'
  - Si tiene cajón: mismas piezas de cajón que bajo_cajones (1 cajón)
  - altoHueco default: 600mm — editable por el usuario
  - **Referencia:** Sección 3.5 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1B.5** Implementar `calcBajoCooktop(module)`:
  - Mismo despiece que bajo_estandar
  - Agregar pieza especial de nota: "Requiere recorte para parrilla — verificar dimensiones del modelo"
  - **Referencia:** Sección 3.6 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1B.6** Implementar `calcEsquineroBajo(module)`:
  - Inputs: widthX, widthY (dos anchos separados), H, D
  - Casco: usa el eje mayor como ancho principal
  - Piezas: 2× Costado, 1× Piso, 2× Amarre, 1× Fondo HDF, 1× Panel ciego
  - Toggle esquineroTipo: 'ciego' | 'carrusel' | 'magic_corner'
  - Puertas según tipo: ciego→1 puerta bisagra 165° | carrusel→2 puertas pequeñas | magic_corner→1 puerta
  - **Referencia:** Sección 3.7 de INSTRUCTIVO_TECNICO_COCINAS.md

---

### 1C — Despiece de alacenas

> Las alacenas SÍ llevan techo de tablero (a diferencia de los bajos).

- [x] **1C.1** Implementar `calcAlacenaEstandar(module)`:
  - Piezas: 2× Costado (D×H), 1× Piso (innerWidth×D), 1× Techo (innerWidth×D), 1× Amarre (innerWidth×80), 1× Fondo HDF
  - Puertas: misma lógica que bajos (1 si W≤500, 2 si W>500)
  - Entrepaños: stepper 0–4, default 2
  - Toggle montaje: 'pared' | 'sobre_bajos'
  - **Referencia:** Sección 3.8 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1C.2** Implementar `calcAlacenaAventos(module)`:
  - Casco idéntico a alacena_estandar
  - Puertas horizontales (abatibles hacia arriba)
  - Toggle aventosHojas: 1 (puerta completa) | 2 (dividida en altura)
  - Si 2 hojas: `hojaH = floor((H-10)/2) - 2`
  - Herraje Aventos siempre isPremium: true
  - **Referencia:** Sección 3.9 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1C.3** Implementar `calcAlacenaCampana(module)`:
  - Input adicional: anchoCampana (default 600mm)
  - lateralW = floor((W - anchoCampana) / 2)
  - Piezas: 2× Costado ext, 2× Panel divisor, 2× Piso lateral, 1× Techo corrido, 2× Fondo HDF, 2× Puerta lateral
  - **Referencia:** Sección 3.10 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1C.4** Implementar `calcAlacenaEsquinera(module)`:
  - Inputs: widthX, widthY, H, D
  - Toggle esquineroTipo: 'con_puertas' | 'carrusel'
  - Bisagra especial 45° para tipo con_puertas
  - **Referencia:** Sección 3.11 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1C.5** Implementar `calcAlacenaProfunda(module)`:
  - Mismo cálculo que alacena_estandar
  - Defaults distintos: H=350mm, D=600mm (profundidad igual a bajos)
  - **Referencia:** Sección 3.12 de INSTRUCTIVO_TECNICO_COCINAS.md

---

### 1D — Despiece de torres

- [x] **1D.1** Implementar `calcTorreDespensa(module)`:
  - Piezas: 2× Costado, 1× Piso, 1× Techo, 2× Amarre, 1× Fondo HDF
  - Entrepaños: stepper 2–8, default 4
  - Puertas: si W>600 → 2 puertas; si H>1500 → puertas dobles en altura (4 puertas total)
  - `doorH = useDualDoor ? floor((H-10)/2)-2 : H-10`
  - **Referencia:** Sección 3.13 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1D.2** Implementar `calcTorreHornos(module)`:
  - Piezas base: 2× Costado, 1× Piso, 1× Techo, 2× Amarre, 1× Fondo HDF
  - Separadores: 1× Sep.sup.microondas, 1× Sep.inf.microondas, 1× Sep.sup.horno
  - Inputs: altoMicro (default 450), altoHorno (default 600)
  - Toggle cajonesIntermedios: stepper 0–3, default 1
  - Si cajones > 0: mismas piezas de cajón que bajo_cajones
  - Puerta superior si queda espacio (hSup > 100mm)
  - **Validación crítica:** `H >= altoMicro + altoHorno + (cajones×150) + (separadores×T) + 2T`
  - Nota obligatoria: "⚠️ Saque trasero 100mm para ventilación — obligatorio"
  - **Referencia:** Sección 3.14 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1D.3** Implementar `calcTorreMicroondas(module)`:
  - Similar a torre_hornos pero con un solo hueco
  - altoMicro default: 450mm
  - El resto: entrepaños arriba y cajones abajo
  - **Referencia:** Sección 3.15 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1D.4** Implementar `calcTorreRefrigerador(module)`:
  - Solo genera marco: 2× Costado, 1× Techo, 1× Fondo HDF
  - No genera frentes (el refri es visible)
  - Nota: "Solo genera el marco/carcasa. El refrigerador no se cotiza."
  - **Referencia:** Sección 3.16 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1D.5** Implementar `calcTorreLimpieza(module)`:
  - Igual que torre_despensa con: numEntrepanos=0, numPuertas=1, W default=450mm
  - **Referencia:** Sección 3.17 de INSTRUCTIVO_TECNICO_COCINAS.md

---

### 1E — Extras de cocina

- [x] **1E.1** Implementar módulo `cubierta`:
  - No genera piezas de tablero — genera línea de costo por m²
  - `area = (W/1000) * (D/1000)` en m²
  - Toggle tipoCubierta: 'postformado' | 'cuarzo' | 'granito' | 'acero'
  - `costo = area × CONFIG.kitchen.preciosCubierta[tipo]`
  - Multiplicador extras: ×4 (igual que otros extras)
  - **Referencia:** Sección 3.18 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1E.2** Implementar módulo `zoclo`:
  - El usuario ingresa metros lineales y alto del zoclo (default 120mm)
  - Genera 1 pieza de tablero: (ml×1000) × altoZoclo para el despiece
  - `costo = ml × CONFIG.kitchen.precioZoclo`
  - Si CONFIG.kitchen.zocloMode === 'auto': calcular ml automáticamente sumando anchos de módulos bajos

- [x] **1E.3** Implementar módulo `panel_lateral`:
  - 1 pieza: D × H, material: selector independiente (puede ser diferente al frente del proyecto)
  - Default material: igual al frente del proyecto

- [x] **1E.4** Implementar módulos `panel_relleno`, `isla` y `gabinete_remate`:
  - panel_relleno: 1 pieza W×H, material casco
  - isla: combinación de 2 bajos espalda con espalda (usar calcBajoEstandar × 2 con D compartida)
  - gabinete_remate: igual que alacena_profunda, se ubica sobre refri o al remate de línea
  - **Referencia:** Sección 3.18 de INSTRUCTIVO_TECNICO_COCINAS.md

---

### 1F — Herrajes automáticos

- [x] **1F.1** Implementar `assignKitchenHardware(module)`:
  - Usar tabla de herrajes por módulo de Sección 4.7 de INSTRUCTIVO_TECNICO_COCINAS.md
  - Cada herraje retorna: `{ id, qty, unitCost, isPremium }`
  - isPremium: `unitCost > CONFIG.pricing.hardwareThreshold` (default $500 MXN)

- [x] **1F.2** Implementar función `bisagrasPerDoor(doorHeight, frontMaterial)`:
  - ≤1000mm → 2 | 1001–2000mm → 3 | >2000mm → 4
  - +1 si material es 'vidrio', 'metal' o 'aluminio'
  - Bisagra especial 165° para esquineros tipo 'ciego' y 'con_puertas'

- [x] **1F.3** Implementar lógica de correderas:
  - Toggle por módulo: 'blum_tandem' | 'economica'
  - Si blum_tandem y cajonWidth > 500mm → usar 'blum_tandem_plus'
  - Blum siempre isPremium: true

- [x] **1F.4** Implementar lógica de patas:
  - Solo bajos y torres: `qty = W > 900 ? 6 : 4`
  - Clips de zoclo: `qty = ceil(W / 300) * 2`

- [x] **1F.5** Implementar herrajes especiales:
  - Aventos (alacena_aventos): 1 par por puerta, isPremium: true, modelo según peso estimado de puerta
  - Carrusel lazy susan (esquinero tipo 'carrusel'): 1 pz, isPremium: true
  - Magic corner (esquinero tipo 'magic_corner'): 1 pz, isPremium: true

---

### 1G — UI del Paso 2 para cocinas

- [x] **1G.1** Agregar `"cocina"` al selector de tipo de proyecto en el Paso 1 del wizard.

- [x] **1G.2** Cuando `type === 'cocina'`, mostrar en el Paso 2:
  - Campo "Ancho del muro (mm)" — opcional, para el indicador de ancho acumulado
  - Selector de material frente del proyecto (1 solo para todo el proyecto, con opción de override por módulo)
  - Selector de material casco del proyecto
  - Grupos de módulos filtrados por CONFIG.kitchen.modules

- [x] **1G.3** Agrupar módulos visualmente en 4 secciones con header de categoría:
  - 🔲 Bajos | 🔼 Alacenas | 🗼 Torres | ➕ Extras
  - Solo mostrar las secciones que tienen al menos 1 módulo enabled en CONFIG

- [x] **1G.4** Implementar la tarjeta de módulo de cocina con todos sus toggles:
  - Campos de dimensiones (W, H, D) con input numérico y dropdown de anchos sugeridos
  - Toggles según `KITCHEN_MODULE_CONTROLS[moduleType]` de Sección 5 de INSTRUCTIVO_TECNICO_COCINAS.md
  - Toggle de corredera (Blum / económica) visible solo en módulos con cajones
  - Toggle de montaje (pared / sobre bajos) solo en alacenas
  - Override de material frente: checkbox "¿Frente diferente al proyecto?" — al marcar, mostrar selector
  - **Referencia:** Sección 5 completa de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1G.5** Implementar el indicador de ancho acumulado:
  - Mostrar debajo de la lista de módulos
  - Acumular bajos y alacenas por separado
  - Torres NO se acumulan con bajos
  - Si el usuario ingresó ancho del muro: mostrar barra de progreso con colores
    - ≤95% del muro → verde ✅
    - 95–100% → amarillo ⚠️
    - >100% → rojo ❌ "Excede el ancho disponible por Xmm"
  - Si no ingresó ancho del muro: mostrar solo el total acumulado en mm
  - **Referencia:** Sección 8 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1G.6** Implementar validaciones en tiempo real:
  - Ancho mínimo 200mm — mostrar error inline
  - Torres mínimo 1800mm de alto
  - Suma de cajones no supera alto interior
  - Suma de huecos en torre_hornos no supera alto total
  - Esquineros: widthX y widthY ambos requeridos
  - **Referencia:** Sección 7.1 de INSTRUCTIVO_TECNICO_COCINAS.md

- [x] **1G.7** Mostrar advertencias (no bloqueantes) en casos especiales:
  - Bajo fregadero o bajo horno con material que no es MDF RH: "⚠️ Zona húmeda — considera MDF RH"
  - Torre hornos/microondas: "⚠️ Verificar saque de ventilación 100mm antes de instalar"

---

### 1H — Paso 3: Resumen de costos de cocinas

- [x] **1H.1** Agregar al resumen de cocinas las líneas específicas:
  - Subtotal carpintería (casco + frente + corte + chapacanto + mano de obra + flete) × 3
  - Subtotal herrajes premium (Blum, Aventos, etc.) — precio = costo / 0.70
  - Subtotal extras (cubierta, zoclo si aplica) × 4
  - Total antes de IVA
  - IVA 16%
  - Total con IVA

- [x] **1H.2** Mostrar resumen de materiales:
  - Láminas de casco necesarias (qty por material y espesor)
  - Láminas de frente necesarias
  - Metros lineales de chapacanto (casco y frente por separado)
  - Fórmula: `sheetsNeeded(parts, mat)` con factor de desperdicio 20%
  - **Referencia:** Sección 9.1 de INSTRUCTIVO_TECNICO_COCINAS.md

---

### 1I — PDF de cocinas

- [x] **1I.1** PDF cliente (minimal — igual que closets):
  - Portada oscura con logo, título "COTIZACIÓN DE COCINA", nombre del proyecto y cliente
  - Descripción narrativa del proyecto
  - Precio: subtotal → IVA → total
  - Términos legales (desde CONFIG.brand.legalTerms)
  - Sin detalle de módulos ni despiece

- [x] **1I.2** PDF interno (despiece completo):
  - Tabla de módulos: tipo, dimensiones, material casco, material frente, precio unitario
  - Por cada módulo: tabla de piezas con nombre, cantidad, largo, ancho, material
  - Tabla de herrajes: herraje, cantidad, costo unitario, subtotal
  - Tabla de extras: cubierta, zoclo, paneles — con m² o ml y precio
  - Resumen de materiales (láminas necesarias)
  - Totales desglosados

---

## BLOQUE 2 — Baños

> Leer Sección 4 de INSTRUCTIVO_TECNICO.md antes de este bloque.

- [ ] **2.1** Definir módulos de baño:
  ```javascript
  BANO_MODULES = {
    vanity_1_puerta:  { label: 'Vanity 1 Puerta',   defaultW: 600, defaultH: 800, defaultD: 450 },
    vanity_2_puertas: { label: 'Vanity 2 Puertas',  defaultW: 900, defaultH: 800, defaultD: 450 },
    vanity_cajones:   { label: 'Vanity con Cajones', defaultW: 600, defaultH: 800, defaultD: 450 },
    espejo_caja:      { label: 'Botiquín / Espejo',  defaultW: 600, defaultH: 700, defaultD: 120 },
    torre_bano:       { label: 'Torre de Baño',      defaultW: 350, defaultH: 2100, defaultD: 350 },
  }
  ```

- [ ] **2.2** Despiece para módulos de baño:
  - Material casco obligatorio: MDF RH (mostrar advertencia si se selecciona melamina normal)
  - Montaje: flotante en muro (escuadras) — sin patas
  - Saque en fondo HDF para tuberías: centrado, U 200×150mm en el frente del cajón superior

- [ ] **2.3** Herrajes de baño: escuadras de muro (qty según peso estimado)

- [ ] **2.4** Agregar `"bano"` al selector y filtrado de UI

---

## BLOQUE 3 — Mueble de TV / Centro de Entretenimiento

> Leer Sección 5 de INSTRUCTIVO_TECNICO.md antes de este bloque.

- [ ] **3.1** Definir módulos de TV:
  ```javascript
  TV_MODULES = {
    consola_patas:    { label: 'Consola con Patas',  defaultW: 1600, defaultH: 400, defaultD: 400 },
    consola_flotante: { label: 'Consola Flotante',   defaultW: 1600, defaultH: 400, defaultD: 400 },
    torre_tv:         { label: 'Torre Lateral TV',   defaultW: 350,  defaultH: 2100, defaultD: 300 },
    repisa_flotante:  { label: 'Repisa Flotante',    defaultW: 800,  defaultH: 40,   defaultD: 300 },
    panel_fondo:      { label: 'Panel de Fondo',     defaultW: 1800, defaultH: 2400, defaultD: 30  },
  }
  ```

- [ ] **3.2** Despiece para `consola_patas` y `consola_flotante`

- [ ] **3.3** Despiece para `repisa_flotante`:
  - Opción A: repisa hueca (cajita) — calcular piezas de la caja
  - Opción B: herraje oculto — cotizar herraje + tablero sólido ≥38mm

- [ ] **3.4** Agregar `"tv"` al selector y filtrado de UI

---

## BLOQUE 4 — Libreros y Muebles Tablered

> Leer Secciones 8, 9 y 10 de INSTRUCTIVO_TECNICO.md antes de este bloque.

- [ ] **4.1** Agregar módulo `librero` con el despiece exacto de Tablered Arauco (9 piezas A–I)
- [ ] **4.2** Agregar módulo `credenza` con el despiece exacto de Tablered Arauco (7 piezas A–G)
- [ ] **4.3** Agregar módulo `mesa_centro` con el despiece de 2 bloques (KIRA + NERO)
- [ ] **4.4** Crear tipos de proyecto `"librero"`, `"credenza"`, `"mesa_centro"` con filtrado UI

---

## BLOQUE 5 — Clósets (extensión de lo existente)

> Solo mejoras sobre lo que ya funciona. NO cambiar la lógica existente.

- [ ] **5.1** Auditar módulos de closet actuales y documentar cuáles están implementados
- [ ] **5.2** Agregar módulos faltantes si los hay (colgador_largo, zapatero, maletero, torre_entrepaños)
- [ ] **5.3** Agregar validación anti-vuelco para muebles > 1500mm de alto

---

## BLOQUE 6 — Puertas de Intercomunicación

> Leer Sección 7 de INSTRUCTIVO_TECNICO.md antes de este bloque.

- [ ] **6.1** Definir módulo de puerta:
  ```javascript
  PUERTA_MODULE = {
    puerta_std: { label: 'Puerta Interior', defaultW: 900, defaultH: 2100, defaultD: 45 }
  }
  ```

- [ ] **6.2** Despiece: bastidor pino + 2 tapas MDF + marco chambrana + tope + bisagras libro + cerradura
- [ ] **6.3** Agregar `"puerta"` al selector y filtrado de UI

---

## BLOQUE 7 — Multi-tenant / Producción

- [ ] **7.1** Pantalla de configuración inicial (primera vez): URL Supabase, Anon Key, nombre empresa, logo
- [ ] **7.2** PDF usa nombre y logo del APP_CONFIG en lugar de los hardcodeados de Renova
- [ ] **7.3** Indicador de estado de conexión a Supabase en el header (🟢 / 🔴)
- [ ] **7.4** Pantalla "Acerca de / Licencia" con versión, cliente y fecha de vencimiento
- [ ] **7.5** Protección básica de licencia (licenseKey en CONFIG, validación simple)

---

## BLOQUE 8 — Calidad y Pulido

- [ ] **8.1** Test end-to-end: crear cotización completa de cocina y verificar despiece, herrajes, PDF y guardado en Supabase
- [ ] **8.2** Revisar y mejorar PDF de cocinas: portada, índice, despiece por módulo, resumen materiales, totales desglosados
- [ ] **8.3** Tooltips en UI: al hacer hover sobre tipo de módulo, mostrar dimensiones estándar y descripción breve
- [ ] **8.4** Calcular m² de tablero y número exacto de láminas necesarias por proyecto

---

## Orden de ejecución recomendado

```
BLOQUE 0 (obligatorio primero)
  ↓
BLOQUE 1A → 1B → 1C → 1D → 1E → 1F → 1G → 1H → 1I  (cocinas completo)
  ↓
BLOQUE 2 (baños)
  ↓
BLOQUE 4 (libreros y Tablered — más fáciles, dan confianza)
  ↓
BLOQUE 3 (TV)
  ↓
BLOQUE 5 (closets — extensión de lo existente)
  ↓
BLOQUE 6 (puertas)
  ↓
BLOQUE 7 (multi-tenant — solo cuando todo lo demás funcione)
  ↓
BLOQUE 8 (calidad y pulido — siempre al final)
```

---

## Notas para Claude Code

- **Lee INSTRUCTIVO_TECNICO_COCINAS.md completo antes de escribir cualquier función de cocinas.**
- **No inventes medidas.** Todas las dimensiones estándar están en los instructivos. Úsalas.
- **No cambies la estructura de `project.data`.** Solo extiende con nuevos tipos de módulo.
- **No rompas closets.** Cada cambio debe pasar la prueba: "¿un proyecto de closet existente sigue funcionando igual?"
- **innerWidth = W - (2 × T) siempre.** Sin excepción. Nunca W - T ni W - 30 hardcodeado.
- **Bajos NO llevan techo. Alacenas y torres SÍ llevan techo.** Error crítico si se confunden.
- **Cajones tienen frente externo (premium) y frente interno (casco).** Son piezas diferentes.
- **Cubierta no es pieza de tablero.** Va a extras como precio por m², no a parts[].
- **Torres con horno: validar siempre que la suma de huecos + cajones + separadores ≤ H.**
- **El sistema Dual Material (Casco + Frente) es la lógica central.** Todo módulo nuevo debe respetarlo.
- **Prioridad absoluta: Bloque 1 (Cocinas).** Es el producto principal para la primera venta (Renova).
