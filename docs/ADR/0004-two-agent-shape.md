# ADR-0004: Tutor and Gardner as two agents on a shared session

**Status:** Accepted
**Date:** 2026-04-29

## Context

Limpide needs to do two things in parallel during a session:

1. **Conversational reasoning** — drive the session loop, ask Socratic questions, judge whether to bridge or deliver, hold the floor with the student.
2. **Analytic reasoning** — read the rolling transcript, extract gaps, classify curiosity signals, evaluate teach-back quality, recognize cross-substrate moments.

These have different latency requirements (interactive vs. debounced), benefit from different models (strong reasoning vs. structured extraction), and must not interfere with each other's outputs. Critically, the analytic output must never reach the student — Foundation rule F3.

agent-core today is single-agent: an `Agent` instance owns its session.

## Decision

Limpide attaches two agents to a single shared session: a **Tutor** on the interactive lane and a **Gardner** on the analytic lane. They share the event stream and the memory handle but have separate identities, system prompts, tool sets, and LLM clients.

agent-core gains a new abstraction: `Session` becomes a first-class object that agents *attach to* rather than own. The exact interface is in `ARCHITECTURE.md`.

The Tutor holds the conversational *floor* by default; Gardner never requests the floor. Visibility flags on emitted events (`public` / `internal` / `agent-only`) prevent Gardner's analysis from reaching the student through the channel adapter — this is enforced at the dispatch layer, not in agent prompts.

Gardner runs on a debounce: it accumulates events and reads the rolling window when triggered by either an event-count threshold, a latency threshold, or an explicit `flush` call from the Playbook before a state transition that needs the analysis (Teach-back evaluation, end-of-session Update).

## Alternatives considered

**One agent with two modes.** Single Agent that alternates between "be the Tutor" and "be the Gardner." Rejected because it loses parallelism (Gardner can't analyze while Tutor is conversing), conflates concerns the prompt would have to police, and forces a single model to do both jobs (which means picking either an over-spec'd model for Gardner's structured extraction or an under-spec'd model for Tutor's reasoning).

**Two completely independent agents with no shared session.** Each runs its own instance of the framework, communicating via SurrealDB. Rejected because the synchronization seams are exactly the kind of thing the framework should hide. Sharing a session at the framework level keeps the abstraction clean.

**A specialized "AgentEnsemble" abstraction now.** A first-class concept in agent-core for multi-agent collaborations. Rejected as premature. The shared-Session model handles the two-agent case cleanly, and we'll learn what the right ensemble abstraction looks like by building more multi-agent Skills first. Two-Agents-on-Session is the pragmatic path; AgentEnsemble can come later if we discover we need it.

## Consequences

**Commits us to:**
- The shared-Session abstraction as the agent-core extension. Other Skills that need multi-agent coordination will use the same mechanism.
- Heterogeneous models: Claude Opus for Tutor (strong reasoning), Claude Haiku for Gardner (structured extraction). agent-core's per-agent LLM client supports this.
- The visibility-flag mechanism on events as the structural enforcement of "Gardner stays internal."

**Precludes:**
- A simpler single-agent loop. We have two, and the orchestration cost is real.
- Deep coupling between the two agents — they communicate only through the event stream and the memory layer. This is intentional; the loose coupling is what makes the parallelism work.

**Opens:**
- Future Skills with multiple agents. A research assistant could be a writer + critic pair. A debate tutor could be two perspectives plus a moderator. The shared-Session abstraction handles them all.
- A cleaner story for "what does this agent know, when?" — the event stream is the source of truth, and each agent reads what it has subscribed to.
- Better debuggability: every state transition is an event, every gap is an event, every cross-substrate recognition is an event. Time-travel debugging on a session is real.

## Status

**Settled** on the two-agent shape and the shared-Session abstraction.

**Tentative** on the specific debounce parameters for Gardner (currently: 4 events or 8 seconds, whichever first; coalesce repeated triggers). These will be tuned as real sessions run.

**Open** on whether a future AgentEnsemble abstraction in agent-core would supersede the shared-Session model. The current model is sufficient and we'll let the abstraction emerge from real multi-agent use cases rather than designing it speculatively.
