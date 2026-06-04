# Architecture Decision Records

Each ADR captures one decision. The format is: context, decision, alternatives considered, consequences. ADRs are append-only — superseded ADRs are marked `Superseded by ADR-NNNN` rather than deleted.

## Index

| ADR | Title | Status |
|---|---|---|
| [0001](0001-database-surrealdb.md) | Use SurrealDB as the single data layer | Accepted |
| [0002](0002-edge-cloudflare-workers.md) | Cloudflare Workers for the edge layer | Accepted |
| [0003](0003-framework-agent-core.md) | Build on agent-core, not from scratch | Accepted |
| [0004](0004-two-agent-shape.md) | Tutor and Gardner as two agents on a shared session | Accepted |
| [0005](0005-aya-migration-to-surrealdb.md) | Migrate AYA from Convex to SurrealDB | Accepted, phased |
| 0006 | Licensing | Open |
| 0007 | Auth provider | Open |
| 0008 | Frontend framework details | Open |
| [0009](0009-probe-gating.md) | Probe gating — deterministic policy over learned strategy | Proposed |
| [0010](0010-concept-depth.md) | Concept depth — program-required floor, student-desired ceiling | Proposed |
| [0011](0011-classifier-calibration.md) | Disclosure classifier calibration and gold-set construction | Proposed |
| [0012](0012-expressive-baseline.md) | Expressive baseline — measuring understanding relative to expected articulation | Proposed |
| [0013](0013-canonical-graph-and-overlays.md) | Canonical concept graph and curriculum overlays | Proposed |
| [0014](0014-safeguarding-and-child-data.md) | Safeguarding and child data protection | Proposed |
| [0015](0015-acceptable-use.md) | Acceptable use of institutional evidence | Proposed |
| [0016](0016-system-evaluation.md) | Evaluating the system | Proposed |
| [0017](0017-latency-cost-budget.md) | Latency and cost budget for the interactive path | Proposed |
| [0018](0018-confidence-model.md) | The confidence model | Proposed |

## Template

```markdown
# ADR-NNNN: Title

**Status:** Proposed | Accepted | Superseded by ADR-NNNN | Rejected
**Date:** YYYY-MM-DD

## Context

What is the situation that requires a decision? What are the constraints?

## Decision

What did we decide? Stated as a fact, not a recommendation.

## Alternatives considered

What other paths did we consider? What were their trade-offs?

## Consequences

What does this decision commit us to? What does it preclude? What new questions does it open?

## Status

Settled / Tentative / Open, per the project conventions.
```
