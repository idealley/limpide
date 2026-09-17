# Measurement

Limpide aims to develop understanding that students can explain, apply, and revisit independently. Explanation trajectories are useful evidence about that process. They are neither the only possible evidence nor sufficient by themselves to establish durable understanding or educational efficacy. [ADR-0021](ADR/0021-validate-tutoring-first.md) supersedes the earlier process-only measurement claim.

## Process and independent outcomes

A completion count or a student's “got it” cannot establish understanding. A process record is richer: it preserves the attempt, support, revision, and application. But a fluent conversation can still reflect rehearsal, leading questions, task familiarity, or evaluator error.

Use two complementary views: what happens during tutoring, and what the learner can do later without it. A delayed explanation or unfamiliar application can supply evidence the original conversation cannot. Keep the reasoning behind judgments and the uncertainty, rather than treating any single score as truth.

## Candidate process signals

**Iteration delta.** Does the next explanation resolve the specific gap in the earlier attempt? Record what support preceded the change. Improvement after the tutor supplied a step differs from independent reconstruction, and neither alone proves retention.

**Recovery under probing.** Can the learner respond to a follow-up, boundary case, or counterexample? Unexpected probes can reveal weaknesses in rehearsed explanations. They can also be leading, ambiguous, or overly demanding in language. Recovery is evidence to calibrate, not near-proof of understanding.

**Self-noticing.** Does the learner identify and repair a gap before the tutor names it? Repeated observations may reveal metacognitive progress. Task difficulty, familiarity with the interaction, and the tutor's timing affect the result; a raw latency is not a comparable ability score.

**Unprompted transfer.** Does the learner apply a principle in a different context without being led? Record the context and whether the connection is substantively valid. Mentioning Euclid during an argument is a candidate connection, not sufficient evidence of transferable reasoning. Far transfer across mathematics, reading, and rhetoric remains a hypothesis to test.

Gardner can propose these annotations from transcripts. Its judgments require calibration and educator review; visibility restrictions do not make a metric valid or immune to gaming.

## Independent validation

For a bounded pilot, define outcomes and rubrics in advance. Use matched but distinct baseline and later tasks, including an unaided revisit about a week later and an unfamiliar application of the same concept. Assess explanation and reasoning as well as correctness. Where feasible, educators judge work without knowing the tutor's score or the teaching route.

Keep assessment items separate from the practice bank. Record support, repeated-item exposure, task versions, and why tasks were selected. A drop in selected difficulty must not masquerade as progress. Explanations can use diagrams, calculations, and brief language where appropriate; the assessment should not primarily test eloquence.

Compare internal judgments with independent outcomes, including disagreements and uncertainty. A student who was confidently classified as understanding but cannot later apply the concept is useful evidence for revising the model.

A small before-and-after pilot supports feasibility and directional learning observations. It does not isolate Limpide's causal effect from classroom teaching, selection, or other practice. Claims of efficacy need an appropriate comparison and study design. Institutional efficacy requires additional evidence; aggregate trajectories alone cannot attribute outcomes to a school.

## Confidence and instructional history

Confidence remains a per-rung, history-derived estimate with evidence weight (ADR-0018, amended by ADR-0021). Preserve attempts, support, task difficulty, timing, and uncertainty so it can be recomputed.

Discovered, partial, delivered, and forced describe the encounter and teaching route. Independent demonstration is stronger evidence than repeating supplied content. A learner can demonstrate understanding after direct explanation; the teaching route must not determine the verdict in advance.

The numerical deltas in `ARCHITECTURE.md`, the exam-mode multiplier, and different decay rates are provisional model assumptions. Do not claim that discovered understanding is more durable because a formula makes it decay more slowly. Estimate or revise such differences against delayed evidence; equal decay must remain a legitimate comparison. Confidence is for tentative selection and formative reporting, not a calibrated probability or efficacy statistic until validated.

F10 still prevents a confidence increase from “got it” alone. The learner must demonstrate reasoning; the tutor's explanation or a multiple-choice selection without supporting reasoning is insufficient. Initial placement can leave concepts unknown and revise estimates later (`BOOTSTRAP-PILOT.md`).

## Goodhart discipline and attribution

Do not reward the Tutor for maximizing a process proxy. Optimizing iteration delta could encourage a weak initial attempt; optimizing recovery could encourage easy or leading probes. Selecting prompts or versions using these same metrics is also an optimization pathway, even without a turn-by-turn reward. Independent outcomes and human review must check that pathway.

Attribute support explicitly. The disclosure classifier and probe records (ADRs 0009 and 0011) can help distinguish independent reconstruction from repetition after a hint. They do not solve attribution by definition. A classifier's “open” label is itself an estimate, and correctness depends on subject context and the learner's prior knowledge.

Keep student-facing feedback useful and avoid score-chasing. Be transparent about assessment purposes and data access. Keeping a score hidden is not a substitute for validation or consent. Teacher-facing views remain formative under ADR-0015.

## Expression and other confounds

The expressive baseline (ADR-0012) is a proposed mitigation, not a demonstrated resolution of the articulation confound. A learner may understand but struggle to explain; probing can also impose language demands. Use alternative representations and educator review, and retain uncertainty when evidence is insufficient.

Keep depth, expression, and disclosure separate. In physics, distinguish mathematical errors from physical misconceptions. In philosophy, assess interpretation and argument quality while allowing defensible disagreement; eloquence or agreement with the tutor is not understanding.

## What the pilot data improves

Repeated exercises can refine an individual learner map, expose ambiguous items, suggest misconceptions, and inform teaching choices. Official curriculum material seeds the shared structure before learner data exists; extraction and LLM-assisted decomposition produce traceable draft mappings and prerequisites for review. Teachers validate inferred relationships and candidate changes. School context and recent assignments locate the starting topic; diagnostic attempts supply evidence about understanding. Correlated mistakes do not establish prerequisite edges, and adapting instruction around a proposed edge does not independently validate it.

The proposed daily-practice experiment and its minimal observations are in `BOOTSTRAP-PILOT.md`. Shared curriculum improvement needs appropriate authorization; learner participation is not automatic permission for research, model training, or network contribution.

## Status

**Settled.** Preserve process evidence and uncertainty; distinguish support from independent demonstration; require independent validation for efficacy claims; protect expression differences; avoid direct proxy optimization; formative use and existing data boundaries.

**Tentative.** The four process signals, their operational definitions, attribution method, expressive-baseline correction, confidence weights/decay, and placement estimates.

**Open.** Educator agreement, external predictive validity, a suitable comparison design, cross-context comparability, and whether process reporting changes teaching enough to justify a paid product.
