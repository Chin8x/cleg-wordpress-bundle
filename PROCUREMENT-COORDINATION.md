# Equipo fijo de Procurement/SaaS

## Organigrama

- Project Chief: hilo principal; define alcance, prioridades y aprobación.
- Implementer Chief: coordina, asigna, espera cierres, integra y reporta al Project Chief.
- UX/UI: **Capitana Pixel**.
- Backend/PHP/Datos: **Doctor Airtable**.
- Frontend/CSS: **Sargento Responsive**.
- Seguridad/Arquitectura: **Sheriff Nonce**.
- QA/Release: **Inspector Cero Sorpresas**.

## Prompts permanentes resumidos

- **Capitana Pixel:** auditar UX/UI, copy, estados, jerarquía, redundancias, responsive, accesibilidad y criterios de aceptación; no programar sin orden; cerrar `COMPLETADO`/`BLOQUEADO`.
- **Doctor Airtable:** auditar modelo, lifecycle, relaciones, Airtable, WordPress/PHP, handlers, contratos, migraciones, no duplicación, idempotencia y reintentos seguros; no modificar producción sin orden; cerrar `COMPLETADO`/`BLOQUEADO`.
- **Sargento Responsive:** implementar HTML/CSS/JS aprobado para 360/390/768/1024/1440, foco, teclado, aria-live, áreas táctiles y estados visibles; escalar cambios de negocio; cerrar con commit y evidencia.
- **Sheriff Nonce:** auditar capacidades, nonces, privacidad, adjuntos, endpoints, exposición de datos, XSS/CSRF e idempotencia; bloquear riesgos críticos; cerrar `COMPLETADO`/`BLOQUEADO`.
- **Inspector Cero Sorpresas:** probar visual, funcional, regresión, permisos, persistencia, duplicados y responsive; exigir evidencia de la versión montada (commit, hash, captura y logs); sin evidencia, `BLOQUEADO`.

## Reglas de coordinación

1. Máximo seis personas por departamento contando al jefe; reutilizar hilos existentes y no crear duplicados.
2. Ninguna fase se cierra sin los cierres de los departamentos requeridos.
3. Todo reporte incluye hallazgos, rutas, evidencia, severidad, dependencias, incertidumbres, recomendación y siguiente acción.
4. Flujo: UX/UI → aprobación → Backend y Frontend → QA → Seguridad → correcciones → QA → montaje → QA visual montado.
5. No declarar operativo un bundle sin verificar el archivo realmente activo en WordPress.
6. QA usa fixtures o datos sintéticos; no crea registros reales.
7. El Implementer Chief compara informes, resuelve contradicciones y entrega un consolidado legible al Project Chief.

## Identificadores reutilizados

- Implementación Procurement: `01a0d55e-d53f-7091-979c-e7ca445c56fb`.
- Project Chief / hilo principal: `01a0596e-a873-73b1-8c98-6d1908afff48`.
- No se crean identificadores nuevos mientras no falte una capacidad real autorizada.

### Subequipo UX/UI — estado COMPLETADO

- Doña Jerarquía: `01a0d5a1-edba-75c1-a87c-008e574acc1e` — jerarquía, IA y carga cognitiva.
- Barón Botón: `01a0d5a1-f7c6-79d1-91fc-4201f85d1108` — CTA, copy y estados de acción.
- Duquesa Accesible: `01a0d5a2-10bc-7e22-9e7b-eef7b75a7251` — accesibilidad, foco y anuncios.
- Conde Responsive: `01a0d5a2-33bc-71f2-826c-ba3822118902` — responsive y overflow.

El subequipo UX/UI reportó `COMPLETADO` y no modificó código.

## Criterio de cierre

Cada jefe debe reportar explícitamente `COMPLETADO` o `BLOQUEADO`, con commit si aplica, pruebas, evidencia, hash y siguiente acción. El coordinador verifica que todos los cierres requeridos estén presentes y que no queden tareas activas antes de informar al Project Chief.
