# Roles & Responsibilities

How the SDD-First SDLC maps onto roles. The framework assumes a modern product organization with cross-functional squads supported by horizontal platforms.

---

## Table of Contents

1. [Role catalog](#role-catalog)
2. [Product](#product)
3. [Engineering](#engineering)
4. [QA](#qa)
5. [Architecture](#architecture)
6. [Platform / DevOps](#platform--devops)
7. [CX / UX](#cx--ux)
8. [AI agents as a "role"](#ai-agents-as-a-role)
9. [Stage-by-stage role contributions](#stage-by-stage-role-contributions)

---

## Role catalog

| Role | Stage of greatest leverage | Single-line summary |
|---|---|---|
| Product Manager | Spec, Clarification, Operations | Owns the *why* and the *what*; accepts shipped work against the spec. |
| Tech Lead | Design, Implementation | Owns technical correctness for a squad's deliverables. |
| Engineer | Implementation, Validation | Authors code and tests; partners with agents. |
| QA Engineer | Validation | Owns acceptance pipelines, drift checks, and quality bar. |
| Architect | Constitution, Design | Stewards cross-cutting concerns and the constitution. |
| Platform / DevOps | Constitution, Deployment, Operations | Builds and runs the platform that makes the SDLC possible. |
| SRE | Deployment, Operations | Owns reliability; partners with squads on SLOs. |
| CX/UX Designer | Spec, Clarification, Validation | Owns user experience and behavior; co-author of user-facing specs. |
| CX/UX Researcher | Spec, Operations | Brings user signal in; closes the feedback loop. |
| Security Engineer | Constitution, Design, Validation | Owns security posture; gates contracts that touch sensitive data. |
| Engineering Manager | All | Owns flow, capacity, and squad health. |

## Product

Product Managers are accountable for the spec, end-to-end. They drive Specification and Clarification, partner with Engineering on feasibility in Design, and accept shipped work in Validation against the same spec they wrote.

In an SDD-First org, PMs write *better* specs than they did under canonical Agile because the spec is structured, lintable, and re-used. They spend less time on tickets and more time on problem framing.

PM-specific responsibilities:

- Authoring the spec; co-authoring with the AI draft agent and tightening with Tech Lead and CX/UX.
- Resolving `MUST-CLARIFY` items with named decisions and impact recorded.
- Defining business KPIs in the telemetry plan.
- Final accept/reject in Validation.
- Surfacing user signal (in partnership with CX) into Operations.

## Engineering

Engineers (and Tech Leads) own the technical lifecycle from Design through Operations.

Tech Leads specifically:

- Co-author the design document with the squad.
- Own G3 (design review) for the squad's specs.
- Allocate tasks across humans and AI agents, including marking `agent_assignable`.
- Maintain technical health — refactor budgets, dependency hygiene, observability bar.
- Mentor; ensure TDD discipline holds.

Engineers:

- Implement tasks under TDD.
- Author and review PRs (including agent-authored PRs — separation of duties applies).
- Own runbooks for code they author.
- Participate in on-call for their squad's services.

## QA

QA's role shifts in an SDD-First org. QA is **not** primarily a manual-test function; it is a **quality engineering** function that:

- Owns the spec-to-test mapping discipline (every AC has a test, every test has a spec).
- Builds and maintains the acceptance test framework (the `spec()` helper, the validation report, the drift check).
- Runs and curates NFR validation suites (load, chaos, accessibility).
- Operates G5 — the validation gate.
- Performs targeted exploratory testing on new surfaces — high-leverage, hypothesis-driven, not regression chasing.

QA pairs with engineers, not gates them. The platform owns the regression suite; QA owns the strategy.

## Architecture

Architects are stewards of cross-cutting concerns. They are *not* an ivory tower — they spend most of their time in design reviews, on critique, and amending the constitution as reality teaches.

Architects:

- Maintain the constitution. Ratify amendments. Author cross-cutting principles.
- Approve high-impact specs and any cross-service or contract-breaking design.
- Run the Architecture Council (a standing forum, not a gating committee).
- Maintain reference architectures and the schema/event registry.
- Mentor Tech Leads. The goal is fewer Architect approvals over time, not more.

Architects must build, even at small dosage, to stay calibrated.

## Platform / DevOps

The Platform team builds and runs the **SDD platform** — the spec management system, the AI agent layer, the validation pipeline, the deploy pipeline, the policy engine. See [SDD Platform Architecture](../architecture/sdd-platform.md) for the system view; this section covers the *role* expectations.

Platform/DevOps:

- Owns developer experience. Inner-loop time, CI flake rate, deploy lead time are *their* metrics.
- Implements policy enforcement for the constitution.
- Operates the AI agent runner, including audit trails.
- Provides paved-road templates: service scaffolds, dashboard templates, runbook templates.
- Pairs with squads to onboard them onto the framework.

A common failure mode is Platform building tools nobody uses. Platform's success is measured by squad adoption and squad-reported pain reduction, not feature ship count.

## CX / UX

CX/UX is a **first-class participant** in the SDLC, not a hand-off step.

UX Designers:

- Co-author user-facing specs from the start.
- Resolve usability-relevant `MUST-CLARIFY` items.
- Produce design artifacts (flows, prototypes) referenced from the spec.
- Sign off in Validation for usability/accessibility.
- Participate in the Operations loop on user-facing features.

CX Researchers:

- Surface user signal into the Specification stage (problems, evidence, pain points).
- Run usability research at design-prototype stage on high-impact specs.
- Close the loop in Operations: did the shipped feature actually solve the problem?

User-facing specs that ship without CX/UX involvement are flagged by the platform; this is policy, not advice.

## AI agents as a "role"

Agents are not employees, but they are **accountable participants** in the SDLC with explicit boundaries.

Agents are assigned tasks the same way humans are. Each agent role has a system prompt fragment and a set of allowed tools:

- **Spec-draft agent** — drafts specs from problem statements; cannot merge.
- **Clarification agent** — surfaces gaps and contradictions; cannot decide.
- **Design-draft agent** — drafts technical designs and task graphs.
- **Critique agent** — independently reviews specs and designs for gaps.
- **Implementation agent** — writes code and tests under TDD; opens PRs.
- **Reviewer agent** — assists human reviewers with specific checks (security, accessibility, drift); does not approve.

Each agent role is documented like a job description, with allowed tools, prohibited tools, escalation triggers, and audit expectations.

## Stage-by-stage role contributions

A summary; the [RACI matrix](raci.md) is the formal version.

| Stage | Drives | Co-authors | Reviews / Approves | Informed |
|---|---|---|---|---|
| Constitution | Architect | Tech Leads, Platform | Architecture Council, CTO | All |
| Specification | PM | Tech Lead, UX | Architect (when applicable), Security (when applicable) | Squad, CX |
| Clarification | PM | Tech Lead, UX, Architect | All co-authors | Squad |
| Design | Tech Lead | Engineers, UX (for UI) | Architect, Platform/SRE, Security | PM |
| Implementation | Engineers | AI agents | Tech Lead, peer engineers | PM, QA |
| Validation | QA | Squad | PM, UX, SRE, Architect (if cross-service) | Platform |
| Deployment | Platform/SRE | Squad | Squad on-call | PM, CX |
| Operations | SRE + Squad | All | — (continuous) | All |

---

[← Overview](../overview.md) · [Squads →](squads.md) · [RACI →](raci.md)
