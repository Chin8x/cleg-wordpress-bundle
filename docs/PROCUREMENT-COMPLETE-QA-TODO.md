# Procurement Complete QA — TODO Checklist

Este checklist acompaña a `PROCUREMENT-COMPLETE-QA.md`. Todo caso no ejecutado debe permanecer como **NO VERIFICABLE** o **abierto**, nunca como PASS.

## Runtime y datos

- [ ] Repetir Bandeja, Cotizaciones, Compras activas, Detalle, Nueva solicitud e Historial post-montaje.
- [x] Verificar `/admin-recibos/`: filtros Día/Proyecto/Persona/Monto/Estado, Filtrar/Limpiar y `receipt_page=2` con Página 2 de 6 · 147 recibos, Anterior/Siguiente.
- [ ] Confirmar una sola PO operativa `1055` y conservar conflictos únicamente en auditoría.
- [ ] Verificar tracking, ETA, recepción parcial/completa y receipt ID sin mutar producción.
- [ ] Probar corrección controlada de persona, proyecto, compra y cantidad en Recibos.
- [ ] Verificar auditoría append-only por edición: valor anterior, valor nuevo, usuario, fecha/hora, motivo y evidencia; rechazar sobrescritura silenciosa.
- [ ] Eliminar fallback de ownership por nombre/login/email; exigir WP user ID + tenant ID + request ID.
- [ ] Probar tenant/request cruzado, nonce ausente/expirado y capabilities con usuarios controlados.
- [ ] Probar idempotencia, record ID, request ID y error de ambigüedad.

## UI y responsive

- [ ] Recorrer cada link, botón, filtro, campo, columna, estado y CTA documentado.
- [ ] Ejecutar screenshots y overflow checks en 1440, 1024, 390 y 360 px.
- [ ] Confirmar wrapping seguro, `min-width:0`, `max-width:100%` y target táctil mínimo de 44 px.
- [ ] Verificar back, forward, refresh y persistencia de filtros/estado.
- [x] Validar reload de Bandeja: carga correcta, 21 solicitudes y contadores conservados.
- [ ] Validar back/forward: la sesión se interrumpe durante la navegación; mantener NO VERIFICABLE hasta repetir en entorno estable.

## Estados y errores

- [ ] Loading.
- [ ] Empty.
- [ ] Error de red/API.
- [ ] Error de validación.
- [ ] Success de lectura/mutación en entorno controlado.

## Seguridad y accesibilidad

- [ ] Matriz de permisos y capabilities.
- [ ] Nonce válido, ausente, expirado y de otro contexto.
- [ ] Ownership WP user + tenant + request.
- [ ] DTO cliente allowlist y ausencia de datos internos.
- [ ] MIME/tamaño/privacidad de adjuntos.
- [ ] `aria-current`, foco, teclado, nombres accesibles y contraste.

## Gates de publicación

- [x] PHP 8.4 `php -l cleg-active-snippets.php`: sin errores de sintaxis.
- [ ] Revisión del Project Chief.
- [ ] Verificación runtime de fixes `59dec3b`, `cc7f710`, `d4d5068`, `b1dcd6c`.
- [ ] QA visual y funcional completa.
- [ ] Actualización de checksums.
- [ ] Commit y push autorizados.
- [ ] Montaje/sync y QA post-montaje.
