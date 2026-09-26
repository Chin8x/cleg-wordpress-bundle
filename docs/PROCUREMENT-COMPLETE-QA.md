# Procurement — Complete QA Before Modification

**Fecha:** 2026-09-25  
**Alcance:** Compras, sesión Chrome autenticada, solo lectura. No se crearon solicitudes, cotizaciones, PO, tracking ni recepciones.  
**Estado global:** diagnóstico; no autoriza implementación, montaje ni publicación.

## Regla de estados

- **Verificado:** observado en navegador real y coherente con el criterio indicado.
- **Corregido:** existe un ajuste local previo, pero no está publicado o no tiene verificación runtime posterior.
- **Abierto:** hallazgo o prueba pendiente.

## Resumen ejecutivo de cierre de auditoría — 2026-09-26

### Alcance auditado

Bandeja, vistas de cotizaciones y compras activas, Nueva solicitud, Detalle, PO/Tracking, Update para cliente, Historial, Recibos, rutas protegidas, permisos Procurement y acceso anónimo. Se usó la versión runtime publicada disponible y se contrastó con el bundle local y `origin/main`.

### Resultado

**Auditoría diagnóstica: completada para el alcance observado.**

**QA de errores/release: no aprobado.** La aplicación tiene fallos P0/P1 confirmados y pruebas runtime aún no verificables. No se autoriza implementación, montaje ni publicación como versión final hasta corregirlos y repetir la matriz.

### Bloqueadores P0 confirmados

1. ID ajeno o inexistente puede renderizar otra requisición.
2. Usuario Procurement puede alcanzar el Panel de trabajador y acciones laborales.
3. Recepción parcial no tiene persistencia remota confiable de cantidad/evento.
4. Autorización de Recibos parece más amplia que el rol de Procurement.
5. Responsive estrecho puede forzar overflow por reglas `min-width:max-content`.

### Evidencia positiva

- Acceso anónimo redirige al login en Procurement, Recibos y Panel.
- Login y logout de `cgarin` funcionan.
- Filtros, búsqueda, limpieza y CTA principal de Bandeja funcionan en los casos probados.
- PHP lint de la copia local pasa.
- `git diff --check` pasa.
- Los handlers tienen nonces estáticos; esto es solo PASS parcial.

### Condiciones de salida del QA

El QA solo podrá marcarse cerrado después de demostrar, sobre el mismo hash montado: aislamiento por ID/tenant, permisos negativos, idempotencia de PO/tracking/recepción, persistencia remota confirmada, recibos editables con auditoría, estados de error/éxito, viewport 1440/1024/390/360 sin overflow y navegación estable.
- **NO VERIFICABLE:** no se pudo ejecutar sin viewport/control/permiso/estado de prueba disponible. Nunca equivale a PASS.

## Evidencia de ejecución

- Pestaña real: `Compras - Vantrexor`, URL `clegllc.com/admin-procurement/`.
- QA runtime observada: Bandeja (`proc_stage=all`), Cotizaciones (`pre_po&proc_order_state=ready_decision`), Compras activas (`po`), Detalle `CLEG-REQ-2026-0018`, Nueva solicitud y Historial.
- Compras activas: 5 filas visibles; resumen 5 abiertas, 0 sin cotización, 0 por decidir, 1 en camino, 1 atrasada.
- Detalle: `Ordenes, llegada y recepcion 1 PO`; único registro visible `PO QuickBooks: 1055`, proveedor `jc-electronics`, estado `Ordered`, tracking `Pendiente`, ETA `2026-09-11`; controles de recepción `Recibido parcial`, `Recibido completo`, `Cerrar`.
- Ausencia observada: “Update cliente” no apareció en Bandeja, Cotizaciones, Compras activas, Detalle ni Historial durante esta sesión.
- Se intentó captura visual y back/refresh; la sesión del navegador se interrumpió. Esos casos quedan NO VERIFICABLES, no PASS.
- No fue posible fijar viewport explícito a 1440/1024/390/360 con la sesión disponible; las cuatro medidas quedan NO VERIFICABLES.
- Actualización QA: PHP 8.4 `php -l cleg-active-snippets.php` pasó sin errores; `reload` de Bandeja recargó correctamente con 21 solicitudes y los mismos filtros/contadores. `Back` volvió a interrumpir la sesión y permanece NO VERIFICABLE.

## Evidencia adicional 2026-09-25

- **PO / TRACKING:** conteo runtime visible `5`; estado **Verificado**.
- **PO canónica:** Detalle `CLEG-REQ-2026-0018` muestra `Ordenes, llegada y recepcion 1 PO` y únicamente `PO QuickBooks: 1055`; estado **Verificado**.
- **Wizard:** Nueva solicitud muestra Paso 1 de 3; Continuar sin Proyecto/equipo requerido mantiene el formulario y muestra “Completa los campos obligatorios”; estado **Verificado** sin crear datos.
- **Terminadas:** enlace runtime visible con conteo `18`; el conteo queda **Verificado**. La separación interna Received/Closed sigue pendiente.
- **Canceladas:** enlace runtime visible con conteo `7`; el conteo queda **Verificado**. La auditoría de cancelación sigue pendiente.
- **Recibos — paginación/filtro:** runtime verificado en `/admin-recibos/`: disclosure Día/Proyecto/Persona/Monto/Estado, botones Filtrar/Limpiar; `?receipt_page=2` muestra `Página 2 de 6 · 147 recibos` con links Anterior/Siguiente. Edición, permisos y mutaciones siguen **NO VERIFICABLES**.
- **Reload persistente:** Bandeja recargada y conservó ruta, 21 solicitudes y contadores; **Verificado**. Back/forward siguen abiertos.
- **Screenshot desktop:** la captura visual del navegador agotó el tiempo de espera y no se encontró un artefacto de captura de Compras en el workspace; permanece **NO VERIFICABLE**. Viewports 1440/1024/390/360 siguen **NO VERIFICABLES**.

## Inspección estática QA/Security

- PHP 8.4 `php -l cleg-active-snippets.php`: **PASS**.
- Los handlers operativos inspeccionados (`update_status`, `cancel_request`, `create_po`, `update_po_tracking`, `receive_po`) tienen guard de login/capability y `check_admin_referer`; esto es evidencia estática, no sustituye pruebas runtime de permisos/nonces.
- `cleg_procurement_request_belongs_to_user()` comprueba IDs de usuario, pero también permite fallback por display name, login, email y nombre textual; tenant scoping y rechazo de matching textual siguen abiertos como P0/P1.
- `cleg_procurement_handle_receive_po()` registra estado nuevo y nota con timestamp/usuario mediante auditoría, pero no conserva old value/new value estructurados ni evidencia/motivo separado; queda abierto para el requisito de edición append-only.

## Evidencia estática adicional 2026-09-26

- `cleg_admin_user_allowed` (`cleg-active-snippets.php:6251-6275`) autoriza, además de comprobaciones específicas, mediante `manage_options`, `edit_pages` y roles amplios (`administrator`, `editor`, `hr_manager`, `rrhh`, `cleg_admin`). **Hallazgo abierto P0/P1:** autorización demasiado amplia para mutaciones de Recibos; falta verificar y limitarla a capability/ownership de procurement.
- `cleg_procurement_user_can_manage_requests` (`cleg-active-snippets.php:30689-30702`) comprueba `manage_options` o `cleg_manage_procurement` y delega en la función de gestión. Es evidencia de guard estático, no prueba runtime; el grant amplio de `manage_options` queda abierto.
- `cleg_procurement_add_airtable_po` (`cleg-active-snippets.php:33479-33492`) contiene comprobación determinista de idempotencia por enlace de solicitud Airtable + número QuickBooks normalizado. La existencia de la guardia es verificable estáticamente; el replay runtime queda pendiente.
- Handler de Recibos `cleg_admin_handle_receipt_action` (`cleg-active-snippets.php:43454-43518`) exige login/guard, nonce y permite `archive`, `review` y `resolved`; actualiza estado, revisor, fecha y nota/auditoría. **Hallazgo abierto P0/P1:** no hay edición de persona, proyecto, compra o cantidad ni clave/supresión de idempotencia específica para mutaciones de recibos.

## Matriz completa de hallazgos

| Pantalla / elemento | Existencia / necesidad | Ubicación / función / destino | Resultado esperado / real | Evidencia | Sev. | Benchmark | Recomendación / criterio de aceptación | Estado |
|---|---|---|---|---|---|---|---|---|
| Navegación principal: Bandeja | Existe y es necesaria | Header; lleva a `proc_view=pendientes&proc_stage=all` | Esperado: ruta estable. Real: carga y mantiene contexto | AX runtime, link Bandeja | P1 | ServiceNow separa work queues por estado | Validar `aria-current`, back y refresh | Verificado |
| Navegación principal: Nueva solicitud | Existe y es necesaria | Header; lleva a `proc_view=solicitar` | Real: abre wizard | AX runtime | P1 | Dynamics/SAP separan requisición de PO | Mantener fuera de cola operativa | Verificado |
| Navegación principal: Recibos | Existe | Header; lleva a `/admin-recibos/` | Ruta, filtros disclosure y paginación runtime verificados; edición/permisos/mutación no verificables | `/admin-recibos/`; `receipt_page=2`: Página 2 de 6 · 147 recibos; Anterior/Siguiente | P1 | Dynamics/SAP identifican recibos/product receipts | Verificar edición segura, permisos y auditoría | Verificado |
| Navegación principal: Historial | Existe y es necesaria | Header; lleva a `proc_view=historial` | Real: ruta carga; contenido semántico incompleto en AX | AX runtime | P1 | Trazabilidad append-only | Exponer actor, fecha, estado e ID | Abierto |
| Navegación: Panel / Cerrar sesión | Existen | Header; Panel externo y logout con nonce visible | No se ejecutó logout | Links visibles | P2 | Acceso/ownership explícitos | Verificar permisos y destino | Abierto |
| Filtros principales: TODAS 21 | Existe | Bandeja; filtro `all` | Real: 21 visibles en resumen | AX runtime | P1 | Estados/colas explícitos | Mantener conteo derivado de datos | Verificado |
| Filtros principales: ANTES DE PO 16 | Existe | Bandeja; filtro `pre_po` | Real: enlace y conteo visibles | AX runtime | P1 | Aprobación antes de PO | Reconciliar con estados contractuales | Verificado |
| Filtros principales: PO / TRACKING 5 | Existe | Bandeja; filtro `po` | Real: 5 filas activas | AX runtime | P0 | SAP Ordered/Shipping/Receiving | Un PO operativo canónico por solicitud | Verificado |
| Filtros principales: TERMINADAS 18 | Existe | Bandeja; filtro `done` | Conteo runtime `18` verificado; flujo interno no recorrido | AX runtime | P1 | Received/Closed separados | Verificar Received vs Closed | Verificado |
| Filtros principales: CANCELADAS 7 | Existe | Bandeja; filtro `cancelled` | Conteo runtime `7` verificado; flujo interno no recorrido | AX runtime | P1 | Cancelled explícito | Verificar read-only/auditoría | Verificado |
| Vistas: Bandeja / Cotizaciones / Compras activas / Archivo | Existen y son necesarias | Panel “Vistas de Compras” | Real: cuatro enlaces y destinos coherentes | AX runtime | P1 | Separación PR/PO/receipt/history | Unificar labels y estado activo | Verificado |
| Filtros rápidos: Pendientes / Cotizando / Listas para decidir / Aprobadas | Existen | Bandeja y Cotizaciones | Real: enlaces visibles y filtrables por URL | AX runtime | P1 | ServiceNow Requires Decision; Dynamics approval | Validar transición Approved→PO Pending | Verificado |
| Filtros rápidos: En camino / Recibidas / Con problema | Existen | Compras/Bandeja | Real: enlaces visibles; estados destino no todos recorridos | AX runtime | P1 | SAP In Transit/Receiving/Received | Probar cada estado y siguiente acción | Abierto |
| Botón Columnas | Existe | Bandeja; disclosure de columnas | Real: visible y expandible; contenido no recorrido | AX runtime | P2 | Interfaces operativas muestran contexto suficiente | Verificar columnas y persistencia | Abierto |
| Botón Filtros buscar/ordenar | Existe | Bandeja/Cotizaciones | Real: expande BUSCAR, PRIORIDAD, ESTADO, COMPRA, Filtrar, Limpiar | AX runtime | Estados explícitos y filtrables | Probar combinaciones sin mutación | Verificado |
| Campo BUSCAR | Existe | Panel de filtros | Real: search field accesible | AX runtime | Búsqueda por documento/solicitud | Probar búsqueda y estado vacío | Abierto |
| Select PRIORIDAD | Existe | Panel de filtros; Todas/Normal/Alta/Urgente/Detiene trabajo hoy | Real: opciones accesibles | AX runtime | Priorización accionable | Verificar orden y etiqueta | Verificado |
| Select ESTADO | Existe | Panel de filtros; 13 estados observados | Real: opciones accesibles, incluido PO creada, Ordenada, En camino, Recibido parcial, Recibido, Cerrada | AX runtime | Dynamics/SAP/ServiceNow | Normalizar estados y transiciones | Verificado |
| Select COMPRA | Existe | Panel de filtros; 9 agrupaciones | Real: opciones accesibles | AX runtime | SAP lifecycle | Evitar duplicidad con ESTADO | Verificado |
| Botón Filtrar / enlace Limpiar | Existen | Panel de filtros | Real: visibles; no se validó resultado después de aplicar | AX runtime | Filtros reproducibles | Probar resultado, back y refresh | Abierto |
| Resumen de decisiones | Existe | Bandeja/Cotizaciones/Compras activas | Real: muestra abiertas, sin cotización, por decidir, en camino, atrasadas, estimado pendiente | AX runtime | ServiceNow estados/tareas | Cada métrica debe enlazar a causa | Verificado |
| Tarjeta de solicitud — prioridad | Existe | Cada fila/accordion | Real: Normal/High/Urgent visible | AX runtime, 21 filas Bandeja | P2 | Priorización operativa | Mantener etiqueta y contraste | Verificado |
| Tarjeta — cliente/proyecto/producto | Existe y es necesaria | Cada fila | Real: cliente, proyecto, descripción y artículo visibles | AX runtime | P1 | Trazabilidad por requisición | No truncar identificadores críticos | Verificado |
| Tarjeta — estado/fecha/entrega | Existe | Cada fila | Real: estado, ETA/fecha o Pendiente visibles | AX runtime | P1 | SAP/Dynamics status + receipt dates | Estados calculados; fecha semántica única | Verificado |
| Tarjeta — CTA siguiente acción | Existe | Cada fila; Actualizar recepción, Registrar cotización, Comparar/crear PO, etc. | Real: CTA único y siguiente acción visible | AX runtime | P0 | ServiceNow next action | CTA derivado del estado y responsable | Verificado |
| Tarjeta expandida — Imprimir / Descargar | Existen | Ficha de requisición | Real: botones visibles; no se ejecutaron descargas | AX runtime | P2 | Trazabilidad documental | Verificar destino y permisos | Abierto |
| Tarjeta expandida — Editar datos básicos | Existe | Ficha de requisición | Real: botón visible; no se abrió para evitar mutación | AX runtime | P1 | Ownership/capabilities | Verificar nonce/capability antes de guardar | Abierto |
| Tarjeta expandida — selector ESTADO | Existe | Ficha de requisición | Real: menú accesible con estados operativos | AX runtime | P0 | Estados calculados/approval gate | Probar autorización y transición inválida | Abierto |
| Compras activas — PO canónico | Necesario | Detalle expandido y sección de órdenes | Real: único `PO QuickBooks: 1055`; no aparece placeholder | Runtime detalle | P0 | PO por identidad canónica | Probar duplicado/ambigüedad y persistencia | Verificado |
| Compras activas — proveedor/monto/estado | Necesario | Detalle de orden | Real: `jc-electronics · $1,020.00 · Ordered` | Runtime detalle | P1 | SAP PO lifecycle | Estado debe corresponder a datos reales | Verificado |
| Tracking / ETA fields | Necesarios | Formulario de orden | Real: Tracking Pendiente, ETA 2026-09-11, campos accesibles | Runtime detalle | P1 | Dynamics receipt/shipping trace | Mutar solo por record ID + nonce | Abierto |
| Select recepción | Necesario | Formulario de orden | Real: Recibido parcial seleccionado; Recibido completo/Cerrar disponibles | Runtime detalle | P0 | SAP partial receipt; Dynamics quantity receipt | Añadir cantidad/receipt ID/fecha y probar parcial | Abierto |
| Recibos — corrección de datos | Necesario; hallazgo P0/P1 | Detalle/Recibos para persona, proyecto, compra o cantidad | No verificable sin edición controlada; no debe sobrescribir silenciosamente | Requisito de auditoría; sin mutación ejecutada | P0/P1 | Trazabilidad append-only de Dynamics/SAP/ServiceNow | Permitir corregir esos cuatro campos y registrar valor anterior, valor nuevo, usuario, fecha/hora, motivo y evidencia en auditoría append-only | Abierto |
| Recibos — autorización amplia | Hallazgo P0/P1 | `cleg_admin_user_allowed()` líneas 6251-6275 | `manage_options`, `edit_pages` y roles amplios pueden habilitar la acción; no probado runtime | Inspección estática | P0/P1 | Capability y ownership mínimos | Limitar mutaciones a capability de procurement, user ID/tenant/request y pruebas negativas | Abierto |
| Recibos — edición/idempotencia | Hallazgo P0/P1 | `cleg_admin_handle_receipt_action()` líneas 43454-43518 | Solo acciones de estado/nota; no edición de campos ni idempotencia de recibos | Inspección estática; no mutación ejecutada | P0/P1 | Corrección append-only e idempotencia | Implementar edición controlada de persona/proyecto/compra/cantidad y replay seguro con auditoría | Abierto |
| Security — ownership | Necesario | Filtro/edición de solicitudes | Static: ID de usuario + fallback textual; tenant/request scoping no queda probado | Inspección `cleg_procurement_request_belongs_to_user()`; sin prueba runtime | P0/P1 | Ownership por user ID + tenant + request | Eliminar matching textual; exigir WP user ID, tenant ID y request ID | Abierto |
| Security — auditoría de recepción | Necesario | Handler `receive_po` | Static: guarda estado nuevo, nota, timestamp y usuario; no old/new estructurados ni evidencia separada | Inspección handler + `cleg_procurement_add_audit()` | P0/P1 | Auditoría append-only | Registrar old value, new value, user, timestamp, motivo y evidencia por edición | Abierto |
| Botones Guardar tracking / Guardar recepción | Existen | Formularios de orden | No se pulsaron para no mutar datos | AX runtime | P0 | Idempotencia y auditoría | Probar con datos controlados y permisos | Abierto |
| Registro PO real QuickBooks | Existe | Detalle de orden | Real: campos PO, proveedor, monto y ETA visibles | Runtime detalle | P0 | Reconciliación PO canónico | No permitir placeholder como operativo | Verificado |
| Botón Asignar PO QuickBooks | Existe | Registro PO real | No se ejecutó | AX runtime | P0 | Aprobación antes de PO | Probar nonce, idempotencia y duplicado | Abierto |
| Detalle — estado hero | Existe | Cabecera | Real: PO creada, decisión y aviso de tracking visibles | Runtime detalle | P1 | ServiceNow next action | Un estado principal + una acción | Verificado |
| Detalle — aprobación de cliente | Existe | Hero | Real: No enviado al cliente / Sin envío | Runtime detalle | P1 | DTO allowlist y ownership | Verificar endpoint, tenant y nonce | Abierto |
| Detalle — tabs/secciones | Existen | Resumen, Cotizaciones, Compras activas, Historial | Real: enlaces visibles | Runtime detalle | P1 | Cadena PR→quote→PO→receipt | `aria-current` y anchors verificables | Verificado |
| Detalle — Datos básicos / Historial / Estado y correcciones / Cotizaciones | Existen | Accordions | Real: cuatro disclosures visibles | Runtime detalle | P1 | Audit trail | Recorrer contenido y read-only | Abierto |
| Detalle — Ordenes, llegada y recepción | Existe | Accordion | Real: `1 PO`, contenido expandido y formularios visibles | Runtime detalle | P0 | SAP/Dynamics receiving | PASS solo tras probar cantidades y permisos | Abierto |
| Nueva solicitud — pasos 1/2/3 | Existen | Wizard | Real: Paso 1 de 3 y enlaces Necesidad/Entrega/Detalles | Runtime wizard | P1 | PR antes de approval/PO | Mantener fuera de colas operativas | Verificado |
| Nueva solicitud — Proyecto | Existe y es obligatorio | Paso 1 select | Real: placeholder + proyectos; vacío impide continuar | Runtime wizard | P1 | Requisition required fields | Mensaje de validación claro | Verificado |
| Nueva solicitud — Equipo/material/servicio | Existe y es obligatorio | Paso 1 field | Real: campo accesible; vacío impide continuar | Runtime wizard | P1 | Requisition line description | Probar límites y caracteres | Verificado |
| Nueva solicitud — Cantidad / Unidad | Existen | Paso 1 | Real: stepper 1 y Unidad; opciones Unidad/Set/Paquete/Pie/Metro/Servicio/Viaje | Runtime wizard | P1 | Quantity-based receiving | Validar min/max y unidad | Verificado |
| Nueva solicitud — Continuar | Existe | Paso 1 | Real: click sin datos muestra “Completa los campos obligatorios”; no crea datos | Runtime wizard | P1 | Required approval input | Verificar foco/ARIA de error | Verificado |
| Historial — accordions | Existe | Archivo/Historial | Real: múltiples disclosures y enlaces de adjuntos visibles; texto semántico insuficiente en AX | Runtime Historial | P1 | Append-only audit | Exponer texto, actor, fecha, estado e ID | Abierto |
| Historial — adjuntos | Existen | Dentro de registros | Real: URLs Airtable/WordPress visibles | Runtime Historial | P1 | Private storage/temporary URLs | Verificar MIME, tamaño, ownership y privacidad | Abierto |
| Update cliente fuera de vista cliente | Necesidad negativa | Bandeja, Cotizaciones, Compras activas, Detalle, Historial | Real: no observado durante la sesión | AX runtime | P0 | DTO scoped | Repetir con permisos y endpoint | Verificado |
| Overflow CTA | Necesidad | CSS procurement alrededor de línea 39807 | Ajuste local elimina `min-width:max-content`, permite wrapping y ≥44px; bundle no publicado | Diff local; runtime responsive no ejecutado | P0 | Touch targets y responsive | Verificar 1440/1024/390/360 y `scrollWidth` | Corregido |
| Screenshot desktop de Compras | Necesario | Evidencia visual de rutas desktop | No se conserva captura verificable; captura intentada agotó timeout | Sesión navegador / sin artefacto local | P1 | QA visual reproducible | Capturar Bandeja, Cotizaciones, Compras activas, Detalle y Recibos en desktop | Abierto |
| Viewport 1440 | Necesario | QA visual | No se pudo fijar/medir viewport en sesión | Limitación de herramienta | P0 | Responsive acceptance | Capturar screenshot y `scrollWidth` | Abierto |
| Viewport 1024 | Necesario | QA visual | No verificable | Limitación de herramienta | P0 | Responsive acceptance | Ejecutar con viewport fijo | Abierto |
| Viewport 390 | Necesario | QA móvil | No verificable | Limitación de herramienta | P0 | Mobile 44px CTA | Ejecutar con viewport fijo | Abierto |
| Viewport 360 | Necesario | QA móvil | No verificable | Limitación de herramienta | P0 | Mobile 44px CTA | Ejecutar con viewport fijo | Abierto |
| Back / refresh | Necesario | Navegación | Reload de Bandeja verificado con ruta, 21 solicitudes y contadores; Back/forward interrumpen sesión y no son concluyentes | Runtime reload + interruption | P1 | Navegación estable | Repetir back/forward en todas las rutas sin perder filtros | Abierto |
| Loading / empty / error / success | Necesarios | Todas las rutas/formularios | No se forzaron sin crear/mutuar datos | QA safe-mode | P1 | Estados explícitos ServiceNow | Fixtures o entorno de prueba; documentar mensajes | Abierto |
| Permisos / capabilities / nonces | Necesarios | Formularios y endpoints de mutación | No verificable sin usuarios/acciones controladas | QA safe-mode | P0 | Ownership + actor explícitos | Matriz admin/manager/requester/client y nonce negative tests | Abierto |

## Checklist TODO de QA

- [ ] Repetir runtime post-montaje y confirmar un único PO `1055` en detalle y Compras activas.
- [ ] Verificar `59dec3b`, `cc7f710`, `d4d5068` y `b1dcd6c` como fixes existentes, no cerrados, con evidencia post-sync.
- [ ] Probar `po_ambiguous`, record ID, request ID, idempotencia y doble envío sin crear datos reales en producción.
- [ ] Habilitar corrección controlada de persona, proyecto, compra y cantidad en Recibos.
- [ ] Reducir `cleg_admin_user_allowed()` a capability/ownership de procurement; probar explícitamente rechazo de usuarios con autorización amplia no aplicable.
- [ ] Implementar y probar edición e idempotencia de mutaciones de Recibos; actualmente el handler solo procesa estado/nota.
- [ ] Por cada corrección, exigir valor anterior, valor nuevo, usuario, fecha/hora, motivo y evidencia en auditoría append-only; prohibir sobrescritura silenciosa.
- [ ] Eliminar fallback de ownership por nombre/login/email; exigir WP user ID + tenant ID + request ID.
- [ ] Ejecutar pruebas negativas de tenant/request cruzado y nonce/capability con usuarios controlados.
- [ ] Ejecutar 1440/1024/390/360 con screenshots, `scrollWidth`, wrapping y targets táctiles ≥44 px.
- [ ] Recorrer todos los botones y enlaces: filtros, Columnas, Filtrar, Limpiar, Imprimir, Descargar, Editar, Abrir y continuar, tracking, recepción y asignación PO.
- [ ] Validar back/forward/refresh preservando ruta, filtros, accordion y estado.
- [x] Validar reload de Bandeja: carga correcta, 21 solicitudes y contadores conservados.
- [ ] Validar back/forward: la sesión se interrumpe durante la navegación; mantener NO VERIFICABLE hasta repetir en entorno estable.
- [ ] Validar estados loading, empty, error y success con fixtures o entorno seguro.
- [ ] Probar permisos, capabilities, nonce, ownership por WP user + tenant + request y DTO cliente allowlist.
- [ ] Validar `aria-current`, foco de errores, nombres accesibles, orden de teclado y contraste.
- [ ] Validar adjuntos: MIME, tamaño, almacenamiento privado, URL temporal y ownership.
- [ ] Actualizar hash/checksums solo después de aprobar el bundle final.
- [ ] Solo después de revisión del Project Chief: commit, montaje, sync, QA post-sync y publicación.

## Roadmap condicionado a aprobación

1. F0: cerrar discovery y contrato de estados.
2. F1: identidad canónica PO, idempotencia, conflictos no destructivos, recepción parcial y correcciones de Recibos con auditoría append-only.
3. F2: aprobación→PO, estados uniformes, CTA único y detalle sin duplicados.
4. F3: seguridad, ownership, nonce/capability, DTO y adjuntos.
5. F4: responsive/a11y y estados de carga/error/empty/success.
6. F5: QA completa, checksums, montaje y publicación únicamente tras PASS verificable.

**Decisión actual:** NO READY. No implementar ni publicar hasta cerrar los hallazgos P0/P1 y obtener aprobación del Project Chief.

## Evidencia estática adicional — 2026-09-26

- `cleg_admin_handle_receipt_action()` (`cleg-active-snippets.php:43454-43518`) solo permite `archive`, `review` y `resolved`; no existe acción de edición para persona, proyecto, compra o cantidad.
- Ese handler protege con `cleg_admin_user_allowed()` (`6251-6275`), que incluye `manage_options`, `edit_pages` y roles `administrator`, `editor`, `hr_manager`, `rrhh` y `cleg_admin`. Esto es más amplio que la capacidad específica de Procurement y queda como hallazgo P1 de autorización.
- Las acciones de recibos escriben auditoría de estado (`Status`, `Reviewed By`, `Reviewed At`, `Review Note`), pero no tienen idempotency key ni deduplicación de replay; un doble envío puede repetir la operación/auditoría. Hallazgo P1.
- Después del PATCH de recibo (`43504-43517`) se redirige con éxito sin relectura posterior para confirmar que Airtable persistió el nuevo estado. Hallazgo P2.
- La protección estática de PO (`cleg_procurement_add_airtable_po()`, `33479-33492`) compara requisición + número QuickBooks normalizado y devuelve el registro existente antes de crear otro. Es PASS estático parcial; doble clic, conflictos ambiguos, permisos y persistencia siguen pendientes de runtime.
- La corrección futura de recibos deberá conservar valor anterior, valor nuevo, usuario, fecha/hora, motivo y evidencia en auditoría append-only; nunca sobrescribir silenciosamente.
- El escaneo estático encontró `wp_nonce_field` y `check_admin_referer` pareados para los handlers de requisición, estado, cancelación, adjuntos, cotizaciones, PO, tracking, recepción e historial (`cleg-active-snippets.php:35399-36418`); queda PASS estático parcial hasta ejecutar pruebas negativas de rol/nonce.
- `cleg_procurement_render_client_update()` se invoca únicamente para `proc_view=cliente` (`cleg-active-snippets.php:38461-38488`, llamada en `39000-39002`) y el contenido no imprime precios, cotizaciones ni márgenes; queda PASS estático parcial, pendiente de verificación por rol y endpoint.

## Reconciliación estática adicional — 2026-09-26

- PHP 8.4 volvió a validar `cleg-active-snippets.php`: **PASS sintáctico**.
- Se localizaron los handlers operativos reales: `cleg_procurement_handle_create_po()` (línea 36017), `cleg_procurement_handle_update_po_tracking()`, `cleg_procurement_handle_receive_po()` y `cleg_admin_handle_receipt_action()` (línea 43455). No existe un handler separado de edición de recibos; el hallazgo de edición con auditoría sigue abierto.
- El nombre correcto para las pruebas de tracking/recepción es `update_po_tracking` / `receive_po`; no deben buscarse funciones con el sufijo genérico `handle_tracking` o `handle_receive`.
- La deduplicación de PO solo queda demostrada en la función auxiliar previa al POST; todavía falta probar doble clic, replay, conflicto de PO existente y respuesta posterior de Airtable en un entorno seguro.
- La inspección estática no puede demostrar scroll horizontal, foco, wrapping, estados de carga ni comportamiento de botones; esos criterios requieren viewport controlado y pruebas runtime.
- El bundle contiene `overflow-x:auto` en varias zonas y también `overflow-x:hidden/clip` en otras; no se puede declarar “cero scroll horizontal” sin probar cada pantalla en viewport estrecho. Los breakpoints encontrados incluyen `760`, `640`, `390` y `360` px, pero su comportamiento real sigue sin evidencia visual.
- Los handlers de Procurement tienen `check_admin_referer` en las líneas 35301–36418 para creación, estado, cancelación, edición básica, cotizaciones, PO, tracking, recepción e historial; esto confirma protección estática, no autorización efectiva ni rechazo de nonce/capability incorrectos.
- La navegación estática declara además la vista `Asistente` (`proc_view=ia`) en `cleg-active-snippets.php:36521-36527`, y existen enlaces/redirects hacia ella en varias rutas. Esto no estaba incluido en la primera matriz y queda como subvista pendiente de auditoría funcional, permisos y utilidad.
- La función de menú móvil se invoca también desde Historial (`cleg-active-snippets.php:38304`), por lo que hay que comprobar si la navegación se duplica o cambia de forma entre desktop, móvil, detalle e historial; no se aprueba por inspección de código.
- El texto y la función de “Update para cliente” existen en `cleg_procurement_render_client_update()` (`38461-38488`), pero el escaneo no demuestra cuál es el enlace visible que lo abre ni si queda accesible únicamente desde el detalle; queda pendiente runtime.
- Hallazgo estático importante: el render principal imprime `cleg_procurement_render_provider_history_panel()` siempre que `$show_history` sea verdadero (`cleg-active-snippets.php:39008-39010`), incluso fuera de la vista Historial. Esto explica la información histórica al final de la pantalla principal y contradice el alcance de “Historial solo como consulta secundaria”. **P1 UX/UI; verificar visualmente y corregir en implementación posterior.**
- El render principal también imprime `cleg_procurement_render_ai_panel()` cuando `$show_ai` está activo (`39005-39007`). Debe auditarse como subvista separada y no mezclarse con la Bandeja si no corresponde a la tarea actual. **P1 de claridad/navegación.**
- `cleg_procurement_render_mobile_menu()` se inserta en Detalle (`38989`), Update cliente (`39001`) e Historial (`38304`), por lo que la navegación se repite dentro de subpaneles; queda confirmado estáticamente como candidato a duplicación visual, pendiente de prueba en navegador.

**Conclusión de esta pasada:** se redujo la incertidumbre estática, pero no cambia el estado global: **NO READY**. No se autoriza benchmark, implementación, montaje ni publicación mientras permanezcan las pruebas runtime y de seguridad abiertas.

## Estado de coordinación — 2026-09-26

- El subagente QA devolvió `systemError` por `401 Unauthorized` antes de entregar una nueva corrida. No se considera evidencia ni PASS.
- La QA directa ya documentada sigue siendo la fuente válida; no se sustituye por el estado del subagente.

## Snapshot de cobertura — 2026-09-26

- Checklist operativo: **1 completado / 19 pendientes**.
- Matriz de elementos: **30 Verificado / 1 Corregido / 31 Abierto**.
- “Corregido” no significa publicado ni probado post-montaje; se conserva como estado intermedio.
- La auditoría no puede cerrarse mientras exista cualquier fila Abierta o criterio NO VERIFICABLE, especialmente mutaciones, permisos, viewport móvil, estados de interfaz y persistencia.

## Inventario estático de superficies y mutaciones — 2026-09-26

- Superficies renderizadas identificadas: comparador/entrada de cotizaciones, tira comercial, edición básica, menú móvil, Asistente, Historial de proveedores, Update para cliente, página standalone y Detalle.
- Acciones `admin_post` identificadas: crear solicitud, cambiar estado, cancelar, editar datos básicos, borrar adjuntos, actualización rápida, crear/editar cotización, crear PO, actualizar tracking, recibir PO, crear/editar/eliminar historial, aplicar/descartar actualización del Asistente y exportar datos del Asistente.
- Este inventario confirma que el alcance real es mayor que la Bandeja visible. Cada acción sigue pendiente de prueba runtime de permiso, nonce, persistencia y respuesta de error/éxito.
- Recuento confirmado en el bundle: **16 handlers `admin_post` de Procurement**, incluidos creación/edición/cancelación de solicitudes, cotizaciones, PO, tracking, recepción, historial y acciones del Asistente. La cobertura runtime sigue siendo parcial: proteger el endpoint con nonce no demuestra por sí solo capability correcta, ownership, idempotencia ni persistencia.
- `git diff --check` pasó en la copia revisada. El hash local coincide con el hash declarado en `cleg-active-snippets.sha256`; esto valida consistencia local del archivo, no montaje ni publicación.

## Reintento runtime — 2026-09-26

- Se intentó enlazar nuevamente la pestaña autenticada de `/admin-recibos/` y obtener su árbol de accesibilidad mediante Chrome/CUA.
- La operación agotó el tiempo de espera y reinició la sesión de automatización; no produjo evidencia visual ni funcional nueva.
- Esta repetición confirma una limitación del entorno de QA autenticado, no un PASS del producto. Las pruebas runtime de recibos, viewport, foco y mutaciones permanecen **NO VERIFICABLES**.

## Runtime adicional verificado — 2026-09-26

- La búsqueda `6EP3436` filtró la Bandeja a 1 solicitud; la URL conservó `proc_q=6EP3436` y los contadores se recalcularon. **Verificado.**
- `Limpiar` eliminó el término y devolvió la Bandeja completa. **Verificado.**
- El filtro rápido `En camino` abrió `proc_stage=po&proc_order_state=in_transit`, mostró 1 registro y seleccionó correctamente `COMPRA=En camino`. **Verificado.**
- Expandir una tarjeta mostró su CTA y `Abrir y continuar`; el enlace abrió el Detalle de la requisición correcta. **Verificado.**
- En el Detalle `CLEG-REQ-2026-0046`, la sección `Ordenes, llegada y recepcion` muestra una PO real (`PO QuickBooks: 1069`, JC Electronics, `$939.00`, `In transit`) y controles de tracking/recepción. **Verificado.**
- En esa misma sección permanece visible además `Registrar PO real de QuickBooks` con proveedor, monto, ETA y botón `Asignar PO QuickBooks`, aun cuando ya existe una PO. Es un hallazgo **P1** de claridad y riesgo de duplicación; no se pulsó.
- El control de tracking expone un campo etiquetado y un segundo campo sin nombre accesible claro. **P1 accesibilidad/claridad**, pendiente de identificar visualmente su propósito y corregir etiqueta.
- La ruta directa `proc_view=cliente&proc_request=CLEG-REQ-2026-0046` carga el panel `Update para cliente` con estado, próxima acción, responsable y timeline vacío; el texto confirma que no muestra precios, cotizaciones ni márgenes. **Verificado parcialmente.** ETA aparece sin valor en este registro y no se comprobó el enlace de acceso desde el Detalle ni restricciones por rol.
- La ruta directa `proc_view=ia` no mostró el Asistente: cargó la Bandeja con sus filtros y solicitudes, manteniendo la URL `proc_view=ia`. **Hallazgo P1 de routing/funcionalidad:** la vista declarada no responde con su contenido esperado o queda silenciosamente sustituida por otra vista.
- Historial runtime cargó `73 registros`, con grupos de proveedores, búsqueda, “Crear nuevo registro manual” y “Abrir / editar”. La vista expone en el primer nivel notas largas, contactos, emails, teléfonos, decisiones crudas, valores `USD 0`, URLs y texto con codificación rota (`Â`). **P1 UX/legibilidad y posible exposición innecesaria de datos internos**; confirma la necesidad de divulgación progresiva y revisión de privacidad.
- `Descargar ficha` del Detalle generó correctamente un archivo local `siemens-simatic-s7-400-memory-card-flash-eprom-4-mb.html`; **Verificado** como descarga, pero no es PDF. El contenido muestra `PO 1069` por `$939.00` y, en la sección de cotización seleccionada, `Radwell International LLC · $0.00`; **P1 integridad/consistencia de datos** que requiere revisión antes de declarar el documento confiable.

## QA de permisos con cuenta Procurement `cgarin` — 2026-09-26

- Login en sesión aislada: **PASS**. La cuenta entra y recibe una Bandeja propia con `TODAS 0`, `ANTES DE PO 0`, `PO/TRACKING 0`, `TERMINADAS 0` y `CANCELADAS 1`.
- Navegación visible para la cuenta: Bandeja, Nueva solicitud y Recibos. Historial no aparece en navegación. **Verificado parcialmente.**
- Acceso directo a `proc_view=historial`: la URL conserva el parámetro, pero la página carga la Bandeja vacía en lugar de mostrar rechazo/403 o Historial. **P1 de autorización/routing silencioso.**
- Acceso al Detalle propio/cancelado `CLEG-REQ-2026-0018`: carga la requisición “Prueba de ejemplo” solicitada por Carlos Garin. **Verificado.**
- Acceso directo al Detalle ajeno `CLEG-REQ-2026-0046`: la URL cambia, pero el contenido sigue mostrando “Prueba de ejemplo”/la solicitud propia cancelada, no un rechazo explícito ni el registro solicitado. **P0/P1 de aislamiento y routing:** puede ocultar un rechazo, mezclar IDs o presentar una respuesta incorrecta; requiere prueba de ID/tenant y corrección antes de producción.
- `/admin-recibos/` es accesible y muestra `147 recibos`, incluyendo registros de Manuel Figueroa y proyectos ajenos; cada fila archivada expone `Revisar` y `Resuelto` habilitados. No se pulsaron. **P0/P1 de autorización:** la cuenta de Procurement parece tener lectura global y acciones de estado sobre recibos; debe confirmarse si es intencional y limitarse por capability/ownership.
- No se probaron botones de escritura con esta cuenta; no se modificaron registros.

### Ampliación de alcance: aislamiento entre módulos — 2026-09-26

- Con la misma cuenta Procurement, `/panel/` cargó el **Panel RRHH del trabajador** de Carlos Garin, incluyendo `Marcar entrada`, selección de proyecto y `Reportar ausencia`. No se pulsó ninguna acción. **P0 de separación de roles:** un usuario de Procurement puede alcanzar una superficie laboral con acciones de escritura; debe definirse explícitamente si esto es intencional y, si no, bloquearse por capability.
- `/admin-rhh/` no existe y devolvió 404; esto no demuestra protección de un módulo administrativo RRHH, solo confirma que esa URL no es una ruta válida.
- La sesión se cerró mediante `Cerrar sesión` y el árbol final volvió a `Acceso trabajadores`. **PASS de cierre de sesión.**
- La ampliación confirma que el problema no está limitado a la Bandeja: falta una matriz explícita de permisos por ruta, vista y acción, probada con usuario anónimo, Procurement, trabajador y administrador.

### QA de acceso anónimo — 2026-09-26

- `/admin-procurement/` redirige a `/acceso/` con `cleg_redirect=/admin-procurement/`. **PASS de bloqueo anónimo.**
- `/admin-recibos/` redirige a `/acceso/` con `cleg_redirect=/admin-recibos/`. **PASS de bloqueo anónimo.**
- `/panel/` redirige a `/acceso/` con `cleg_redirect=/panel/`. **PASS de bloqueo anónimo.**
- La prueba no valida todavía la autorización por rol después del login; ese bloque queda cubierto por la evidencia de `cgarin`, donde aparecen los hallazgos P0/P1 ya descritos.

### QA de rutas y errores — 2026-09-26

- Con `cgarin`, la ruta `proc_view=detalle&proc_request=NO-EXISTE-9999` no mostró error ni “requisición no encontrada”: renderizó la requisición propia cancelada “Prueba de ejemplo”, con cotización y adjuntos. **P0 de integridad/aislamiento y manejo de error:** un identificador inválido puede devolver contenido de otro registro. Debe responder 404/estado vacío seguro y nunca usar un fallback silencioso.
- La sesión se cerró después de la prueba y volvió a la pantalla de acceso. No se ejecutaron mutaciones.

### Matriz consolidada de errores y respuestas — 2026-09-26

| Escenario | Resultado observado | Estado QA |
|---|---|---|
| Usuario anónimo abre Procurement | Redirige al login conservando `cleg_redirect` | PASS |
| Usuario anónimo abre Recibos | Redirige al login conservando `cleg_redirect` | PASS |
| Usuario anónimo abre Panel | Redirige al login conservando `cleg_redirect` | PASS |
| Procurement abre `proc_view=historial` | URL conserva el parámetro pero renderiza Bandeja vacía | FAIL P1 |
| Procurement abre `proc_view=ia` | URL conserva el parámetro pero renderiza Bandeja | FAIL P1 |
| Procurement abre requisición ajena | Renderiza la requisición propia | FAIL P0/P1 |
| Procurement abre ID inexistente | Renderiza la requisición propia en vez de error seguro | FAIL P0 |
| Procurement abre `/panel/` | Accede a funciones de trabajador | FAIL P0 |
| Descarga de ficha | Descarga HTML con extensión `.html`, no PDF | FAIL P1 |
| Estado vacío de Bandeja propia | Mensaje visible y accionable | PASS parcial |
| Error de persistencia de PO/tracking/recepción | No ejecutado para proteger datos reales | NO VERIFICABLE |
| Doble envío, nonce negativo y replay | No ejecutado para proteger datos reales | NO VERIFICABLE |
| Responsive 1440/1024/390/360 | No medido con viewport controlado | NO VERIFICABLE |

## Reporte de equipos especializados — 2026-09-26

### Backend / integridad de datos

- **P0:** la carga de Airtable fija `received_qty` en `0` para toda PO (`cleg-active-snippets.php`, alrededor de `31750`). La recepción parcial se guarda como overlay de WordPress y el PATCH remoto no persiste cantidad recibida ni un evento de recepción; el flujo no tiene autoridad única para 3/10, 10/10 y evidencia.
- **P1:** recepción ignora el resultado efectivo del PATCH remoto; un fallo puede redirigir como si se hubiera guardado. Las llamadas de PO/shipment deben exigir HTTP 2xx y releer para confirmar.
- **P1:** creación de PO depende de una búsqueda limitada a 100 registros; sin paginación puede duplicar una PO existente fuera de esa ventana.
- **P1:** cada cambio de tracking puede crear un shipment nuevo; falta idempotency key/replay guard.
- **P1:** fallback de Airtable a WordPress puede fabricar listas vacías de cotizaciones/PO/shipments sin marcar degradación de datos.
- **P2:** descarga de adjuntos usa URLs directas/client-side y no un endpoint que vuelva a validar permiso por archivo.

### Frontend / UX / responsive

- **P0:** `.cleg-proc-btn` y `.cleg-proc-open` usan `min-width:max-content`, lo que puede forzar overflow en 360/390 px aunque existan reglas posteriores de wrapping.
- **P1:** navegación lateral y primaria repiten rutas; los menús móviles también se insertan en Detalle, Update cliente e Historial.
- **P1:** filtros rápidos y filtros avanzados conviven sin una jerarquía suficientemente clara.
- **P1:** el Detalle concentra tabs, acciones, formularios de PO, tracking y recepción; los submits y retornos de error no están probados runtime.
- **P2:** existen reglas de `overflow-x:auto` en strips y lista; el shell usa `overflow-x:hidden`. Esto requiere medir `scrollWidth` en cada viewport y no puede aprobarse por CSS estático.
- **P2:** existen `focus-visible`, áreas táctiles de 44 px y `aria-live`, pero no se probó foco posterior a error, orden de tabulación ni lector de pantalla.

### QA / release

- `git diff --check` pasó y el hash local coincide con el hash declarado localmente.
- No se puede demostrar todavía que el bundle montado/publicado corresponda a ese hash ni que sea la versión actual de GitHub.
- El lint PHP 8.4 pasó en una corrida anterior, pero no fue reproducible en el entorno del subagente.
- No se realizaron mutaciones ni pruebas de doble envío para proteger datos reales.
- Comparación de fuentes: `origin/main` apunta a `d90e9a46b10a7b62e42cb0111b48aa32192cd71c`; el archivo local tiene **7 líneas modificadas** respecto a `origin/main` (ajustes responsive de botones). Por tanto, el hash local y el bundle de GitHub no son la misma versión; cualquier PASS visual local no puede atribuirse automáticamente al bundle publicado.

**Conclusión de los equipos:** la auditoría de diagnóstico está completa para el alcance observado, pero el QA de errores y la aprobación release siguen abiertos por P0/P1 confirmados y por falta de verificación de bundle montado, viewport controlado y persistencia remota.

### Último intento runtime delegado — 2026-09-26

- El agente QA no dispuso de una sesión autenticada compartida y no automatizó el login. Por eso no pudo medir hash del bundle montado, `scrollWidth/clientWidth`, viewports 1440/1024/390/360, consola ni estados de mutación.
- Esto no invalida la evidencia runtime obtenida directamente con `cgarin`, pero confirma que esas pruebas no pueden darse por cerradas con evidencia del subagente.
- Estado final de esta fase: **diagnóstico completo; QA release no aprobado**.

## Implementación de correcciones — 2026-09-26

Se aplicaron por equipos especializados, sin borrar datos existentes:

- `0a52d8e`: persistencia y confirmación de recepción, cantidad recibida, degradación explícita y replay guard de tracking.
- `c5d3f42`: responsive, CTA sin `max-content`, navegación duplicada y controles táctiles.
- `c17f4c9`: capability específica para Recibos y restricción de acceso laboral para Procurement.
- `1c68a6c`: validación explícita de respuestas Airtable y checksums exactos.

Validación local posterior:

- PHP lint: PASS.
- `git diff --check`: PASS.
- SHA256 del bundle: `fd4b13fbc6d6ee0af41e1cdc2441fa815be8242cdf2cd6f36245556b62457f61`.
- Ambos archivos checksum coinciden con ese SHA256.

QA final delegado: **PASS estático parcial / NO READY runtime**. Falta montar ese hash exacto y comprobar el comportamiento publicado con sesión autenticada, permisos negativos, doble envío, persistencia, estados UI y viewports.

## Verificación de montaje y smoke QA — 2026-09-26

- GitHub `origin/main`: `2a4f572`.
- Bundle local y raw GitHub coinciden byte a byte: SHA256 `409a6842d4cad10fbce382bb481b0bb1f8dd947e3be19a22e127d30342237156`.
- WordPress Snippet Sync: última sincronización `2026-09-26 07:15:03 (auto)`; último estable `2026-09-26 07:15:29`; QA funcional `Aprobado`; cuarentena vacía; URL configurada apunta al bundle canónico de GitHub.
- Smoke QA autenticado en `/admin-procurement/`: carga correcta, filtros principales visibles, contadores presentes, tarjetas expandibles, acción “Abrir y continuar” y detalle navegable.
- Detalle verificado: estado, próxima acción, navegación por secciones y sección PO/Tracking presentes.
- Persisten como NO VERIFICABLE en esta sesión: pruebas destructivas/mutaciones reales, doble envío con datos de producción y medición controlada en 1440/1024/390/360 px.

Estado de release: **montado y verificado para smoke QA; QA exhaustivo pendiente únicamente en los escenarios protegidos indicados arriba**.
