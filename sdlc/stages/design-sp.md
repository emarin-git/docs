# Etapa 3 — Design

El Design toma una spec clarificada y produce un plan técnico: arquitectura, contratos, cambios al modelo de datos, secuenciación, y un desglose concreto de tareas listo para implementación.

---

## Tabla de Contenidos

1. [Propósito](#propósito)
2. [Inputs y outputs](#inputs-y-outputs)
3. [Estructura del documento de diseño](#estructura-del-documento-de-diseño)
4. [Desglose de tareas](#desglose-de-tareas)
5. [Rol del agente de IA](#rol-del-agente-de-ia)
6. [Quality gate G3](#quality-gate-g3)
7. [Mejores prácticas y anti-patterns](#mejores-prácticas-y-anti-patterns)

---

## Propósito

Cerrar la brecha entre el *qué* (spec) y el *cómo* (código) sin caer en la implementación. El diseño debe ser lo suficientemente concreto para que las tareas de implementación puedan programarse, paralelizarse y asignarse a humanos o agentes de IA, pero lo suficientemente abstracto para sobrevivir a refactors menores.

Crucialmente, esta etapa produce el **task graph** — la unidad de trabajo que la etapa de implementación consume.

## Inputs y outputs

| | |
|---|---|
| **Inputs** | `spec.md` aprobada, log de clarificación, constitution, contexto de arquitectura del sistema, schema registry |
| **Outputs** | `specs/<feature-id>/design.md`, diffs de contratos (OpenAPI / proto / SQL DDL), `tasks.md` task graph, estrategia de pruebas |
| **Dueños** | Tech Lead (lidera), Architect (revisa preocupaciones cross-cutting), ingenieros del Squad (co-autores y consumidores) |
| **Cadencia** | Por feature; 1–5 días dependiendo de la complejidad |

## Estructura del documento de diseño

```markdown
# Design — SPEC-2026-0142: Resumable file uploads
Version: 1  •  Status: review

## 1. Summary
One paragraph: the chosen approach and why.

## 2. Context & constraints
- Constitution clauses applied: A2 (event-driven), S1 (PII), Q1 (coverage)
- Existing systems touched: upload-service, object-store, billing-events bus

## 3. Architecture
- Diagram (Mermaid) of the components changed and added.
- Component-by-component delta.

## 4. Contracts
- API: diff against the OpenAPI spec, semver impact stated.
- Events: schema changes registered; consumer impact analyzed.
- DB: DDL diff; migration plan; rollback plan.

## 5. Data model
- New tables, columns, indexes; retention policy; PII tagging.

## 6. Sequencing & dependencies
- Order of merge: what must land first to avoid breakage.
- Backfill needs.

## 7. Test strategy
- Unit: TDD by default; coverage targets.
- Contract: consumer-driven contract tests against new endpoints/events.
- Integration: scenarios that exercise FR-1..FR-N.
- Acceptance: each AC mapped to an automated check.
- Non-functional: load profile, chaos drills, accessibility audit if user-facing.

## 8. Risks & mitigations
- Top 3 risks; for each: mitigation, detection, fallback.

## 9. Rollout plan
- Feature flag(s); progressive delivery profile; kill switch; revert criteria.

## 10. Alternatives considered
- A short list with the reason each was rejected. Future readers will thank you.
```

El design referencia la spec por IDs estables (FR-1, AC-3) — nunca los repite.

## Desglose de tareas

Una lista plana o poco profunda de tareas, cada una de ≤1 día de esfuerzo, cada una enlazable, cada una con dependencias explícitas. La plataforma almacena esto como un grafo.

```markdown
# Tasks — SPEC-2026-0142

- [ ] T-01 Add `upload_sessions` table + migration  (covers FR-1, FR-3)
- [ ] T-02 Implement chunk-write endpoint           (covers FR-1, FR-2; depends T-01)
- [ ] T-03 Implement resume endpoint                 (covers FR-1, AC-1; depends T-01, T-02)
- [ ] T-04 Implement TTL cleanup job                 (covers FR-3, AC-2)
- [ ] T-05 Client SDK changes for resumption         (covers FR-2; depends T-02, T-03)
- [ ] T-06 Telemetry counters/histograms             (covers telemetry plan)
- [ ] T-07 Acceptance test scaffolding               (covers AC-1..AC-5)
- [ ] T-08 Load test scenario (100 MB/s sustained)  (covers NFR-2)
- [ ] T-09 Runbook + on-call notes                   (covers operations)
```

Cada tarea tiene front-matter que el agente y CI consumen:

```yaml
id: T-03
spec: SPEC-2026-0142
covers: [FR-1, AC-1]
depends_on: [T-01, T-02]
acceptance:
  - test: tests/upload/resume.spec.ts::resumes_within_2s
estimated_effort: 0.5d
agent_assignable: true   # may be implemented by an AI agent under review
```

Las tareas marcadas `agent_assignable: false` (p. ej., paths sensibles a seguridad, migraciones irreversibles) requieren autoría solo-humana.

## Rol del agente de IA

Dos loops distintos de agente en esta etapa:

1. **Design draft agent.** Dada la spec + constitution + contexto de arquitectura, redacta el documento de diseño y un task graph inicial. Siempre revisado por el Tech Lead — es un punto de partida, no final.
2. **Critique agent.** Agente independiente corrido con un prompt explícito de "encuentra qué está mal": modos de fallo faltantes, NFRs no manejados, contratos que entran en conflicto con vecinos, problemas de secuenciación, brechas en la estrategia de pruebas. Sus hallazgos abren issues contra el draft.

Los critique agents son más valiosos cuando se corren *separadamente* del drafting agent, idealmente con diferentes prompts/contextos, para evitar mode collapse.

## Quality gate G3

Híbrido:

- **Automatizado.** Los contratos pasan lint limpio (OpenAPI/proto válidos, clasificación semver correcta), pasa la verificación de compatibilidad del schema-registry, el DDL tiene un rollback, cada tarea referencia la spec, cada AC está cubierto por al menos una tarea.
- **Humano.** Aprobación del Architect si es cross-service o nuevo contrato público; aprobación del Tech Lead; revisión de Platform/SRE si los NFRs cambian la postura del SLO; revisión de Security si la clasificación de datos o auth cambia.

Si un design revela que la spec está mal, regresa a Specification. Eso es normal y no es un modo de fallo.

## Mejores prácticas y anti-patterns

**Mejores prácticas.**

Diseña *el contrato primero*, luego la implementación. Los contratos son para siempre (o al menos dolorosos de cambiar); el código interno es barato. Haz real la sección de alternativas — es una de las partes más valiosas del diseño para futuros mantenedores y para revisión. Mantén las tareas lo suficientemente pequeñas para que el inner TDD loop quepa dentro de una tarea. Corre el critique agent antes de la revisión del Architect; deja que los humanos pasen su tiempo en juicio, no en encontrar brechas. Toma en serio la secuenciación — un diseño hermoso con un mal merge order bloqueará al squad.

**Anti-patterns.**

Designs que re-litigan la spec. Designs que dan la vuelta a los NFRs ("haremos que sea rápido"). Designs que son una gigantesca tarea de 8 semanas haciéndose pasar por una feature. Designs que omiten migraciones o planes de rollback porque "eso lo manejamos después". Designs auto-generados por un agente y mergeados sin crítica humana — el modo de fallo son patrones genéricos convergentes que no encajan bien con nada.

---

[← Clarification](clarification-sp.md) · [Siguiente: Implementation →](implementation-sp.md)
