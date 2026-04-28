# SDD-First SDLC Framework

A production-ready Software Development Life Cycle built around **Spec-Driven Development (SDD)** as the primary paradigm, with **Test-Driven Development (TDD)** embedded at the implementation layer and **AI-assisted workflows** as a first-class concern.

This documentation set is structured like a documentation repo. Treat it as the canonical source of truth for how engineering work flows through the organization.

---

## Table of Contents

1. [What is SDD-First?](#what-is-sdd-first)
2. [Why this framework exists](#why-this-framework-exists)
3. [Core principles](#core-principles)
4. [Where SDD fits (and where it doesn't)](#where-sdd-fits-and-where-it-doesnt)
5. [SDD vs TDD vs Agile vs Waterfall](#sdd-vs-tdd-vs-agile-vs-waterfall)
6. [Document map](#document-map)
7. [How to use this framework](#how-to-use-this-framework)

---

## What is SDD-First?

**Spec-Driven Development (SDD)** treats specifications as *executable, versioned, first-class artifacts* that drive plans, tasks, code, and tests. Instead of specs being write-once requirement documents that get archived, the spec is the source of truth that AI agents and humans both operate against. When the spec changes, downstream artifacts (plans, tasks, tests, code) regenerate or are flagged for reconciliation.

This framework integrates SDD with TDD. SDD answers *what* and *why*; TDD answers *how* at the code level. The spec describes the feature behavior; tests describe the unit/contract behavior. Both are required.

The framework is informed by current (2026) SDD tooling and practice — most notably GitHub Spec Kit (Specify → Plan → Tasks → Implement) and AWS Kiro (Requirements → Design → Tasks) — generalized into an end-to-end SDLC suitable for a real organization.

## Why this framework exists

Traditional Agile delivers iteratively but is weak on traceability and tends to produce specs as throwaway tickets. Waterfall produces heavy specs that rarely match the implementation. AI-assisted coding amplifies both failure modes: agents happily generate plausible code from vague prompts, producing fast output that nobody can verify.

SDD-First exists to close those gaps. Specs become the contract between humans, AI agents, tests, and operations. The SDLC, the org structure, and the platform are all designed to keep that contract honest.

## Core principles

The framework is built on seven non-negotiable principles, encoded in the [Constitution](sdlc/stages/constitution.md):

1. **Spec is the contract.** No production code is written without a merged spec.
2. **Specs are executable.** Specs generate plans, tasks, test stubs, and scaffolds.
3. **Tests are the floor, not the ceiling.** TDD at unit level; spec-derived acceptance tests at feature level.
4. **AI agents are accountable.** Agents operate inside the spec; human review is mandatory at named gates.
5. **Traceability is end-to-end.** Every line of code links to a task, every task to a spec, every spec to a constitutional principle.
6. **Drift is an incident.** Spec/code/test drift is detected automatically and treated as a defect.
7. **Specs scale down.** The full lifecycle is mandatory for features; lightweight paths exist for trivial work.

## Where SDD fits (and where it doesn't)

**SDD is most effective for:**

- Greenfield products and new features in existing products
- Cross-team or cross-service features where contracts matter
- Anything an AI agent will materially help build
- Compliance-sensitive domains (regulated industries, security-critical systems)
- Long-lived systems where institutional memory decays faster than the code

**SDD is overkill for:**

- One-line bug fixes and typo corrections
- Hotfixes under active incident
- Throwaway prototypes and spikes (use a "Spike" lightweight path)
- Pure refactors that preserve behavior (covered by tests, not specs)

The framework provides an explicit **lightweight path** for the second category — see [Lifecycle](sdlc/lifecycle.md#lightweight-paths).

## SDD vs TDD vs Agile vs Waterfall

| Dimension | Waterfall | Agile (canonical) | TDD | SDD-First (this framework) |
|---|---|---|---|---|
| Source of truth | Requirements doc | Backlog / tickets | Test suite | Versioned spec repo |
| Granularity | Project | Story | Function/class | Feature → Task → Test → Code |
| Change handling | Change request | Re-prioritize | Refactor | Re-spec → regenerate |
| AI agent fit | Poor | Mediocre | Good (codegen from tests) | Native |
| Traceability | Heavy upfront, decays | Weak | Strong at unit level | End-to-end |
| Best at | Fixed-scope projects | Discovery | Code correctness | Features in AI-augmented orgs |
| Worst at | Change | Long-horizon planning | Cross-cutting features | Trivial tasks |

SDD does not replace TDD. SDD wraps TDD: the spec produces acceptance tests *and* drives the unit-test cycle that TDD specializes in.

## Document map

```
/docs
  overview.md ............................... you are here
  /sdlc
    lifecycle.md ............................ end-to-end flow + lightweight paths
    /stages
      constitution.md ....................... principles, governance, AI agent rules
      specification.md ...................... what/why; functional + non-functional
      clarification.md ...................... resolving ambiguity before design
      design.md ............................. technical plan, architecture, contracts
      implementation.md ..................... TDD-driven build, agent workflows
      validation.md ......................... acceptance, quality gates, drift checks
      deployment.md ......................... release, progressive delivery
      operations.md ......................... observability, feedback, learning loop
  /organization
    roles.md ................................ Product, Eng, QA, Arch, Platform, CX/UX
    squads.md ............................... cross-functional team model
    raci.md ................................. responsibility matrix per stage
  /architecture
    sdd-platform.md ......................... reference platform architecture
  /best-practices
    policies.md ............................. per-stage practices, anti-patterns, scale guidance
```

## How to use this framework

- **New to SDD?** Read this overview, then [Lifecycle](sdlc/lifecycle.md), then the [Constitution](sdlc/stages/constitution.md).
- **Adopting in an existing org?** Start with [Roles](organization/roles.md) and [Squads](organization/squads.md) to find your minimum-viable rollout.
- **Building the platform?** Go straight to [SDD Platform Architecture](architecture/sdd-platform.md).
- **Writing or reviewing specs?** Read [Specification](sdlc/stages/specification.md) and [Clarification](sdlc/stages/clarification.md), then [Policies](best-practices/policies.md).

---

[Next: SDLC Lifecycle →](sdlc/lifecycle.md)
