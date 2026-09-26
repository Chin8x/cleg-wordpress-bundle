# Procurement Phase 1 — Discovery, Audit and TODO

**Estado:** Discovery consolidado; implementación, montaje y publicación pausados hasta revisión/aprobación del Project Chief.

**Regla de lectura:** “Implementado” describe existencia previa en el bundle, no cierre. Los fixes deben considerarse **existentes, pendientes de verificación** hasta completar QA runtime, permisos, responsive y evidencia.

## Matriz por pantalla y elemento

| Pantalla / elemento | Encontrado | Benchmark | Decisión | To-do | Implementado | QA visual | QA funcional | Evidencia |
|---|---|---|---|---|---|---|---|---|
| Bandeja — navegación y colas | Bandeja, Cotizaciones/Pre-PO y Compras activas/PO aparecen como rutas/colas relacionadas; labels y jerarquía no son totalmente uniformes. | Dynamics/SAP separan requisición aprobada, PO y recepción; ServiceNow expone estados explícitos. | Mantener colas separadas, con contrato de estados compartido. | Unificar nomenclatura y estado activo; validar `aria-current`. | Parcial; existente, pendiente de verificación. | QA desktop previa: carga y filas visibles. | Falta matriz de permisos, estados y navegación completa. | QA previa de Bandeja; informes UX/Responsive. |
| Bandeja — fila de solicitud | Conserva solicitud, líneas/proyecto, fecha, riesgo y CTA; riesgo de métricas/IDs repetidos. | ServiceNow calcula estado desde líneas/tareas y muestra la siguiente acción. | Una fila por solicitud y un CTA derivado del estado. | Eliminar duplicidad semántica; documentar responsable y siguiente acción. | Parcial; pendiente de verificación. | Desktop observado. | Falta comprobar todos los estados y acciones. | QA Inspector; informe UX. |
| Cotizaciones — decisión pre-PO | Ruta `pre_po+ready_decision`; “Requiere decisión” debe distinguir bloqueo real de simple existencia de cotización. | Dynamics/SAP: requisición aprobada antes de crear/enviar PO; ServiceNow usa “Requires Decision”. | Mostrar decisión solo cuando cotización está lista y no hay dependencia abierta. | Definir transición `Approved → Requires Decision → PO Pending`. | Existente; pendiente de verificación. | Ruta cargó en QA previa. | Falta validar reglas de transición y permisos. | QA previa: 6 abiertas/decisión, 0 sin cotización, 0 vencidas. |
| Compras activas — PO/tracking | Muestra proveedor, PO, estado, ETA, tracking, recepción y acciones. Existió riesgo de PO placeholder/duplicado. | SAP: Ordered, Confirming, Confirmed, Shipping, Shipped, Receiving, Received; recepción parcial mantiene PO abierto. | Una sola entidad PO operativa por relación; conflictos solo en auditoría. | Verificar que `1055` sea el único PO operativo; validar tracking y recepción por record ID. | `59dec3b`, `cc7f710`, `d4d5068` existentes, pendientes de verificación. | Carga desktop previa; responsive pendiente. | P0 aún no certificado post-`d4d5068`. | Última QA previa mostró `Pendiente` y `1055`; evidencia no concluyente. |
| Compras activas — recepción | Formulario de recepción presente; estado/cantidad parcial requieren contrato explícito. | Dynamics registra cantidad, fecha e identificador de recibo; SAP mantiene abierto el PO hasta completar. | Soportar `Partially Received` y `Received`, nunca cerrar por recepción parcial. | Añadir/validar cantidad, receipt ID, fecha y siguiente acción. | Parcial; pendiente de verificación. | Responsive 390/360 pendiente. | Falta prueba parcial/completa y autorización. | Benchmark Dynamics/SAP; informe Backend. |
| Alertas | Bloque debe existir solo con alertas accionables. | ServiceNow usa tareas/condiciones explícitas como “Awaiting Task Completion”. | Ocultar cuando count=0; cada alerta enlaza al registro y explica la acción. | Probar cero, una y múltiples alertas; revisar enlaces. | Parcial; pendiente de verificación. | Falta evidencia específica. | Falta prueba de enlaces y conteos. | Informe UX; requisito de aceptación. |
| Nueva requisición — wizard | Wizard de 3 pasos, validación y ARIA; debe quedar fuera de la cola operativa. | Dynamics/SAP tratan la requisición como documento previo a aprobación/PO. | Mantener separación creación vs. decisión/ejecución. | Validar estados Draft/Review/Approved y errores/empty states. | Existente; pendiente de verificación. | Responsive pendiente, especialmente 360 px. | Falta validar submit, nonce y capabilities. | Informe Responsive; QA pendiente. |
| Detalle — hero y siguiente acción | Hay hero de estado, tabs y acciones; riesgo de mezclar decisión y logística. | ServiceNow deriva siguiente acción de estado/tareas; Dynamics separa aprobación y confirmación. | Un estado principal y una única próxima acción visible. | Definir responsable, estado y CTA por transición. | Existente; pendiente de verificación. | Ruta cargó; falta evidencia 1440/1024/390/360. | Última QA previa mostró 2 PO; no PASS. | `CLEG-REQ-2026-0018`; QA previa. |
| Detalle — cadena de trazabilidad | Debe enlazar solicitud→cotización→PO→tracking→recepción sin duplicar entidades. | Dynamics/SAP preservan identificadores, cantidades y fechas en documentos/receipts. | Mostrar cadena completa y append-only history. | Verificar IDs Airtable, PO canónico y receipt IDs. | `59dec3b`, `cc7f710`, `d4d5068` existentes, pendientes. | Pendiente. | Pendiente de verificación post-sync. | Informe Backend; benchmark oficial. |
| Detalle — Update cliente | Update cliente debe vivir solo en detalle/vista cliente; antes apareció fuera de esa vista. | DTO externo debe exponer solo información aprobada y contextual. | Mantenerlo fuera de Bandeja, Compras activas e Historial. | Verificar ausencia en rutas no cliente y allowlist del DTO. | `b1dcd6c` existente, pendiente de verificación. | QA previa confirmó ausencia en tres rutas tras sync. | Falta repetir con bundle actual y probar ownership/nonce. | QA previa; informe Security. |
| Historial | Auditoría secundaria con actor, fecha, estado e identificador. | Dynamics conserva diarios de cantidades/fechas; ServiceNow calcula estados desde trazabilidad. | Append-only; nunca sustituir estado operativo. | Validar orden, actor, timestamps y filtros. | Existente; pendiente de verificación. | Carga previa confirmada. | Falta prueba de inmutabilidad y permisos. | QA previa de Historial. |
| Adjuntos | Adjuntos presentes como capacidad; faltan límites/almacenamiento privado documentados. | Buenas prácticas de procurement requieren trazabilidad y acceso controlado. | MIME/tamaño limitados, almacenamiento privado y URLs temporales. | Definir política y pruebas de descarga/ownership. | Parcial; pendiente. | Pendiente. | Pendiente. | Informe Security. |
| Responsive — 1024/720/390/360 | Reglas existentes para grids, accordions y navegación; riesgo de overflow de CTA. | Interfaces benchmark mantienen estado/acción legibles en cada viewport. | `min-width:0`, `max-width:100%`, wrapping seguro y targets táctiles ≥44 px. | QA visual con screenshots y `scrollWidth`; registrar regresiones. | Ajuste local de overflow existente, sin verificar ni publicar. | Pendiente. | Pendiente. | P0 alrededor de línea 39807; informe Responsive. |
| Seguridad — ownership y mutaciones | Debe usarse usuario WP + tenant + request; mutaciones por record ID y nonce/capability. | Estados/tareas auditables requieren actor y autorización explícitos. | Prohibir matching textual; rechazar ambigüedad. | Probar permisos, nonce, tenant y `po_ambiguous`. | P0 parcial; pendiente de verificación. | No aplica. | Pendiente de pruebas negativas. | Informe Security/Backend. |

## Contrato funcional propuesto

- Estados mínimos: `Draft`, `Review`, `Approved`, `Requires Decision`, `PO Pending`, `Ordered`, `In Transit`, `Partially Received`, `Received`, `Closed`, `Canceled`.
- Regla dura: una requisición debe estar aprobada antes de generar/enviar un PO.
- PO canónico: Airtable record ID + request record ID + número normalizado; duplicados se conservan como conflicto/auditoría.
- Recepción: cantidad, receipt ID y fecha; la recepción parcial mantiene el PO abierto.
- CTA, responsable y siguiente acción se calculan desde estado y tareas reales.

## Benchmark oficial

- [Microsoft Dynamics — Purchase requisitions workflow](https://learn.microsoft.com/en-us/dynamics365/supply-chain/procurement/purchase-requisitions-workflow)
- [Microsoft Dynamics — Approve and confirm purchase orders](https://learn.microsoft.com/en-us/dynamics365/supply-chain/procurement/purchase-order-approval-confirmation)
- [Microsoft Dynamics — Product receipt against purchase orders](https://learn.microsoft.com/vi-vn/dynamics365/supply-chain/procurement/product-receipt-against-purchase-orders)
- [SAP Ariba — Purchase orders](https://help.sap.com/docs/buying-invoicing/shopping-guide-for-business-purchases/working-with-purchase-orders)
- [SAP Ariba — Receiving overview](https://help.sap.com/docs/buying-invoicing/purchasing-guide-for-procurement-professionals/about-receiving)
- [ServiceNow — Purchase requisition, purchase order, and sourcing request states](https://www.servicenow.com/docs/r/source-to-pay-operations/sourcing-and-procurement-operations/pr-po-sr-states.html?contentId=20JITDQpF9H4CUyKfGrwLg)

## Roadmap condicionado a aprobación

1. **F0 — Discovery/contrato:** cerrar los cinco informes, estados, matriz y decisión interna.
2. **F1 — Datos:** PO canónico, idempotencia, conflictos no destructivos y recepción parcial.
3. **F2 — Flujo/UI:** aprobación→PO, estados uniformes, CTA único, detalle sin duplicados y alertas accionables.
4. **F3 — Seguridad:** ownership WP+tenant+request, nonce/capability, DTO allowlist y adjuntos privados.
5. **F4 — Responsive/a11y:** 1440/1024/720/390/360, `aria-current`, estados loading/error/empty y CTA ≥44 px.
6. **F5 — QA/montaje:** pruebas negativas, runtime post-sync, hash, evidencia visual/funcional y publicación solo tras PASS.

## Registro de fixes existentes

| Fix | Estado de auditoría |
|---|---|
| `59dec3b` — identidad canónica/idempotencia de PO | Existente, pendiente de verificación |
| `cc7f710` — ocultación de placeholders operativos | Existente, pendiente de verificación |
| `d4d5068` — filtro defensivo de placeholders en relaciones | Existente, pendiente de verificación |
| `b1dcd6c` — Update cliente fuera de vista cliente | Existente, pendiente de verificación |

**Decisión actual:** NO READY. No implementar, montar ni publicar hasta revisión/aprobación del Project Chief y cierre de QA P0/P1.
