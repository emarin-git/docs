# Best Practices, Policies & Anti-patterns

The cumulative ground-truth of how to run the SDD-First SDLC well — at any scale, in any topology. This file is the curriculum new squads are onboarded against and the reference for retros.

---

## Table of Contents

1. [Cross-cutting principles](#cross-cutting-principles)
2. [Per-stage best practices](#per-stage-best-practices)
3. [Anti-patterns catalog](#anti-patterns-catalog)
4. [Policies (recommended defaults)](#policies-recommended-defaults)
5. [Scaling guidance](#scaling-guidance)
6. [Multi-repo / microservices](#multi-repo--microservices)
7. [AI-assisted workflows](#ai-assisted-workflows)
8. [Adoption and rollout](#adoption-and-rollout)

---

## Cross-cutting principles

These hold at every stage:

- **The spec wins.** When code, tests, or behavior disagree with the spec, the platform raises a drift defect — and the resolution is either to update the spec or to fix the code, never to silently let them diverge.
- **Make policies executable.** A policy that isn't enforced is a vibe. If a rule matters, it runs in CI, the policy engine, or the deploy pipeline.
- **Small, frequent, reversible.** Small specs, small PRs, small rollouts. Reversibility is a property of the system, not a hope.
- **AI agents under boundary, not on a leash.** Agents work well inside explicit constraints; they fail badly under vague oversight. Define the constraints, automate the audit, then *trust* the bounded behavior.
- **Optimize for the inner loop.** Inner-loop time predicts adoption better than any policy. If the inner loop is slow, your TDD will collapse and your specs will rot.

## Per-stage best practices

### Constitution

**Practices.** Keep it short and enforceable. Every clause has a "how would I notice if this were violated" answer. Amendments are batched monthly with rationale and migration plans. The constitution is loaded as agent context — write it like the agent will read it (because it will).

**Policies.**

- All amendments require Architecture Council ratification, recorded in the version history.
- Machine-readable policies file must lint clean before constitution amendment merges.
- Quarterly review of waivers — patterns of waivers indicate the rule is wrong, not that the team is.

### Specification

**Practices.** Co-author with the AI draft agent; tighten with humans. Write the *out-of-scope* section first. Use stable IDs everywhere. Keep specs ≤ ~5 pages of authored content (excluding generated/linked); larger specs split into linked children.

**Policies.**

- G1 automated lints must pass for any spec to enter Clarification.
- Specs touching public contracts require Architect approval.
- Specs touching user-facing behavior require UX approval; specs handling sensitive data require Security approval.
- Specs without testable ACs fail G1.

### Clarification

**Practices.** Time-box to ≤3 working days. Run the clarify-agent for completeness; humans for judgment. Record decisions, not discussions. If a decision implies a spec change, edit the spec — don't leave it in the log alone.

**Policies.**

- Zero open `MUST-CLARIFY` markers required for G2.
- Clarification log is append-only and auditable.
- Agents may not amend specs autonomously.

### Design

**Practices.** Design contracts first; implementation later. Make tasks ≤1 day. Run the critique agent before Architect review. Always document alternatives considered. Plan migration and rollback before merging the design.

**Policies.**

- Every AC must be covered by ≥1 task.
- Every task must reference its spec.
- DDL changes require an explicit rollback plan.
- Cross-service designs require contract diff review.
- `agent_assignable: false` is required for security-critical, irreversible-migration, and crypto/auth tasks.

### Implementation

**Practices.** TDD by default. PRs ≤300 lines. Agent-authored work reviewed by a *different* human than the prompter. Pair humans with agents on novel work. Property-based tests for invariants in NFRs. Runbooks updated alongside code, not after.

**Policies.**

- Agents may not merge to main.
- Coverage delta on changed lines ≥80% (or constitution-set threshold).
- TDD-exempt tasks require recorded justification.
- Every PR cites task ID, spec ID, and constitutional clauses.
- Architectural lints, contract checks, and security scans block merge.

### Validation

**Practices.** Automate the report; never hand-write it. Treat drift as a defect with severity. Run NFR validation as soon as design produces enough surface to instrument. Make the validation report part of release notes.

**Policies.**

- Drift report status `clean` or explicit waiver required for G5.
- Every AC in the spec must have ≥1 passing test.
- Contract-breaking changes require a waiver merged before G5 closes.
- NFR misses fail G5 unless an exception is approved by the relevant role (SRE for performance, Security for security, etc.).

### Deployment

**Practices.** Progressive by default. Decouple deployment from release with flags. One-button rollback. Provision dashboards in code. Run rollback drills quarterly per service.

**Policies.**

- All releases use progressive delivery unless an exception is documented.
- Pinned versions in deploy specs; no `:latest`.
- SBOMs and image signatures are mandatory.
- Each release has a runbook with explicit revert criteria.
- Friday after-noon production releases require Director-level approval (cultural policy; adapt for your timezone/topology).

### Operations

**Practices.** Treat the spec as living documentation. Postmortem actions classified into spec/constitution/runbook/platform — never "be more careful." User signal flows into the spec backlog continuously. Spec health is reviewed quarterly per service.

**Policies.**

- Postmortem actions older than 60 days are themselves an incident.
- Each service has a documented owner; orphan services are blocked at deploy.
- SLO breaches that exceed error budget pause non-essential changes to the affected service until burn rate normalizes.

## Anti-patterns catalog

A non-exhaustive list of the most common ways this framework fails when applied carelessly.

### Spec anti-patterns

The "implementation in disguise" spec — prescribes class names and SQL; no longer a contract, just code in prose.
The "wishlist" spec — multiple barely-related desires; impossible to validate.
The "frozen" spec — never amended after approval; drift accumulates silently.
The "retro-spec" — written after code lands; describes what was built, not what was wanted.
The "shadow constitution" — spec contradicts constitution but slips through reviewer fatigue.

### Process anti-patterns

**Gate inflation.** Every problem becomes a new gate. Eventually nobody can ship; teams route around the gates.
**Gate theatre.** Gates exist but are routinely bypassed; the bypass becomes the norm.
**Lightweight-path abuse.** Behavior changes shipped as patches because Standard path is too slow — the symptom is a too-slow Standard path, not lazy engineers.
**Clarification by deferral.** "We'll figure it out in design" — the same problem now contaminates a more expensive stage.

### Agent anti-patterns

**Agents merging their own work.** Eliminates the human-judgment gate entirely.
**Agents writing both tests and the code that satisfies them in one session.** Collusion risk; tests pass while the spec is unmet.
**Hidden agent edits.** Agent-authored changes presented as human PRs without provenance — destroys auditability.
**Generic agent output.** Tracking effort by lines of code generated instead of specs satisfied — pure cargo cult.

### Org anti-patterns

**Component teams.** Frontend and backend halves of a feature in different teams. Specs become hand-off bait.
**Shared services with no owner.** Ambiguous ownership = stale specs, ambiguous on-call, slow incident response.
**Architects who don't build.** Drift between principle and practice; constitution becomes irrelevant.
**Permanently-temporary embeds.** Security/UX engineers permanently embedded in one squad become single points of failure.

### Platform anti-patterns

**Building tools nobody uses.** Platform measured by ship count rather than adoption.
**Opaque agent runners.** No audit, no traceability — destroys the value proposition of SDD.
**One-shot CI.** Pipelines that run end-to-end on every PR but cache nothing — inner-loop time explodes.
**Drift "warnings" with no consequence.** If drift never blocks, it never gets fixed.

## Policies (recommended defaults)

A starter set you can adopt and amend. Codify them in the constitution and back them with policy-engine rules.

| Policy | Default | Notes |
|---|---|---|
| Spec quality lint | Required for G1 | All FR/NFR/AC must have stable IDs |
| Coverage delta | ≥ 80% on changed lines | Per repo; raise for libraries, lower for prototypes |
| PR size | ≤ 300 lines diff | Soft guideline, hard cap at 800 with justification |
| Reviewer required | ≥1 human, not the prompter | Separation of duties for agent-authored work |
| Contract compatibility | Backward-compatible by default | Breaking changes require waiver |
| Release strategy | Progressive (canary → ramp) | Exceptions documented |
| Image policy | Pinned, signed, SBOM | Enforced at deploy |
| Postmortem follow-up | Closed within 60 days | Tracked as platform metric |
| Spec health review | Quarterly per service | Tracked in service catalog |
| Constitution amendment cadence | Monthly batch | Emergency path for incidents |
| Agent permissions | Default-deny | Constitution explicitly enumerates allowed tools per role |

## Scaling guidance

**Small (≤30 engineers).** Adopt 80% of the framework. Lightweight constitution; one shared platform engineer; AI agents as a shared fleet; manual RACI; minimum viable drift detector. Skip tribes; everyone is one squad-of-squads.

**Medium (30–150 engineers).** Full framework. Dedicated Platform team. Tribes with shared on-call rotations and partial constitutions. Multi-tenant agent runner with per-tribe cost budgets. Architecture Council formalized; meets bi-weekly.

**Large (150+ engineers).** Federate. Org-level constitution stays small and absolute (security, compliance, contracts). Tribe-level constitutions extend it. Multiple platform teams (org platform + tribe platforms). Spec management explicitly cross-repo with global traceability graph. AI agent fleets per tribe sharing tooling and audit infrastructure. Quarterly boundary audits become non-negotiable; misaligned squads get re-chartered.

A common scaling failure: copying a successful squad's habits and trying to enforce them centrally. What worked is usually the *autonomy*, not the specific habits.

## Multi-repo / microservices

The framework treats specs and code as **co-located**: the spec for a feature lives next to the code that implements it. For multi-repo / microservice estates:

- **Cross-repo specs** are first-class. The spec lives in the *driving* repo (usually the user-facing one), and links into the schemas, registries, and contributing services it touches.
- **Schema registry is global.** Per-repo schemas don't scale; you'll get drift between consumers and producers within weeks.
- **Traceability graph is global.** Specs, designs, tasks, PRs, and deploys must be queryable across repos — that is the whole point.
- **Ownership.yaml is global.** Every service, every public schema, every contract has exactly one owning squad. Audit quarterly.
- **Contract tests run cross-repo.** Consumer-driven contract testing (e.g., Pact) is a baseline, not optional, in microservice estates.

A common failure mode in microservices is treating the spec stage as a per-repo concern. Specs that touch multiple services need a single owning author and explicit consulted-squad approvals at G3. The platform must make this a default, not paperwork.

## AI-assisted workflows

Agents amplify whatever process you have. The key practices:

**Treat agents like new hires with super-powers and no judgment.** Boundaries explicit, work reviewed, no merge authority, audit trail mandatory.

**Use multiple, separate agents for adversarial roles.** Don't have the same agent draft a design *and* critique it. Different prompts, different model providers when feasible.

**Let agents do drudge work.** Scaffolds, schema migrations, telemetry plumbing, refactors with strong test coverage, mechanical translations. They are excellent here.

**Don't let agents own novel cross-cutting work.** Performance work without a profiler in the loop, security-critical paths, novel architectural choices — humans drive, agents assist.

**Cite everything.** Every agent-authored artifact references its source spec ID and constitutional clauses. This is non-negotiable; it's what makes the audit trail work.

**Measure what matters.** Track specs satisfied per cycle, drift defects per release, agent-authored PRs reverted, agent-authored security findings caught in review. Don't track lines of code generated.

**Continuously evaluate agent fitness.** When new model versions ship, run them on a benchmark of recent tasks before promoting them in the runner. Quality regressions on production workflows are real and costly.

## Adoption and rollout

Adopting this framework in an existing org is itself a project. Recommended phasing:

**Phase 0 — Constitution (week 1–2).** Author a v0.1 constitution from your existing standards. Get Architecture and CTO signoff. Make it the loaded context for any AI tools already in use.

**Phase 1 — One squad, one feature (weeks 3–6).** Pick a willing squad and run a single feature through the full lifecycle. Build the minimum spec linter, drift checker, and agent integration to support it. Keep manual where automation would be slow to build.

**Phase 2 — Platform foundations (weeks 4–12, parallel).** Build the spec source-of-truth, the lint/drift detector, the agent runner, and the policy engine into a credible platform. Pair with the pilot squad.

**Phase 3 — Expand to a tribe (months 3–6).** Onboard 3–5 more squads. Refine the constitution based on what the pilot revealed. Migrate existing in-flight work into the framework only at natural boundaries (next major feature), never mid-feature.

**Phase 4 — Org-wide adoption (months 6–18).** Full RACI, gates enforced, full traceability graph. Run quarterly boundary audits. Track adoption with metrics: % of features through Standard path, drift defects per release, postmortem-to-amendment cycle time.

The most common failure is mandating the framework before the platform supports it — teams cargo-cult the process without the automation, and the policy work eats their capacity. Build the platform alongside, in lockstep.

---

[← Overview](../overview.md) · [Architecture ↑](../architecture/sdd-platform.md) · [Lifecycle ↑](../sdlc/lifecycle.md)
