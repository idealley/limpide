# ADR-0005: Migrate AYA from Convex to SurrealDB

**Status:** Accepted, phased
**Date:** 2026-04-29

## Context

AYA today runs on Convex. ADR-0001 commits Limpide to SurrealDB. Running both means maintaining two backends, two query languages, two deployment stories, and a synchronization seam between AYA's primitives (Tasks, Events, Foundation rules) and Limpide's data (curriculum graph, student subgraph, gap records).

The simplification we're after only happens if AYA also migrates.

## Decision

AYA migrates from Convex to SurrealDB. The migration is phased to de-risk it:

**Phase 1.** A new SurrealDB-backed Memory implementation in agent-core. Existing Memory backends (InMemory, Convex HTTP, FalkorDB) continue to work. No AYA changes. Limpide can ship on this.

**Phase 2.** Limpide ships in production using the SurrealDB Memory backend with its own SurrealDB instance. AYA's other primitives stay on Convex. Patterns are proven under real load.

**Phase 3.** AYA's Tasks, Events, Foundation, and other primitives port to SurrealDB in one cohesive pass. The boundaries between AYA primitives are tight enough that piecemeal migration would create more seams than it removes.

**Phase 4.** Convex deprecated for AYA. Existing data migrated. Convex's reactive query subscriptions are replaced by Cloudflare Workers (ADR-0002) proxying SurrealDB live queries.

Phase 1 starts immediately. Phase 2 follows the Limpide MVP launch. Phases 3 and 4 are months out and depend on Phase 2 going well.

## Alternatives considered

**Keep AYA on Convex; only Limpide on SurrealDB.** The path of least resistance, and it's where Phase 1 + Phase 2 leaves us anyway. Rejected as the *long-term* state because two-database operation is a permanent tax. Acceptable as a transitional state.

**Migrate AYA to SurrealDB first, then build Limpide.** Reversed order. Rejected because Limpide is the project with momentum and a real near-term deliverable, while AYA's migration is months of work with no immediate user-facing benefit. Better to prove SurrealDB on Limpide first.

**Migrate AYA piecemeal — Tasks first, then Events, then Foundation.** Rejected because the AYA primitives reference each other heavily. A Task references its Foundation; an Event is tied to a Task; the Policy Gate reads Foundation and emits Events. Migrating one at a time would require building cross-database adapters that get thrown away.

**Adopt a different unifying database (not SurrealDB).** Considered as part of ADR-0001. SurrealDB won that decision; this ADR follows.

## Consequences

**Commits us to:**
- Investing in the SurrealDB-backed Memory implementation as a real, well-tested backend, since it's the foundation for the broader AYA migration.
- A multi-month project to migrate AYA, with Phase 3 being the heaviest lift.
- Coordinated changes to AYA's documentation, API, and any external integrations during Phase 4.

**Precludes:**
- Convex-specific features in any new AYA work. New code targets SurrealDB-compatible patterns even while running on Convex.
- Adding a third database. The simplification thesis only works with one.

**Opens:**
- Embedded deployments of AYA. SurrealDB embedded means AYA can run on a desktop or in an offline-first mode, which Convex couldn't support.
- Graph-native queries across AYA primitives. The Foundation → Playbook → Task relationships are graph-shaped and SurrealQL handles them naturally.
- A cleaner deployment story for self-hosting institutions.

## Risks

**The migration takes longer than expected.** Likely. Database migrations always do. The phased approach contains the risk: Phases 1 and 2 deliver immediate value to Limpide regardless of whether Phases 3 and 4 ever happen.

**Production issues with SurrealDB at AYA's scale.** SurrealDB 3.0 is fresh. Phases 1 and 2 run Limpide on it first, which gives us early-warning signal before AYA's broader workload depends on it.

**Frontend reactive-state regression during Phase 4.** Convex's reactive queries are excellent and Workers + SurrealDB live queries is more glue. A regression in user-perceived responsiveness is possible. Mitigation: Phase 4 includes performance benchmarks, and we keep Convex running in parallel during cutover until parity is verified.

## Status

**Settled** on the trajectory.

**Tentative** on the Phase 3 and Phase 4 timing — these depend on Phase 2 going well and on what we learn from real production load.

**Open** on whether AYA's external API (the parts other applications depend on) needs a stability guarantee during migration. This is a product decision, not a technical one, and it depends on AYA's external commitments at the time of Phase 3.
