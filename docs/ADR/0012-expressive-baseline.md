# ADR-0012: Expressive baseline — measuring understanding relative to expected articulation

**Status:** Proposed
**Date:** 2026-05-29

## Context

Limpide measures understanding *through explanation* (`MEASUREMENT.md`): the student teaches back, and the quality of the teach-back drives confidence. But expressive ability is unevenly distributed and is itself one of the three capabilities Limpide trains (rhetoric, `VISION.md`). A student who understands but cannot yet articulate — a young child, a non-native speaker, a student with a language difference, or simply a shy one — produces weak teach-back signals and is **systematically under-credited**. The measurement would then conflate *can't explain* with *doesn't understand*, and would under-measure exactly the students who most need help. This is a validity threat to the core thesis, not a minor calibration issue.

The motivating example (a parent and a three-year-old): asked why the baby-monitor screen shows a warning, the child explains that the camera is off so the screen's antenna can't reach the camera's antenna. No one expects a three-year-old to explain radio waves. The point is that he understands the camera must be on and that the screen *communicates* with it — and that is discovered understanding *at his level*. Separately, the same child says a swing slows "because gravity, the big mass attracts the small one" and keeps going "because of inertia" — words whose mechanism he does not yet hold. That is not a failure either; it is shape-without-mechanism, the seed of the curiosity that one day becomes "is the big thing attracting the small one, or catching up with it?" — and eventually, Einstein.

A human tutor handles both cases automatically by calibrating expectation to the learner. The system has no such calibration today; Gardner judges teach-back against an implicit absolute standard.

## Decision

Understanding is judged **relative to an expected level of expression**, never against an absolute or adult-expert rubric.

**An expressive baseline enters the student model.** It estimates the articulation level expected of this student, anchored initially by developmental stage / grade and refined from observed language over time. It may differ by substrate (a student may express mathematical and narrative reasoning differently). It is stored with its source (declared vs. inferred) so its own reliability can be tracked.

**Gardner evaluates teach-back relative to the baseline.** The same explanation maps to different outcomes depending on the expected level: "the camera must be on and the screen talks to it" is `discovered` for a three-year-old and `partial` for a fifteen-year-old. The baseline sets the *bar for what counts as adequate articulation*; it does not change what the concept is.

**Conceptual grasp is decoupled from expressive fluency, and recovery-under-probing is the discriminator.** The machinery already built does the real work of telling "understands but can't say it" from "doesn't understand": probe the glossed step (ADR-0009); if the student reconstructs it — even haltingly, even searching for words — that is understanding; if they cannot, that is a gap, regardless of eloquence. The baseline adjusts the *expectation of expression*; recovery-under-probing adjudicates the *grasp*. Halting expression that recovers under a probe is credited; fluent expression that collapses under a probe is not.

**Productive partial understanding is a curiosity asset, not a deficit.** Shape-without-mechanism ("there's something attracting") is recorded as a `CuriositySignal` (the system already has this object) and as a positive trajectory marker, not a low score. It is the entry point to the depth path, and the system treats it as such.

These three axes are distinct and must not be conflated — conflating them is the trap:

| Axis | Question | Set by | ADR |
|---|---|---|---|
| **Depth** | how deep must the understanding go? | program (floor) + student (ceiling) | 0010 |
| **Expression** | how fluently do we expect it said? | developmental level | 0012 (this) |
| **Disclosure** | how much does a probe reveal? | the probe itself | 0009 |

A curious three-year-old has low required depth *and* low expressive baseline — adjusted down independently. A gifted but non-native fifteen-year-old has high required depth *and* low expressive baseline — and that divergence is the whole point: their language struggle must not be read as conceptual weakness. The expressive baseline therefore **never caps depth** — a student who expresses haltingly can still be taken as deep as their `desiredLevel` and forks allow (ADR-0010). Expression adjusts expectation, not permission.

This is added as **Foundation rule F11** (`PEDAGOGY.md`): *judge understanding relative to expected expression; never penalize articulation for its own sake.*

## Alternatives considered

**A single absolute teach-back rubric for all students.** Simplest. Rejected — it is the validity threat itself, under-measuring every expressively weaker student and mistaking eloquence for grasp.

**Lower the bar uniformly by age.** Coarse age-banding instead of a per-student baseline. Rejected — it misses the non-native or language-different student *within* an age band, and the gifted-but-inarticulate student whose depth and expression diverge. The baseline must be per-student and ideally per-substrate.

**Model expression by lowering the required depth rung (ADR-0010).** Treat "young" as "shallow." Rejected — depth and expression are orthogonal. A three-year-old can pursue a concept deep via curiosity while still expressing haltingly; an eloquent teenager can be shallow. Collapsing the two axes destroys exactly the distinction this ADR exists to protect.

## Consequences

**Commits us to:**
- An `expressiveBaseline` on the student model, anchored by developmental stage and refined from observation, possibly per-substrate.
- Gardner's teach-back evaluation being calibrated to the baseline, with recovery-under-probing as the grasp discriminator (the architectural hook already exists via ADR-0009).
- Recording shape-without-mechanism as a curiosity signal and positive trajectory marker.
- Foundation rule F11 and its Policy-Gate encoding.

**Precludes:**
- A universal teach-back rubric, and any path where halting articulation alone lowers confidence.
- The expressive baseline acting as a ceiling on depth.

**Opens:**
- How the baseline is initialized (declared grade/stage vs. cold observation) and how fast it updates.
- Whether it is genuinely per-substrate, and how to set it for students with language differences **without stereotyping** — a fairness concern that echoes the disclosure-classifier calibration problem (ADR-0011): the baseline is itself an estimate and can be biased, so it needs its own validation.
- Whether the baseline should ever be visible to the student or only internal (F3 instincts suggest internal).

## Status

**Settled.** Understanding judged relative to expected expression, not absolute. The decoupling of grasp from fluency with recovery-under-probing as discriminator. Partial understanding as a curiosity asset. The three orthogonal axes (depth / expression / disclosure). Foundation rule F11. The baseline never caps depth.

**Tentative.** The representation of `expressiveBaseline` (scalar vs. per-substrate), its initialization, and its update rule. Whether F11 needs a distinct Policy-Gate encoding or rides on the existing teach-back evaluation gate.

**Open.** Bias validation of the baseline (it is an estimate that could systematically mis-set expectations for some groups). Per-substrate baselines. Baseline visibility to the student.
