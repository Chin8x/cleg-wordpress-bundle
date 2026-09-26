# Procurement changelog

## 2026-09-26 — QA implementation pass

- Seguridad: aislamiento por tenant e identidad de solicitante, rutas de detalle inexistentes sin fallback silencioso y salida a Panel limitada a capacidades operativas.
- Backend: recepción parcial con `received_qty`, confirmación read-after-write, fallback degradado explícito, idempotencia de PO/shipment y confirmación de tracking.
- Recibos: edición con motivo/evidencia obligatorios, auditoría append-only, transiciones de estado permitidas y modo solo lectura para capacidades sin gestión.
- Frontend: navegación duplicada controlada y formulario de corrección de recibos usable en móvil.
- Verificación local: `git diff --check` y `php -l cleg-active-snippets.php` pasan.
- Bundle local: SHA256 `409a6842d4cad10fbce382bb481b0bb1f8dd947e3be19a22e127d30342237156`; checksums actualizados.

### Pendiente de release

No se marca PASS montado: la sesión Chrome autenticada perdió el puente CUA antes de la QA post-montaje. Falta verificar el hash realmente activo, rutas/roles negativos, scrollWidth/clientWidth en 1440/1024/390/360 y estados runtime sin mutaciones.
