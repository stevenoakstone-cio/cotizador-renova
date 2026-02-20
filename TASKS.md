# TASKS.md — Plan de Desarrollo Cotizador Pro
> Para usar con Claude Code + plugin GetShitDone.
> Leer CONTEXT.md y INSTRUCTIVO_TECNICO.md antes de empezar cualquier tarea.
> No romper closets. No separar archivos. Todo en cotizador-template.html salvo que se indique.

---

## BLOQUE 0 — Fundación (hacer primero, todo lo demás depende de esto)

- [ ] **0.1** Leer `CONTEXT.md` e `INSTRUCTIVO_TECNICO.md` completos antes de empezar.

- [ ] **0.2** Auditar el código de closets en `cotizador-template.html`:
  - Identificar dónde están definidos los módulos de closet (objeto/array con los tipos)
  - Identificar la función de despiece automático (donde se calculan las `parts`)
  - Identificar el sistema de filtrado de UI por tipo de mueble
  - Identificar la función de asignación automática de herrajes
  - Documentar los hallazgos en comentarios en el código o en un archivo `AUDIT.md`

- [ ] **0.3** Crear la función genérica `calculateParts(moduleType, dimensions, material)` que:
  - Recibe tipo de módulo, dimensiones (w, h, d) y material
  - Aplica la fórmula base: `innerWidth = width - 30`
  - Retorna array de piezas `[{ name, qty, w, h, thickness, material }]`
  - Es el núcleo que todos los módulos nuevos usarán
  - **Referencia:** Sección "Cálculo de piezas (despiece automático)" en CONTEXT.md

- [ ] **0.4** Crear la función `assignHardware(moduleType, dimensions, parts)` que:
  - Asigna automáticamente herrajes según tipo y dimensiones
  - Regla de bisagras: altura ≤ 1000mm → 2 bisagras; 1000–2000mm → 3; >2000mm → 4
  - Regla de correderas: ancho ≤ 500mm → estándar; >500mm → Tandem Plus con freno
  - Regla de patas: módulos bajos de cocina → 4 patas (2 si comparte con módulo contiguo)
  - Regla de precio: herraje ≤ $500 MXN → factor ×3; ≥ $501 → precio ÷ 0.70
  - **Referencia:** Sección "Reglas de negocio" en CONTEXT.md

---

## BLOQUE 1 — Cocinas (prioridad máxima)

> Leer Sección 3 de INSTRUCTIVO_TECNICO.md antes de este bloque.

### 1A — Módulos Bajos de Cocina

- [ ] **1A.1** Definir el objeto de configuración para módulos bajos de cocina:
  ```javascript
  COCINA_MODULES_BAJOS = {
    base_1_puerta:   { label: 'Base 1 Puerta',   defaultW: 600, defaultH: 720, defaultD: 580 },
    base_2_puertas:  { label: 'Base 2 Puertas',  defaultW: 600, defaultH: 720, defaultD: 580 },
    fregadero:       { label: 'Fregadero',        defaultW: 600, defaultH: 720, defaultD: 580 },
    cajonera_2:      { label: 'Cajonera 2 Cajones', defaultW: 500, defaultH: 720, defaultD: 580 },
    cajonera_3:      { label: 'Cajonera 3 Cajones', defaultW: 500, defaultH: 720, defaultD: 580 },
    cajonera_4:      { label: 'Cajonera 4 Cajones', defaultW: 500, defaultH: 720, defaultD: 580 },
    esquinero_ciego: { label: 'Esquinero Ciego',  defaultW: 900, defaultH: 720, defaultD: 580 },
    esquinero_l:     { label: 'Esquinero en L',   defaultW: 900, defaultH: 720, defaultD: 580 },
    horno_base:      { label: 'Módulo Horno',     defaultW: 600, defaultH: 720, defaultD: 580 },
  }
  ```

- [ ] **1A.2** Implementar despiece automático para `base_1_puerta` y `base_2_puertas`:
  - Casco: 2 costados + 1 piso + 2 amarres (sup e inf) + 1 fondo HDF 3mm + 1 entrepaño opcional
  - Frente: puertas según `width ≤ 500 → 1 puerta; > 500 → 2 puertas`
  - Ancho puerta = `(innerWidth / doorQty) - 2mm` por puerta
  - Alto puerta = `height - 10mm`
  - **Referencia:** Sección 3.7 de INSTRUCTIVO_TECNICO.md

- [ ] **1A.3** Implementar despiece automático para `fregadero`:
  - Sin techo (solo 2 amarres de 80mm)
  - Sin entrepaño interior
  - Saque en fondo trasero para tuberías (notar en el despiece como observación)
  - 2 puertas siempre
  - **Referencia:** "Módulo Fregadero" en Sección 3.3 de INSTRUCTIVO_TECNICO.md

- [ ] **1A.4** Implementar despiece automático para `cajonera_2`, `cajonera_3`, `cajonera_4`:
  - Casco: 2 costados + 1 piso + 1 amarre + 1 fondo HDF
  - Por cada cajón: 2 costados + 1 frente interno + 1 fondo HDF
  - Frentes de cajón (premium): 1 por cajón
  - Alturas de cajones para módulo de 720mm:
    - Superior: 150mm | Medios: 200mm | Inferior: 270mm (ajustar para que sumen 720mm)
  - Herraje: 1 par de correderas Blum Tandem por cajón
  - **Referencia:** "Módulo Cajonera" en Sección 3.3 de INSTRUCTIVO_TECNICO.md

- [ ] **1A.5** Implementar despiece automático para `horno_base`:
  - Casco con entrepaño a 595mm del piso (hueco del horno)
  - 1 cajón inferior de 150mm
  - Nota en despiece: "Saque trasero 100mm para ventilación — obligatorio"
  - Nota en despiece: "Verificar medidas del horno antes de fabricar"

- [ ] **1A.6** Implementar herrajes automáticos para módulos bajos de cocina:
  - Todos llevan patas regulables (4 pz por módulo independiente)
  - Todos llevan zoclo (1 ml por módulo)
  - Cajoneras: correderas Blum Tandem (1 par por cajón)
  - Puertas: bisagras cup 35mm (2 por puerta)
  - Esquinero en L: bisagras 165° en lugar de estándar

### 1B — Módulos Altos de Cocina (Alacenas)

- [ ] **1B.1** Definir objeto de configuración para alacenas:
  ```javascript
  COCINA_MODULES_ALTOS = {
    alacena_std:      { label: 'Alacena Estándar',   defaultW: 600, defaultH: 800, defaultD: 350 },
    alacena_aventos:  { label: 'Alacena Aventos',    defaultW: 600, defaultH: 800, defaultD: 350 },
    alacena_campana:  { label: 'Sobre Campana/Refri', defaultW: 600, defaultH: 400, defaultD: 300 },
  }
  ```

- [ ] **1B.2** Implementar despiece para `alacena_std`:
  - Casco: 2 costados + 1 piso + 1 techo + 1 fondo HDF + 1-2 entrepaños
  - 1 o 2 puertas (mismo criterio que módulos bajos)
  - Sin patas (va anclada a muro)
  - **Referencia:** "Alacena Estándar" en Sección 3.4 de INSTRUCTIVO_TECNICO.md

- [ ] **1B.3** Implementar despiece para `alacena_aventos`:
  - 1 sola puerta elevable (no 2 laterales)
  - Nota en despiece: "Sistema Aventos — seleccionar modelo según peso del frente"
  - Agregar campo para seleccionar modelo Aventos (HK, HL, HS)

### 1C — Torres / Columnas

- [ ] **1C.1** Definir objeto de configuración para torres:
  ```javascript
  COCINA_MODULES_TORRES = {
    torre_hornos:   { label: 'Torre de Hornos',  defaultW: 600, defaultH: 2100, defaultD: 580 },
    torre_despensa: { label: 'Torre Despensa',   defaultW: 600, defaultH: 2100, defaultD: 580 },
  }
  ```

- [ ] **1C.2** Implementar despiece para `torre_hornos`:
  - Estructura de piso a techo
  - Nichos: microondas 450mm + horno 600mm + cajón inferior 150mm + puertas arriba
  - Nota obligatoria: "Saque trasero 100mm para ventilación"
  - **Referencia:** "Torre de Hornos" en Sección 3.5 de INSTRUCTIVO_TECNICO.md

- [ ] **1C.3** Implementar despiece para `torre_despensa`:
  - Entrepaños regulables cada 32mm (sistema cremallera)
  - 2 puertas (superior + inferior)
  - **Referencia:** "Torre Despensa" en Sección 3.5 de INSTRUCTIVO_TECNICO.md

### 1D — Piezas Extra de Cocina

- [ ] **1D.1** Agregar piezas extra al cotizador de cocinas:
  - **Zoclo**: se calcula como ml lineales totales (suma del ancho de todos los módulos bajos)
  - **Vista lateral (panel)**: tablero del material premium, tamaño = costado visible del casco
  - **Panel de relleno**: tira de material entre módulo y pared (ancho variable 5–100mm)
  - **Cubierta**: cotizar por m² (postformado, cuarzo, granito — precio diferenciado)
  - Estas piezas deben poder agregarse manualmente desde la UI, no son automáticas

### 1E — UI de Cocinas

- [ ] **1E.1** Agregar `"cocina"` al selector de tipo de proyecto (Paso 1 del wizard)

- [ ] **1E.2** Implementar filtrado de UI: cuando `type === "cocina"`, mostrar SOLO:
  - Los 3 grupos de módulos (Bajos, Alacenas, Torres) en el panel de configuración
  - Materiales filtrados por categoría "casco" y "frente" (no clóset-específicos)
  - Las piezas extra de cocina (zoclo, vista lateral, cubierta, panel de relleno)

- [ ] **1E.3** Agrupar los módulos de cocina visualmente en el UI por categoría:
  - Sección "Módulos Bajos" — con ícono y color diferenciador
  - Sección "Alacenas" — idem
  - Sección "Torres" — idem
  - Sección "Piezas Extra" — idem

- [ ] **1E.4** Agregar validaciones de cocina:
  - Advertencia si hay módulo de horno sin nota de ventilación
  - Advertencia si se usa MDF normal (no RH) en módulo de fregadero
  - Advertencia si el ancho de un módulo no está en la lista de anchos estándar (300/400/450/500/600/750/800/900/1000mm)

### 1F — Totales y PDF de Cocinas

- [ ] **1F.1** Agregar al resumen de cocinas:
  - Subtotal de cascos (material económico)
  - Subtotal de frentes (material premium)
  - Subtotal de herrajes
  - Subtotal de piezas extra (zoclo, cubierta, etc.)
  - Total general con IVA

- [ ] **1F.2** Agregar al PDF de cocinas:
  - Tabla de módulos con tipo, dimensiones, material, frente y precio unitario
  - Tabla de despiece con todas las piezas calculadas
  - Tabla de herrajes con cantidades
  - Lista de piezas extra
  - Nota técnica al pie: "Medidas sujetas a verificación en sitio"

---

## BLOQUE 2 — Baños

> Leer Sección 4 de INSTRUCTIVO_TECNICO.md antes de este bloque.

- [ ] **2.1** Definir módulos de baño:
  ```javascript
  BANO_MODULES = {
    bajo_lavabo:  { label: 'Bajo Lavabo',      defaultW: 800,  defaultH: 550, defaultD: 500 },
    cajonera_bano:{ label: 'Cajonera Baño',    defaultW: 400,  defaultH: 550, defaultD: 500 },
    botikin:      { label: 'Botiquín/Espejo',  defaultW: 600,  defaultH: 800, defaultD: 150 },
  }
  ```

- [ ] **2.2** Implementar despiece para `bajo_lavabo`:
  - Material: **MDF RH obligatorio** — forzar en el selector de materiales
  - Cajón superior con saque en U: 220mm × 180mm centrado
  - Montaje flotante: agregar nota "Anclar con escuadras metálicas — no usar solo yeso"

- [ ] **2.3** Agregar `"bano"` al selector y filtrado de UI igual que cocinas

- [ ] **2.4** Validación: si `type === "bano"` y el material seleccionado NO es MDF RH → mostrar advertencia roja prominente

---

## BLOQUE 3 — Mueble de TV / Centro de Entretenimiento

> Leer Sección 5 de INSTRUCTIVO_TECNICO.md antes de este bloque.

- [ ] **3.1** Definir módulos de TV:
  ```javascript
  TV_MODULES = {
    consola_patas:   { label: 'Consola con Patas',   defaultW: 1600, defaultH: 400, defaultD: 400 },
    consola_flotante:{ label: 'Consola Flotante',    defaultW: 1600, defaultH: 400, defaultD: 400 },
    torre_tv:        { label: 'Torre Lateral TV',    defaultW: 350,  defaultH: 2100, defaultD: 300 },
    repisa_flotante: { label: 'Repisa Flotante',     defaultW: 800,  defaultH: 40,   defaultD: 300 },
    panel_fondo:     { label: 'Panel de Fondo',      defaultW: 1800, defaultH: 2400, defaultD: 30  },
  }
  ```

- [ ] **3.2** Despiece para `consola_patas` y `consola_flotante`:
  - Igual que módulo base de cocina pero sin zoclo
  - Consola con pistas: incluir pistones de gas si tiene tapa abatible (≥150N)

- [ ] **3.3** Despiece para `repisa_flotante`:
  - Opción A: repisa hueca (cajita) — calcular piezas de la caja
  - Opción B: herraje oculto — cotizar herraje + tablero sólido ≥38mm
  - El usuario elige el método

- [ ] **3.4** Agregar `"tv"` al selector y filtrado de UI

---

## BLOQUE 4 — Libreros y Muebles Tablered

> Leer Secciones 8, 9 y 10 de INSTRUCTIVO_TECNICO.md antes de este bloque.

- [ ] **4.1** Agregar módulo `librero` con el despiece exacto de Tablered Arauco:
  - 9 piezas (A–I) con sus medidas específicas
  - Cubrecanto KIRA: 59.2 ml
  - 8 bisagras bidimensionales
  - 6 regatones niveladores

- [ ] **4.2** Agregar módulo `credenza` con el despiece exacto de Tablered Arauco:
  - 7 piezas (A–G) con sus medidas específicas
  - Cubrecanto NERO: 49 ml
  - 8 bisagras bidimensionales
  - 4 patas metálicas 10cm

- [ ] **4.3** Agregar módulo `mesa_centro` con el despiece de 2 bloques:
  - Bloque 1 KIRA: 5 piezas (A–E)
  - Bloque 2 NERO: 4 piezas (F–I)
  - 1 pistón gas 150N (obligatorio)
  - 2 bisagras bidimensionales
  - 4 patas 5cm

- [ ] **4.4** Crear tipo de proyecto `"librero"`, `"credenza"`, `"mesa_centro"` con filtrado UI correspondiente

---

## BLOQUE 5 — Clósets (extensión de lo existente)

> Solo mejoras sobre lo que ya funciona.

- [ ] **5.1** Auditar módulos de closet actuales y documentar cuáles están implementados

- [ ] **5.2** Agregar los módulos faltantes según INSTRUCTIVO_TECNICO.md Sección 6:
  - `colgador_largo` (si no existe)
  - `colgador_corto_doble` (si no existe)
  - `zapatero` (si no existe)
  - `maletero` (si no existe)
  - `torre_entrepaños` (si no existe)

- [ ] **5.3** Agregar validación: si hay módulo `colgador_largo` o `torre_entrepaños` con alto > 1,500mm → advertencia "Anclar mueble a pared — anti-vuelco obligatorio"

---

## BLOQUE 6 — Puertas de Intercomunicación

> Leer Sección 7 de INSTRUCTIVO_TECNICO.md antes de este bloque.

- [ ] **6.1** Definir módulo de puerta:
  ```javascript
  PUERTA_MODULE = {
    puerta_std: { label: 'Puerta Interior', defaultW: 900, defaultH: 2100, defaultD: 45 }
  }
  ```

- [ ] **6.2** Despiece de puerta:
  - Bastidor de pino (perímetro + peinazos)
  - 2 tapas MDF (cara exterior + cara interior)
  - Marco (chambrana): 3 piezas × 2 lados = 6 tiras
  - Tope: 1 tira
  - Herrajes: 3 bisagras tipo libro + 1 cerradura
  - Número de bisagras: auto-calcular según tipo de relleno (honey-comb → 3; peinazos → 3; sólida → 4)

- [ ] **6.3** Agregar `"puerta"` al selector y filtrado de UI

---

## BLOQUE 7 — Multi-tenant / Producción

> Este bloque prepara la app para ser vendida como SaaS a cada carpintería.

- [ ] **7.1** Agregar pantalla de configuración inicial (primera vez que se abre la app):
  - Input: URL de Supabase
  - Input: Anon Key de Supabase
  - Input: Nombre de la empresa (reemplaza "Renova" en el PDF)
  - Input: Logo (upload de imagen)
  - Botón "Guardar y conectar" — guarda en localStorage bajo `APP_CONFIG`

- [ ] **7.2** Hacer que el PDF use el nombre y logo del `APP_CONFIG` en lugar de los hardcodeados de Renova

- [ ] **7.3** Agregar indicador de estado de conexión a Supabase en el header:
  - 🟢 Conectado a la nube
  - 🔴 Modo offline (solo localStorage)
  - Al hacer clic: muestra info de la cuenta conectada

- [ ] **7.4** Agregar pantalla "Acerca de / Licencia":
  - Versión de la app
  - Nombre del cliente (la carpintería que compró la licencia)
  - Fecha de vencimiento de licencia (hardcodeada en el HTML por ahora)
  - Botón para contactar soporte

- [ ] **7.5** Agregar protección básica de licencia:
  - Un campo `licenseKey` en el `APP_CONFIG`
  - Al iniciar la app, verificar que la key sea válida (validación simple contra un hash hardcodeado)
  - Si la licencia venció o no existe → pantalla de bloqueo con mensaje de contacto

---

## BLOQUE 8 — Calidad y Pulido

- [ ] **8.1** Test end-to-end manual: crear una cotización completa de cocina desde cero y verificar:
  - El despiece es correcto (piezas, medidas, cantidades)
  - Los herrajes se asignan automáticamente y son correctos
  - El PDF generado es profesional y tiene toda la información
  - El proyecto se guarda correctamente en Supabase

- [ ] **8.2** Revisar y mejorar el PDF de cocinas:
  - Portada con nombre del proyecto, cliente, fecha y logo
  - Índice de módulos
  - Tabla de despiece por módulo
  - Resumen de materiales (m² de cada tablero necesario)
  - Resumen de herrajes (total de bisagras, correderas, patas, etc.)
  - Totales con desglose de casco / frente / herrajes / extras

- [ ] **8.3** Agregar tooltips o ayuda contextual en la UI para los módulos nuevos:
  - Al pasar el mouse sobre un tipo de módulo: mostrar dimensiones estándar y descripción breve

- [ ] **8.4** Optimizar el cálculo de m² de tablero necesario:
  - Dado el despiece completo de un proyecto, calcular cuántos tableros completos (1220×2440mm) se necesitan
  - Mostrar el número de tableros por diseño/material
  - Esto permite cotizar el material exacto sin desperdicio

---

## Orden de ejecución recomendado

```
BLOQUE 0 (obligatorio primero)
  ↓
BLOQUE 1A → 1B → 1C → 1D → 1E → 1F  (cocinas completo)
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

- **No inventes medidas.** Todas las dimensiones estándar están en `INSTRUCTIVO_TECNICO.md`. Úsalas.
- **No cambies la estructura de `project.data`.** Solo extiende con nuevos tipos de módulo.
- **No rompas closets.** Cada cambio debe pasar la prueba: "¿un proyecto de closet existente sigue funcionando igual?"
- **Pregunta antes de hacer cambios grandes** a la arquitectura del HTML monolítico.
- **El sistema Dual Material (Casco + Frente) es la lógica central.** Todo módulo nuevo debe respetarlo.
- **Prioridad absoluta: Bloque 1 (Cocinas).** Es el producto principal para la primera venta.
