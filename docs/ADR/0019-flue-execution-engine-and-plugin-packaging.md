# ADR-0019: Run on Flue inside the AYA envelope; ship Limpide as an AYA plugin

**Status:** Accepted
**Date:** 2026-08-24
**Supersedes:** ADR-0003 (Build on agent-core, not from scratch)
**Amends:** ADR-0004 (Tutor and Gardner as two agents on a shared session)
**Relates to:** AYA ADR-0014 (Converge AYA on Flue as the execution engine), AYA ADR-0017 (Trusted plugin packages), AYA ADR-0018 (Scheduled Playbook clock), AYA `docs/design/2026-08-22-plugin-host-and-verticals.md`

## Context

ADR-0003 committed Limpide to `@aegilo/agent-core` and to shipping as an AYA *Skill*, with a set of agent-core extensions (shared `Session`, `AttachedAgent`, lanes, floor, event visibility) that Limpide would contribute back.

Two things changed on the AYA side between April and August 2026, and Limpide's documents did not follow:

1. **AYA retired agent-core and converged on Flue** (AYA ADR-0014, accepted 2026-06-17). agent-core's differentiator was the voice/modality layer; voice was cut; what remained was a parallel re-implementation of a generic agent framework. Flue (`@flue/runtime`, currently 1.0.0-beta.1) is now the motor: the agentic loop, tools, skills, subagents, sandboxes, model providers, streaming, workflow orchestration. AYA is the *envelope*: the eight primitives on SurrealDB, the Policy Gate, the append-only Event Journal, multi-tenancy and IAM, Renderables. The seam between them is the **Dispatch adapter**, which is refs-only: a Task crosses into Flue as a JSON payload of ids, and the adapter reconstructs tenant-scoped repositories, the gate, and the journal sinks from those refs. No Flue types leak into the primitives.

2. **AYA became a plugin host** (AYA ADR-0017, accepted 2026-08-22). The host boots with zero plugins. A vertical product — Email EA, Aegilo, and explicitly "later Limpide" — is a resolvable package that exports an `AppBundle` plus contribution hooks: widgets, namespaced renderable kinds, policy evaluators, plugin-private Surreal schema, worker BFF routes, tools, Flue workflows, named loops, and playbook seeds (including scheduled ones, via the host clock of ADR-0018). A plugin may *propose* Foundation amendments but never writes Foundation. "Skill" in AYA now means a Flue skill — a versioned procedure the agent follows — not a product.

The consequence for Limpide: the word "Skill" in our documents means something else now, the framework we designed extensions for no longer exists, and the multi-agent extension we planned to contribute has no home. The question is whether the design survives the substrate change. It does — because the design was always stated in terms of the AYA *primitives* (Foundation, Playbook, Task, Event, Memory, Dispatch, Policy Gate, Ontology), and those are exactly the part AYA kept.

## Decision

**Limpide runs on Flue as the execution engine, inside the AYA envelope, and ships as an AYA plugin package (`@aegilo/plugin-limpide` or equivalent; name tentative).** Nothing is built on agent-core. No Limpide-specific agent runtime is written.

### 1. The primitive mapping stands unchanged

The table in `ARCHITECTURE.md` (Foundation → F-rules as gate rules; Playbook → the six-state session loop; Task → a tutoring session; Event → transcript and state log; Memory → curriculum graph and student subgraph; Dispatch → lanes; Policy Gate → deterministic ALLOW/DENY/REQUIRE_REVIEW; Ontology → canonical vocabulary) was the substance of ADR-0003 and it is unaffected. It is the reason this ADR is a substrate swap and not a redesign.

### 2. Tutor and Gardner are Flue agents; the tutoring session is a Task admitted through the Dispatch adapter

- **A tutoring session is an AYA Task** with the standard lifecycle. When a student message arrives, Dispatch leases the Task and admits it to Flue through the refs-only adapter (`{ entityId, accountId, taskId, principal, correlationId, … }`), exactly as AYA's chat path does today (`worker/src/workflows/chat-dispatch-adapter.ts`).
- **The Tutor is a Flue agent** (`createAgent`) with a persistent Flue session per tutoring session, Opus-class model, the Limpide tool set, and the session-loop Playbook attached as a Flue skill. It holds the conversational floor by construction: it is the only agent whose output is landed as a `public` message.
- **The Gardner is a Flue agent that runs as a named loop / workflow, not as a subagent of the Tutor.** The reasons are the same as in ADR-0004: different latency (debounced vs interactive), different model (Haiku-class), and F3 — Gardner's output must never reach the student. A Flue subagent shares the parent's sandbox and returns its result *into the parent's transcript*, which is the wrong direction for F3. Instead, Gardner is a separate Flue session triggered by the worker on the debounce conditions of ADR-0004 (event count, latency threshold, or an explicit flush before a state transition), reading the Event Journal and writing gap records, curiosity signals, attributions, and teach-back evaluations back through the Limpide repositories. Its journal events carry `visibility: 'internal'`.
- **The shared session is the AYA Task plus the Event Journal**, not a framework object. The `Session` / `AttachedAgent` / `Dispatcher` interfaces in the former "Required extensions to agent-core" section of `ARCHITECTURE.md` are retired as *framework* contracts and retained as the *plugin-side* contract Limpide's worker code implements over Flue sessions: `awaitLane('analytic')` becomes "wait for the Gardner loop to drain for this Task"; the floor is "only the Tutor's session output is landed as public"; event visibility is a field on Limpide's journal events, enforced where events are landed, not in prompts.
- **The disclosure classifier** (ADR-0009, ADR-0011) is a Flue workflow with a structured `result` schema on the hot path, invoked from the Tutor's probe tool before a probe is landed. It is deliberately not an agent with a session.

### 3. The Policy Gate is AYA's; Limpide contributes evaluators

Foundation rules F1–F12, the transition gate (F10), `evaluateProbe` (ADR-0009), and the safeguarding gate (F12, ADR-0014) are **policy evaluators the Limpide plugin registers with the host's `PolicyGateRegistry`** — the same seam Email EA uses. They are invoked from inside the Flue workflow around tool calls, outbound events, and state transitions ("the envelope in miniature", AYA ADR-0014 §2). Gate initialisation errors fail closed. The propose-vs-enact discipline on the gate's own evolution (ADR-0009) is preserved: Gardner proposes, a human enacts, and a plugin may not write Foundation.

### 4. What the plugin contributes, and what it may not

Contributes: `AppBundle` (the Limpide app manifest), the Tutor/Gardner/classifier Flue definitions and workflows, Limpide tools (curriculum read, probe emission, gap and encounter writes, memory queries), policy evaluators, plugin-private Surreal schema (`limpide_*` tables — curriculum graph, programs, student subgraph, gaps, pinned gaps, curiosity signals, probes), namespaced renderable kinds (`limpide.*` — inline visualisations, the pinned-gap sidebar), widgets, playbook seeds (the session loop; scheduled playbooks for decay, agenda audits, rollups), and declared event types.

May not: add primitives; write Foundation; put credentials in the browser; bypass the Dispatch adapter with an in-process shortcut.

### 5. Deployment shape follows the host

Three processes, as for every AYA vertical: the AYA host (frontend + `@aya/platform` + `@aya/runtime`), the worker running Flue, and SurrealDB. Limpide has no fourth process. Single-tenant / on-prem is packaging of the same three processes, not a fork (host design doc, "SaaS vs on-prem"). The edge layer follows AYA's actual shape (ADR-0020).

## Alternatives considered

**Keep agent-core alive for Limpide only.** Rejected. AYA deleted it (flip, not migration — `AGENTS.md`); Limpide would be the sole maintainer of a framework whose only remaining differentiator was cut, and would lose the Dispatch adapter, Policy Gate registry, and plugin host it wants to reuse.

**Build Limpide directly on Flue without the AYA envelope.** Tempting for a microsite or a first pilot: `createAgent` + a Surreal tool + a Hono `app.ts` is a weekend. Rejected as the *product* architecture because the envelope is exactly what the pedagogy needs — the Policy Gate is what makes F-rules deterministic rather than prompt suggestions, the Event Journal is what makes the process signals of `MEASUREMENT.md` exist at all, and the four-scope data model (`ARCHITECTURE.md`) is a host feature. Flue on its own is single-tenant and ungoverned by design (AYA ADR-0014 §1). A throwaway standalone-Flue prototype for content authoring or a diagnostic microsite is fine; it is not Limpide.

**Gardner as a Flue subagent of the Tutor.** Rejected (see §2): a subagent's result returns into the parent's transcript, which is exactly the leak F3 forbids, and it couples Gardner's cadence to the Tutor's turn.

**A LangGraph / Mastra / bespoke runtime.** Already rejected in ADR-0003; the reasoning holds and is now stronger, since the AYA org has one framework rather than two.

## Consequences

**Commits us to:**
- Flue as the engine, at whatever version AYA pins. Limpide follows AYA's Flue upgrades; it does not pin separately.
- The refs-only Dispatch adapter as the *only* way a tutoring Task reaches an agent. No in-process injection of live repositories into workflows.
- The AYA plugin contract (ADR-0017): manifest, contribution hooks, plugin-private schema, namespaced renderables. Limpide UI does not land in AYA's `src/` before the host's plugin extraction (F-042) exists — the same sequencing rule Aegilo is under.
- Rewriting ADR-0004's mechanism in these terms (done in the amended ADR-0004), while keeping its decision.

**Precludes:**
- Contributing multi-agent "Session" extensions upstream — there is no upstream to contribute to. Ensemble patterns now live in plugin code over Flue sessions and the Event Journal.
- Any Limpide-owned cron, queue, or scheduler. Scheduled work is a playbook with a `schedule` projection ticked by the host clock (AYA ADR-0018).

**Opens:**
- Flue's batteries for free: OTel traces (which `docs/ADR/0017-latency-cost-budget.md` needs), structured results for the classifier and Gardner, sandboxes if a future substrate needs code execution (the `code` teach-scaffold channel), MCP for external curriculum sources.
- Limpide as the second commercial plugin against a published host — Aegilo proves the packaging, Limpide inherits it.
- The path for a standalone Flue prototype (content authoring, probe generation per `methods/synthetic-probe-generation.md`, a diagnostic microsite) that later lands its output into the plugin without a rewrite, because the data contracts are the plugin's Surreal schema either way.

## Status

**Settled.** Flue as engine, AYA as envelope, Limpide as a plugin, Tutor/Gardner as separate Flue sessions joined by the Task and the Event Journal, gate evaluators registered with the host.

**Tentative.** The package name. Whether the Gardner loop is a Flue *workflow* (one run per debounce trigger, with run history) or a long-lived Flue *agent session* per tutoring Task (cheaper context reuse, no run history); the choice is an implementation detail behind the plugin's own contract and should be made when AYA's named-loop API is used in anger. Whether the disclosure classifier runs as a Flue workflow or as a plain provider call from the probe tool — a workflow buys observability, a plain call buys latency.

**Open.** Whether Limpide's plugin-private schema uses `RELATE` graph edges for prerequisites and cross-substrate links (natural for the traversal queries) or record links (simpler for the overlay model of ADR-0013). Decide with the first curriculum import.
