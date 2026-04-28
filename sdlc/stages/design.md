# Stage 3 — Design

Design takes a clarified spec and produces a technical plan: architecture, contracts, data model changes, sequencing, and a concrete task breakdown ready for implementation.

---

## Table of Contents

1. [Purpose](#purpose)
2. [Inputs and outputs](#inputs-and-outputs)
3. [Design document structure](#design-document-structure)
4. [Task breakdown](#task-breakdown)
5. [AI agent role](#ai-agent-role)
6. [Quality gate G3](#quality-gate-g3)
7. [Best practices and anti-patterns](#best-practices-and-anti-patterns)

---

## Purpose

Bridge the gap from *what* (spec) to *how* (code) without slipping into implementation. The design must be concrete enough that implementation tasks can be scheduled, parallelized, and assigned to humans or AI agents, but abstract enough to survive minor refactors.

Crucially, this stage produces the **task graph** — the unit of work that the implementation stage consumes.

## Inputs and outputs

| | |
|---|---|
| **Inputs** | Approved `spec.md`, clarification log, constitution, system architecture context, schema registry |
| **Outputs** | `specs/<feature-id>/design.md`, contract diffs (OpenAPI / proto / SQL DDL), `tasks.md` task graph, test strategy |
| **Owners** | Tech Lead (drives), Architect (reviews for cross-cutting concerns), Squad engineers (co-author and consume) |
| **Cadence** | Per feature; 1–5 days depending on complexity |

## Design document structure

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

The design references the spec by stable IDs (FR-1, AC-3) — it never restates them.

## Task breakdown

A flat-or-shallow list of tasks, each ≤1 day of effort, each linkable, each with explicit dependencies. The platform stores this as a graph.

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

Each task has front-matter the agent and CI consume:

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

Tasks marked `agent_assignable: false` (e.g., security-sensitive paths, irreversible migrations) require human-only authorship.

## AI agent role

Two distinct agent loops at this stage:

1. **Design draft agent.** Given the spec + constitution + architecture context, drafts the design document and an initial task graph. Always reviewed by the Tech Lead — it is a starting point, not final.
2. **Critique agent.** Independent agent run with an explicit "find what's wrong" prompt: missing failure modes, unhandled NFRs, contracts that conflict with neighbors, sequencing issues, gaps in test strategy. Its findings open issues against the draft.

Critique agents are most valuable when run *separately* from the drafting agent, ideally with different prompts/contexts, to avoid mode collapse.

## Quality gate G3

Hybrid:

- **Automated.** Contracts lint clean (OpenAPI/proto valid, semver classification correct), schema-registry compatibility check passes, DDL has a rollback, every task references the spec, every AC is covered by at least one task.
- **Human.** Architect approval if cross-service or new public contract; Tech Lead approval; Platform/SRE review if NFRs change SLO posture; Security review if data classification or auth changes.

If a design reveals that the spec is wrong, loop back to Specification. That is normal and not a failure mode.

## Best practices and anti-patterns

**Best practices.**

Design *the contract first*, then the implementation. Contracts are forever (or at least painful to change); internal code is cheap. Make the alternatives section real — it is one of the most valuable parts of the design for future maintainers and for review. Keep tasks small enough that the inner TDD loop fits inside one task. Run the critique agent before Architect review; let humans spend their time on judgment, not gap-finding. Treat sequencing seriously — a beautiful design with a bad merge order will block the squad.

**Anti-patterns.**

Designs that re-litigate the spec. Designs that hand-wave NFRs ("we'll make it fast"). Designs that are one giant 8-week task masquerading as a feature. Designs that omit migrations or rollback plans because "we'll handle that later." Designs auto-generated by an agent and merged without human critique — the failure mode is convergent generic patterns that fit nothing well.

---

[← Clarification](clarification.md) · [Next: Implementation →](implementation.md)
