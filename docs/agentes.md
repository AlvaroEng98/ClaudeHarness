# Agentes en claude-code-harness — Referencia de Implementación

> Documento de referencia para replicar el modelo de sub-agentes del harness
> en una configuración propia de Claude Code.
>
> Fuente analizada: `Chachamaru127/claude-code-harness` (v2.x)
> Complemento conceptual: `Gentleman-Programming/gentle-ai` (modelo orquestador/delegación)

---

## Índice

1. [Qué es un sub-agente en este contexto](#1-qué-es-un-sub-agente-en-este-contexto)
2. [Dónde viven los agentes](#2-dónde-viven-los-agentes)
3. [Estructura de un archivo de agente](#3-estructura-de-un-archivo-de-agente)
4. [Los 6 agentes del harness](#4-los-6-agentes-del-harness)
5. [Cómo se invocan los agentes](#5-cómo-se-invocan-los-agentes)
6. [Reglas de delegación — cuándo el orquestador delega](#6-reglas-de-delegación--cuándo-el-orquestador-delega)
7. [Separación de responsabilidades: workers vs reviewers](#7-separación-de-responsabilidades-workers-vs-reviewers)
8. [Relación agentes ↔ skills ↔ hooks](#8-relación-agentes--skills--hooks)
9. [Plantilla base para un agente propio](#9-plantilla-base-para-un-agente-propio)
10. [Checklist de implementación](#10-checklist-de-implementación)

---

## 1. Qué es un sub-agente en este contexto

Un sub-agente en el harness **no es un modelo separado ni un proceso externo**.
Es una definición en Markdown que describe:

- **Rol** — qué responsabilidad única tiene este agente.
- **Scope** — qué archivos o capas del proyecto puede tocar.
- **Skills precargadas** — qué `SKILL.md` carga antes de actuar.
- **Herramientas permitidas** — lista explícita de `allowed-tools`.
- **Criterio de completitud** — cuándo se considera que terminó su trabajo.

Claude Code los ejecuta mediante el `Task tool` (AgentTool).
El orquestador invoca al sub-agente, espera su resultado y decide el siguiente paso.

```
Orquestador (orchestrator agent)
    │
    ├─► Worker agent      → escribe código
    ├─► Reviewer agent    → revisa seguridad / performance / calidad
    └─► Planner agent     → genera spec.md y Plans.md
```

---

## 2. Dónde viven los agentes

```
.claude/
└── agents/                    ← directorio estándar de Claude Code
    ├── harness-worker.md
    ├── harness-reviewer.md
    ├── harness-planner.md
    ├── harness-security-reviewer.md
    ├── harness-performance-reviewer.md
    └── harness-quality-reviewer.md
```

> En el repo original el directorio es `agents/` en la raíz del plugin.
> Al instalarlo, el harness los copia a `~/.claude/agents/` (global)
> o a `.claude/agents/` dentro del proyecto (workspace scope).

---

## 3. Estructura de un archivo de agente

Todo archivo de agente es un `.md` con frontmatter YAML + cuerpo de instrucciones.

```markdown
---
name: nombre-del-agente
description: >
  Una o dos frases que describen cuándo Claude debe invocar este agente.
  El orquestador lee esto para decidir a quién delegar.
allowed-tools:
  - Read
  - Write
  - Bash
  - Grep
  # Listar SOLO las herramientas que este agente necesita.
  # Principio de mínimo privilegio.
---

# Nombre del agente

## Rol

Descripción concisa de la responsabilidad única de este agente.

## Skills precargadas

Antes de actuar, lee los siguientes archivos de skill:

- `.claude/skills/impl/SKILL.md`
- `.claude/skills/verify/SKILL.md`

## Instrucciones

1. Paso concreto uno.
2. Paso concreto dos.
3. Paso concreto tres.

## Criterio de completitud

El agente finaliza cuando:
- [ ] Condición A se cumple.
- [ ] Condición B se cumple.

## Lo que este agente NO hace

- No toca archivos de infraestructura (`infra/`, `k8s/`, `terraform/`).
- No modifica `.env` ni archivos de secretos.
- No hace commit ni push directamente.
```

---

## 4. Los 6 agentes del harness

### 4.1 `harness-worker` — Agente de implementación

| Campo | Valor |
|---|---|
| **Invocado por** | `/work`, `/harness-loop` |
| **Responsabilidad** | Implementar el task activo de `Plans.md` |
| **Skills** | `impl/`, `verify/`, `ci/` |
| **Herramientas** | Read, Write, Bash, Grep, MultiEdit |
| **NO hace** | Review, commit, modificar `.env` o infra |

El worker es el único agente que escribe código de producción.
Lee `Plans.md` para saber qué tarea ejecutar y marca el ítem como completado.

---

### 4.2 `harness-reviewer` — Agente de revisión (orquestador de reviews)

| Campo | Valor |
|---|---|
| **Invocado por** | `/harness-review` |
| **Responsabilidad** | Orquestar los 3 sub-revisores en paralelo |
| **Skills** | `harness-review/` |
| **Herramientas** | Read, Grep, Task (para lanzar sub-agentes) |
| **NO hace** | Escribir código, hacer commit |

Este agente **no revisa directamente** — lanza a los tres agentes de revisión
especializada en paralelo y consolida sus resultados en un único reporte.

```
harness-reviewer
    ├─► harness-security-reviewer   (paralelo)
    ├─► harness-performance-reviewer (paralelo)
    └─► harness-quality-reviewer    (paralelo)
```

---

### 4.3 `harness-security-reviewer` — Revisor de seguridad

| Campo | Valor |
|---|---|
| **Invocado por** | `harness-reviewer` (paralelo) |
| **Responsabilidad** | Detectar vulnerabilidades, inyecciones, auth débil |
| **Skills** | `harness-review/security/` |
| **Herramientas** | Read, Grep |
| **Scope** | Solo lectura — nunca escribe |

Analiza: inyección SQL/NoSQL, autenticación, exposición de secretos,
permisos de archivos, inputs no sanitizados.

---

### 4.4 `harness-performance-reviewer` — Revisor de performance

| Campo | Valor |
|---|---|
| **Invocado por** | `harness-reviewer` (paralelo) |
| **Responsabilidad** | Detectar cuellos de botella, N+1, queries lentas |
| **Skills** | `harness-review/performance/` |
| **Herramientas** | Read, Grep |
| **Scope** | Solo lectura |

Analiza: consultas ORM ineficientes, falta de índices,
ausencia de caché, loops sobre colecciones grandes.

---

### 4.5 `harness-quality-reviewer` — Revisor de calidad

| Campo | Valor |
|---|---|
| **Invocado por** | `harness-reviewer` (paralelo) |
| **Responsabilidad** | Revisar tipado, convenciones, cobertura de tests |
| **Skills** | `harness-review/quality/` |
| **Herramientas** | Read, Grep |
| **Scope** | Solo lectura |

Analiza: ausencia de type hints, funciones sin tests,
nombres inconsistentes, imports no utilizados, complejidad ciclomática.

---

### 4.6 `harness-planner` — Agente de planificación

| Campo | Valor |
|---|---|
| **Invocado por** | `/plan-with-agent`, `/harness-setup` |
| **Responsabilidad** | Generar `spec.md` y `Plans.md` desde un request |
| **Skills** | `planning/` |
| **Herramientas** | Read, Write, Bash (solo lectura del proyecto) |
| **NO hace** | Escribir código de producción |

Produce dos artefactos:
- `spec.md` — descripción del problema y decisiones de diseño.
- `Plans.md` — lista ordenada de tasks para el worker.

---

## 5. Cómo se invocan los agentes

### Desde un comando slash

```markdown
<!-- En .claude/commands/harness-review.md -->
---
name: harness-review
description: Lanza revisión paralela de seguridad, performance y calidad.
---

Invoca al agente harness-reviewer para revisar los cambios actuales.
El agente lanzará sub-revisores en paralelo y consolidará los resultados.
```

### Desde el cuerpo de otro agente (delegación explícita)

```markdown
## Instrucciones del harness-reviewer

1. Lee el diff de los archivos modificados desde el último commit.
2. Lanza en paralelo:
   - Agente `harness-security-reviewer` con el diff como contexto.
   - Agente `harness-performance-reviewer` con el diff como contexto.
   - Agente `harness-quality-reviewer` con el diff como contexto.
3. Espera los tres resultados.
4. Consolida en un único reporte con secciones: Seguridad / Performance / Calidad.
5. Indica si hay hallazgos que bloquean el merge (severidad Alta o Crítica).
```

### Disparadores automáticos (vía hooks)

```json
// .claude-plugin/hooks.json
{
  "PostToolUse": {
    "pattern": "Write|MultiEdit",
    "agent": "harness-worker",
    "action": "run_tests"
  },
  "Stop": {
    "agent": "harness-worker",
    "action": "generate_session_summary"
  }
}
```

---

## 6. Reglas de delegación — cuándo el orquestador delega

El orquestador mantiene su thread delgado. Delega cuando:

| Condición | Agente destino |
|---|---|
| Task nuevo en `Plans.md` | `harness-worker` |
| Cambios en 2+ archivos no triviales | `harness-reviewer` (post-work) |
| Request de nueva feature / módulo | `harness-planner` |
| Commit o PR listo para merge | `harness-reviewer` (obligatorio) |
| Sesión larga con contexto acumulado | Nuevo `harness-worker` con contexto fresco |
| Detección de bug de seguridad | `harness-security-reviewer` aislado |

> **Principio clave**: el orquestador nunca escribe código de producción directamente.
> Siempre delega al worker. Esto preserva la separación entre decisión e implementación.

---

## 7. Separación de responsabilidades: workers vs reviewers

```
┌─────────────────────────────────────────────────────────────┐
│                     WORKERS (escriben)                       │
│  harness-worker     harness-planner                         │
│  allowed: Write, MultiEdit, Bash                             │
│  scope: src/, app/, Plans.md, spec.md                       │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                   REVIEWERS (solo leen)                      │
│  harness-reviewer   harness-security-reviewer               │
│  harness-performance-reviewer   harness-quality-reviewer    │
│  allowed: Read, Grep                                         │
│  scope: cualquier archivo, pero NUNCA Write                 │
└─────────────────────────────────────────────────────────────┘
```

Un reviewer que descubre un problema **reporta** — no corrige.
La corrección vuelve al ciclo como un nuevo task en `Plans.md`.

---

## 8. Relación agentes ↔ skills ↔ hooks

```
Ciclo Plan → Work → Review
│
├── /plan-with-agent
│       └── harness-planner [skill: planning/]
│               └── genera spec.md + Plans.md
│
├── /work
│       └── harness-worker [skill: impl/, verify/, ci/]
│               ├── lee Plans.md → ejecuta task
│               ├── hook PostToolUse → corre tests automáticamente
│               └── marca task como completado
│
└── /harness-review
        └── harness-reviewer [skill: harness-review/]
                ├── harness-security-reviewer  ─┐
                ├── harness-performance-reviewer ├─ paralelo
                └── harness-quality-reviewer   ─┘
                        └── consolida reporte final
```

Los skills son el **conocimiento**. Los agentes son el **rol**.
Los hooks son el **timing**. Los comandos son el **punto de entrada**.

---

## 9. Plantilla base para un agente propio

Copia este template en `.claude/agents/<nombre-agente>.md`:

```markdown
---
name: <nombre-agente>
description: >
  [Una o dos frases. El orquestador lee esto para decidir cuándo invocar
  este agente. Sé específico sobre el dominio y el trigger.]
allowed-tools:
  - Read
  - Grep
  # Agrega Write, Bash, MultiEdit solo si este agente necesita modificar archivos.
  # Si es un reviewer, NUNCA agregues Write.
---

# <Nombre del agente>

## Rol

[Una frase. Qué responsabilidad única tiene este agente en el ciclo de desarrollo.]

## Contexto de activación

Este agente se activa cuando:
- [Condición 1 — ej: hay un task de tipo X en Plans.md]
- [Condición 2 — ej: el orquestador detecta cambios en archivos del dominio Y]

## Skills precargadas

Lee estos archivos antes de actuar:

```
.claude/skills/<categoria>/SKILL.md
```

## Instrucciones

1. [Paso 1 — concreto y accionable]
2. [Paso 2]
3. [Paso 3]

## Paths permitidos

- Puede leer: `<glob>`
- Puede escribir: `<glob>` *(omitir si es reviewer)*

## Paths prohibidos

- `.env`, `.env.*`
- `infra/`, `k8s/`, `terraform/`
- `credentials/`, `secrets/`

## Criterio de completitud

- [ ] [Condición de done 1]
- [ ] [Condición de done 2]

## Lo que este agente NO hace

- [Restricción explícita 1]
- [Restricción explícita 2]
```

---

## 10. Checklist de implementación

Antes de considerar el sistema de agentes funcional en tu harness:

### Estructura
- [ ] Directorio `.claude/agents/` creado en el proyecto.
- [ ] Al menos un worker y un reviewer definidos.
- [ ] Cada agente tiene `name`, `description` y `allowed-tools` en frontmatter.

### Separación
- [ ] Los reviewers no tienen `Write` ni `MultiEdit` en `allowed-tools`.
- [ ] Los workers no tienen acceso a paths protegidos (`.env`, `infra/`).
- [ ] El orquestador no escribe código directamente.

### Skills
- [ ] Cada agente referencia las skills relevantes en su cuerpo.
- [ ] El directorio `.claude/skills/` tiene al menos las categorías que usan los agentes.

### Integración
- [ ] El comando `/harness-review` invoca al reviewer, no al worker.
- [ ] `Plans.md` existe en la raíz del proyecto como artefacto vivo.
- [ ] Los hooks de `PostToolUse` están configurados si se quiere CI local automático.

### Validación
- [ ] Ejecutar `/harness-setup` o equivalente para inicializar el contexto.
- [ ] Hacer una prueba con tarea pequeña: `/plan-with-agent` → `/work` → `/harness-review`.
- [ ] Confirmar que el reviewer no modificó ningún archivo del proyecto.

---

*Referencia: `Chachamaru127/claude-code-harness` · Modelo de delegación: `Gentleman-Programming/gentle-ai`*