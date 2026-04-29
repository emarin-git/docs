# Matriz RACI

Asignaciones de Responsable / Accountable / Consultado / Informado por etapa. **R** = hace el trabajo, **A** = en última instancia responsable (una sola persona), **C** = consultado antes, **I** = informado después.

Esta RACI es la fuente de verdad. Las adaptaciones a nivel de squad están permitidas pero deben ser explícitas en el charter del squad.

---

## Tabla de Contenidos

1. [Cómo leer la matriz](#cómo-leer-la-matriz)
2. [RACI por etapa](#raci-por-etapa)
3. [Decisiones transversales](#decisiones-transversales)
4. [Rutas de escalada](#rutas-de-escalada)

---

## Cómo leer la matriz

- **R** debe hacer el trabajo.
- **A** es el nombre único en la línea — hay exactamente un A por fila.
- **C** debe ser consultado *antes* de que se finalice una decisión.
- **I** debe ser informado *después* de que se tome la decisión.
- Una celda vacía significa: no está en el ciclo por defecto para esta fila.

Los roles están abreviados: PM (Product Manager), TL (Tech Lead), Eng (Engineer), QA, Arch (Architect), Plat (Platform/DevOps + SRE), UX (UX Designer), Sec (Security), CX (CX Research).

## RACI por etapa

### Etapa 0 — Constitution

| Actividad | PM | TL | Eng | QA | Arch | Plat | UX | Sec | CX |
|---|---|---|---|---|---|---|---|---|---|
| Autorar / enmendar constitution | C | C | I | C | **R / A** | C | I | C | I |
| Autorar políticas legibles por máquina | I | C | I | I | C | **R / A** | I | C | |
| Ratificar enmienda | I | C | I | I | **R / A** | C | I | C | |
| Aplicar en CI / deploy | | I | I | I | C | **R / A** | | C | |

### Etapa 1 — Specification

| Actividad | PM | TL | Eng | QA | Arch | Plat | UX | Sec | CX |
|---|---|---|---|---|---|---|---|---|---|
| Autorar spec | **R / A** | R | C | C | C | | R (cara al usuario) | C (sensible) | C |
| Definir requisitos funcionales | **R / A** | C | C | I | | | C | | C |
| Definir requisitos no funcionales | C | **R / A** | C | C | C | C | C | C | |
| Definir acceptance criteria | R | C | I | **R / A** | | | C | | C |
| Lint de calidad del spec (automatización G1) | I | I | I | I | | **R / A** | | | |
| Aprobación del spec (G1 humano) | **R / A** | R | | | C | | C | C | I |

### Etapa 2 — Clarification

| Actividad | PM | TL | Eng | QA | Arch | Plat | UX | Sec | CX |
|---|---|---|---|---|---|---|---|---|---|
| Ejecutar sesión de clarification | **R / A** | R | C | C | C | | C | C | C |
| Resolver ítems `MUST-CLARIFY` | **R / A** | R | I | I | C | | C | C | C |
| Mantener clarification log | **R / A** | I | I | I | | I | I | I | I |
| Cerrar G2 | **R / A** | C | | I | | I | | | |

### Etapa 3 — Design

| Actividad | PM | TL | Eng | QA | Arch | Plat | UX | Sec | CX |
|---|---|---|---|---|---|---|---|---|---|
| Autorar documento de diseño | I | **R / A** | R | C | C | C | C (UI) | C (sensible) | I |
| Autorar task breakdown | I | **R / A** | R | C | | | I | | |
| Definir test strategy | I | R | R | **R / A** | | | C | C | |
| Cambios de contratos | I | R | C | I | **R / A** (cross-servicio) | C | | C | |
| Design review (G3) | C | R | C | C | **R / A** (cross-servicio) | C | C (UI) | C (sensible) | |

### Etapa 4 — Implementation

| Actividad | PM | TL | Eng | QA | Arch | Plat | UX | Sec | CX |
|---|---|---|---|---|---|---|---|---|---|
| Implementar tarea | I | C | **R** | I | | | | | |
| Disciplina TDD | I | **A** | R | C | | | | | |
| Autorar/ejecutar agentes | I | C | **R** | | | A | | | |
| Code review | I | R | R | C | | | C (UI) | C (sensible) | |
| Aprobar merge | I | **R / A** | R | | | | | C (sensible) | |
| Build completo (G4) | I | A | R | I | | R | | | |

### Etapa 5 — Validation

| Actividad | PM | TL | Eng | QA | Arch | Plat | UX | Sec | CX |
|---|---|---|---|---|---|---|---|---|---|
| Ejecutar pipeline de validation | I | C | C | **R / A** | | R | | C | |
| Revisión del drift report | C | R | C | **R / A** | C (cross-servicio) | C | | C | |
| Validación NFR | C | R | C | **R / A** | | R | | | |
| Signoff de aceptación (G5) | **R / A** | R | | R | C (cross-servicio) | R (operabilidad) | R (cara al usuario) | R (sensible) | C |

### Etapa 6 — Deployment

| Actividad | PM | TL | Eng | QA | Arch | Plat | UX | Sec | CX |
|---|---|---|---|---|---|---|---|---|---|
| Definir rollout plan | C | R | C | C | C (cross-servicio) | **R / A** | C (cara al usuario) | C | I |
| Ejecutar progressive rollout | I | R | R | I | | **R / A** | | | I |
| Aprobar gate G6 | I | C | | | | **R / A** | | C | |
| Decisiones de rollback / kill-switch | C | R | R | I | | **R / A** | | | C |

### Etapa 7 — Operations

| Actividad | PM | TL | Eng | QA | Arch | Plat | UX | Sec | CX |
|---|---|---|---|---|---|---|---|---|---|
| Operar servicio / on-call | I | R | **R / A** | | | C | | | |
| Gestión de SLO | C | R | C | | C | **R / A** | | | |
| Respuesta a incidentes | C | R | R | | | **R / A** | | C | |
| Postmortem | C | **R / A** | R | C | C (cross-servicio) | R | | C | C |
| Enmiendas al spec por incidentes | **R / A** | R | C | C | C (transversal) | | C | C | C |
| Triage de señal de usuario | **R / A** | C | I | | | | C | | R |

## Decisiones transversales

Algunas decisiones no encajan en una sola etapa. Valores por defecto:

| Decisión | A | R | C | I |
|---|---|---|---|---|
| Agregar nuevo dominio de producto / squad | VP Eng + VP Product | Squad EM | Architecture Council, Platform | Todos |
| Nueva superficie de API pública | Architect | Squad TL | Security, Platform | PM, Squad |
| Waiver constitucional | Architect | TL solicitante | Security si aplica | Todos los Squads (transparencia) |
| Declaración de patch-path en un PR | Autor | Autor | Revisor | Squad TL |
| Rollback de release en incidente | Incident Commander | On-call SRE | Squad TL, PM | Todos |
| Cambio de permisos de agente IA | Architect + Platform Lead | Platform | Security | Todos |

## Rutas de escalada

Los conflictos se resuelven al nivel más bajo que tenga autoridad:

1. **Dentro de un squad** — Tech Lead + PM + UX co-deciden. EM media si es necesario.
2. **Entre squads en un contrato** — TL del squad principal + TL del squad secundario. Architect media.
3. **Estándares transversales o conflicto constitucional** — Architecture Council.
4. **Estratégico / cross-funcional** — VP Eng + VP Product + (VP Design cuando aplica).
5. **Override de seguridad** — Security Engineer puede bloquear un release; solo el CTO puede anular.

El marco evita explícitamente los patrones de "todo va al CTO". Si la cadencia de escalada aumenta, examina el límite, no las personas.

---

[← Roles](roles-sp.md) · [Squads ↑](squads-sp.md)
