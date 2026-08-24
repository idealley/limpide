# ADR-0004: Tutor and Gardner as two agents on a shared session

**Status:** Accepted; amended by ADR-0019 (2026-08-24)
**Date:** 2026-04-29 (amended 2026-08-24)

> **Amendment note.** The *decision* — two agents with separate identities, models, tool sets, and latency profiles, joined by a shared event stream, with Gardner structurally unable to reach the student — stands. The *mechanism* was originally an agent-core extension (a first-class `Session` that agents attach to). agent-core is retired (ADR-0019); the mechanism is now expressed over Flue sessions and AYA's Task + Event Journal. The Decision section below is rewritten accordingly; the original agent-core wording is preserved in git history.

## Context

Limpide needs to do two things in parallel during a session:

1. **Conversational reasoning** — drive the session loop, ask Socratic questions, judge whether to bridge or deliver, hold the floor with the student.
2. **Analytic reasoning** — read the rolling transcript, extract gaps, classify curiosity signals, evaluate teach-back quality, recognize cross-substrate moments.

These have different latency requirements (interactive vs. debounced), benefit from different models (strong reasoning vs. structured extraction), and must not interfere with each other's outputs. Critically, the analytic output must never reach the student — Foundation rule F3.

Flue's unit of execution is a session that runs one operation at a time; a Flue *subagent* runs in its own session but returns its result into the parent's transcript. Neither, on its own, is the shape Limpide needs: the two agents must run concurrently, and one of them must never speak into the other's transcript.

## Decision

Limpide runs two Flue agents against one AYA tutoring Task: a **Tutor** on the interactive lane and a **Gardner** on the analytic lane. They share the Task's Event Journal and the Limpide repositories but have separate Flue sessions, identities, system prompts, tool sets, and models.

- **The shared session is the AYA Task plus its Event Journal**, not a framework object. Every student turn, Tutor turn, state transition, probe, gap, and evaluation is a journal event on the Task. Both agents read from it; each writes to it with its own role.
- **Tutor** is a Flue agent with a persistent session per tutoring Task, admitted through the refs-only Dispatch adapter on each student message (ADR-0019 §2). It holds the conversational *floor* by construction: only the Tutor session's output is landed as a `public` message. It never delegates to Gardner as a subagent.
- **Gardner** is a Flue agent run by the worker as a named loop, triggered on a debounce — an event-count threshold, a latency threshold, or an explicit `flush` from the Playbook before a state transition that needs the analysis (Teach-back evaluation, end-of-session Update). It reads the rolling window of the journal and writes gap records, curiosity signals, teach-back evaluations, and probe attributions through the Limpide repositories. Its journal events carry `visibility: 'internal'`.
- **Visibility is enforced where events are landed**, not in prompts: the worker lands `public` events as messages/renderables; `internal` and `agent-only` events never leave the journal. This is the structural enforcement of F3.
- **`awaitLane('analytic')`** — the Playbook's way of waiting for Gardner without polling — is implemented as "wait until the Gardner loop has drained the journal for this Task up to the current sequence number". The Playbook then reads the latest gap records and decides the transition, with the Policy Gate's transition evaluator (F10) having the last word.

## Alternatives considered

**One agent with two modes.** Single agent that alternates between "be the Tutor" and "be the Gardner." Rejected because it loses parallelism (Gardner can't analyze while Tutor is conversing), conflates concerns the prompt would have to police, and forces a single model to do both jobs (which means picking either an over-spec'd model for Gardner's structured extraction or an under-spec'd model for Tutor's reasoning).

**Gardner as a Flue subagent of the Tutor** (`defineAgentProfile` + `session.task`). The idiomatic Flue delegation shape, and rejected for exactly that reason: a subagent's result returns into the parent's transcript, which is the F3 leak; and the subagent runs when the parent decides, which ties Gardner's cadence to the Tutor's turn instead of to the debounce.

**Two completely independent processes communicating only through SurrealDB.** Rejected because the synchronisation seams (drain, flush, sequence numbers) are exactly what the worker's lease queue and the journal already provide; a second process adds an operational surface without adding isolation the journal doesn't already give.

**A specialized "AgentEnsemble" abstraction now.** Rejected as premature, as before. Two sessions on one Task is the pragmatic path; if a second ensemble plugin (writer + critic, debate pair + moderator) needs the same thing, the pattern can be lifted into a shared helper then.

## Consequences

**Commits us to:**
- Two Flue sessions per active tutoring Task, with the Task id as the join key and the journal as the only channel between them.
- Heterogeneous models: Opus-class for Tutor (strong reasoning), Haiku-class for Gardner (structured extraction, with a `result` schema). Flue's per-agent `model` supports this directly.
- The visibility field on Limpide's journal events, and its enforcement at the landing step in the worker.

**Precludes:**
- A simpler single-agent loop. We have two, and the orchestration cost is real.
- Deep coupling between the two agents — they communicate only through the journal and the repositories. This is intentional; the loose coupling is what makes the parallelism work.

**Opens:**
- Future ensemble plugins with multiple agents on one Task. A research assistant could be a writer + critic pair. A debate tutor could be two perspectives plus a moderator. The Task-plus-journal join handles them all; a floor with explicit handoffs is a small addition when a second *speaking* agent appears.
- A cleaner story for "what does this agent know, when?" — the journal is the source of truth, and each agent reads the window it subscribed to.
- Better debuggability: every state transition is an event, every gap is an event, every cross-substrate recognition is an event; Flue's OTel traces attach to each session. Time-travel debugging on a session is real.

## Status

**Settled** on the two-agent shape, on the Task + Event Journal as the shared session, and on landing-time visibility enforcement.

**Tentative** on the specific debounce parameters for Gardner (currently: 4 events or 8 seconds, whichever first; coalesce repeated triggers). These will be tuned as real sessions run. Also tentative whether Gardner is one long-lived Flue session per Task or one workflow run per trigger (ADR-0019, Tentative).

**Open** on whether a shared ensemble helper should be lifted out of the Limpide plugin once a second multi-agent plugin exists. Let it emerge from real use rather than designing it speculatively.
