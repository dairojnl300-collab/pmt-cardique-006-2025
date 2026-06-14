# PROMPT ESPECIALIZADO — CLAUDE CODE
## Mejora del Checklist PMT · Inspección de Campo · LP-CARDIQUE-006-2025

---

## CONTEXTO DEL PROYECTO

Estoy trabajando en una **aplicación web offline HTML de una sola página** para inspección de campo del **Plan de Manejo de Tránsito (PMT)** del contrato **LP-CARDIQUE-006-2025** — Ciénaga de la Virgen, Cartagena. La app ya existe y funciona, pero necesita mejoras significativas de UX, lógica y funcionalidad.

La app está construida en **HTML + CSS + JavaScript vanilla** (sin frameworks), diseñada para uso en móvil en campo, almacena el estado en `localStorage`, y genera un informe HTML/PDF imprimible. El archivo de salida debe ser un único `.html` auto-contenido.

---

## DESCRIPCIÓN TÉCNICA DEL ARCHIVO ACTUAL

El archivo `PMT_Inspección_de_Campo.html` tiene:

### Estructura de Datos (`var D = {...}`)
- **3 frentes de obra:** `calicanto` (779 m), `chaplundun` (350 m), `ricaurte` (831 m)
- **6 secciones por frente:** Señalización 🚧 | Zona de Acopio 📦 | Rutas de Acceso 🛣️ | Personal y Paleteros 👷 | Peatones y Ciclistas 🚶 | Indicadores 📊
- **80 ítems en total** (30 + 25 + 25), cada uno con `id`, texto `t`, y flag `c` (1 = crítico)
- **24 ítems marcados como críticos** (flag `c:1`)

### Lógica de Estado
- `checks[id]` — ítem inspeccionado (booleano)
- `cumple[id]` — resultado: `true` (CUMPLE) | `false` (NO CUMPLE) | `undefined` (sin calificar)
- `notas[frente]` — observaciones de texto libre por frente
- `inc[frente]` — registro de incidente vial por frente: tipo, descripción, hora, acción tomada
- `openSecs[secId]` — secciones desplegadas
- Persistencia completa en `localStorage` con clave `pmt_cv3_cumple_real`

### UI Actual
- **Header naranja** con título, subtítulo y badge de contrato
- **Barra de progreso global** (% de ítems inspeccionados)
- **Tabs + Nav bottom** para navegar entre frentes, Resumen y Estadísticas
- **Panel por frente:** card con coordenadas GPS + secciones colapsables con ítems
- **Por ítem:** checkbox → al marcar, aparecen botones CUMPLE / NO CUMPLE + botones editar/eliminar
- **Panel Resumen:** cards de estado por frente + botones de exportar (Ver Informe, PDF, WhatsApp, Enviar checklist)
- **Panel Estadísticas:** stats cards, barras de cumplimiento por frente, tabla detalle de ítems
- **Reporte PDF:** overlay con HTML imprimible que lista todos los ítems con sus estados

### Paleta de Colores (CSS vars)
```css
--or: #E85D04   /* naranja primario */
--vd: #2DC653   /* verde CUMPLE */
--rj: #E63946   /* rojo NO CUMPLE / Crítico */
--am: #FFBE0B   /* amarillo PARCIAL */
--bg: #111827   /* fondo oscuro */
--bg2: #1F2937
--cd: #1E293B   /* card background */
--br: #334155   /* border */
--tx: #E8E8F0   /* texto principal */
```

---

## PROBLEMAS IDENTIFICADOS Y MEJORAS REQUERIDAS

### 🔴 PRIORIDAD ALTA — Errores / Bugs

**1. Bug crítico: el checkbox NO se marca visualmente al tocar**
El elemento `<div class="chk">` recibe el `✓` vía `chkContent` en `buildContent()`, pero al hacer `togCheck()` el DOM ya renderizado no actualiza el texto ni la clase `.done` correctamente en todos los casos. Revisar y corregir `togCheck()` para que:
- Actualice `checks[id]`, llame `save()`, `upd()`.
- Actualice visualmente: añadir/quitar clase `.done` al `.item` y texto `✓` al `.chk`.
- Muestre/oculte el `.cumple-row` con display flex/none.

**2. Bug: "Peatones y Ciclistas" solo tiene 3 ítems pero es la sección más crítica**
La sección carece de ítems sobre: accesibilidad para personas con discapacidad, señalización de paso peatonal provisional en obra nocturna, y separación física entre zona de trabajo y circulación peatonal. Agregar mínimo 2 ítems nuevos a esta sección en los 3 frentes.

**3. Bug visual: el emoji del ícono de sección "Rutas de Acceso" se rompe**
El código genera `'🛣️ Rutas de Acceso'` y luego hace `s.lbl.slice(0,2)` para el ícono, lo que corta mal el emoji de dos caracteres Unicode. Corregir el parseo del emoji del label para todos los labels de sección.

---

### 🟡 PRIORIDAD MEDIA — UX y Funcionalidad

**4. Campo "Inspector" debe poder guardar múltiples inspectores**
Actualmente es un solo input de texto. Reemplazar por un campo que permita agregar hasta 3 nombres (Residente, Inspector Interventoría, Supervisor CARDIQUE) con labels, y que todos aparezcan en el reporte PDF.

**5. Añadir campo "Hora de inspección" por frente**
Cada frente debe tener un campo `<input type="time">` para registrar la hora exacta de la inspección de ese frente. Debe aparecer en el reporte PDF y en el resumen de WhatsApp.

**6. Añadir campo de clasificación de No Conformidades**
Cuando el usuario marca "NO CUMPLE", debe aparecer un menú con las categorías de no conformidad:
- `NC-S`: Señalización incompleta/ausente
- `NC-E`: EPP/Personal incumplido
- `NC-A`: Acceso/Rutas obstruidas
- `NC-P`: Peatones/Ciclistas en riesgo
- `NC-D`: Documentación faltante
Este campo debe ser opcional pero registrarse en el estado y aparecer en el PDF.

**7. Botón "Limpiar frente" por panel**
Agregar un botón discreto por frente que permita resetear solo ese frente (checks, cumple, notas, incidente) sin afectar los demás. Pedir confirmación antes.

**8. Indicador visual de ítems críticos sin cumple definido**
Los ítems marcados como `c:1` (críticos) que estén inspeccionados pero sin calificación (sin CUMPLE/NO CUMPLE) deben mostrar una advertencia visual distinta (badge amarillo parpadeante o borde rojo punteado) para recordar al inspector que deben ser calificados.

**9. Sección "Incidente Vial" mejorada**
Actualmente el incidente solo registra tipo, descripción, hora y acción. Agregar:
- Campo de **gravedad**: Leve / Moderado / Grave
- Checkbox de **"¿Se reportó a autoridades?"**
- Que en el reporte PDF, los incidentes graves aparezcan destacados con fondo rojo

---

### 🟢 PRIORIDAD NORMAL — Mejoras de Reporte y Export

**10. Mejorar el reporte de WhatsApp**
La función `compartirTexto()` genera un texto plano. Mejorar el formato para incluir:
```
🚧 PMT | LP-CARDIQUE-006-2025
📅 Fecha: [fecha] | ⏰ [hora global]
👷 Inspector: [nombres]
📊 Avance: [X]% ([N]/80 ítems)

✅ CUMPLEN: [N] | ❌ NO CUMPLEN: [N] | ⏳ SIN CALIFICAR: [N]

🔵 CALICANTO ([X]%): [estado emoji]
🟡 CHAPLUNDÚN ([X]%): [estado emoji]
🟢 RICAURTE ([X]%): [estado emoji]

⛔ CRÍTICOS NO CONFORMES: [lista de ítems críticos con NO CUMPLE]

📋 Incidentes: [N] reportados
```

**11. El reporte PDF debe incluir:**
- Hora de inspección por frente
- Categoría de No Conformidad junto a cada ítem "NO CUMPLE"
- Tabla de resumen de no conformidades críticas al final (solo los críticos con NO CUMPLE)
- Footer con: versión del documento, referencia normativa INVIAS 2024, fecha/hora de generación

**12. Añadir "Score de conformidad" al Panel Estadísticas**
Calcular y mostrar una **calificación ponderada** donde:
- Ítems críticos valen **2x** en el cálculo de conformidad
- Mostrar la fórmula: `(CUMPLEN_CRITICOS*2 + CUMPLEN_NORMALES) / (CRITICOS*2 + NORMALES) * 100`
- Mostrar con colores: ≥90% Verde / 70-89% Amarillo / <70% Rojo

---

### 🔵 MEJORAS DE CÓDIGO / ARQUITECTURA

**13. Refactorizar `buildContent()` que tiene código duplicado**
Los 3 frentes comparten exactamente la misma estructura de secciones. El código genera HTML repetido. Crear una función `buildFrenteHTML(k)` que reciba la clave del frente y retorne el HTML del panel completo.

**14. Separar los datos `var D` a una constante bien documentada al inicio del script**
El objeto `D` está embebido en el bloque `<script>`. Dejarlo al inicio con comentarios claros sobre la estructura de cada campo.

**15. Agregar función de validación antes de exportar**
Antes de generar el PDF o enviar por WhatsApp, verificar:
- Si hay ítems inspeccionados sin calificación (CUMPLE/NO CUMPLE) → mostrar alerta con conteo
- Si no hay nombre de inspector → mostrar alerta
- Si hay ítems críticos con NO CUMPLE → pedir confirmación explícita antes de enviar

---

## INSTRUCCIONES PARA CLAUDE CODE

1. **El output debe ser un único archivo HTML auto-contenido** (`PMT_Inspeccion_Campo_v2.html`), sin dependencias externas — todo CSS y JS embebido.

2. **Mantener compatibilidad total** con el `localStorage` existente: no cambiar la clave `pmt_cv3_cumple_real` ni la estructura de los campos `checks`, `cumple`, `notas`, `inc`. Para los campos nuevos (hora por frente, categoría NC, gravedad incidente, inspectores múltiples), agregar claves nuevas en el objeto guardado.

3. **Prioridad de implementación sugerida:**
   - Primero: bugs 1, 2, 3 (funcionalidad rota)
   - Segundo: mejoras 4, 5, 6, 8 (UX campo)
   - Tercero: mejoras 10, 11, 12 (reportes)
   - Cuarto: mejoras 7, 9, 13, 14, 15 (polish y arquitectura)

4. **No cambiar la paleta de colores** ni el estilo visual general (dark mode, naranja CARDIQUE). Sí mejorar la tipografía y spacing si hay oportunidades claras.

5. **Probar mentalmente el flujo de uso:** Inspector llega a campo → abre el archivo en Chrome móvil → selecciona frente → registra hora de inspección → marca ítems → califica CUMPLE/NO CUMPLE → clasifica NC si aplica → agrega observación → registra incidente si hubo → va a Resumen → exporta PDF o WhatsApp.

6. **El archivo resultante debe funcionar offline al 100%** — sin llamadas a CDN, sin fetch, solo `localStorage`.

---

## ARCHIVOS DE REFERENCIA

El archivo HTML a mejorar está disponible como: `PMT_Inspeccion_de_Campo.html`

La estructura de datos completa de `var D` incluye los 3 frentes con sus 6 secciones y 80 ítems. Si necesitas la lista completa de IDs de ítems, está disponible en el archivo fuente.

---

*Contrato: LP-CARDIQUE-006-2025 | Interventoría: Consorcio Inter CRD Ciénaga | PMT v01 · INVIAS 2024*
