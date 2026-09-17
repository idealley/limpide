# ADR-0018: The confidence model

**Status:** Proposed
**Date:** 2026-05-29

## Context

Confidence is the most load-bearing quantity in the system. It drives which concept the Tutor selects next (the Select state picks the lowest-confidence node whose prerequisites are solid), whether a concept counts as "done," what the institution sees as progress, and what decays over time. Yet it is currently a single scalar (`confidence: number` on `StudentConcept`) with a first-cut delta table that explicitly says "reopening this requires an ADR" (`ARCHITECTURE.md`). This is that ADR.

Several decisions already constrain it:

- **`MEASUREMENT.md`** establishes that the *history of how confidence was earned is the measurement, and the number is a lossy summary*; that understanding is process, not state; and that the signal must never be placed on the Tutor's optimisation loop (Goodhart).
- **ADR-0010** introduced the per-concept understanding ladder and flagged per-rung confidence as open.
- **ADR-0012** requires that the outcome be judged relative to the student's expressive baseline, with recovery-under-probing as the grasp discriminator.
- **Foundation rules** F10 (no rise without teach-back) and F2 (discovered outweighs delivered) bound the update.

The motivating failure of the scalar: a child who is sure "something attracts the swing" but blank on the mechanism has high confidence at a shallow rung and none at a deep one. A single number smears those into a misleading middle.

## Decision

**1. Confidence is per-rung, not scalar.** It is a value per ladder rung (ADR-0010), not one number per concept. "Done for a program" means the student is at or above the program's required rung with adequate confidence *at that rung*. This resolves the smearing and makes depth and mastery legible together.

**2. Confidence is derived from encounter history, never primary.** The encounter log is the source of truth; confidence is a cached, always-recomputable summary of it. It is never written directly. This is what keeps it from becoming a gameable state metric and keeps every value auditable back to the encounters that produced it — the `MEASUREMENT.md` commitment, made structural.

**3. Each rung carries its credibility.** A rung's confidence is reported with how much evidence backs it (an observation weight; a Beta-style estimate is the natural form), so "0.6 from one encounter" is distinguished from "0.6 from ten." The Select state reads both the estimate and its credibility, so a single lucky discovery is not mistaken for solidity.

**4. The update function**, per rung:
- The evidence from an encounter is its outcome class — discovered / partial / delivered / forced / retreated — scaled by mode (exam-driven multiplies positive evidence by 0.7), with discovered outweighing delivered (F2).
- The outcome is classified **relative to the expressive baseline** (ADR-0012): a young or non-native student's halting-but-recovered explanation is classified as discovered, because recovery-under-probing — not eloquence — is the grasp signal. The baseline shapes the *classification*, not a separate multiplier.
- No rung's confidence rises without a teach-back (F10).

**5. Decay encodes the thesis.** Untouched confidence decays — and *discovered understanding decays more slowly than delivered*, because discovered understanding "survives forgetting" and delivered does not (`VISION.md`). The decay rate is a function of how the rung's confidence was earned. This writes the platform's central distinction directly into the math, rather than treating all confidence as equally durable.

**6. Confidence is not on the optimisation loop.** It is measured and used for selection and reporting; the Tutor is never rewarded for raising it (`MEASUREMENT.md`). Reaffirmed here because it is the quantity most tempting to optimise.

**7. Prerequisite readiness stays separate.** The Select state's "prerequisites sufficiently solid" check reads prerequisite confidence, but confidence does **not** propagate through the graph as a derived value. Readiness is a separate gate, not a folded-in term — keeping the two decoupled avoids a fragile propagation model where one shaky estimate cascades.

## Alternatives considered

**Keep the scalar.** One number per concept. Rejected — it smears rungs (the swing example) and is inconsistent with the ladder ADR-0010 already introduced.

**Confidence as primary stored state.** Write and trust the number directly. Rejected — it becomes a gameable state metric and loses auditability; the encounter history must remain the truth, with confidence derived.

**Point estimate without credibility.** A bare number per rung. Rejected — it cannot tell a lucky single success from established understanding, which the Select state needs.

**Uniform decay.** All confidence forgets at one rate. Rejected — it erases the discovered/delivered distinction that is the whole thesis; durability *is* the difference between the two.

**Propagate confidence through prerequisites.** Derive a concept's confidence partly from its prereqs'. Rejected for now — fragile and hard to reason about; the separate readiness check is sufficient.

## Consequences

**Commits us to:**
- Per-rung, observation-weighted confidence, derived from the encounter log and never written directly.
- An update keyed on outcome class × mode, classified relative to the expressive baseline, gated by F10 and weighted by F2.
- Differential decay: discovered slower than delivered.
- Confidence kept off the optimisation loop; prerequisite readiness kept as a separate gate.

**Precludes:**
- A scalar confidence; directly-written confidence; uniform decay; treating confidence as a comparative cross-student score (it is exposed only formatively — ADR-0015).

**Opens:**
- The exact statistical form (Beta vs. alternatives) and whether credibility itself decays as evidence ages.
- The decay half-lives per outcome class (the discovered-vs-delivered durability ratio).
- How per-rung confidence aggregates into a single concept-level view for operator displays without reintroducing the smearing.
- Cross-student normalisation given that confidence is baseline-relative (ties to ADRs 0011 and 0016 calibration, and the acceptable-use limits of ADR-0015).

## Status

**Settled.** Per-rung, not scalar. Derived from encounter history, never primary. Observation-weighted credibility. Outcome × mode update, classified relative to the expressive baseline, F10/F2-bound. Differential decay (discovered slower than delivered). Not on the optimisation loop. Prerequisite readiness as a separate gate.

**Tentative.** The statistical form, the per-outcome decay constants, and the per-rung-to-concept aggregation for display. The first-cut delta table in `ARCHITECTURE.md` is retained as the starting magnitudes, now interpreted per rung.

**Open.** Whether credibility decays with evidence age. Cross-student normalisation under baseline-relative confidence.

## Amendment — 2026-09-10

[ADR-0021](0021-validate-tutoring-first.md) takes precedence over conflicting commitments above; the original text is retained as decision history.

- **Unchanged.** Per-rung confidence derived from encounter history, with observation-weighted credibility; F10; kept off the optimisation loop; prerequisite readiness as a separate gate.
- **Update weights are provisional (decision 4).** Amended F2 distinguishes independent demonstration from repetition of supplied content; it does not give discovery an intrinsic bonus, and a learner can demonstrate understanding after direct explanation. The exam-mode 0.7 multiplier and the discovered-over-delivered weighting are hypotheses to compare against alternatives, including no mode discount.
- **Differential decay is a hypothesis (decision 5).** The rejection of uniform decay is withdrawn: equal decay is a legitimate comparison, and any durability difference must be estimated from delayed independent evidence rather than written into the formula.
- **Baseline-relative classification needs validation.** See the ADR-0012 amendment.
- **Confidence is not a calibrated probability or efficacy statistic** until validated; it serves tentative selection and formative reporting.
