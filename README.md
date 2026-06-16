# harness-agents

Plugin de Claude Code que orquesta agentes especializados para desarrollo de software. Implementa tres flujos de trabajo con gates de aprobación, revisión paralela y separación estricta entre planificación, construcción y revisión.

## Instalación

```bash
claude plugin install .
```

## Flujos

| Flujo | Cuándo usarlo | Comandos |
|-------|---------------|----------|
| **Consulta** | Preguntas, exploración de código | (respuesta directa) |
| **Express** | Bugfix acotado, ≤3 archivos, causa obvia | `/work` + `/harness-review` |
| **Completo** | Features, refactoring, decisiones de diseño | `/plan-with-agent` → `/work` → `/harness-review` |

## Comandos

| Comando | Descripción |
|---------|-------------|
| `/plan-with-agent` | Genera `spec.md` y `Plans.md` con el Planificador |
| `/work` | Ejecuta el siguiente task pendiente de `Plans.md` con el Constructor |
| `/harness-review` | Revisión paralela de seguridad, performance y calidad |

## Agentes

| Agente | Rol | Herramientas |
|--------|-----|--------------|
| `planificador` | Explora codebase y genera spec + plan | Read, Write, Bash (lectura), Grep, Glob |
| `constructor` | Único agente que escribe código | Read, Write, Edit, Bash, Grep, Glob |
| `revisor` | Orquesta los 3 revisores en paralelo | Read, Grep, Agent |
| `revisor-seguridad` | Detecta vulnerabilidades | Read, Grep |
| `revisor-performance` | Detecta cuellos de botella | Read, Grep |
| `revisor-calidad` | Revisa tipado, tests, convenciones | Read, Grep |

## Principios

- El orquestador **nunca escribe código** — delega siempre al Constructor.
- Los revisores son **solo lectura** — detectan, no corrigen.
- Los gates son **condiciones de parada obligatoria** — no se omiten.
- Un `## BLOQUEO DE DISEÑO` del Constructor escala al Planificador, no al orquestador.

## Skills incluidos

- `harness-artifacts` — formato de artefactos y registro de versiones
- `harness-complete-flow` — protocolo detallado del Flujo Completo
- `harness-express-flow` — protocolo detallado del Flujo Express
- `harness-backtrack` — protocolo de retroceso cuando un agente emite `## RECHAZO`
- `issues-creator` — convierte un plan en tickets del issue tracker
- `skill-redactor` — crea nuevos skills con estructura correcta
- `handoff` — genera documento de traspaso entre sesiones
- `grill-me` — entrevista al desarrollador sobre su diseño
- `caveman` — modo de comunicación comprimido (menos tokens)
