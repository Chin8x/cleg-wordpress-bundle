# CLEG Foundation — Estrategia de testing

## Capas

- Unitarias: normalización, filtros, importes, fechas, transiciones e idempotencia.
- Integración: callbacks WordPress/Airtable simulado, permisos, nonce, errores y efectos finales.
- E2E: solicitud → cotización → PO → tracking → recepción.
- Visual: 360, 390, 768 y 1440 px; carga, vacío, error y éxito.
- Accesibilidad: teclado, foco, etiquetas, anuncios, contraste y zoom 200%.

## Gates del piloto

P0: aislamiento entre tenants, permisos, nonce, doble envío, timeout tras escritura, PO duplicada, tracking y recepción repetidos.

P1: filtros, paginación, adjuntos, estados, búsqueda y responsive.

No se acepta el piloto con BLOCKER/CRITICAL/HIGH abiertos. Cada resultado debe registrar commit, SHA del bundle, entorno, datos sintéticos y evidencia. No se usan datos de producción.
