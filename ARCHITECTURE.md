# CLEG Foundation — Arquitectura objetivo

Fuente actual: `cleg-active-snippets.php` en `main`, bundle WordPress generado. La estructura modular siguiente es objetivo; no se afirma que exista hoy en GitHub.

## Capas

`Shortcode/entrada → autorización → presenter → componentes → servicio → adaptador Airtable`.

- Presentación no consulta Airtable ni decide permisos.
- Servicios aplican reglas y transiciones.
- Adaptador normaliza respuestas y errores.
- WordPress mantiene URLs y shortcodes durante la migración.

## Organización lógica futura

```text
src/foundation/{tokens,components,shell,runtime}
src/modules/{worker,hr,requests,payroll,documents,field-ops,projects,procurement,tenant-admin}
tests/{unit,integration,e2e,visual,accessibility}
```

Cada módulo tendrá vista, presenter, actions, estilos y scripts acotados. El generador seguirá produciendo el bundle canónico.

## Navegación

Registro único: `destino → shortcode → módulo → capacidad → renderizador`. Se conservan URLs antiguas mediante adaptadores. El menú no sustituye la autorización del servidor.

## Migración

1. Contratos y tokens sin cambiar flujos.
2. Componentes y cliente de errores compartidos.
3. Piloto Compras detrás de URLs actuales.
4. Shell y navegación única.
5. Migración módulo a módulo con rollback.
6. Retiro de duplicados solo con evidencia de consumidores cero.
