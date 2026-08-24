# ADR-0020: The edge layer follows AYA — Surreal live queries, worker BFF, host clock

**Status:** Accepted
**Date:** 2026-08-24
**Supersedes:** ADR-0002 (Cloudflare Workers for the edge layer)
**Relates to:** ADR-0019, ADR-0007, AYA ADR-0018 (Scheduled Playbook clock), AYA `docs/architecture.md`

## Context

ADR-0002 chose Cloudflare Workers for three jobs Convex used to do for AYA: reactive subscriptions to the frontend, webhook receivers, and scheduled background work — plus an auth and rate-limiting boundary in front of the agent process. It was written in April 2026 when AYA still ran on Convex and the Convex exit was planned around a Workers relay.

AYA did not exit Convex that way. The shape that shipped (AYA `docs/architecture.md`, `AGENTS.md`):

- **The browser subscribes to SurrealDB live queries directly**, authenticated with a short-lived Surreal token minted by the worker from the user's Logto identity (`VITE_SURREAL_TOKEN_ENDPOINT`, `aya_user` RECORD access resolving `$auth` from `$token.aya_sub`). Row-level permissions live in Surreal, not in a relay.
- **The Node worker hosts the BFF routes** (Hono), the webhook receivers (inbound email, GSOC), the Surreal lease queues, and the Flue processor. There is no separate edge tier.
- **Scheduled work is a host Dispatch feature**: a playbook's `schedule` projection compiles to due work that the worker leases, heartbeats, and journals (AYA ADR-0018). Convex crons were deleted; nothing replaced them with a third-party scheduler.
- Auth is Logto (AYA ADR-0008); quota and policy checks run in the worker and fail closed.

Limpide, as a plugin against this host (ADR-0019), cannot sensibly run a private Workers edge in front of a host that has none. It would duplicate auth, split the WebSocket story, and put a Limpide-only scheduler beside the host clock.

## Decision

**Limpide has no edge tier of its own. It uses AYA's shape:**

1. **Reactive frontend state** = SurrealDB live queries from the browser, over the host's token endpoint and Surreal permissions. Limpide's plugin-private tables carry the same `$auth`-scoped PERMISSIONS clauses as the host tables; the four data scopes of `ARCHITECTURE.md` are enforced there and at the gate, not in a relay.
2. **Request/response and webhooks** = worker BFF routes contributed by the Limpide plugin (ADR-0017 "worker BFF routes"). Session start, program enrollment, teacher views, and any future inbound channel (an LMS callback, for instance) are routes in the worker, behind the host's auth and `httpProtection`.
3. **Scheduled work** = playbooks with a `schedule` projection, seeded by the plugin and ticked by the host clock. Confidence decay (ADR-0018), pinned-gap cooldown advancement, agenda audits, and end-of-day rollups are each a scheduled playbook. Every tick writes a heartbeat event; an external monitor watches freshness (AYA ADR-0018 §4). Limpide ships no cron.
4. **The long-running agent loop** stays in the worker process, as ADR-0002 already said; that part of its reasoning survives.

## Alternatives considered

**Keep Workers as designed (ADR-0002).** Rejected: it would make Limpide the only AYA vertical with an edge tier, duplicate Logto-backed auth in a second place, and require a Workers-side scheduler beside the host clock, which AYA ADR-0018 forbids for plugins ("They do not ship a private cron").

**Workers in front of the host for everything (adopt it at the host level).** Out of Limpide's scope; that would be an AYA ADR. Nothing here precludes it — if AYA later fronts the worker with Cloudflare, Limpide inherits it.

**Cloudflare Containers for the worker.** Still a legitimate future deployment target for the *worker process* (Flue builds for Node or Cloudflare); it is a hosting choice, not an architecture choice, and belongs in deployment docs.

## Consequences

**Commits us to:**
- Surreal permissions as the enforcement point for the personal / cohort / institutional / network scopes on the read path. The schema work in `ARCHITECTURE.md` ("the schema distinguishes the scopes from day one") is now also permission work from day one.
- Host-issued Surreal tokens; Limpide mints none of its own.
- Scheduled playbooks as the *only* scheduling mechanism.

**Precludes:**
- Durable Objects and any Cloudflare-specific state. The tentative "Workers Durable Objects design for WebSocket fan-out" in `ARCHITECTURE.md` is dropped.
- A Limpide deployment that is *not* an AYA deployment. (The single-binary / embedded topology remains: it is the AYA edge profile — embedded Surreal, local single-user identity — with the Limpide plugin listed.)

**Opens:**
- One fewer vendor and one fewer runtime to reason about for safeguarding and child-data compliance (ADR-0014): the DPIA describes three processes, not four.
- Self-hosting for institutions becomes the same story as AYA on-prem, which Aegilo's GSOC clients will already have exercised.

## Status

**Settled.** No Limpide edge tier; live queries + worker BFF + host clock.

**Tentative.** Whether teacher/operator views (cohort scope) are served by live queries with permission clauses or by BFF routes that aggregate server-side. Aggregation-at-read favours BFF; reactivity favours live queries; the cohort scope's "no drill-down to a named individual" rule (`ARCHITECTURE.md`) is easier to guarantee in a BFF route. Decide when the first teacher view is built (Phase 2 of the scope phasing).
