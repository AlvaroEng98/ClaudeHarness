---
name: harness-complete-flow
description: Fases 1-4 del Flujo Completo del Harness con GATE 1 y GATE 2. Cargar al clasificar una solicitud como Completo.
---

## Flujo Completo

### Fase 1 — Planificación

1. Invocar agente `planificador` con la descripción de la tarea del Desarrollador.
2. El Planificador explora el codebase y produce:
   - `spec.md` — descripción del problema y decisiones de diseño
   - `Plans.md` — lista ordenada de tasks atómicos para el Constructor
3. Presentar `Plans.md` al Desarrollador.

> **GATE 1**: No avanzar hasta recibir aprobación explícita del Desarrollador. Si el Desarrollador solicita modificaciones: retornar al Planificador con el motivo. Si rechaza 5 veces consecutivas sin aprobar: suspender y pedir instrucciones.

### Fase 2 — Construcción

Para cada task `[ ]` en `Plans.md`, en orden:

1. Si el task está marcado con `[APROBACIÓN REQUERIDA]`:
   - Presentar al Desarrollador y esperar confirmación. **GATE adicional**.
2. Invocar agente `constructor` (`/work`).
3. El Constructor lee `Plans.md`, ejecuta el primer task pendiente y lo marca `[x]`.
4. Si el Constructor emite `## BLOQUEO DE DISEÑO`:
   - Invocar `planificador` con la situación bloqueante.
   - El Planificador actualiza `Plans.md` con la resolución.
   - Reanudar construcción desde el task bloqueado.

### Fase 3 — Revisión

1. Invocar agente `revisor` (`/harness-review`) sobre todos los archivos modificados.
2. El Revisor lanza `revisor-seguridad`, `revisor-performance` y `revisor-calidad` en paralelo y consolida.
3. Presentar resultado al Desarrollador.

> **GATE 2**: Si el Revisor emite `BLOQUEADO`:
> - Agregar tasks de corrección al final de `Plans.md`.
> - Volver a Fase 2 con los tasks nuevos.

### Fase 4 — Cierre

Solo cuando el Revisor emite `APROBADO`:

1. Notificar al Desarrollador que el flujo está completo.
2. Listar archivos creados/modificados/eliminados.
3. Indicar si quedan tasks opcionales o de seguimiento en `Plans.md`.
