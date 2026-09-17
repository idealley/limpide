# ADR-0009: Probe gating — deterministic policy over learned strategy

**Status:** Proposed
**Date:** 2026-05-29

## Context

`MEASUREMENT.md` establishes that understanding is measurable only through process, and names four teach-back signals — iteration delta, recovery under probing, self-noticing latency, and unprompted transfer. The strongest of these, recovery under probing, carries an unsolved problem: **attribution.** When a student recovers after a Tutor probe, how much of the recovery belongs to the student and how much to the probe? A probe that opens a question ("why does that have to be true?") and a probe that hands over the answer ("doesn't multiplying by –1 twice land you back where you started?") can share an identical *intent* and produce opposite attributions. Without separating these, every downstream confidence number — and the institutional efficacy evidence in `STRATEGY.md` that is built from those numbers — is inflated by the system's own prompting.

A second, related need: the depth to which the Tutor pursues "why" must terminate. Unbounded Socratic recursion bottoms out in foundations for every concept (why 1+1=2 → Peano → philosophy of mathematics) and is unusable. The recursion needs a floor, and the natural floor is the depth the student's *program* requires for the concept — with the existing fork mechanism (`PEDAGOGY.md`) as the student's opt-in to go deeper.

A third need: we want the system to improve its probing over time (self-learning) without letting that learning erode the Foundation rules. The failure mode this whole project pushes against (the Klarna shape) reappears here: if the component being measured can also move its own guardrail, it will optimize the guardrail away.

The Policy Gate already exists (`ARCHITECTURE.md`) as deterministic ALLOW / DENY / REQUIRE_REVIEW gates over tool calls, outbound events, and state transitions, reading the Foundation rules from the Foundation primitive. Today it gates state transitions (F10 denies Apply→Update without a teach-back). It does not yet gate the *content of what the Tutor asks the student.*

## Decision

We introduce a **probe object** and a six-step gated pipeline for every diagnostic Tutor utterance. Conversational and bridging utterances are not probes and do not enter the pipeline; only diagnostic moves (the Teach-back probes, "push one step further," mirror-don't-correct) do.

A probe carries a student-facing part and internal annotations:

1. **`utterance`** — what the student sees or hears. Visibility `public`.
2. **`intent`** — what the probe is trying to open, declared by the Tutor at probe time. Visibility `internal`. Cheap and honest because the Tutor knows its own aim; near-impossible to reconstruct post hoc from the utterance alone.
3. **`disclosure`** — how much of the answer the utterance reveals, assigned by a separate classifier (see below), not by the Tutor.

The pipeline:

1. **Tutor proposes** the probe with its declared `intent`.
2. **Disclosure classifier** — a fast, inline component with Gardner-like understanding but distinct from Gardner — reads the `utterance` and assigns a structured `disclosure` label (`open | hint | leading | near_answer`) plus a free-text reason. The classifier is **program-agnostic**: it judges only the intrinsic revealing-power of the words, not whether the probe is appropriate for this student or program. This makes it a pure function of the text — testable in isolation, reusable across every program and student, small and cheap.
3. **Policy Gate** evaluates the labeled probe deterministically against program graph and student parameters: `ALLOW | DENY | REQUIRE_REVIEW`. This is the single place where context-dependent judgment lives — program-required depth, session mode, student level, the F1 not-yet-noticed-gap rule. The gate keys off **structured fields only** (the `disclosure` enum, recursion depth, mode, student params); the classifier's free-text reason is audit metadata and never an input to the decision, or determinism and auditability are lost.
4. **Tutor reworks** on `DENY`, seeing the denial reason, and regenerates — exactly as it already handles the F10 transition denial.
5. **Gardner reviews** the full 1–4 record as part of its normal analytic pass: it performs attribution (recovery *from* an opening probe vs. repetition *of* a leaked answer), updates confidence accordingly, and **proposes** rule or parameter changes where the data warrants.
6. **Proposals are gated by blast radius** before taking effect. Parameter nudges within hard-coded bounds (patience curves, per-program depth defaults) may auto-apply, logged and reversible. Anything that changes a rule's shape, or touches a Foundation rule, is `REQUIRE_REVIEW` for a human. Gardner proposes; it never enacts changes to the rules themselves. The human review reads *evidence*, not a bare proposal: the regression result and offline replay from the evaluation stack (`docs/ADR/0016-system-evaluation.md`) accompany every proposal.

This yields a four-layer stack, in which learning is confined to the upper layers and can never rewrite the lower ones:

1. **Foundation rules** — hand-written, never learned. The constitution (F1–F10).
2. **Policy (deterministic, e.g. OPA/Rego)** — auditable rules, parameterized. Enforces the constitution plus program/level constraints.
3. **Learned parameters** — values the policy reads (patience curves, depth defaults, the existing `learnedAfterNAttempts` signal), tuned from data but bounded by the rules.
4. **Strategy (the LLM Tutor)** — fully adaptive probe generation, but every output passes the gate before reaching the student.

Program-governed depth is enforced at step 3 as a terminating floor on the why-recursion: the chain bottoms out at the program-required depth for the concept by default; going deeper requires an active fork. The depth floor reads from the curriculum graph and the student's program/level.

## Alternatives considered

**Attribution from transcript analysis alone.** Let Gardner infer, after the fact, how much each probe gave away. Rejected as unreliable: the probe's intent is not recoverable from its surface text, and reconstructing it invites exactly the hindsight bias attribution is meant to remove. Declaring intent at probe time is cheap insurance against this.

**Let the Tutor classify its own disclosure.** Simpler — no separate classifier. Rejected because it reintroduces the Goodhart trap: the agent whose probes are being measured has an incentive to under-report how much it leaked, the same way it would sandbag its own iteration delta. Separation of the classifier from the Tutor preserves the Tutor/Gardner separation-of-powers the system already rests on (ADR-0004, F1, F3).

**Make the classifier part of Gardner.** Reuse the analytic agent. Rejected on latency grounds: disclosure must be labeled *before* the probe reaches the student, on the interactive path, while Gardner is deliberately debounced and off the hot path. Folding classification into Gardner would either block the Tutor or push disclosure labeling to post hoc, defeating the pre-send gate. A distinct, fast, narrow classifier keeps Gardner reflective.

**Let Gardner enact rule changes directly (full self-learning).** Maximally adaptive. Rejected because it wires the learner into its own guardrail — the Klarna failure mode at the meta-level. The propose-vs-enact split with blast-radius review is the safe form of self-learning.

**A non-deterministic (LLM) gate.** Let a model judge appropriateness holistically. Rejected because the gate is the safety layer and must be auditable and reproducible. The soft judgment (what does this probe mean / how much does it reveal) is isolated in the classifier; the gate's decision *given* the label is deterministic.

## Consequences

**Commits us to:**
- A new `Probe` entity in the data model and an `evaluateProbe` method on the `Policy` interface (`ARCHITECTURE.md`).
- A third inline model role — the disclosure classifier — distinct from Tutor and Gardner, on the interactive path, small and synchronous (Haiku-class or a fine-tuned small/deterministic classifier as its job is narrow).
- The four-layer policy/learning stack as the discipline for all future self-learning in Limpide, not just probes.
- Logging probe objects with outcomes as the shared substrate for both attribution (MEASUREMENT.md) and Gardner's rule-proposal loop — the same data serves both.

**Precludes:**
- The Tutor self-reporting disclosure, and any path by which learning silently modifies a Foundation rule or a policy rule's shape.
- A gate decision that branches on free text. Structured fields only.

**Opens:**
- The exact representation of "program-required depth" is resolved in ADR-0010: depth is an index into a per-concept understanding ladder, with the program supplying the required rung (floor) and the student an optional desired rung (ceiling), `desiredLevel >= requiredLevel`.
- Classifier reliability and calibration drift — the classifier is now load-bearing for attribution, so its error rate bounds the trustworthiness of every confidence number. The calibration methodology and gold-set construction are specified in ADR-0011.
- The attribution function itself — how `disclosure` + the student's subsequent production combine into a credit assignment, and whether that can be validated against an external ground truth (a delayed re-test, a teacher's independent read) without reintroducing the state-metric failure mode.

## Status

**Settled.** The probe object (intent declared by Tutor, disclosure assigned by a separate classifier). The six-step gated pipeline. The classifier being program-agnostic and distinct from Gardner. The gate keying off structured fields only. The four-layer stack and the propose-vs-enact / blast-radius discipline on rule evolution. Program-governed depth as the terminating floor with fork as the opt-in.

**Tentative.** The disclosure enum values (`open | hint | leading | near_answer`). The classifier's model and whether it is prompted, fine-tuned, or partially deterministic. The blast-radius thresholds that separate auto-applicable parameter nudges from human-review rule changes. The representation of program-required depth.

**Open.** The attribution function and its external validation. Classifier calibration methodology. Whether the gate's own evolution log needs to be surfaced to institutions as part of the efficacy evidence (it is, after all, a record of how the system's pedagogy changed over time).

## Amendment — 2026-09-10

[ADR-0021](0021-validate-tutoring-first.md) takes precedence over conflicting commitments above; the original text is retained as decision history.

- **Measurement premise.** The context's claim that understanding is measurable only through process is superseded: `MEASUREMENT.md` pairs process evidence with delayed independent outcomes, and institutional efficacy reporting is a possible later offering once validity and demand are established.
- **F1 as a gate input (step 3).** The not-yet-noticed-gap rule is replaced by amended F1: prefer probes that let the student notice, with timely explanation or correction when probing is unproductive or help is requested. How the gate encodes this less mechanical rule is open.
- **Attribution.** Probe records and disclosure labels help separate independent reconstruction from repetition after a hint; they do not solve attribution by definition. External validation against delayed unaided work, listed as open above, is required before attribution-based claims.
