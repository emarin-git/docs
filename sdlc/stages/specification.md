# Stage 1 — Specification

The spec is **the contract**. Everything downstream — design, tasks, code, tests, acceptance — derives from and references the spec.

---

## Table of Contents

1. [Purpose](#purpose)
2. [Inputs and outputs](#inputs-and-outputs)
3. [What a spec must contain](#what-a-spec-must-contain)
4. [Spec template](#spec-template)
5. [Spec quality standards](#spec-quality-standards)
6. [Quality gate G1](#quality-gate-g1)
7. [Best practices and anti-patterns](#best-practices-and-anti-patterns)

---

## Purpose

Capture *what* is being built and *why*, in enough detail that:

- A reviewer can identify gaps before any code is written.
- An AI agent can generate an initial design and task breakdown.
- Acceptance tests can be derived directly from acceptance criteria.
- A future engineer can reconstruct intent from the spec alone.

A spec describes **behavior, not implementation**. Implementation is the next stage's problem.

## Inputs and outputs

| | |
|---|---|
| **Inputs** | Customer signal (CX), product strategy, constitution, NFR baselines, prior related specs |
| **Outputs** | `specs/<feature-id>/spec.md`, structured front-matter, linked acceptance criteria |
| **Owners** | Product Manager (drives), Tech Lead (co-author), CX/UX (co-author for user-facing work) |
| **Cadence** | Per feature; expect 0.5–3 days of authoring + review |

## What a spec must contain

Every spec has nine required sections. Missing any of these fails G1.

1. **Front-matter** — feature ID, status, owners, links to related specs and constitution clauses.
2. **Problem & motivation** — the user/business problem in plain language. Cite evidence.
3. **Personas & scenarios** — who is affected, what they're trying to do.
4. **Functional requirements** — behaviors, in numbered, testable form.
5. **Non-functional requirements (NFRs)** — performance, availability, security, accessibility, cost.
6. **Acceptance criteria** — Given/When/Then statements that map to automatable tests.
7. **Out of scope** — explicit non-goals. Often the most useful section.
8. **Open questions / clarifications needed** — `MUST-CLARIFY` markers consumed by Stage 2.
9. **Risks & assumptions** — what could go wrong, what we're betting on.

Two optional but recommended sections: **Telemetry plan** (what we'll measure) and **Rollout plan sketch** (canary, flags, kill switch).

## Spec template

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

The structured front-matter is critical: the spec platform indexes it for traceability and the AI agents key off it.

## Spec quality standards

A spec must satisfy these properties; the spec platform lints for them automatically.

- **Atomic and addressable.** Every requirement and acceptance criterion has a stable ID (`FR-1`, `AC-3`). Stable IDs survive renumbering — never reuse IDs.
- **Testable.** Every AC is derivable to an automated test. "Should be fast" is not testable; "p99 ≤ 300ms at 200 RPS" is.
- **Bounded.** A spec covers one feature. If it spans multiple deployable units, split it (and link the children).
- **Self-contained on intent.** A reader unfamiliar with the system should understand *why*, even if some *how* requires reading linked context.
- **Consistent with the constitution.** No requirement may contradict a constitutional principle without an explicit, approved waiver in the front-matter.
- **Reviewed by all three roles.** Product (intent), Engineering (feasibility), CX/UX (usability) for any user-visible feature.

## Quality gate G1

G1 is hybrid (automation + human):

**Automated checks:**

- Front-matter schema valid.
- All required sections present.
- Every FR/NFR/AC has a stable ID.
- ACs lint as Given/When/Then.
- Linked constitution clauses exist.
- No orphan `MUST-CLARIFY` left untriaged for >5 business days.

**Human checks:**

- PM signoff that the problem is real and worth solving.
- Tech Lead signoff that scope is feasible in the proposed cadence.
- UX/CX signoff for user-facing changes.
- Architect signoff if public contracts or cross-service work is affected.

## Best practices and anti-patterns

**Best practices.**

Co-author with the AI agent. Have the agent produce a first-draft spec from a problem statement, then have humans tighten the FRs and ACs. The agent is excellent at structure and completeness checks; humans are essential for product judgment. Write the **out-of-scope** section first — it's where misunderstandings hide. For any cross-cutting concern, link an existing canonical spec rather than re-specifying. Keep the spec living: amend it during implementation when reality contradicts assumptions, rather than letting the code drift away.

**Anti-patterns.**

The "implementation in disguise" spec that prescribes specific class names and SQL — that belongs in design. The "wishlist" spec that's a backlog of vague desires; specs are atomic. The "frozen" spec that nobody updates after it's approved, leading to drift. Specs written *after* the code lands ("retro-specs") — they describe what was built, not what was wanted, and are useless as contracts. Specs that quietly contradict the constitution and rely on reviewer fatigue to slip through.

---

[← Constitution](constitution.md) · [Next: Clarification →](clarification.md)
