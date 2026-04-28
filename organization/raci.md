# RACI Matrix

Per-stage Responsibility / Accountable / Consulted / Informed assignments. **R** = does the work, **A** = ultimately answerable (single person), **C** = consulted before, **I** = informed after.

This RACI is the source of truth. Squad-level adaptations are allowed but must be explicit in the squad's charter.

---

## Table of Contents

1. [Reading the matrix](#reading-the-matrix)
2. [Stage-by-stage RACI](#stage-by-stage-raci)
3. [Cross-cutting decisions](#cross-cutting-decisions)
4. [Escalation paths](#escalation-paths)

---

## Reading the matrix

- **R** must do the work.
- **A** is the single name on the line — there is exactly one A per row.
- **C** must be consulted *before* a decision is finalized.
- **I** must be informed *after* the decision is made.
- An empty cell means: not in the loop by default for this row.

Roles are abbreviated: PM (Product Manager), TL (Tech Lead), Eng (Engineer), QA, Arch (Architect), Plat (Platform/DevOps + SRE), UX (UX Designer), Sec (Security), CX (CX Research).

## Stage-by-stage RACI

### Stage 0 — Constitution

| Activity | PM | TL | Eng | QA | Arch | Plat | UX | Sec | CX |
|---|---|---|---|---|---|---|---|---|---|
| Author / amend constitution | C | C | I | C | **R / A** | C | I | C | I |
| Author machine-readable policies | I | C | I | I | C | **R / A** | I | C | |
| Ratify amendment | I | C | I | I | **R / A** | C | I | C | |
| Enforce in CI / deploy | | I | I | I | C | **R / A** | | C | |

### Stage 1 — Specification

| Activity | PM | TL | Eng | QA | Arch | Plat | UX | Sec | CX |
|---|---|---|---|---|---|---|---|---|---|
| Author spec | **R / A** | R | C | C | C | | R (user-facing) | C (sensitive) | C |
| Define functional requirements | **R / A** | C | C | I | | | C | | C |
| Define non-functional requirements | C | **R / A** | C | C | C | C | C | C | |
| Define acceptance criteria | R | C | I | **R / A** | | | C | | C |
| Spec quality lint (G1 automation) | I | I | I | I | | **R / A** | | | |
| Spec approval (G1 human) | **R / A** | R | | | C | | C | C | I |

### Stage 2 — Clarification

| Activity | PM | TL | Eng | QA | Arch | Plat | UX | Sec | CX |
|---|---|---|---|---|---|---|---|---|---|
| Run clarification session | **R / A** | R | C | C | C | | C | C | C |
| Resolve `MUST-CLARIFY` items | **R / A** | R | I | I | C | | C | C | C |
| Maintain clarification log | **R / A** | I | I | I | | I | I | I | I |
| Close G2 | **R / A** | C | | I | | I | | | |

### Stage 3 — Design

| Activity | PM | TL | Eng | QA | Arch | Plat | UX | Sec | CX |
|---|---|---|---|---|---|---|---|---|---|
| Author design document | I | **R / A** | R | C | C | C | C (UI) | C (sensitive) | I |
| Author task breakdown | I | **R / A** | R | C | | | I | | |
| Define test strategy | I | R | R | **R / A** | | | C | C | |
| Contract changes | I | R | C | I | **R / A** (cross-service) | C | | C | |
| Design review (G3) | C | R | C | C | **R / A** (cross-service) | C | C (UI) | C (sensitive) | |

### Stage 4 — Implementation

| Activity | PM | TL | Eng | QA | Arch | Plat | UX | Sec | CX |
|---|---|---|---|---|---|---|---|---|---|
| Implement task | I | C | **R** | I | | | | | |
| TDD discipline | I | **A** | R | C | | | | | |
| Author/run agents | I | C | **R** | | | A | | | |
| Code review | I | R | R | C | | | C (UI) | C (sensitive) | |
| Approve merge | I | **R / A** | R | | | | | C (sensitive) | |
| Build complete (G4) | I | A | R | I | | R | | | |

### Stage 5 — Validation

| Activity | PM | TL | Eng | QA | Arch | Plat | UX | Sec | CX |
|---|---|---|---|---|---|---|---|---|---|
| Run validation pipeline | I | C | C | **R / A** | | R | | C | |
| Drift report review | C | R | C | **R / A** | C (cross-service) | C | | C | |
| NFR validation | C | R | C | **R / A** | | R | | | |
| Acceptance signoff (G5) | **R / A** | R | | R | C (cross-service) | R (operability) | R (user-facing) | R (sensitive) | C |

### Stage 6 — Deployment

| Activity | PM | TL | Eng | QA | Arch | Plat | UX | Sec | CX |
|---|---|---|---|---|---|---|---|---|---|
| Define rollout plan | C | R | C | C | C (cross-service) | **R / A** | C (user-facing) | C | I |
| Run progressive rollout | I | R | R | I | | **R / A** | | | I |
| Approve gate G6 | I | C | | | | **R / A** | | C | |
| Rollback / kill-switch decisions | C | R | R | I | | **R / A** | | | C |

### Stage 7 — Operations

| Activity | PM | TL | Eng | QA | Arch | Plat | UX | Sec | CX |
|---|---|---|---|---|---|---|---|---|---|
| Run service / on-call | I | R | **R / A** | | | C | | | |
| SLO management | C | R | C | | C | **R / A** | | | |
| Incident response | C | R | R | | | **R / A** | | C | |
| Postmortem | C | **R / A** | R | C | C (cross-service) | R | | C | C |
| Spec amendments from incidents | **R / A** | R | C | C | C (cross-cutting) | | C | C | C |
| User signal triage | **R / A** | C | I | | | | C | | R |

## Cross-cutting decisions

Some decisions don't fit a single stage. Defaults:

| Decision | A | R | C | I |
|---|---|---|---|---|
| Adding a new product domain / squad | VP Eng + VP Product | Squad EM | Architecture Council, Platform | All |
| New public API surface | Architect | Squad TL | Security, Platform | PM, Squad |
| Constitutional waiver | Architect | Requesting TL | Security if relevant | All Squads (transparency) |
| Patch-path declaration on a PR | Author | Author | Reviewer | Squad TL |
| Release rollback in incident | Incident Commander | On-call SRE | Squad TL, PM | All |
| AI agent permission change | Architect + Platform Lead | Platform | Security | All |

## Escalation paths

Conflicts are resolved at the lowest level that has authority:

1. **Within a squad** — Tech Lead + PM + UX co-decide. EM mediates if needed.
2. **Across squads on a contract** — primary squad's TL + secondary squad's TL. Architect mediates.
3. **Cross-cutting standards or constitution conflict** — Architecture Council.
4. **Strategic / cross-functional** — VP Eng + VP Product + (VP Design when relevant).
5. **Security override** — Security Engineer can block a release; only CTO can override.

The framework explicitly avoids "everything goes to the CTO" patterns. If escalation cadence rises, examine the boundary, not the people.

---

[← Roles](roles.md) · [Squads ↑](squads.md)
