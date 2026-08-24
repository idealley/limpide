# ADR-0002: Cloudflare Workers for the edge layer

**Status:** Superseded by ADR-0020 (2026-08-24)
**Date:** 2026-04-29

> **Superseded.** AYA exited Convex without a Workers relay: the browser subscribes to SurrealDB live queries directly, the Node worker hosts the BFF routes and webhooks, and scheduled work is the host's scheduled-playbook clock (AYA ADR-0018). Limpide follows that shape and has no edge tier of its own — see `0020-edge-layer-follows-aya.md`. The text below is kept as decision history; the one part that survives is that the agent loop stays in the worker process.

## Context

Migrating from Convex to SurrealDB (ADR-0001 and ADR-0005) loses three things Convex provided out of the box:

1. Reactive query subscriptions to the frontend.
2. HTTP webhook receivers (for inbound email, OAuth callbacks, future channels).
3. Scheduled background work.

We need replacements for each. We also need an auth and rate-limiting layer between the frontend and the agent process.

## Decision

Cloudflare Workers handle the edge layer. Specifically:

- **WebSocket fan-out** via Durable Objects. Frontend connects to a Worker; the Worker proxies SurrealDB live queries with auth in between.
- **Webhook receivers.** Workers verify signatures, transform payloads, and forward to SurrealDB or enqueue agent work.
- **Scheduled triggers** via Cron Triggers. Gardner background work, confidence decay calculations, end-of-day rollups.
- **Auth and rate limiting.** Workers as the front line, before any traffic reaches the agent process or SurrealDB.

The agent process — the long-running Tutor and Gardner runtime — stays as a Node service. Workers' CPU time limit per request is too short for a multi-LLM-call agent loop.

## Alternatives considered

**Vercel Edge Functions.** Similar capability profile to Workers. Rejected because Vercel's pricing model is less predictable for high-traffic WebSocket workloads and because Workers' Durable Objects are a better fit for stateful WebSocket connections than Vercel's edge runtime.

**A traditional Node backend (Express, Fastify) handling everything.** Rejected because we still need the long-running agent process anyway, and adding a second Node process for edge concerns just doubles the operational surface without gaining much.

**Skip the edge layer entirely.** Frontend talks directly to the agent process which talks directly to SurrealDB. Rejected because we lose graceful handling of webhooks, scheduled work, and the auth boundary, and because exposing the agent process directly to the public internet is a security regression.

**Cloudflare Containers for the agent process too.** Considered. Cloudflare Containers are now generally available and would let us keep the entire stack on Cloudflare. Tentatively yes, eventually — but not for the MVP. For now, the agent process is a Node service deployed wherever convenient (Fly.io, Railway, a VM); migration to Containers is a future-friendly option, not a current commitment.

## Consequences

**Commits us to:**
- Cloudflare as a vendor for edge concerns. Vendor lock-in is real but the alternatives are similar.
- Durable Objects' programming model. It's good but it's specific.
- Writing more glue than Convex required for reactive frontend state. SurrealDB's live queries plus Workers' WebSocket relay is more code than Convex's reactive queries out of the box.

**Precludes:**
- Pure self-hosted deployments without Cloudflare. The edge layer is Cloudflare-specific. (Workaround: for institutions that need self-hosting, the embedded SurrealDB topology with the frontend talking directly to the agent process can work, at the cost of not having the edge benefits.)

**Opens:**
- Global low-latency for the frontend. Workers run at every Cloudflare PoP.
- A clean operational story: edge concerns at Cloudflare, reasoning at the agent process, data at SurrealDB. Each piece does what it's best at.
- Easy webhook integrations for future channels (Slack, Discord, school LMS systems).

## Status

**Settled** for the hosted topology.

**Open** for the embedded / self-hosted topology — those deployments may skip the edge layer entirely, with the frontend talking directly to the agent process. The architecture document notes this. The data model and agent shape do not depend on Workers; the edge layer is replaceable.
