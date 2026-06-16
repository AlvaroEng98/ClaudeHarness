---
name: harness-backtrack
description: Protocolo de backtracking del Harness. Cargar solo cuando un agente emite ## RECHAZO o ## BLOQUEO DE DISEÑO.
---

## Protocolo de backtracking

Cuando un agente emite una señal de rechazo o bloqueo:

1. **Notificar al Desarrollador** antes de ejecutar el backtracking:
   - Motivo del retroceso
   - Fase destino
   - Artefactos que serán invalidados
2. **Marcar artefactos invalidados**: añadir al inicio de cada archivo posterior a la fase destino:
   ```
   > [INVALIDADO] Este artefacto fue invalidado el YYYY-MM-DD por: <motivo>.
   ```
3. **Re-ejecutar desde la fase destino** con los artefactos preservados como contexto.

### Mapa de backtracking

| Señal | Fase destino |
|-------|-------------|
| Investigador `## RECHAZO` | Aclarar con Desarrollador → re-ejecutar Investigador |
| Especificador `## RECHAZO` | Re-ejecutar Especificador con aclaraciones, o retornar a Proposer |
| Diseñador `## RECHAZO` | Re-ejecutar Especificador con los requisitos afectados |
| Constructor `## BLOQUEO DE DISEÑO` | Re-ejecutar Diseñador con la situación bloqueante |
| Reviewer criterios no cumplidos | Re-ejecutar Constructor con los criterios fallidos |
| Developer solicita revisión de fase | Retornar a esa fase invalidando las posteriores |
