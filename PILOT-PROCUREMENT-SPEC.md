# Piloto aprobado para especificación — Compras

## Alcance

Bandeja, Nueva requisición, Detalle, PO/Tracking, Update para cliente e Historial. Recibos queda fuera del piloto inicial.

## Flujo principal

`Borrador → Enviada → Cotizando → Por decidir → Aprobada → PO enviada → En camino → Recibida parcial → Recibida → Cerrada`.

La bandeja muestra artículo, proyecto, estado, fecha/ETA, prioridad, próxima acción y un único botón: **Abrir y continuar**. El detalle muestra una sola tarjeta por PO real y conserva duplicados para auditoría.

## Criterios de aprobación de especificación

- Una sola navegación y una acción principal por registro.
- Permisos por operación y por recurso.
- PO idempotente; reintentos no duplican; varias POs distintas siguen permitidas.
- Éxito visible solo después de persistencia confirmada.
- Sin scroll horizontal ni recortes en 360/390 px.
- Update para cliente excluye precios, márgenes, proveedor y notas internas.
- El flujo crítico funciona con teclado y estados de carga/error/éxito.

## Ciclo

UX/UI → aprobación de esta especificación → frontend/backend → QA → auditoría → correcciones → QA → verificación del Supervisor. No se despliega a producción.

## Pendientes explícitos de Fase 1

- Adjuntos privados: no se implementan todavía; los adjuntos siguen el comportamiento existente y requieren una revisión de aislamiento por tenant.
- Cambios de permisos: no se amplían ni rediseñan en esta fase; queda pendiente una matriz por operación y recurso.
- Idempotencia completa: no se implementa todavía para todas las escrituras; la PO y las operaciones repetibles requieren una fase específica antes del piloto.
- No se crean registros ni se envían formularios reales durante la validación de esta fase.
