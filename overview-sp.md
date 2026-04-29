# Marco SDLC SDD-First

Un Software Development Life Cycle listo para producción construido alrededor de **Spec-Driven Development (SDD)** como paradigma principal, con **Test-Driven Development (TDD)** integrado en la capa de implementación y los **flujos de trabajo asistidos por IA** como una preocupación de primera clase.

Este conjunto de documentación está estructurado como un repositorio de documentación. Trátalo como la fuente canónica de verdad sobre cómo fluye el trabajo de ingeniería a través de la organización.

---

## Tabla de Contenidos

1. [¿Qué es SDD-First?](#qué-es-sdd-first)
2. [Por qué existe este marco](#por-qué-existe-este-marco)
3. [Principios fundamentales](#principios-fundamentales)
4. [Dónde encaja SDD (y dónde no)](#dónde-encaja-sdd-y-dónde-no)
5. [SDD vs TDD vs Agile vs Waterfall](#sdd-vs-tdd-vs-agile-vs-waterfall)
6. [Mapa de documentos](#mapa-de-documentos)
7. [Cómo usar este marco](#cómo-usar-este-marco)

---

## ¿Qué es SDD-First?

**Spec-Driven Development (SDD)** trata las especificaciones como *artefactos ejecutables, versionados y de primera clase* que impulsan los planes, las tareas, el código y las pruebas. En lugar de que las specs sean documentos de requisitos que se escriben una vez y se archivan, la spec es la fuente de verdad contra la que operan tanto los agentes de IA como los humanos. Cuando la spec cambia, los artefactos derivados (planes, tareas, pruebas, código) se regeneran o se marcan para reconciliación.

Este marco integra SDD con TDD. SDD responde *qué* y *por qué*; TDD responde *cómo* a nivel de código. La spec describe el comportamiento de la feature; las pruebas describen el comportamiento unitario/de contrato. Ambos son requeridos.

El marco está informado por las herramientas y prácticas actuales de SDD (2026) — notablemente GitHub Spec Kit (Specify → Plan → Tasks → Implement) y AWS Kiro (Requirements → Design → Tasks) — generalizadas en un SDLC end-to-end adecuado para una organización real.

## Por qué existe este marco

El Agile tradicional entrega de forma iterativa pero es débil en trazabilidad y tiende a producir specs como tickets desechables. Waterfall produce specs pesadas que rara vez coinciden con la implementación. La codificación asistida por IA amplifica ambos modos de fallo: los agentes generan con gusto código plausible a partir de prompts vagos, produciendo salidas rápidas que nadie puede verificar.

SDD-First existe para cerrar esas brechas. Las specs se convierten en el contrato entre humanos, agentes de IA, pruebas y operaciones. El SDLC, la estructura organizacional y la plataforma están todos diseñados para mantener ese contrato honesto.

## Principios fundamentales

El marco está construido sobre siete principios no-negociables, codificados en la [Constitution](sdlc/stages/constitution-sp.md):

1. **La spec es el contrato.** No se escribe código de producción sin una spec mergeada.
2. **Las specs son ejecutables.** Las specs generan planes, tareas, stubs de pruebas y scaffolds.
3. **Las pruebas son el piso, no el techo.** TDD a nivel unitario; pruebas de aceptación derivadas de la spec a nivel de feature.
4. **Los agentes de IA son responsables.** Los agentes operan dentro de la spec; la revisión humana es obligatoria en gates nombrados.
5. **La trazabilidad es end-to-end.** Cada línea de código se enlaza a una tarea, cada tarea a una spec, cada spec a un principio constitucional.
6. **El drift es un incidente.** El drift entre spec/código/pruebas se detecta automáticamente y se trata como un defecto.
7. **Las specs escalan hacia abajo.** El ciclo de vida completo es obligatorio para features; existen rutas ligeras para trabajo trivial.

## Dónde encaja SDD (y dónde no)

**SDD es más efectivo para:**

- Productos greenfield y nuevas features en productos existentes
- Features cross-team o cross-service donde los contratos importan
- Cualquier cosa que un agente de IA ayudará materialmente a construir
- Dominios sensibles a compliance (industrias reguladas, sistemas críticos para seguridad)
- Sistemas de larga vida donde la memoria institucional decae más rápido que el código

**SDD es excesivo para:**

- Correcciones de bugs de una línea y correcciones de typos
- Hotfixes durante un incidente activo
- Prototipos desechables y spikes (usa una ruta ligera "Spike")
- Refactors puros que preservan el comportamiento (cubiertos por pruebas, no por specs)

El marco proporciona una **ruta ligera** explícita para la segunda categoría — ver [Lifecycle](sdlc/lifecycle-sp.md#rutas-ligeras).

## SDD vs TDD vs Agile vs Waterfall

| Dimensión | Waterfall | Agile (canónico) | TDD | SDD-First (este marco) |
|---|---|---|---|---|
| Fuente de verdad | Documento de requisitos | Backlog / tickets | Suite de pruebas | Repo de specs versionadas |
| Granularidad | Proyecto | Story | Función/clase | Feature → Task → Test → Code |
| Manejo de cambios | Change request | Re-priorizar | Refactor | Re-spec → regenerar |
| Encaje con agentes IA | Pobre | Mediocre | Bueno (codegen desde tests) | Nativo |
| Trazabilidad | Pesada al inicio, decae | Débil | Fuerte a nivel unitario | End-to-end |
| Mejor para | Proyectos de alcance fijo | Descubrimiento | Corrección de código | Features en organizaciones aumentadas con IA |
| Peor para | Cambio | Planificación a largo horizonte | Features cross-cutting | Tareas triviales |

SDD no reemplaza a TDD. SDD envuelve a TDD: la spec produce pruebas de aceptación *y* maneja el ciclo de pruebas unitarias en el que TDD se especializa.

## Mapa de documentos

```
/docs
  overview-sp.md ............................ estás aquí
  /sdlc
    lifecycle-sp.md ......................... flujo end-to-end + rutas ligeras
    /stages
      constitution-sp.md .................... principios, gobernanza, reglas de agentes IA
      specification-sp.md ................... qué/por qué; funcional + no-funcional
      clarification-sp.md ................... resolver ambigüedad antes del diseño
      design-sp.md .......................... plan técnico, arquitectura, contratos
      implementation-sp.md .................. construcción guiada por TDD, flujos de agentes
      validation-sp.md ...................... aceptación, quality gates, drift checks
      deployment-sp.md ...................... release, entrega progresiva
      operations-sp.md ...................... observabilidad, feedback, ciclo de aprendizaje
  /organization
    roles-sp.md ............................. Product, Eng, QA, Arch, Platform, CX/UX
    squads-sp.md ............................ modelo de equipos cross-funcionales
    raci-sp.md .............................. matriz de responsabilidad por etapa
  /architecture
    sdd-platform-sp.md ...................... arquitectura de plataforma de referencia
  /best-practices
    policies-sp.md .......................... prácticas por etapa, anti-patterns, guía de escala
```

## Cómo usar este marco

- **¿Nuevo en SDD?** Lee este overview, luego [Lifecycle](sdlc/lifecycle-sp.md), luego la [Constitution](sdlc/stages/constitution-sp.md).
- **¿Adoptando en una organización existente?** Empieza con [Roles](organization/roles-sp.md) y [Squads](organization/squads-sp.md) para encontrar tu rollout mínimo viable.
- **¿Construyendo la plataforma?** Ve directamente a [SDD Platform Architecture](architecture/sdd-platform-sp.md).
- **¿Escribiendo o revisando specs?** Lee [Specification](sdlc/stages/specification-sp.md) y [Clarification](sdlc/stages/clarification-sp.md), luego [Policies](best-practices/policies-sp.md).

---

[Siguiente: SDLC Lifecycle →](sdlc/lifecycle-sp.md)
