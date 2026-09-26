# Procurement changelog

## 2026-09-26 — QA implementation pass

- Seguridad: aislamiento por tenant e identidad de solicitante, rutas de detalle inexistentes sin fallback silencioso y salida a Panel limitada a capacidades operativas.
- Backend: recepción parcial con `received_qty`, confirmación read-after-write, fallback degradado explícito, idempotencia de PO/shipment y confirmación de tracking.
- Recibos: edición con motivo/evidencia obligatorios, auditoría append-only, transiciones de estado permitidas y modo solo lectura para capacidades sin gestión.
- Frontend: navegación duplicada controlada y formulario de corrección de recibos usable en móvil.
- Verificación local: `git diff --check` y `php -l cleg-active-snippets.php` pasan.
- Bundle local: SHA256 `409a6842d4cad10fbce382bb481b0bb1f8dd947e3be19a22e127d30342237156`; checksums actualizados.

### Montaje y QA post-montaje

- Publicado en `origin/main` como `2a4f572`; el auto-sync de WordPress confirmó el mismo SHA en activo, last-good y candidate: `409a6842d4cad10fbce382bb481b0bb1f8dd947e3be19a22e127d30342237156`.
- Status privado: ejecución habilitada, bundle cargado, sin cuarentena y QA funcional aprobado; timestamp del sitio `2026-09-26 07:15:34`.
- QA autenticado sin mutaciones: Procurement, detalle, ID inexistente, Recibos, filtros, estado vacío, notices de error/éxito y Panel.
- Pendiente: repetir la matriz exacta de viewports 1440/1024/390/360; la capacidad visual disponible solo permitió medir `1920x855` (`clientWidth=1905`, `scrollWidth=1905`).
