---
name: work
description: Ejecuta el siguiente task pendiente de Plans.md usando el Constructor.
---

Invoca al agente `constructor` para ejecutar el siguiente task pendiente.

El constructor leerá `Plans.md`, tomará el primer `[ ]` task, lo implementará y lo marcará `[x]`.
Si no hay tasks pendientes en `Plans.md`, informar al Desarrollador.
