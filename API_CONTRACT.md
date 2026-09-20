# CLEG Foundation — Contrato lógico de API

No se cambia todavía ningún endpoint existente. Los adaptadores actuales deben conservar sus URLs.

## Piloto Compras

Operaciones: crear solicitud, listar/detallar, actualizar, registrar cotización, aprobar/rechazar, emitir PO, actualizar tracking, registrar recepción y cancelar.

Toda escritura exige `idempotency_key`; toda actualización exige `expected_version`. `tenant_id` y `actor_id` se derivan de la sesión, nunca del cliente.

Respuesta común:

```json
{"operation_id":"opaque","status":"succeeded","resource_id":"opaque","version":3,"correlation_id":"opaque"}
```

Error común: `code`, `message`, `retryable`, `correlation_id` y errores por campo.

## Reglas

- Fechas civiles `YYYY-MM-DD`; instantes ISO-8601 UTC; importes decimales y moneda explícita.
- 401 sesión, 403 permiso, 404 recurso, 409 conflicto/idempotencia, 422 validación, 429 límite, 503 dependencia.
- Paginación con cursor opaco; nunca truncamiento silencioso.
- Adjuntos validados por tipo real, tamaño, pertenencia al tenant y permiso de descarga.
- Namespace REST futuro, solo si hace falta: `cleg-procurement/v1`.
