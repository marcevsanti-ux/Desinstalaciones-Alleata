# Informe Técnico — Alleata · Sistema de Gestión de Desinstalaciones

**Versión del portal administración:** v5.6 · 26/03/26
**Versión del portal distribuidor:** v1.1.1 · 09/03/26
**Empresa:** [Alleata](https://alleata.com.ar)
**Fecha del informe:** 26 de marzo de 2026

## 1. Descripción General

El sistema está compuesto por dos aplicaciones web independientes:

| Archivo | Rol |
|---|---|
| index.html | Panel de administración / operaciones — visión global de todas las OTs |
| distri.html | Portal del distribuidor — interfaz móvil para gestión de OTs asignadas |

## 2. Stack Tecnológico

| Capa | Tecnología |
|---|---|
| Frontend (admin) | React 18 (CDN), Babel Standalone, XLSX.js |
| Frontend (distribuidor) | HTML5 + CSS3 vanilla, JavaScript puro |
| Fuente de datos | Google Sheets (CSV export) |
| Backend / integración | Google Apps Script (webhook REST) |
| Almacenamiento de evidencias | Google Drive (carpeta Alleata · Evidencias) |
| Georreferenciación | API Georef Argentina |
| Hosting | Estático — GitHub Pages |

## 3. Campos reales del sheet de OTs (Hoja13)

| Campo | Descripción |
|---|---|
| WorkOrderNumber | Número de orden de trabajo |
| Nombre_del_comercio__c | Nombre del comercio |
| Status | Estado: Pendiente, En Proceso, Exitoso, Extraviada, Despachado |
| Logistica__c | Distribuidor logístico asignado |
| Categoria_Zona__c | Zona geográfica |
| Direcci_n_de_env_o__City__s | Ciudad de envío (prioritaria) |
| Direcci_n_de_env_o__Street__s | Dirección de envío (prioritaria) |
| CP_extraido | Código Postal |
| Contacto__c | Contacto del comercio |
| Horario__c | Horario de atención |

## 4. Panel de Administración (index.html v5.6)

### 4.1. KPIs con botón Listar

Header con 6 contadores en tiempo real: TOTAL, PENDIENTES, EN PROCESO, DESPACHADAS, EXITOSAS, EXTRAVIADAS. Cada contador tiene botón Listar que abre tabla completa con vista de impresión.

### 4.2. Tabs

| Tab | Función |
|---|---|
| Dashboard | Mapa de provincias, tabla con filtros, búsqueda |
| Distribuidores | Gestión de logísticos, cálculo de distancias |
| Indicadores | KPIs de actividad en tiempo real |
| Importar | Importación masiva de Exitosos desde xlsx Salesforce |
| Config | URLs de planillas y webhook |

### 4.3. Importar desde Salesforce

1. Usuario sube xlsx exportado de Salesforce
2. App extrae OTs con Status=Exitoso
3. Un solo request al webhook con toda la lista
4. Apps Script cruza contra Hoja13 y actualiza solo las que existen
5. OTs históricas no presentes en Hoja13 se ignoran

### 4.4. Modal de OT

Al clic en cualquier OT: datos completos, edición de Status directo en Hoja13, botón WhatsApp, Google Maps, inferencia de CP via API Georef.

### 4.5. Lógica geográfica

Normalización avanzada para Argentina: clasificación AMBA/Interior por haversine (radio 70km), subzonas Norte/Sur/Oeste, clasificación por coordenadas lat/lon para Patagonia y Cuyo, unificación de ciudades con acentos.

## 5. Portal del Distribuidor (distri.html v1.1.1)

### 5.1. Estados disponibles

| Estado | Detalle |
|---|---|
| Contactado | Registro de contacto |
| No encontrado | Con motivo obligatorio (chips) + foto opcional |
| Exitoso | Retiro exitoso |
| Confirmado retiro | Con fecha y horario |

### 5.2. Evidencia fotográfica

Solo disponible en estado No encontrado. El distribuidor puede tomar foto con cámara o seleccionar de galería. Las fotos se suben a Google Drive y la URL queda registrada en Estados_Distri.

### 5.3. Tabs

OTs / Agenda / Hoja de Ruta / Indicadores

## 6. Backend — Google Apps Script

| Endpoint | Acción |
|---|---|
| GET sin params | Devuelve todos los estados |
| GET ?distri=Nombre | Filtra por distribuidor |
| GET ?action=regenerarSolapas | Regenera solapas individuales |
| POST action:updateStatus | Actualiza Status en Hoja13 |
| POST action:importExitosos | Importación masiva con cruce contra Hoja13 |
| POST estado distribuidor + foto | Escribe en Estados_Distri, sube foto a Drive |

## 7. Estados y Colores

| Estado | Color |
|---|---|
| Pendiente | amarillo #f59e0b |
| En Proceso | azul #3b82f6 |
| Exitoso | verde #10b981 |
| Extraviada | rojo #ef4444 |
| Despachado | violeta #8b5cf6 |

## 8. Oportunidades de mejora

1. Autenticación: ambas apps son accesibles sin login
2. React via CDN penaliza performance inicial
3. Configuración en localStorage se pierde al limpiar navegador
4. Sin manejo de conflictos de escritura simultánea
5. Dependencia de estructura fija del sheet de Salesforce
