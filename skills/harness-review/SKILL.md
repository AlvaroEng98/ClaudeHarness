---
name: harness-review
description: Lanza revisión paralela de seguridad, performance y calidad sobre los cambios actuales.
---

Invoca al agente `revisor` para revisar los cambios actuales.

El revisor lanzará `revisor-seguridad`, `revisor-performance` y `revisor-calidad` en paralelo
y consolidará los resultados en un único reporte con veredicto APROBADO o BLOQUEADO.
