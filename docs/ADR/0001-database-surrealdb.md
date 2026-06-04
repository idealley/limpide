# ADR-0001: Use SurrealDB as the single data layer

**Status:** Accepted
**Date:** 2026-04-29

## Context

Limpide's data is genuinely multi-model:

- The curriculum is a graph (prereq edges, cross-substrate links).
- Concepts and transcripts are documents.
- Gap records, encounters, and student profiles are relational.
- Curiosity classification will eventually need vector search.
- Sessions need real-time event-driven updates pushed to the frontend.

The naive stack would be Postgres for relational, with pgvector for embeddings, plus a graph layer (Neo4j or similar), plus a document store (or JSONB in Postgres), plus some pub/sub for reactivity. Four systems. Four query languages. Four operational stories. Synchronization seams between them.

AYA today uses Convex, which is a good unified backend for some of these concerns but has limitations: Convex is its own ecosystem with its own deployment model, doesn't offer graph traversal as a primitive, and its real-time queries are excellent but not portable to other deployment topologies (embedded, edge, offline-first).

## Decision

Limpide uses SurrealDB 3.x as its single data layer.

SurrealDB unifies relational, document, graph, time-series, and vector data in one engine with one query language (SurrealQL). It runs as a single Rust binary, embeddable as a library or deployable as a server or distributed cluster. Live queries over WebSocket provide real-time reactivity equivalent to what Convex offers.

AYA migrates to SurrealDB on the same trajectory; see ADR-0005.

## Alternatives considered

**Postgres + pgvector + a graph layer.** The conservative choice. Mature, well-understood, abundant tooling. Rejected because the multi-system complexity dominates a one-person team's budget and because graph traversal in SQL is painful enough that the curriculum graph would suffer.

**Stay on Convex.** Convex is genuinely good for what it does. Rejected because Convex doesn't offer graph traversal, doesn't support embedded deployment for desktop or offline-first paths, and locks AYA into a specific deployment ecosystem. The flexibility cost of staying on Convex is higher than the migration cost over the project lifetime.

**Neo4j as primary store.** Strong on graphs, weak on everything else. Rejected because the relational and document data wouldn't fit naturally and we'd need a second store anyway.

**FaunaDB / DGraph / TigerGraph.** Considered briefly. Each strong in some axis, none unifying all four data shapes Limpide needs in one engine.

## Consequences

**Commits us to:**
- Investing in SurrealQL fluency. The query language is novel and the team needs to learn it.
- Filing issues against a 3.0-fresh database. Production deployments at Tencent, Volvo, Walmart de-risk this somewhat but not fully.
- A specific 3.x version pin. Upgrades will be deliberate, not automatic.

**Precludes:**
- A vendor-supported managed offering as mature as Postgres-on-RDS. SurrealDB Cloud exists but is younger.
- Some tooling: ORMs, migration frameworks, and BI tools assume SQL. We'll work without them or build minimal versions.

**Opens:**
- The path to embedded deployment (desktop, offline-first) for free.
- Graph-native curriculum traversal queries that would be pages of SQL.
- Vector search in the same engine when curiosity classification needs it.

**Honest caveats noted in the architecture document:**
- SurrealDB 3.0's new streaming query engine currently covers read-only statements only. Write-heavy workloads use the older execution path.
- Live query subscriptions are powerful but the ergonomics in the React ecosystem are younger than Convex's. We'll write more glue.

## Status

**Settled.** Reopening requires concrete production-blocking issues with SurrealDB that can't be worked around.
