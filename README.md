# Limpide

> **Understanding is what survives explaining it.**

A tutoring platform built on the principle that understanding is *discovered*, not delivered — and that *understanding* is a different thing than *learning*. Learning is information transferred. Understanding is information that has become part of how the student thinks. Limpide exists to produce the second.

## Status

Early design. No code yet. This repository currently holds the design documents that will guide implementation.

## What this is

Limpide is a Skill running on the AYA / `@aegilo/agent-core` substrate. It is a tutor that asks the student to explain what they think they know, surfaces gaps through Socratic probing, and makes the student feel the difference between thinking they understand and actually understanding. It honors urgent needs (the test on Friday) while patiently building the deeper capability behind them.

The verb that matters is **understand**, not learn. Limpide does not aim to transfer information efficiently — that's what books and search engines are for. Limpide aims to produce the kind of understanding that survives the student's attempt to explain it, that can be rebuilt from first principles, that changes how the student reads the next thing they encounter.

The platform's name comes from the French *limpide* (clear, transparent, untroubled). The name describes what the student becomes during a good session — not what the system is or does.

## Architecture at a glance

- **Two agents**, attached to a shared session: a **Tutor** (interactive, conversational, runs the session loop) and a **Gardner** (analytic, reads the transcript, extracts gaps and curiosity signals, never speaks to the student directly).
- **One database**: SurrealDB, embedded for desktop / single-tenant deployments, server mode for hosted use.
- **Edge layer**: Cloudflare Workers for webhooks, scheduled work, and WebSocket fan-out to the frontend.
- **Long-running agents** stay as Node processes (eventually possibly Cloudflare Containers) — Workers are too short-lived for the agent loop.
- **Frontend**: React + Vite, talks to the agents via Workers, subscribes to SurrealDB live queries for reactive state.

The full architecture is in [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

## The documents

| Document | What it covers | Read it when |
|---|---|---|
| [`docs/VISION.md`](docs/VISION.md) | The educational thesis. Why this platform exists, what it's pushing against, what success looks like. | You're trying to remember why a particular rule is non-negotiable, or you're explaining the project to someone who needs the *why* before the *how*. |
| [`docs/PEDAGOGY.md`](docs/PEDAGOGY.md) | What the system does, in pedagogical terms. The session loop, the recursive fork mechanism, the pinned-gap agenda, the cross-substrate triad work, the urgency modes. | You're designing a new feature and need to check whether it serves the pedagogy. You're writing a system prompt for Tutor or Gardner and need to know what behaviors to encode. |
| [`docs/MEASUREMENT.md`](docs/MEASUREMENT.md) | How the system knows it's working. Why understanding is measurable only through process, the four teach-back signals, and the Goodhart discipline that keeps them honest. | You're deciding what to instrument, what counts as a confidence increase, or how to report efficacy without reintroducing the metrics that destroy understanding. |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | How the system is built. Agent-core extensions, SurrealDB schemas, Workers split, data model in TypeScript interfaces, Policy Gate integration. | You're writing code, reviewing a PR, or making a technical decision that touches multiple components. |
| [`docs/STRATEGY.md`](docs/STRATEGY.md) | The commercial shape. The three-scale model (individual, institutional, network), what institutions actually buy, the three moats, and pricing posture. | You're explaining how Limpide sustains itself, sizing a deployment, or making a decision where commercial incentive meets pedagogy. |
| [`docs/ADR/`](docs/ADR/) | Architecture Decision Records. Each ADR captures one decision, the alternatives considered, and the reasoning. | You're considering changing something. Find the ADR that established it, understand why, then decide whether the reasoning still holds. |

## Reading order, by purpose

- **First contact** (you, six months from now, having forgotten everything): `VISION.md` → `PEDAGOGY.md` → `MEASUREMENT.md` → `ARCHITECTURE.md` → relevant ADRs. Read `STRATEGY.md` when the question is commercial rather than pedagogical.
- **Building a feature**: `PEDAGOGY.md` (what should it do?) → `MEASUREMENT.md` (how will we know it works?) → `ARCHITECTURE.md` (where does it live?) → ADRs that touch the affected components.
- **AI agent context-loading** (Tutor, Gardner, code-generation tools): all documents in order. They are designed to be read in full by an LLM with reasonable context window.

## Project status conventions

Throughout these documents:

- **Settled** — the decision is made and code should reflect it. Reopening requires a new ADR.
- **Tentative** — the current best guess, but the design has not been pressure-tested. Expect revision.
- **Open** — the question has been raised but not answered. Do not build against this.

## License

Not yet decided. See [`docs/ADR/0006-licensing.md`](docs/ADR/0006-licensing.md) (when written) for the open question.

## Related repositories

- [`@aegilo/agent-core`](https://github.com/aegilo/agent-core) — the agent framework Limpide runs on. Limpide-specific extensions live here as ADRs and PRs.
- [AYA](https://github.com/aegilo/aya) — the broader assistant platform Limpide is a Skill within. Limpide drives some of AYA's evolution (e.g., the SurrealDB migration).
