# Stage 2 — Clarification

Clarification is a dedicated stage, not a side-effect of design. Its purpose is to resolve every `MUST-CLARIFY` and surface every hidden assumption *before* anyone commits to architecture or schedules work.

---

## Table of Contents

1. [Purpose](#purpose)
2. [Inputs and outputs](#inputs-and-outputs)
3. [How it works](#how-it-works)
4. [Clarification log](#clarification-log)
5. [AI agent role](#ai-agent-role)
6. [Quality gate G2](#quality-gate-g2)
7. [Best practices and anti-patterns](#best-practices-and-anti-patterns)

---

## Purpose

Most failed features fail at the seams: an unstated assumption, a non-functional requirement nobody owned, a "we'll figure it out" that became "we never figured it out." A separate Clarification stage catches these systematically.

It is also the stage where AI agents add the most measurable value: agents are excellent at probing for gaps, contradictions, and missing edge cases.

## Inputs and outputs

| | |
|---|---|
| **Inputs** | `spec.md` with `MUST-CLARIFY` and `SHOULD-CLARIFY` markers; constitution; related specs |
| **Outputs** | Updated `spec.md` with all `MUST-CLARIFY` resolved; `clarifications.md` log of every Q/A and decision; flagged spec changes |
| **Owners** | Product Manager (drives), Tech Lead, Architect (for cross-cutting questions), CX/UX (for user-affecting questions) |
| **Cadence** | Time-boxed: clarification target is 1–3 working days |

## How it works

Three workstreams run in parallel:

1. **Human clarification.** PM convenes a short clarification session (or async thread) per cluster of related questions. Decisions and rationale go into the clarification log.
2. **Agent-led probing.** An AI agent reads the spec and the constitution, and produces a structured list of gaps, contradictions, and likely-missing edge cases. The agent's findings are reviewed and either folded into the spec, dispatched as new `MUST-CLARIFY` items, or explicitly dismissed with rationale.
3. **Cross-spec reconciliation.** The platform searches for related specs that touch the same surfaces and surfaces conflicts (e.g., two specs both proposing different rate-limit semantics on the same endpoint).

When clarification produces material changes — new FRs, removed scope, NFR changes — the spec is updated and the spec re-runs G1. Iteration is normal here.

## Clarification log

A simple, append-only log keyed to the spec ID. Lives at `specs/<feature-id>/clarifications.md`.

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

The log is immutable history. If a clarified decision is later reversed, that's a *new* clarification entry with a link to the original.

## AI agent role

The dedicated **clarification agent** is given:

- The spec (canonical text + structured form).
- The constitution.
- Related and adjacent specs.
- The schema registry / API catalog.
- Anonymized failure modes from prior incidents tagged to similar surfaces.

It produces structured output, scored by confidence, into three buckets:

- **Probable contradiction** — "FR-3 says reject after 24h but NFR-2 implies retention for analytics; reconcile."
- **Probable gap** — "Spec says nothing about authentication for the resume endpoint."
- **Probable edge case** — "What if the client sends chunks out of order?"

The agent is *not* allowed to silently amend the spec. It opens questions; humans decide.

## Quality gate G2

Automated and strict:

- Zero open `MUST-CLARIFY` markers in the spec.
- Every entry in the clarification log has a recorded decision and an owner.
- The spec, post-clarification, re-passes G1.

`SHOULD-CLARIFY` items may carry forward into Design but must be explicitly tracked.

## Best practices and anti-patterns

**Best practices.**

Time-box clarification aggressively. The right answer is usually one of: yes, no, or "we'll defer with these explicit assumptions." Long clarification cycles are usually a smell that the spec is too big — split it. Always involve CX/UX for any user-visible question; tech-only clarification frequently misses the user-impact of a "small" decision. Write decisions in the language of the spec ("AC-7 is added" rather than "we decided to handle out-of-order chunks") so traceability survives.

**Anti-patterns.**

Letting clarification become a multi-week consensus exercise — that's a planning failure, not a clarification need. Resolving clarifications verbally without recording them — they will resurface as bugs. Allowing the agent to edit the spec without human approval. "Clarifying" by agreeing to "discuss in design," which just defers the problem and contaminates the design stage.

---

[← Specification](specification.md) · [Next: Design →](design.md)
