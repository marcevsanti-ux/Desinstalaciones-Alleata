# Informe Técnico — Alleata · Sistema de Gestión de Desinstalaciones

**Versión del portal distribuidor:** v1.1.1 · 09/03/26  
**Empresa:** [Alleata](https://alleata.com.ar)  
**Fecha del informe:** 23 de marzo de 2026

---

## 1. Descripción General

El sistema está compuesto por **dos aplicaciones web** independientes que trabajan en conjunto para gestionar el proceso de desinstalación de equipos en comercios:

| Archivo | Rol |
|---|---|
| `index.html` | **Panel de administración / operaciones** — visión global de todas las Órdenes de Trabajo (OTs) |
| `distri.html` | **Portal del distribuidor** — interfaz móvil para que cada logístico gestione sus OTs asignadas |

Ambas apps consumen datos desde **Google Sheets** a través de exportaciones CSV públicas y se sincronizan con Google Apps Script mediante un **webhook REST**.

---

## 2. Stack Tecnológico

| Capa | Tecnología |
|---|---|
| Frontend (admin) | React 18 (CDN via `unpkg`), Babel Standalone, XLSX.js |
| Frontend (distribuidor) | HTML5 + CSS3 vanilla, JavaScript puro |
| Fuente de datos | Google Sheets (CSV export) |
| Backend / integración | Google Apps Script (webhook) |
| Georreferenciación | API Georef Argentina (`apis.datos.gob.ar/georef`) |
| Tipografía | System UI (admin) / Plus Jakarta Sans (portal) |
| Hosting implícito | Estático — puede servirse desde cualquier CDN o hosting de archivos |

---

## 3. Arquitectura de Datos

### Google Sheets — Hojas utilizadas

```
Spreadsheet ID: 1vxBwULsCev2TqGohkVSFE0GqHma3Ww69yRPxVjNaECY

  gid=1843553678  →  Hoja de OTs (Órdenes de Trabajo)
  gid=107031751   →  Hoja de Distribuidores
```

### Modelo de campos — OT

| Campo | Descripción |
|---|---|
| `OT` | Número de orden de trabajo |
| `Comercio` | Nombre del comercio a desinstalar |
| `Cuenta` | Cuenta asociada (puede diferir del comercio) |
| `Estado` | Estado de la OT: `Pendiente`, `En Proceso`, `Despachado`, `Extraviada` |
| `Logistica` | Distribuidor logístico asignado |
| `Zona` | Zona geográfica (`CABA`, `GBA`, etc.) |
| `Provincia` | Provincia (texto libre, normalizado por la app) |
| `Ciudad` | Ciudad del comercio |
| `Calle` | Dirección completa |
| `CP` | Código Postal |
| `Contacto` | Nombre/teléfono de contacto |
| `Horario` | Horario de atención del comercio |
| `GestionEstado` | Estado de gestión con el distribuidor |

---

## 4. Módulos del Panel de Administración (`index.html`)

### 4.1. Navegación por pestañas

La app tiene las siguientes secciones principales:

1. **OTs** — tabla general de órdenes de trabajo con filtros avanzados
2. **Indicadores** — KPIs y actividad de distribuidores (consume el webhook)
3. **Config** — configuración de URLs de planillas y webhook

### 4.2. Filtros y búsqueda

- Búsqueda de texto libre por OT, comercio, ciudad, dirección
- Filtro por **sector geográfico** (AMBA Norte, Sur, Oeste, CABA, Interior, provincias)
- Filtro por **estado** de la OT
- Filtro por **distribuidor logístico**
- **Vista por ciudad / zona de distribuidor** con cálculo de distancia en km

### 4.3. Lógica geográfica

El sistema incluye lógica avanzada de normalización geográfica para Argentina:

- **`normProv()`** — normaliza el nombre de provincia con acentos y variantes
- **`getAmbaSubzona()`** — clasifica localidades del AMBA en: `CABA`, `Zona Norte`, `Zona Sur`, `Zona Oeste` usando sets de más de 150 localidades
- **`getSector()`** — determina si una OT es AMBA o Interior, con lógica prioritaria: CP → campo Zona del sheet → ciudad conocida
- **`isEnRadioAmba()`** — verificación por código postal usando un mapa de coordenadas con fórmula de Haversine (radio ~50 km desde CABA)
- **`LOCALIDAD_CP`** — diccionario de ~100 localidades argentinas con su CP

### 4.4. Modal de detalle de OT

Al hacer clic en una OT se abre un modal completo con:
- Datos del comercio y dirección
- Estado actual con badge de color
- **Inferencia de CP** vía API Georef (si el CP está vacío o incompleto)
- **Edición de estado** con envío al webhook (POST con `action: updateStatus`)
- Botón de WhatsApp directo al contacto
- Enlace a Google Maps con la dirección

### 4.5. Vista de distribuidores y mapa

- Selector de distribuidor con cálculo de distancia haversine entre las OTs y la ubicación del distribuidor
- Ordenamiento por distancia
- Modal "Listar" con tabla imprimible de OTs por distribuidor

### 4.6. Sincronización con Google Sheets

- **Actualizar solapas**: botón que llama al webhook con `?action=regenerarSolapas` para regenerar las hojas individuales por distribuidor en Google Sheets
- Alternativa manual: ejecutar `actualizarTodasLasSolapas()` desde Google Apps Script

### 4.7. Tab Indicadores

Panel de KPIs con datos en tiempo real desde el webhook:
- Tarjetas por estado: `Contactado`, `No encontrado`, `Exitoso`, `Confirmado retiro`
- Tarjetas por distribuidor con conteo de acciones
- Tabla filtrable con búsqueda por OT, comercio y distribuidor
- Actualización manual con botón 🔄

---

## 5. Módulos del Portal del Distribuidor (`distri.html`)

### 5.1. Identificación del distribuidor

La app se identifica con el distribuidor mediante un parámetro en la URL o cookie. El header muestra el nombre del distribuidor y estadísticas de sus OTs:

- **Total** de OTs asignadas
- **Pendientes**
- **Contactadas**
- **Exitosas**

### 5.2. Listado de OTs (tab principal)

- Cards con info completa de cada OT: número, comercio, ciudad, dirección, contacto, horario, observaciones previas
- Filtros por estado: `Todos`, `Pendiente`, `Contactado`, `No encontrado`, `Exitoso`, `Confirmado retiro`
- Búsqueda por OT o nombre de comercio
- Indicador visual por estado en el borde de la card (color-coded)

### 5.3. Acciones sobre cada OT

Desde cada card, el distribuidor puede registrar:

| Acción | Estado resultante |
|---|---|
| 📞 Contactado | Contactado |
| ❌ No encontrado | No encontrado |
| ✅ Exitoso | Retirado exitosamente |
| 📦 Confirmado retiro | Confirmado retiro (con fecha) |

Al seleccionar un estado, se abre un **modal de confirmación** que permite:
- Cambiar o confirmar el estado
- Agregar observaciones de texto libre
- Seleccionar fecha de retiro (para "Confirmado retiro")
- Registrar múltiples intentos de "No encontrado" (con chips de fecha)

Los cambios se envían al webhook de Google Apps Script via `fetch POST`.

### 5.4. Tab Agenda

Calendario mensual con:
- Vista de grilla semanal (7 columnas)
- Días con eventos destacados con borde de color
- Panel de detalle al hacer clic en un día
- Tipos de evento: contactos agendados, retiros confirmados
- Estados visuales: Hoy (verde), Vencido (rojo), Contactado (azul), Confirmado (amarillo)

### 5.5. Tab Hoja de Ruta

- Calendario adicional orientado a planificar la ruta del día
- Panel de detalle con las OTs programadas para la fecha seleccionada

### 5.6. Toast y feedback

- Sistema de notificaciones tipo "toast" (aparece en la parte inferior, desaparece automáticamente)
- Spinner de carga al inicializar
- Estados de loading/error explícitos

---

## 6. Comunicación con el Backend

El sistema usa un **webhook de Google Apps Script** como backend serverless:

```
GET  {webhookUrl}                      → obtener estados de distribuidores
GET  {webhookUrl}?action=regenerarSolapas → regenerar hojas por distribuidor
POST {webhookUrl}  body: { action: "updateStatus", ot: "123", status: "En Proceso" }
POST {webhookUrl}  body: { action, ot, estado_distri, obs, fecha }  → actualización del distribuidor
```

Las URLs se configuran desde la pestaña **Config** del panel admin y se guardan en `localStorage` del navegador.

---

## 7. Colores y Estados

### Estados de OT (admin)

| Estado | Color |
|---|---|
| Pendiente | `#f59e0b` (amarillo) |
| En Proceso | `#3b82f6` (azul) |
| Extraviada | `#ef4444` (rojo) |
| Despachado | `#10b981` (verde) |

### Zonas geográficas

| Zona | Fondo | Texto |
|---|---|---|
| CABA | azul claro | azul oscuro |
| GBA | verde claro | verde oscuro |
| PBA | amarillo | marrón |
| Interior | violeta claro | violeta oscuro |

---

## 8. Observaciones y Posibles Mejoras

### Fortalezas
- **Sin servidor propio** — arquitectura completamente serverless usando Google como backend
- **Lógica geográfica robusta** para el contexto argentino (AMBA, provincias, CP)
- **Dos interfaces especializadas**: una para operaciones y otra optimizada para móvil del distribuidor
- Inferencia automática de CP vía API pública del Estado argentino

### Oportunidades de mejora

1. **Seguridad**: las URLs de Google Sheets son públicas; no hay autenticación en el panel admin
2. **React via CDN + Babel en runtime**: penaliza la performance inicial; en producción conviene un build estático (Vite/CRA)
3. **Configuración en `localStorage`**: si el usuario limpia el navegador, pierde la configuración del webhook
4. **Sin manejo de conflictos de escritura**: si dos distribuidores actualizan la misma OT simultáneamente, el último que escribe gana
5. **Dependencia de Google Sheets**: cualquier cambio en la estructura de la planilla puede romper el parseo CSV
6. **El portal del distribuidor no tiene autenticación**: cualquier persona con el link ve las OTs del distribuidor

---

## 9. Flujo de Trabajo Resumido

```
Google Sheets (datos maestros)
        │
        │ CSV export (lectura)
        ▼
Panel Admin (index.html)          Portal Distribuidor (distri.html)
   - Ve todas las OTs               - Ve sus OTs asignadas
   - Filtra por zona/estado          - Registra gestiones (contacto, retiro)
   - Ve indicadores                  - Agenda y hoja de ruta
   - Configura webhook               │
        │                            │
        └────────────┬───────────────┘
                     │ POST/GET
                     ▼
            Google Apps Script (webhook)
                     │
                     │ Escribe
                     ▼
            Google Sheets (actualiza estados)
```

---

*Informe generado el 23 de marzo de 2026 a partir del análisis del código fuente de `index.html` y `distri.html`.*
