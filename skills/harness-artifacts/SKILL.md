---
name: harness-artifacts
description: Formato de artefactos y registro de versiones del Harness. Cargar en Flujo Completo al persistir artefactos por primera vez.
---

## Formato de artefactos

Cada feature usa su propia carpeta:

```
docs/harness/<nombre-de-la-feature>/
├── investigation-report.md
├── proposals.md
├── requirements.md
├── design.md
├── tasks.md
└── review-report.md
```

El nombre de la carpeta usa kebab-case descriptivo de la feature (ej. `autenticacion-oauth`, `exportacion-pdf`).

## Registro de versiones

El archivo `docs/harness/.harness-version.md` registra los cambios al Harness. Ante cualquier modificación de agentes o flujo, añadir una entrada con: versión incremental, fecha y resumen de cambios.
