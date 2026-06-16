---
name: harness-express-flow
description: Fases E1-E3 del Flujo Express del Harness. Cargar al clasificar una solicitud como Express.
---

## Flujo Express

### Fase E1 — Confirmación

Antes de construir, presentar al Desarrollador:
- Resumen del problema (máx. 5 líneas)
- Lista exacta de archivos a modificar
- Descripción concreta de los cambios a realizar

> **GATE Express**: No continuar sin confirmación explícita del Desarrollador.
>
> Si durante la exploración se descubre que el scope supera 3 archivos o implica cambios de interfaz: notificar y proponer escalar a Flujo Completo.

### Fase E2 — Construcción

1. Invocar agente `constructor` con:
   - Descripción exacta del fix
   - Lista de archivos a modificar
2. Si el Constructor emite `## BLOQUEO DE DISEÑO`:
   - Escalar a Flujo Completo desde Fase 1.
   - El análisis de E1 sirve como contexto inicial para el Planificador.

### Fase E3 — Revisión

1. Invocar agente `revisor` (`/harness-review`) pasando **únicamente**:
   - Lista de ficheros modificados por el Constructor
   - Descripción original del cambio como criterio de aceptación
2. Presentar resultado al Desarrollador.
3. Si el Revisor emite `BLOQUEADO`: retornar al Constructor con los criterios fallidos.

No persistir artefactos en `docs/harness/`. Si se escala a Completo: crear carpeta en ese momento.
