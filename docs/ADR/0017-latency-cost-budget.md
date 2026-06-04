# ADR-0017: Latency and cost budget for the interactive path

**Status:** Proposed
**Date:** 2026-05-29

## Context

The interactive path has grown to three model roles: the **Tutor** (Claude Opus, strong reasoning) holds the conversation; the **disclosure classifier** (ADR-0009) sits *synchronously* before every diagnostic probe; and **Gardner** (Claude Haiku) runs analytically. Two of these are on or near the hot path. There is no performance or cost budget anywhere, and "real-time conversational tutoring on Opus plus an inline classifier" is an unproven latency story and an unmodelled cost. A tutor that feels sluggish loses the student; a session that is too expensive does not scale.

## Decision

Set explicit budgets and the lane discipline that keeps the path fast.

**Latency.**
- The Tutor streams; the student should see a first token quickly (target on the order of one to two seconds), with the full turn following. Perceived responsiveness comes from streaming, not from waiting for completion.
- The disclosure classifier must fit *inside* the probe-dispatch window — it gates whether a probe is sent, so its latency is added directly to the turn. It must be small and fast (Haiku-class or smaller; a fine-tuned or partially-deterministic classifier is preferred precisely because it removes a slow model call from the hot path — ADR-0011 already floats this).
- Gardner stays **off** the hot path: debounced and analytic (ADR-0004), it never blocks the Tutor's turn. This is existing design; the budget reaffirms it as a hard constraint, not an optimisation.

**Cost.**
- A per-session cost ceiling, with Opus Tutor turns expected to dominate. The heterogeneous model choice (Opus Tutor, Haiku Gardner, Haiku-class classifier — `ARCHITECTURE.md`) is already a cost decision; this ADR makes the budget explicit rather than emergent.
- The depth path (longer sessions, more probes, more forks) is inherently more expensive than exam-mode surface work; the budget is expressed per session-type, not as a flat number.

**Levers, in preference order:** stream first tokens; make the classifier small/deterministic enough to shrink or remove its hot-path call; cache and reuse curriculum-graph content (concepts, ladders, scaffolds are slow-changing); use the smallest model that holds quality for each role; keep Gardner and all background work off the interactive lane.

The numbers here are targets to design against, not measured facts; they are confirmed or revised by load and cost testing before the pilot.

## Alternatives considered

**Measure later.** Ship and profile in production. Rejected — three model roles with two on the hot path is a known, foreseeable risk; targets set now shape the classifier and streaming decisions while they are still cheap to change.

**All-Opus for simplicity.** One strong model for every role. Rejected on cost and latency — Gardner's structured extraction and the classifier's narrow judgment do not need Opus, and putting Opus on the synchronous classifier step would blow the probe-dispatch budget.

**Drop the synchronous classifier; label disclosure post hoc.** Remove the hot-path call entirely. Rejected — disclosure must be known *before* a probe reaches the student for the gate to work (ADR-0009); post-hoc labelling defeats the pre-send gate. The right move is to make the classifier fast, not to move it.

## Consequences

**Commits us to:**
- A streaming Tutor with a first-token latency target.
- A disclosure classifier constrained to fit the probe-dispatch window — a hard input to the ADR-0011 choice of classifier model.
- Gardner and background work off the interactive lane as a budget constraint.
- A per-session-type cost ceiling.

**Precludes:**
- A slow or large model on the synchronous classifier step.
- Blocking the Tutor turn on Gardner.

**Opens:**
- The actual numbers — first-token target, classifier budget, per-session cost ceilings — pending load and cost testing.
- Whether the classifier can be made deterministic enough to remove a model call from the hot path entirely.
- Caching strategy for curriculum content and its invalidation on graph edits.

## Status

**Settled.** That there is a budget; streaming; the classifier must fit the probe-dispatch window; Gardner stays off the hot path; per-session-type cost ceilings; the lever order.

**Tentative.** All specific numbers.

**Open.** Whether a deterministic classifier can remove the hot-path model call. The caching/invalidation design.
