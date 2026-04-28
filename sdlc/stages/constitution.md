# Stage 0 — Constitution

The constitution is a **versioned, repository-resident document** that defines the principles, policies, and constraints every spec, design, and AI agent must respect. It is not a wiki page. It is checked into the repo, peer-reviewed, and loaded as persistent context for AI agents at the start of every session.

---

## Table of Contents

1. [Purpose](#purpose)
2. [Inputs and outputs](#inputs-and-outputs)
3. [What goes in the constitution](#what-goes-in-the-constitution)
4. [Authoring template](#authoring-template)
5. [Governance and amendments](#governance-and-amendments)
6. [AI agent rules](#ai-agent-rules)
7. [Quality gate G0](#quality-gate-g0)
8. [Best practices and anti-patterns](#best-practices-and-anti-patterns)

---

## Purpose

The constitution exists to:

- Give AI agents and humans a shared, durable context for *how this product is built*.
- Prevent every spec from re-litigating the same architectural decisions.
- Make principles auditable and enforceable in CI, not just aspirational.
- Decay-proof institutional knowledge (it lives next to the code, not in someone's head).

If a principle isn't enforceable or referenceable, it doesn't belong in the constitution.

## Inputs and outputs

| | |
|---|---|
| **Inputs** | Org engineering standards, security/compliance requirements, product strategy, prior postmortems, regulatory constraints |
| **Outputs** | `constitution.md` (versioned), `policies/*.yaml` (machine-readable rules), AI agent system prompt fragments |
| **Owners** | Architecture Council (ratification); CTO/VP Eng (final approval); Platform team (enforcement) |
| **Cadence** | Standing document; ratified amendments batched monthly |

## What goes in the constitution

Seven sections, each mandatory:

1. **Product principles** — non-negotiable product values (e.g., "we never silently lose user data").
2. **Architectural principles** — boundaries, layering rules, allowed dependencies (e.g., "domain layer has no I/O").
3. **Technology baseline** — language versions, framework choices, default datastore, the standard message bus.
4. **Security & compliance** — data classification, secrets handling, regulated-data rules.
5. **Quality bar** — coverage thresholds, latency/availability SLOs, accessibility minimums.
6. **AI agent operating rules** — what agents may do unsupervised, what requires human approval (see below).
7. **Process rules** — review thresholds, gate definitions, lightweight-path criteria.

Each section ends with **enforcement**: how the rule is checked (CI lint, policy engine, human review) and **exceptions**: how to request a waiver.

## Authoring template

```markdown
# Constitution — <product / repo name>
Version: 2026.04.01  •  Last ratified: 2026-04-15  •  Owner: Architecture Council

## 1. Product principles
- P1. We never silently lose user data.
  - Enforcement: any write path without durable acknowledgement requires CR-DATA waiver.
- P2. ...

## 2. Architectural principles
- A1. Domain layer has no I/O. Side effects live in adapters.
  - Enforcement: `arch-lint` rule `no-io-in-domain`. CI fails on violation.
- A2. Services communicate via versioned, schema-registered events.
  - Enforcement: schema registry CI check.

## 3. Technology baseline
- TypeScript 5.x, Node 22 LTS, Postgres 16, Kafka 3.x, OpenTelemetry.
- Default web framework: Hono. Default ORM: Drizzle. Default test runner: Vitest.

## 4. Security & compliance
- S1. PII is tagged at schema level; queries against PII tables require `pii-reviewer` approval.
- S2. Secrets only via the secret broker; no plaintext in env files in repos.

## 5. Quality bar
- Q1. Unit coverage ≥ 80% on changed lines (delta coverage).
- Q2. p99 API latency ≤ 300ms unless waived in spec NFRs.
- Q3. WCAG 2.2 AA for all user-facing surfaces.

## 6. AI agent operating rules
- AI1. Agents may write code, tests, and specs. Agents may not merge to main.
- AI2. Agents may run read-only tools without approval. Write tools require human-in-the-loop.
- AI3. Agents must cite the spec section ID for every code change.

## 7. Process rules
- R1. Standard path is default. Patch path requires explicit declaration.
- R2. Spec changes affecting public contracts require Architect approval.
```

The constitution is human-readable Markdown plus a machine-readable companion (`policies/*.yaml`) so CI and the spec platform can consume it directly.

## Governance and amendments

Amendments follow the same path as a spec: PR, review, ratification by Architecture Council, version bump. Amendments must include:

- Rationale and the problem the current rule fails to handle.
- Migration plan for existing specs and code (or an explicit grandfather clause).
- Effective date.

Emergency amendments (e.g., during a security incident) may be ratified asynchronously by any two Architecture Council members and confirmed at the next scheduled meeting.

## AI agent rules

This section deserves its own emphasis because AI agents will read it before every session.

The constitution explicitly defines:

- **Permissions** — which tools agents may invoke, and which require human approval.
- **Boundaries** — paths, repos, or environments off-limits (e.g., production secrets, customer data).
- **Mandatory citations** — every agent-produced artifact must reference its source spec ID and constitutional clause.
- **Refusal cases** — when an agent must stop and ask (ambiguous spec, missing context, conflict with constitution).
- **Audit hooks** — every agent action is logged with prompt, tools called, files touched, and human approver if any.

These rules ship as a system-prompt fragment loaded by the agent runner. They are not advisory — the runner enforces them.

## Quality gate G0

The constitution must pass G0 before the repo is unblocked for spec work:

- All seven sections present and non-empty.
- Each principle has explicit enforcement.
- Machine-readable policies file lints clean.
- Two Architecture Council members have signed off.

Constitutional drift (the Markdown says one thing, the YAML says another, CI enforces a third) is itself an incident.

## Best practices and anti-patterns

**Best practices.**

The constitution is short. Aim for a document a new senior engineer can read on day one. If yours is over ~30 pages, you have policy bloat — split rarely-changing standards into linked sub-docs. Every principle should answer "how would I notice if this were violated?" If you can't answer, it's a value statement, not a principle, and it doesn't belong here. Keep examples and rationale inline with each principle so the agent loading this file as context sees the *why* alongside the *what*.

**Anti-patterns.**

A constitution that is just a list of preferences ("we like clean code"). A constitution that contradicts itself across sections. A constitution that's never amended (suggests it isn't being read). A constitution that's amended weekly (suggests it's a backlog dumping ground). Stuffing tactical decisions ("use lodash 4.x") in the constitution instead of in dependency manifests.

---

[← Lifecycle](../lifecycle.md) · [Next: Specification →](specification.md)
