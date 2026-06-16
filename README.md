# harness-agents

Plugin de Claude Code que orquesta agentes especializados para desarrollo de software. Separa estrictamente planificación, construcción y revisión — Claude actúa como orquestador y delega toda escritura de código al Constructor.

Útil cuando quieres que Claude siga un proceso disciplinado: no code sin spec aprobada, no merge sin revisión paralela de seguridad + performance + calidad.

## Requisitos

- Claude Code ≥ 1.0

## Instalación

```bash
# 1. Agregar el marketplace (una sola vez)
claude plugin marketplace add AlvaroEng98/ClaudeHarness

# 2. Instalar el plugin
claude plugin install harness-agents@alvaroeng
```

<details>
<summary>Instalación local (desarrollo)</summary>

```bash
git clone https://github.com/AlvaroEng98/ClaudeHarness
cd ClaudeHarness
claude plugin install .
```

</details>

## Inicio rápido

**Consulta** — preguntas o exploración, sin comandos:
> "¿Qué hace el módulo de autenticación?"

**Express** — bugfix acotado (≤3 archivos, causa obvia):
1. Claude confirma el problema y archivos afectados **[GATE]**
2. `/harness-agents:work` — Constructor aplica el fix
3. `/harness-agents:harness-review` — revisión paralela de seguridad, performance y calidad

**Completo** — features, refactoring, decisiones de diseño:
1. `/harness-agents:plan-with-agent` — Planificador genera `spec.md` y `Plans.md`
2. Aprobar `Plans.md` **[GATE 1]**
3. `/harness-agents:work` — Constructor ejecuta cada task (repetir por task)
4. `/harness-agents:harness-review` — revisión paralela **[GATE 2]**

## Flujos

| Flujo | Cuándo usarlo | Gates | Artefactos |
|-------|---------------|-------|------------|
| **Consulta** | Preguntas, exploración de código | Ninguno | Ninguno |
| **Express** | Bugfix acotado, ≤3 archivos, causa obvia | 1 (confirmación) | Ninguno |
| **Completo** | Features, refactoring, decisiones de diseño | 2 | `spec.md`, `Plans.md` |

> Toda duda entre Express y Completo resuelve como **Completo**.

## Comandos

| Comando | Cuándo usarlo |
|---------|---------------|
| `/harness-agents:plan-with-agent` | Inicio de Flujo Completo — genera `spec.md` y `Plans.md` |
| `/harness-agents:work` | Ejecuta el siguiente task pendiente de `Plans.md` |
| `/harness-agents:harness-review` | Revisión paralela al finalizar Express o Completo |

## Artefactos

| Archivo | Quién lo genera | Contenido |
|---------|-----------------|-----------|
| `spec.md` | Planificador | Descripción técnica del problema, decisiones de diseño |
| `Plans.md` | Planificador | Lista de tasks pendientes; el Constructor los ejecuta uno a uno |

Ambos viven en la raíz del repositorio. El orquestador solo escribe estos dos archivos.

## Agentes

| Agente | Rol | Herramientas |
|--------|-----|--------------|
| `planificador` | Explora codebase y genera spec + plan | Read, Write, Bash (lectura), Grep, Glob |
| `constructor` | Único agente que escribe código | Read, Write, Edit, Bash, Grep, Glob |
| `revisor` | Orquesta los 3 revisores en paralelo | Read, Grep, Agent |
| `revisor-seguridad` | Detecta vulnerabilidades | Read, Grep |
| `revisor-performance` | Detecta cuellos de botella | Read, Grep |
| `revisor-calidad` | Revisa tipado, tests, convenciones | Read, Grep |

## Skills incluidos

Skills del protocolo principal:

- `plan-with-agent` — invoca al Planificador para generar spec + plan
- `work` — invoca al Constructor para ejecutar el siguiente task
- `harness-review` — revisión paralela de seguridad, performance y calidad
- `harness-complete-flow` — protocolo detallado del Flujo Completo (fases + gates)
- `harness-express-flow` — protocolo detallado del Flujo Express
- `harness-backtrack` — protocolo de retroceso cuando un agente emite `## RECHAZO` o `## BLOQUEO DE DISEÑO`
- `harness-artifacts` — formato de artefactos y registro de versiones

Skills utilitarios:

- `issues-creator` — convierte un plan en tickets del issue tracker (vertical slices)
- `skill-redactor` — crea nuevos skills con estructura correcta
- `handoff` — genera documento de traspaso entre sesiones
- `grill-me` — entrevista al desarrollador sobre su diseño antes de planificar
- `caveman` — modo de comunicación comprimido (~75% menos tokens)

## Principios

- El orquestador **nunca escribe código** — delega siempre al Constructor.
- Los revisores son **solo lectura** — detectan, no corrigen.
- Los gates son **condiciones de parada obligatoria** — no se omiten.
- Un `## BLOQUEO DE DISEÑO` del Constructor escala al Planificador, no al orquestador.
- Escalar de Express a Completo requiere notificar al Desarrollador — nunca silenciosamente.
