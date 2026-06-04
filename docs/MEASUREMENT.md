# Measurement

This document answers a question the rest of the project deliberately leaves open: if Limpide is structurally committed to *understanding* rather than *learning*, and understanding is the thing most ed-tech destroys precisely by trying to measure it, how does Limpide know whether it is working?

The short answer is that the earlier framing — "understanding is unmeasurable" — was imprecise. Understanding is not unmeasurable. It is *measurable only through process*, and process measurement has different properties, and different dangers, than the state measurement that produces the failure mode `VISION.md` describes. This document makes that distinction load-bearing.

## State versus process

The metrics that destroy understanding are **state** metrics. They take a snapshot of what the student holds at a moment in time: concepts mastered, confidence score, percent complete, a number that says *the student understands springs*. State metrics are gameable in a specific and fatal way — the thing being scored is a static claim, and the system cannot distinguish a true claim from a rehearsed one. The student says "got it," the score rises, the next concept loads. This is the Klarna customer-service shape: tickets-resolved was measurable, relationship-quality was not, the measurable proxy was optimized and the unmeasurable thing was destroyed.

The signals teach-back produces are **process** signals. They score a trajectory rather than a state: not *does the student understand* but *how did this student's understanding move over the course of trying to explain it*. A student can assert a state. A student cannot easily fake a process — the shape of how an explanation improves across attempts, how a student responds to a probe they did not anticipate, who noticed the gap first. The architecture already commits to this in passing: `PEDAGOGY.md` says confidence "is not a single number, it's a weighted history of how each encounter went." That sentence is a process-measurement commitment. This document names it and builds on it.

The reframed claim, then:

> Understanding is measurable only through process. Process metrics must be kept honest about Goodhart's law — measured, but never placed on the system's optimization loop.

## The signals teach-back produces

The Feynman move — *explain it to me as if I'm someone who's never seen one* — is not only the pedagogy. It is the assessment, produced as a byproduct of the thing the student already needed. Four signals fall out of it. All four are things Gardner can extract from a transcript; none of them require asking the student to perform for a score.

**1. Iteration delta.** When the student's first explanation hits a wall and they try again, does explanation N+1 actually cover the gap that explanation N exposed? This is the cleanest measure of the core pedagogical claim — that understanding is what survives the attempt to explain it. A student who can re-explain *through* the gap they just hit has internalized something; a student who repeats the same gap in new words has not. The signal is the delta across attempts, not the quality of any single attempt.

**2. Recovery under probing.** This is the strongest signal the system has, for a structural reason: *the student did not author the question.* A volunteered explanation can be rehearsed. A response to an unanticipated probe — the Tutor's "you're saying multiplying two negatives flips the sign, why does that have to be true?" — cannot be. When Gardner surfaces a glossed-over step and the Tutor probes it neutrally (PEDAGOGY, Teach-back behavior 2 and 3), the student either rebuilds from underneath or collapses. Rebuilding under an unforeseen probe is near-proof that the understanding is structural rather than memorized. The probe is pedagogy and assessment in a single act.

**3. Self-noticing latency.** `VISION.md` lists, among the person-shaped outcomes, "notice when their own argument has a gap before someone else points it out." That outcome is directly observable: in any given exchange, who flagged the gap first — the student or the system? A student whose self-noticing latency falls over weeks is developing the metacognitive capability the platform exists to produce. This is measurable without any quiz, and it degrades gracefully (a student who never self-notices is simply at latency = "system always first," not a failure to be scored).

**4. Unprompted transfer.** The cross-substrate moment — the student quoting Euclid while writing an argument, recognizing Socratic structure in a proof — is, per Foundation rule F8, *acknowledged quietly and never praised*. F8 is a behavioral rule for the Tutor; it is not a prohibition on Gardner recording the moment internally. Spontaneous transfer is the highest-value signal in the system because it demonstrates that a capability has detached from the substrate it was trained in. It is rare, it cannot be solicited without destroying it, and its frequency over time is the truest measure of the depth path working.

## What this maps onto

These four signals are not a new measurement system bolted onto the existing one. They are the qualitative account of what the confidence math in `ARCHITECTURE.md` is already trying to capture. The discovered / partial / delivered / forced ladder is a process model wearing the costume of a number:

- *Discovered* (+0.40) is iteration delta closing plus, often, recovery under probing.
- *Partial discovered* (+0.25) is iteration delta closing on the second attempt.
- *Delivered* (+0.15) and *forced* (+0.05) record that no process signal fired — the understanding was given, not moved through.

So the work this document implies is not "add metrics." It is "make sure the confidence deltas are driven by these four signals and nothing else," and "expose the trajectory, not just the running total, to anyone who reads the data." A confidence number that has thrown away the history of how it was earned has collapsed back into a state metric. The history is the measurement; the number is a lossy summary of it.

Foundation rule F10 (confidence cannot rise without teach-back) is the structural guarantee that the state metric can never be written without the process having occurred. F10 is what makes the whole measurement story trustworthy: there is no path in the system by which a score rises that does not pass through a student explaining something.

## The Goodhart discipline

The moment a process metric becomes a target, it begins to rot in process-specific ways. Two dangers are sharp enough to state as rules.

**Do not put the proxy on the optimization loop.** If "iteration delta" becomes the quantity the Tutor is rewarded for maximizing, the system learns to manufacture steep improvement curves — and the cheapest way to manufacture a steep curve is a weak first explanation followed by a strong second one. A Tutor optimizing for measured improvement has an incentive to let the student flail early. The discipline: these signals are *measured and reported*; they are not fed back as a reward the Tutor chases turn-to-turn. The Tutor optimizes for the Foundation rules; the signals observe whether that is working.

**Attribute the recovery.** When measured improvement happens after a probe, part of the credit belongs to the probe, not the student — and a system that does not separate these is partly scoring its own prompting. Gardner must distinguish "the student recovered *from* a probe that opened the question" from "the probe handed the student the answer and they repeated it." Only the first is a signal about the student. This attribution is hard and is the central open problem of this document; getting it wrong inflates every downstream number.

A third, quieter danger: any signal the student learns the system *wants* will be performed rather than produced — the exact reasoning behind F8. This is why self-noticing latency and unprompted transfer are kept internal (F3) and never surfaced to the student as a score to chase. The measurement is honest only as long as it is invisible to the person being measured.

## The articulation confound

Measuring understanding *through explanation* has a built-in hazard: explanation ability is unevenly distributed, and it is itself one of the three capabilities Limpide trains. A student who understands but cannot yet articulate — a young child, a non-native speaker, a student with a language difference, a shy one — throws weak teach-back signals and is systematically under-credited. Left unaddressed, the measurement conflates *can't explain* with *doesn't understand*, and under-measures the very students who most need help. This is a validity threat to the whole thesis, not a calibration detail.

The resolution (specified in `docs/ADR/0012-expressive-baseline.md`): understanding is judged **relative to an expected level of expression**, never against an absolute. An *expressive baseline* on the student model, anchored by developmental stage and refined from observation, sets the bar for what counts as adequate articulation. The same explanation — "the camera must be on, and the screen talks to it" — is `discovered` for a three-year-old and `partial` for a fifteen-year-old.

Crucially, the signals already defined do the discriminating work. **Recovery under probing** separates "understands but can't say it" from "doesn't understand": a student who reconstructs a glossed step under a probe — even haltingly, even searching for words — has the grasp; one who collapses does not, regardless of eloquence. The baseline adjusts the *expectation of expression*; recovery-under-probing adjudicates the *grasp*. And shape-without-mechanism ("there's something attracting") is not a low score — it is a curiosity signal and a positive trajectory marker, the seed of the depth path.

This means three axes must be held separate, because conflating them is the failure mode: **depth** (how deep the understanding must go — ADR-0010), **expression** (how fluently we expect it said — ADR-0012), and **disclosure** (how much a probe reveals — ADR-0009). A gifted non-native student has high depth and low expression, and that divergence is the point: their language struggle must never be read as conceptual weakness, so expression never caps depth.

## Why this matters beyond the individual

These four signals are also the answer to a commercial question, and that is not a coincidence. "Structured, longitudinal evidence of how understanding actually develops" — the thing `STRATEGY.md` identifies as the institutional product — *is* the accumulated record of iteration deltas, recovery rates, self-noticing latencies, and transfer frequencies across a cohort. A school cannot buy this elsewhere because no other system produces it as a byproduct of doing what students already need; everyone else is producing state snapshots. The measurement thesis and the institutional business model are the same fact seen from two angles. The process signals make the depth path real for the student and make the institutional scale sellable to the buyer. See `STRATEGY.md` for that argument; this document is its foundation.

## Status of this document

**Settled.** The state-versus-process distinction. The claim that understanding is measurable only through process. The four signals as the qualitative account of the confidence math. The Goodhart discipline: signals are measured and reported, never placed on the Tutor's optimization loop. The commitment that the history of how confidence was earned is the measurement, and the number is a lossy summary. That understanding is judged relative to an expressive baseline, with grasp (recovery-under-probing) decoupled from fluency, and depth / expression / disclosure held as three separate axes (`docs/ADR/0012-expressive-baseline.md`).

**Tentative.** The relative weighting of the four signals when they conflict (a student with strong recovery under probing but flat iteration delta — which dominates?). Whether self-noticing latency is best expressed as a latency, a rate, or a categorical ladder. The exact mapping from observed signal to confidence delta beyond the first-cut ladder in `ARCHITECTURE.md`.

**Open.** The attribution problem — separating the student's recovery from the system's prompting — now has a proposed mechanism rather than an open void: probe objects that carry a Tutor-declared `intent` and a classifier-assigned `disclosure` level, gated deterministically, with Gardner assigning credit (student / shared / system) on the analytic pass. This resolves the earlier fork (transcript-alone vs. instrument-the-intent) in favor of instrumenting intent at probe time. The design is in `docs/ADR/0009-probe-gating.md`. What remains genuinely open is narrower: the **attribution function** itself — how `disclosure` plus the student's subsequent production combine into a credit assignment; the **calibration of the disclosure classifier**, which is now load-bearing for every confidence number, so its error rate bounds the trustworthiness of the whole measurement layer (the calibration approach and gold-set construction are in `docs/ADR/0011-classifier-calibration.md`); and whether any of these signals can be validated against an external ground truth (a delayed re-test, a teacher's independent assessment) without reintroducing the state-metric failure mode the whole document is built to avoid.
