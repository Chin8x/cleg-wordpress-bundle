# CLEG Foundation — Design System

Estado: propuesta de Fase 1. No modifica la interfaz actual.

## Tokens base

- Fondo: `#f5f7f9`; superficie: `#ffffff`; texto: `#20252b`; secundario: `#65707c`.
- Borde: `#d8e0e7`; acción: `#c75000`; éxito: `#11744f`; error: `#a82424`.
- Espaciado: 4, 8, 12, 16, 24, 32 y 48 px; radio base: 8 px.
- Tipografía de sistema; base 16 px; auxiliar 14 px; títulos 24/32 px; cifras tabulares.

## Componentes globales

Shell, navegación, cabecera, botón primario/secundario/destructivo, campo con ayuda/error, selector, filtro, tarjeta, tabla adaptable, paginación, estado, adjuntos, pasos, diálogo y mensajes `aria-live`.

Cada componente define foco visible, deshabilitado, carga, error y estado vacío cuando aplique. Los estados nunca dependen solo del color.

## Reglas

- Clases prefijadas `cleg-`; nada de selectores globales genéricos.
- Áreas táctiles mínimas de 44 px.
- Cero scroll horizontal accidental en 360/390 px.
- Móvil: una columna y acción principal visible. Escritorio: contenido compacto y jerarquizado.
