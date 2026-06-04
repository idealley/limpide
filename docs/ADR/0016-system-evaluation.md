# ADR-0016: Evaluating the system

**Status:** Proposed
**Date:** 2026-05-29

## Context

The measurement work to date evaluates the *student* (`MEASUREMENT.md`). There is a parallel, unaddressed question: how do we know the *system* is any good — that a Tutor prompt, a Playbook change, or a Gardner-proposed rule (ADR-0009) actually improves outcomes rather than regressing them? Without an answer, the blast-radius human review in ADR-0009 is gut-feel, and "self-learning" cannot be trusted to learn the right thing.

Three properties make this genuinely hard:

1. **The target is process, not answers.** A good Socratic tutor frequently *withholds* the answer; its correct move is often a probe, not a solution. So the system cannot be scored on "did it produce the right answer." Quality is whether the student did the discovering — the same process target, now pointed at the system.
2. **You cannot freely experiment on children.** A/B testing pedagogy on real minors raises the safeguarding and articulation concerns of ADRs 0012 and 0014. No child may be made the *worse-off* control of an experiment.
3. **The evaluator depends on the things being evaluated.** The process signals are produced by the disclosure classifier (ADR-0011) and Gardner's judgments, which are themselves estimates. "The system improved" can be confounded by "the classifier drifted." The evaluation inherits the calibration problem.

## Decision

A four-layer evaluation stack, from cheapest and most deterministic to most expensive and most real. A change must clear the lower layers before reaching the higher ones.

**1. Foundation-rule and guardrail regression (deterministic, must-pass).** A fixed suite of scenarios with known-correct gate behaviour: a distress disclosure must trip F12; an exam-mode answer request must be delivered and honestly recorded; a leaky probe must be denied; confidence must not rise without teach-back (F10). Any candidate Tutor/Gardner/rule version must pass this suite deterministically before anything else. This is the safety net for the self-learning loop — a Gardner-proposed rule cannot even reach human review until it passes regression.

**2. Offline replay against process signals.** Replay logged sessions (consented, per ADR-0014) against a candidate version and compare on the signals already defined in `MEASUREMENT.md`: did the candidate produce more *discovered* vs *delivered*, lower disclosure on its probes, faster self-noticing, more recovery-under-probing — without violating Foundation rules? The student-measurement signals double as the system's evaluation metric. This is the primary quality measure.

**3. Simulated-student pre-screening.** For changes with no historical analogue to replay, dry-run them against simulated students — LLM learners role-playing a level and a gap, the same construction used for synthetic probe grounding (ADR-0011) and as in corpora like MathDial. Simulated students are *pre-screening only*, never ground truth: they share the synthetic-data limitation (distribution gap from real learners), so they catch regressions cheaply but do not certify a change.

**4. Online evaluation, ethically bounded.** Only after a change clears layers 1–3. Constraints: no child is given a known-worse experience as a control; prefer shadow evaluation, holdouts, and gradual rollout over hard A/B; the process signals are the outcome metric, held to the Goodhart discipline (the proxy is measured, never directly optimised — `MEASUREMENT.md`); changes roll back on regression against the signals or any Foundation-rule violation.

**Calibration tracked as a confound throughout.** Because layers 2–4 read signals produced by the classifier and Gardner, every evaluation result is reported alongside the classifier's calibration state (ADR-0011). A claimed improvement that coincides with classifier drift is treated as unproven until the drift is ruled out. This is the meta-evaluation discipline: validate the evaluator before trusting its verdict.

**The stack is the evidence gate for the self-learning loop.** ADR-0009 lets Gardner *propose* rule changes and routes shape-changing proposals to human `REQUIRE_REVIEW`. This ADR supplies what that review reads: a proposal arrives with its regression result (layer 1) and an offline-replay showing it would have improved process signals on historical sessions without violating Foundation rules (layer 2). The human reviews *evidence*, not a bare proposal — which is what closes the "blast-radius review is gut-feel" gap.

Versioning already supports this: the Playbook is versioned and encounters log which version they ran under (`ARCHITECTURE.md`), so outcome shifts can be attributed to specific changes.

## Alternatives considered

**Optimise a single system-quality score.** One number to rank versions. Rejected — no scalar captures Socratic quality, and putting a score on the optimiser is the Goodhart failure the project exists to avoid.

**Free A/B testing on students.** The standard product approach. Rejected — it requires a worse-off control made of children, which the safeguarding posture forbids; holdouts and gradual rollout get most of the signal without that.

**Human expert rating only.** Have educators score sessions. Rejected as the *primary* method — it does not scale and is itself uncalibrated — though human spot-checks remain part of the loop, as in ADR-0011.

**Answer-accuracy metrics.** Score whether the tutor produces correct solutions. Rejected — the tutor's job is not to answer; this measures the opposite of what Limpide values.

## Consequences

**Commits us to:**
- A maintained regression suite of Foundation-rule and guardrail scenarios as a hard gate.
- An offline replay harness scored on the `MEASUREMENT.md` process signals.
- A simulated-student rig for pre-screening, explicitly non-authoritative.
- Ethically-bounded online evaluation (holdout/gradual rollout, no worse-off child control, roll back on regression).
- Reporting evaluation results against classifier calibration state, and using layers 1–2 as the evidence packet for ADR-0009's blast-radius review.

**Precludes:**
- Shipping any pedagogy, prompt, or rule change unvalidated.
- Making a child the degraded arm of an experiment.
- Trusting a measured improvement that coincides with unverified classifier drift.

**Opens:**
- Simulated-student fidelity — how close is close enough to be useful for pre-screening.
- Detecting classifier drift versus real improvement when the two move together.
- The specific holdout design that is both informative and ethical.
- The bar: what level of offline-replay improvement, at what confidence, justifies an online trial or a rule adoption.

## Status

**Settled.** The four-layer stack and its ordering. Process signals (not answer-accuracy, not a single score) as the quality measure. No worse-off child control. Calibration tracked as a confound. The harness as the evidence gate for the self-learning loop.

**Tentative.** The contents of the regression suite. The replay metric weighting. The simulated-student construction. The roll-out/roll-back thresholds.

**Open.** Simulated-student fidelity validation. Drift-vs-improvement disambiguation. The ethical holdout design. The adoption bar for layer-2 evidence.
