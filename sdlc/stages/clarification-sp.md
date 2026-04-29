# Etapa 2 — Clarification

La clarificación es una etapa dedicada, no un efecto secundario del diseño. Su propósito es resolver cada `MUST-CLARIFY` y revelar cada supuesto oculto *antes* de que alguien se comprometa con la arquitectura o programe trabajo.

---

## Tabla de Contenidos

1. [Propósito](#propósito)
2. [Inputs y outputs](#inputs-y-outputs)
3. [Cómo funciona](#cómo-funciona)
4. [Log de clarificación](#log-de-clarificación)
5. [Rol del agente de IA](#rol-del-agente-de-ia)
6. [Quality gate G2](#quality-gate-g2)
7. [Mejores prácticas y anti-patterns](#mejores-prácticas-y-anti-patterns)

---

## Propósito

La mayoría de las features fallidas fallan en las costuras: un supuesto no declarado, un requisito no-funcional que nadie poseía, un "ya lo resolveremos" que se convirtió en "nunca lo resolvimos". Una etapa de Clarification separada captura esto sistemáticamente.

También es la etapa donde los agentes de IA agregan más valor medible: los agentes son excelentes para sondear brechas, contradicciones y casos límite faltantes.

## Inputs y outputs

| | |
|---|---|
| **Inputs** | `spec.md` con marcadores `MUST-CLARIFY` y `SHOULD-CLARIFY`; constitution; specs relacionadas |
| **Outputs** | `spec.md` actualizada con todos los `MUST-CLARIFY` resueltos; log `clarifications.md` de cada Q/A y decisión; cambios a la spec marcados |
| **Dueños** | Product Manager (lidera), Tech Lead, Architect (para preguntas cross-cutting), CX/UX (para preguntas que afectan al usuario) |
| **Cadencia** | Con tiempo limitado: el objetivo de clarificación es 1–3 días hábiles |

## Cómo funciona

Tres flujos de trabajo corren en paralelo:

1. **Clarificación humana.** El PM convoca una sesión corta de clarificación (o un hilo asíncrono) por grupo de preguntas relacionadas. Las decisiones y la justificación van al log de clarificación.
2. **Sondeo liderado por agentes.** Un agente de IA lee la spec y la constitution, y produce una lista estructurada de brechas, contradicciones y casos límite probablemente faltantes. Los hallazgos del agente se revisan y se incorporan en la spec, se despachan como nuevos items `MUST-CLARIFY`, o se descartan explícitamente con justificación.
3. **Reconciliación cross-spec.** La plataforma busca specs relacionadas que toquen las mismas superficies y revela conflictos (p. ej., dos specs que proponen semánticas diferentes de rate-limit en el mismo endpoint).

Cuando la clarificación produce cambios materiales — nuevos FRs, alcance removido, cambios de NFR — la spec se actualiza y la spec re-corre G1. La iteración es normal aquí.

## Log de clarificación

Un log simple, append-only, asociado al spec ID. Vive en `specs/<feature-id>/clarifications.md`.

```markdown
# Clarification log — SPEC-2026-0142

## Q1. Do we charge for storage of incomplete uploads after 24h?
- Raised by: spec author (alex@), 2026-04-22
- Resolution: No. Incomplete uploads are deleted at the 24h TTL; never billed.
- Decided by: alex@ (PM), priya@ (Eng), with input from finance.
- Spec impact: FR-3 updated to specify deletion at TTL; NFR added on cleanup SLO.
- Decided: 2026-04-23

## Q2. (agent-raised) What happens if the client sends chunks out of order?
- Raised by: clarify-agent, 2026-04-22
- Resolution: Out-of-order chunks accepted; server reconstructs by offset.
- Spec impact: FR-5 added.
- Decided: 2026-04-23

## Q3. (agent-raised) What is the authentication model for the resume endpoint?
- Raised by: clarify-agent, 2026-04-22
- Resolution: Bearer token same as upload session; resume tokens scoped to that session only.
- Spec impact: NFR-S2 added; constitution clause §S1 referenced.
- Decided: 2026-04-23
```

El log es historia inmutable. Si una decisión clarificada se revierte después, eso es una *nueva* entrada de clarificación con un enlace a la original.

## Rol del agente de IA

El **clarification agent** dedicado recibe:

- La spec (texto canónico + forma estructurada).
- La constitution.
- Specs relacionadas y adyacentes.
- El schema registry / catálogo de APIs.
- Modos de fallo anonimizados de incidentes previos etiquetados a superficies similares.

Produce salida estructurada, calificada por confianza, en tres categorías:

- **Probable contradicción** — "FR-3 dice rechazar después de 24h pero NFR-2 implica retención para analytics; reconcilia."
- **Probable brecha** — "La spec no dice nada sobre autenticación para el endpoint de resume."
- **Probable caso límite** — "¿Qué pasa si el cliente envía chunks fuera de orden?"

El agente *no* tiene permitido enmendar silenciosamente la spec. Abre preguntas; los humanos deciden.

## Quality gate G2

Automatizado y estricto:

- Cero marcadores `MUST-CLARIFY` abiertos en la spec.
- Cada entrada en el log de clarificación tiene una decisión registrada y un dueño.
- La spec, post-clarificación, re-pasa G1.

Los items `SHOULD-CLARIFY` pueden llevarse al Design pero deben rastrearse explícitamente.

## Mejores prácticas y anti-patterns

**Mejores prácticas.**

Limita el tiempo de clarificación agresivamente. La respuesta correcta usualmente es: sí, no, o "lo aplazaremos con estos supuestos explícitos". Ciclos largos de clarificación usualmente son una señal de que la spec es demasiado grande — divídela. Siempre involucra a CX/UX para cualquier pregunta visible al usuario; la clarificación solo-tech frecuentemente pierde el impacto al usuario de una decisión "pequeña". Escribe las decisiones en el lenguaje de la spec ("se agrega AC-7" en lugar de "decidimos manejar chunks fuera de orden") para que la trazabilidad sobreviva.

**Anti-patterns.**

Dejar que la clarificación se convierta en un ejercicio de consenso de varias semanas — eso es un fallo de planificación, no una necesidad de clarificación. Resolver clarificaciones verbalmente sin registrarlas — resurgirán como bugs. Permitir que el agente edite la spec sin aprobación humana. "Clarificar" acordando "discutir en design", lo cual solo difiere el problema y contamina la etapa de design.

---

[← Specification](specification-sp.md) · [Siguiente: Design →](design-sp.md)
