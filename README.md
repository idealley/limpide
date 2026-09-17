# Limpide

> **Understanding is what survives explaining it.**

A tutoring platform for understanding that students can explain, apply, and revisit independently. Limpide combines guided discovery, useful explanation, and practice, with a patient memory of the foundations each learner is still building.

## Status

Early design. No code yet. This repository currently holds the design documents that will guide implementation. The current direction is to validate a focused tutoring offer and buyer demand first; the network is a future business goal ([ADR-0021](docs/ADR/0021-validate-tutoring-first.md)). A placement session plus daily mathematics practice is a [proposed bootstrap pilot](docs/BOOTSTRAP-PILOT.md).

## What this is

Limpide is a plugin on the AYA host, running on Flue as the execution engine (ADR-0019). It is a tutor that asks the student to explain what they think they know, surfaces gaps through Socratic probing, and makes the student feel the difference between thinking they understand and actually understanding. It honors urgent needs (the test on Friday) while patiently building the deeper capability behind them.

The aim is **understanding**: ideas the student can reconstruct, connect, and use beyond the current exercise. Explanation trajectories help reveal this; delayed independent tasks help test whether it lasts. Teaching methods and confidence estimates remain hypotheses to validate, while learner dignity and agency are commitments.

The platform's name comes from the French *limpide* (clear, transparent, untroubled). The name describes what the student becomes during a good session — not what the system is or does.

## Architecture at a glance

- **A plugin, not a process.** Limpide is a package listed in an AYA host's config; it contributes agents, tools, workflows, policy evaluators, plugin-private schema, widgets, renderables, and playbooks to the host's three processes (frontend, worker, SurrealDB). It adds no process of its own.
- **Two agents** on one tutoring Task: a **Tutor** (Flue agent, interactive, runs the session loop, holds the floor) and a **Gardner** (Flue agent, analytic, debounced, reads the Event Journal, extracts gaps and curiosity signals, never speaks to the student — enforced where events are landed, not in prompts).
- **Flue is the motor, AYA is the envelope.** Flue runs the agentic loop; AYA's primitives (Task, Event Journal, Playbook, Foundation, Policy Gate, …) govern it. A Task crosses into Flue as refs only, through the host's Dispatch adapter.
- **One database**: SurrealDB — host tables plus Limpide's plugin-private tables; embedded for desktop / single-tenant deployments, server mode for hosted use.
- **No edge tier.** The browser subscribes to SurrealDB live queries directly under Surreal's permission model; the worker hosts the BFF routes; scheduled work is playbooks ticked by the host clock.
- **Identity**: Logto, through the host; consent is Limpide data, not an identity claim.
- **Frontend**: the AYA React + Vite host with Limpide widgets and `limpide.*` renderables.

The full architecture is in [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

## The documents

| Document | What it covers | Read it when |
|---|---|---|
| [`docs/VISION.md`](docs/VISION.md) | The educational thesis. Why this platform exists, what it's pushing against, what success looks like. | You're trying to remember why a particular rule is non-negotiable, or you're explaining the project to someone who needs the *why* before the *how*. |
| [`docs/PEDAGOGY.md`](docs/PEDAGOGY.md) | What the system does, in pedagogical terms. The session loop, the recursive fork mechanism, the pinned-gap agenda, the cross-substrate triad work, the urgency modes. | You're designing a new feature and need to check whether it serves the pedagogy. You're writing a system prompt for Tutor or Gardner and need to know what behaviors to encode. |
| [`docs/MEASUREMENT.md`](docs/MEASUREMENT.md) | How the system knows it's working. Candidate teach-back signals, independent delayed assessments, attribution, and the discipline needed to validate confidence estimates. | You're deciding what to instrument, what counts as a confidence increase, or what evidence is needed before reporting efficacy. |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | How the system is built. The plugin on the AYA host, the two-Flue-sessions-on-one-Task execution shape, plugin-private SurrealDB schema, data model in TypeScript interfaces, Policy Gate evaluators, data scopes, deployment topologies. | You're writing code, reviewing a PR, or making a technical decision that touches multiple components. |
| [`docs/STRATEGY.md`](docs/STRATEGY.md) | The commercial hypotheses: first payer, useful institutional feedback, potential differentiation, and the network as a deferred business goal. | You're explaining how Limpide sustains itself, sizing a deployment, or making a decision where commercial incentive meets pedagogy. |
| [`docs/BOOTSTRAP-PILOT.md`](docs/BOOTSTRAP-PILOT.md) | Proposed placement and daily-practice offer, graph/data boundaries, independent assessment, and subject sequencing. | You are discussing the first learner offer or preparing teacher conversations. |
| [`docs/ADR/`](docs/ADR/) | Architecture Decision Records. Each ADR captures one decision, the alternatives considered, and the reasoning. | You're considering changing something. Find the ADR that established it, understand why, then decide whether the reasoning still holds. |

## Reading order, by purpose

- **First contact** (you, six months from now, having forgotten everything): `VISION.md` → `PEDAGOGY.md` → `MEASUREMENT.md` → `ARCHITECTURE.md` → relevant ADRs. Read `STRATEGY.md` when the question is commercial rather than pedagogical.
- **Building a feature**: `PEDAGOGY.md` (what should it do?) → `MEASUREMENT.md` (how will we know it works?) → `ARCHITECTURE.md` (where does it live?) → ADRs that touch the affected components.
- **AI agent context-loading** (Tutor, Gardner, code-generation tools): all documents in order. They are designed to be read in full by an LLM with reasonable context window.

## Project status conventions

Throughout these documents:

- **Settled** — the decision is made and code should reflect it. Reopening requires a new ADR.

Limpide's documents were written in April 2026 against a planned substrate (agent-core, Cloudflare Workers, a phased Convex→SurrealDB migration). AYA's substrate then moved under them — Flue, no edge tier, an outright SurrealDB + Logto flip, a plugin host — and the documents were realigned on 2026-08-24 (ADR-0007, ADR-0019, ADR-0020; ADR-0002/0003 superseded, ADR-0004 amended, ADR-0005 marked completed). If a Limpide document and an AYA source of truth (`aya/AGENTS.md`, `aya/docs/architecture.md`, `aya/docs/ADR/`) disagree on the substrate, AYA is right and the Limpide document needs an ADR.
- **Tentative** — the current best guess, but the design has not been pressure-tested. Expect revision.
- **Open** — the question has been raised but not answered. Do not build against this.

## License

Not yet decided. Licensing ADR-0006 remains an open placeholder in the [ADR index](docs/ADR/README.md).

## Related repositories

- [AYA](https://github.com/aegilo/aya) — the host Limpide is a plugin against: the eight primitives on SurrealDB, Logto identity, the Policy Gate, the worker running Flue, and the plugin contract (AYA ADR-0017). Limpide's ADRs cite AYA's as "AYA ADR-NNNN".
- [Flue](https://flueframework.com) — the execution engine (`@flue/runtime`) AYA's worker runs agents on. Limpide's Tutor, Gardner, and classifier are Flue agents and workflows; Limpide has no framework of its own.
- `@aegilo/agent-core` — retired (AYA ADR-0014). Referenced only in superseded ADRs kept as history.
