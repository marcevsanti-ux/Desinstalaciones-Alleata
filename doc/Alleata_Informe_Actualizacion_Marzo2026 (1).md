# Alleata · Informe de Actualización — Marzo 2026

**Fecha:** 29 de marzo de 2026  
**App:** Gestión de Desinstalaciones v5.25  
**Planilla importada:** WorkOrder-27_3_2026.xlsx  
**Sheet:** [DESINSTALACIONES - MARZO -2026](https://docs.google.com/spreadsheets/d/1imDRJmnJn-W-yT23i3DdLgue_RuYn3XitL5Rnt2A2H8)  
**App:** [marcevsanti-ux.github.io/Desinstalaciones-Alleata](https://marcevsanti-ux.github.io/Desinstalaciones-Alleata/)

---

## Resultado de la importación

| Indicador | Antes | Después |
|---|---|---|
| **TOTAL** | 1.645 | **2.062** |
| **PENDIENTES** | 1.051 | **1.194** |
| **EN PROCESO** | 127 | **42** |
| **DESPACHADAS** | 82 | **101** |
| **REPROGRAMAR** | — | **244** |
| **EXTRAVIADAS** | 213 | **6** |
| **OTROS** (Cancelado, Devuelta a Depósito, Fallido, etc.) | — | **475** |

> Las **78 OTs Exitosas** se mantienen en la base como registro histórico pero no aparecen en el dashboard ni en el TOTAL — ya fueron recuperadas.

---

## Detalle de cambios de estado (401 registros)

| Estado anterior | → Nuevo estado | Cantidad |
|---|---|---|
| Extraviada | → Reprogramar | 205 |
| En Proceso | → Despachado | 37 |
| Despachado | → **Exitoso** ✓ | 37 |
| En Proceso | → Pendiente | 33 |
| Pendiente | → **Exitoso** ✓ | 24 |
| Pendiente | → Reprogramar | 21 |
| En Proceso | → **Exitoso** ✓ | 15 |
| Despachado | → Reprogramar | 13 |
| Pendiente | → Despachado | 5 |
| Despachado | → Pendiente | 5 |
| Extraviada | → **Exitoso** ✓ | 2 |
| Pendiente | → Cancelado | 2 |
| Despachado | → Cancelado | 2 |
| **Total** | | **401** |

---

## Registros nuevos agregados (649)

| Estado | Cantidad |
|---|---|
| Cancelado | 428 |
| Pendiente | 150 |
| Devuelta a Depósito | 41 |
| Despachado | 24 |
| Reprogramar | 5 |
| Fallido | 1 |
| **Total** | **649** |

---

## OTs que pasaron a Exitoso (78)

OT #5416 · OT #7110 · OT #7688 · OT #9045 · OT #12637 · OT #15576 · OT #15620 · OT #16448 · OT #16797 · OT #16912 · OT #17532 · OT #17761 · OT #17890 · OT #17938 · OT #18080 · OT #18133 · OT #18270 · OT #18610 · OT #18626 · OT #18653 · OT #18724 · OT #18728 · OT #18777 · OT #18793 · OT #18796 · OT #18797 · OT #18812 · OT #18831 · OT #18864 · OT #18880 · OT #18918 · OT #19013 · OT #19068 · OT #19071 · OT #19114 · OT #19136 · OT #19162 · OT #19163 · OT #19164 · OT #19166 · OT #19175 · OT #19187 · OT #19204 · OT #19216 · OT #19232 · OT #19238 · OT #19240 · OT #19261 · OT #19267 · OT #19269 · OT #19281 · OT #19291 · OT #19308 · OT #19314 · OT #19325 · OT #19337 · OT #19354 · OT #19362 · OT #19363 · OT #19364 · OT #19397 · OT #19398 · OT #19400 · OT #19423 · OT #19430 · OT #19440 · OT #19448 · OT #19455 · OT #19460 · OT #19463 · OT #19468 · OT #19474 · OT #19481 · OT #19482 · OT #19491 · OT #19498 · OT #19502 · OT #19507

---

---

# Instructivo — Cómo actualizar la base mensualmente

## Lo que necesitás

- Planilla exportada de Salesforce (`.xlsx`) con todas las WorkOrders
- Acceso al Google Sheet (link arriba)
- Acceso a la app (link arriba)
- Acceso a Claude en [claude.ai](https://claude.ai)

---

## Paso a paso

### 1. Exportar planilla de Salesforce
Bajá el reporte de WorkOrders desde Salesforce en formato `.xlsx`. Asegurate de que incluya: `Id`, `WorkOrderNumber`, `Status`, `record_type_name_clickup__c`, `Logistica__c`, `Nombre_del_comercio__c` y los demás campos habituales.

### 2. Revisar el estado actual de la app
Anotá los indicadores del dashboard antes de actualizar para comparar después.

### 3. Subir la planilla a la tab Importar
En la app, andá a **Importar** y subí el `.xlsx`. La app muestra automáticamente los cambios detectados y la proyección de indicadores.

### 4. Exportar el informe PDF
Hacé clic en **📄 Exportar PDF** para guardar el registro antes de continuar.

### 5. Generar el TSV actualizado con Claude
Abrí una nueva conversación en Claude y pasale:
- La planilla nueva de Salesforce (`.xlsx`)
- El TSV actual de la base (`Hoja13_completa.tsv`)

Pedile: **"Actualizá el TSV con los estados de la planilla nueva"**

Claude devuelve un `Hoja13_completa.tsv` con todos los estados actualizados y los registros nuevos incorporados.

### 6. Importar el TSV en Google Sheets
1. Abrí el Google Sheet → pestaña **hoja 13 nueva**
2. **Archivo → Importar**
3. Subí el `Hoja13_completa.tsv`
4. Configuración: **Reemplazar hoja actual · Tabulador · ✅ Convertir texto**
5. **Importar datos**

### 7. Recargar la app
Presioná **F5**. Los indicadores deben reflejar los cambios.

### 8. Verificar
Compará con la proyección del paso 3. Deben coincidir.

---

## Notas importantes

- Solo se procesan **Desinstalaciones** — otros tipos se ignoran
- Las OTs **Exitosas** no aparecen en el dashboard — son históricas
- El TSV no debe tener filas vacías ni saltos de línea — Claude los limpia automáticamente
- Guardá siempre el PDF del informe como registro de auditoría

---

## Versiones del sistema

| Componente | Versión | Fecha |
|---|---|---|
| Portal administración (index.html) | v5.25 | 29/03/2026 |
| Portal distribuidor (distri.html) | v1.1.1 | 09/03/2026 |
| Google Sheet | hoja 13 nueva · gid=201120448 | — |
| Apps Script webhook | v3.2 | 29/03/2026 |
| Sheet ID | 1imDRJmnJn-W-yT23i3DdLgue_RuYn3XitL5Rnt2A2H8 | — |

---

*Alleata · Sistema de Gestión de Desinstalaciones · Marzo 2026*
