# Etapa 1 — Specification

La spec es **el contrato**. Todo lo que sigue — design, tasks, código, pruebas, aceptación — se deriva de la spec y la referencia.

---

## Tabla de Contenidos

1. [Propósito](#propósito)
2. [Inputs y outputs](#inputs-y-outputs)
3. [Qué debe contener una spec](#qué-debe-contener-una-spec)
4. [Template de spec](#template-de-spec)
5. [Estándares de calidad de spec](#estándares-de-calidad-de-spec)
6. [Quality gate G1](#quality-gate-g1)
7. [Mejores prácticas y anti-patterns](#mejores-prácticas-y-anti-patterns)

---

## Propósito

Capturar *qué* se está construyendo y *por qué*, con suficiente detalle para que:

- Un revisor pueda identificar brechas antes de escribir código.
- Un agente de IA pueda generar un diseño inicial y un desglose de tareas.
- Las pruebas de aceptación puedan derivarse directamente de los criterios de aceptación.
- Un futuro ingeniero pueda reconstruir la intención solo con la spec.

Una spec describe **comportamiento, no implementación**. La implementación es problema de la siguiente etapa.

## Inputs y outputs

| | |
|---|---|
| **Inputs** | Señal del cliente (CX), estrategia de producto, constitution, líneas base de NFRs, specs relacionadas previas |
| **Outputs** | `specs/<feature-id>/spec.md`, front-matter estructurado, criterios de aceptación enlazados |
| **Dueños** | Product Manager (lidera), Tech Lead (co-autor), CX/UX (co-autor para trabajo user-facing) |
| **Cadencia** | Por feature; se esperan 0.5–3 días de autoría + revisión |

## Qué debe contener una spec

Cada spec tiene nueve secciones requeridas. Si falta cualquiera de ellas, falla G1.

1. **Front-matter** — feature ID, status, dueños, enlaces a specs relacionadas y cláusulas de la constitution.
2. **Problema y motivación** — el problema del usuario/negocio en lenguaje claro. Cita evidencia.
3. **Personas y escenarios** — quién está afectado, qué están tratando de hacer.
4. **Requisitos funcionales** — comportamientos, en forma numerada y testable.
5. **Requisitos no-funcionales (NFRs)** — performance, disponibilidad, seguridad, accesibilidad, costo.
6. **Criterios de aceptación** — declaraciones Given/When/Then que mapean a tests automatizables.
7. **Fuera de alcance** — no-objetivos explícitos. A menudo la sección más útil.
8. **Preguntas abiertas / clarificaciones requeridas** — marcadores `MUST-CLARIFY` consumidos por la Etapa 2.
9. **Riesgos y supuestos** — qué podría salir mal, en qué estamos apostando.

Dos secciones opcionales pero recomendadas: **Plan de telemetría** (qué mediremos) y **Boceto de plan de rollout** (canary, flags, kill switch).

## Template de spec

```markdown
---
id: SPEC-2026-0142
title: Resumable file uploads for large media
status: draft  # draft | clarifying | approved | implementing | shipped | retired
owners:
  product: alex@
  engineering: priya@
  ux: jordan@
related:
  constitution: [P1, A2, S1]
  specs: [SPEC-2025-0099]
  parent_epic: EPIC-2026-0007
---

# Resumable file uploads for large media

## 1. Problem & motivation
Creators uploading >500 MB videos lose progress on flaky networks. 14% of upload sessions
abandon at >2 retries (source: telemetry Q1'26). Cost to the business: ~$X/month in support
load and ~Y% drop in conversion for the publish flow.

## 2. Personas & scenarios
- **Creator on hotel Wi-Fi** uploads a 2 GB conference talk; connection drops every few minutes.
- **Mobile creator** uploads 800 MB from a transit-tunnel network with intermittent connectivity.

## 3. Functional requirements
FR-1. Uploads of any size MUST be resumable across a single network drop of up to 30 minutes.
FR-2. The client MUST surface upload progress with byte-level granularity.
FR-3. The server MUST reject resume attempts after 24 hours from the last chunk.
FR-4. ...

## 4. Non-functional requirements
NFR-1. Resume operation latency p99 ≤ 1.5s.
NFR-2. Chunked write path MUST sustain 100 MB/s per upload session at server.
NFR-3. PII handling per Constitution §S1.

## 5. Acceptance criteria
AC-1. Given a 1 GB upload at 50% progress, when the network drops for 5 minutes and recovers,
      then the upload resumes from the last acknowledged chunk within 2 seconds.
AC-2. Given a paused upload older than 24 hours, when the client attempts to resume,
      then the server returns `410 Gone` and the client surfaces a "session expired" error.
AC-3. ...

## 6. Out of scope
- Multi-device upload handoff (separate spec SPEC-2026-0151).
- Server-side transcoding behavior changes.

## 7. Open questions
- [MUST-CLARIFY] Do we charge for storage of incomplete uploads after 24h?
- [SHOULD-CLARIFY] Should resume tokens be encrypted-at-rest beyond default volume encryption?

## 8. Risks & assumptions
- ASSUMPTION: Existing object store supports multi-part with TTL on parts.
- RISK: Mobile clients may aggressively kill background tasks; needs investigation.

## 9. Telemetry plan
- Counter `upload.resume.attempts` by outcome (resumed | expired | failed).
- Histogram `upload.resume.latency`.
```

El front-matter estructurado es crítico: la spec platform lo indexa para trazabilidad y los agentes de IA se basan en él.

## Estándares de calidad de spec

Una spec debe satisfacer estas propiedades; la spec platform les hace lint automáticamente.

- **Atómica y direccionable.** Cada requisito y criterio de aceptación tiene un ID estable (`FR-1`, `AC-3`). Los IDs estables sobreviven a renumeraciones — nunca reutilices IDs.
- **Testable.** Cada AC es derivable a una prueba automatizada. "Debe ser rápido" no es testable; "p99 ≤ 300ms a 200 RPS" sí lo es.
- **Acotada.** Una spec cubre una feature. Si abarca múltiples unidades desplegables, divídela (y enlaza las hijas).
- **Auto-contenida en intención.** Un lector no familiarizado con el sistema debe entender *por qué*, incluso si parte del *cómo* requiere leer contexto enlazado.
- **Consistente con la constitution.** Ningún requisito puede contradecir un principio constitucional sin un waiver explícito y aprobado en el front-matter.
- **Revisada por los tres roles.** Product (intención), Engineering (factibilidad), CX/UX (usabilidad) para cualquier feature visible al usuario.

## Quality gate G1

G1 es híbrido (automatización + humano):

**Verificaciones automatizadas:**

- Schema del front-matter válido.
- Todas las secciones requeridas presentes.
- Cada FR/NFR/AC tiene un ID estable.
- Los ACs hacen lint como Given/When/Then.
- Las cláusulas enlazadas de la constitution existen.
- Sin `MUST-CLARIFY` huérfano sin triagear durante >5 días hábiles.

**Verificaciones humanas:**

- Signoff del PM de que el problema es real y vale la pena resolver.
- Signoff del Tech Lead de que el alcance es factible en la cadencia propuesta.
- Signoff de UX/CX para cambios visibles al usuario.
- Signoff del Architect si los contratos públicos o el trabajo cross-service están afectados.

## Mejores prácticas y anti-patterns

**Mejores prácticas.**

Co-autora con el agente de IA. Que el agente produzca un primer borrador de spec a partir de un planteamiento del problema, luego haz que los humanos aprieten los FRs y ACs. El agente es excelente para revisiones de estructura y completitud; los humanos son esenciales para el juicio de producto. Escribe la sección **fuera de alcance** primero — es donde se esconden los malentendidos. Para cualquier preocupación cross-cutting, enlaza una spec canónica existente en lugar de re-especificar. Mantén la spec viva: enmiéndala durante la implementación cuando la realidad contradiga los supuestos, en lugar de dejar que el código se aleje.

**Anti-patterns.**

La spec "implementación disfrazada" que prescribe nombres de clases específicos y SQL — eso pertenece al diseño. La spec "wishlist" que es un backlog de deseos vagos; las specs son atómicas. La spec "congelada" que nadie actualiza después de aprobarla, llevando a drift. Specs escritas *después* de que el código aterrice ("retro-specs") — describen lo que se construyó, no lo que se quería, y son inútiles como contratos. Specs que silenciosamente contradicen la constitution y dependen de la fatiga del revisor para colarse.

---

[← Constitution](constitution-sp.md) · [Siguiente: Clarification →](clarification-sp.md)
