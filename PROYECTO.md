# Requirements Document

## Introduction

Este documento define los requisitos del **Harness de Agentes para Claude Code**: una metodología de trabajo estructurada que establece cómo Claude Code debe operar en cualquier proyecto de desarrollo de software. El harness define varios agentes con responsabilidades exclusivas, sus interacciones, restricciones y el flujo de trabajo entre ellos. El sistema está diseñado para ser evolutivo y adaptarse con el tiempo.

El objetivo principal es garantizar que el trabajo esté bien especificado, técnicamente diseñado, planificado, implementado y revisado de forma ordenada, con el desarrollador tomando siempre las decisiones clave.

## Glossary

- **Harness**: El conjunto completo de agentes, reglas de interacción y flujo de trabajo que define la metodología de operación de Claude Code.
- **Orquestador**: Agente punto de entrada único que coordina el trabajo de todos los demás agentes.
- **Investigador**: Agente responsable de explorar el problema, el codebase y el contexto relevante.
- **Proposer**: Agente responsable de generar propuestas de solución y presentarlas al desarrollador para su decisión.
- **Especificador**: Agente responsable de definir con precisión los requisitos de la solución elegida.
- **Diseñador**: Agente responsable de crear el diseño técnico de la solución.
- **Planificador**: Agente responsable de generar el plan de tareas de implementación.
- **Constructor**: Agente responsable de escribir y modificar código.
- **Reviewer**: Agente independiente responsable de revisar la implementación final.
- **Desarrollador**: El usuario humano que interactúa con el Harness a través del Orquestador.