# ADR-0011: Disclosure classifier calibration and gold-set construction

**Status:** Proposed
**Date:** 2026-05-29

## Context

ADR-0009 makes the disclosure classifier load-bearing: it labels every diagnostic probe (`open | hint | leading | near_answer`), the Policy Gate decides on that label, and Gardner attributes recovery credit from it. Its error rate therefore bounds the trustworthiness of every downstream confidence number and, through `STRATEGY.md`, the institutional efficacy evidence built on them. A miscalibrated classifier silently corrupts the whole measurement layer (`MEASUREMENT.md`). So calibration is not an implementation detail; it is a first-class correctness concern.

One reframe makes this tractable. Because the classifier is **program-agnostic** (ADR-0009) — it judges only the intrinsic revealing-power of the probe text, not whether the probe is appropriate for a student — its target is an objective-ish property of text, *not* a proxy for understanding. That means it can be validated against a human gold standard without reintroducing the state-metric failure mode the project exists to avoid. The dangerous thing to ground-truth is the confidence deltas; the safe thing is "did the classifier judge this utterance's disclosure correctly." The classifier is precisely the part we *can* calibrate cleanly, which is a payoff of the program-agnostic decision.

A gold set has to come from somewhere, and the cold-start problem is real: there are no Limpide transcripts before launch. There are public tutoring-dialogue corpora, but they are math-only and use coarser taxonomies than our four-rung scale.

## Decision

### Calibration approach

**A human gold set is the anchor.** A held-out set of probes labeled by educators on the four-rung disclosure scale. Because the scale is ordinal, agreement is scored with **quadratic-weighted kappa**, not raw accuracy — being off by one rung is a minor error, off by three is a serious one, and the metric must reflect that.

**Gardner is the continuous drift monitor, at no extra labeling cost.** Gardner already reviews the probe record with full hindsight (ADR-0009 step 5): it sees the probe *and* what the student did next. It flags outcome-contradictions — e.g. a probe labeled `open` after which the student parroted back near-verbatim what the probe contained almost certainly leaked more than `open`. Accumulated contradictions are a running, label-free miscalibration signal and the trigger for recalibration. The slow reflective agent audits the fast inline one — the same separation of powers the system already runs on.

**Errors are biased conservative — round up when uncertain.** The two error directions are asymmetric. Under-labeling (calling a leaky probe `open`) lets the gate pass it *and* makes Gardner over-credit the student — confidence inflates falsely, the Klarna failure in both consumers at once. Over-labeling (calling a clean probe `leading`) only reworks a good probe (mild friction) and under-credits the student (conservative confidence). So when the classifier is uncertain, it rounds toward **more** disclosure. This is a values-driven target, not a statistical one: the system prefers making the Tutor try again over crediting the student falsely. It also reinforces the pedagogy — treating borderline-leaky probes as too leaky pressures the Tutor toward genuinely low-disclosure probes. (Note the two quantities are distinct: real disclosure should be *low*; our *estimate when unsure* rounds *high*.)

**Recalibration is blast-radius-gated** (ADR-0009 layer discipline). A small threshold nudge within bounds may auto-apply, logged and reversible; a wholesale shift in the label distribution is `REQUIRE_REVIEW`, because every downstream number moves with it.

### Gold-set construction (three stages)

1. **Seed (cold start).** Re-label a slice of **MathDial** onto the four-rung scale — its SCAFFOLDING / TELLING / GENERIC annotations bisect our scale (scaffolding ≈ `open`/`hint`, telling ≈ `leading`/`near_answer`), giving labelers a head start within each bucket. Add author-time labels on the curriculum graph's existing `teachBackProbes`: authors rate disclosure as they write each probe, which is cheap and grows the set with the curriculum. This covers the mathematical substrate.
2. **Scale up synthetically, grounded in real teaching content.** Have the Tutor model generate the *same* probe at all four disclosure levels for a given concept + gap. Matched quadruples are ideal training signal because the relative ordering is true by construction. For the **textual and rhetorical** substrates — which no public dataset covers — the synthetic probes are grounded in the canonical texts Limpide already teaches (e.g. Plato's *Meno* and *Republic*): because the concept and its understanding-ladder (ADR-0010) are authored against a real interpretive target, a probe's disclosure relative to that target is well-defined. See the non-mathematical substrate strategy below.
3. **Refine from real transcripts.** Once piloting, label a sample of real Tutor probes. Highest fidelity, and it closes the domain gap the external and synthetic data leave open.

### Non-mathematical substrate strategy

Public tutoring corpora cluster in math, language, and code; the interpretive substrates Limpide most needs — textual (close reading) and rhetorical (argument) — have no public annotated dataset. The plan, by substrate:

- **Mathematical** — seed from MathDial (real, 1:1, scaffolding/telling annotated).
- **Language** — seed from the Teacher–Student Chatroom Corpus (TSCC, real 1:1 English tutoring, dialogue-act annotated) and CIMA (1:1, four tutor dialogue-act categories).
- **Procedural / code** — seed from the Socratic Debugging benchmark, which is explicitly built around guiding a novice to find their own bug without being told — a clean disclosure target.
- **Rhetorical (adjacent signal)** — ArgRewrite's annotated argumentative-essay revisions inform how arguments develop, complementing probe data.
- **Textual and rhetorical (the real gap)** — built, not borrowed: primarily stage-2 synthetic contrastive probes grounded in the canonical texts above, validated later against a small real-transcript sample.

Platonic dialogue is *not* used as raw disclosure labels: it is literary and rhetorically leading by design (the interlocutor mostly assents), so it skews to the `leading`/`near_answer` end and lacks authentic learner gaps. Its value is as grounding for synthetic probes and as a library of rhetorical question-forms, not as gold labels.

Any use of video (e.g. recorded interpretive tutorials to validate real phrasing) is restricted to Creative-Commons-licensed or API-permitted content. Scraping platforms in violation of their terms is out of scope — not on principle but on exposure: a shipped product carries the ToS, contract, and copyright risk directly, and CC/API-licensed content sidesteps it at low cost.

### Dataset sources

- **MathDial** — ~3k one-to-one teacher–student math tutoring dialogues, teacher moves annotated SCAFFOLDING / TELLING / GENERIC; built explicitly to study the telling-vs-scaffolding trade-off (which is our attribution problem by another name). Public. Primary math seed. https://arxiv.org/abs/2305.14536 · https://github.com/eth-nlped/mathdial
- **TalkMoves** — 567 K-12 math lesson transcripts annotated for ten accountable-talk discursive moves (incl. *press for reasoning*, a clean low-disclosure exemplar). Whole-class, not 1:1; not a disclosure scale. Secondary — realistic phrasing and weak supervision. https://arxiv.org/abs/2204.09652 · https://aclanthology.org/2022.lrec-1.497/
- **Teacher–Student Chatroom Corpus (TSCC)** — real one-to-one English-language tutoring chatrooms, dialogue-act annotated, free for research. Language-substrate seed. https://arxiv.org/pdf/2011.07109 · https://aclanthology.org/2020.nlp4call-1.2.pdf
- **CIMA** — open-access 1:1 tutoring (English→Italian translation) with tutor turns annotated into four dialogue-act categories. Narrow but annotated; language-substrate seed. https://www.researchgate.net/publication/343302209_CIMA_A_Large_Open_Access_Dialogue_Dataset_for_Tutoring
- **Socratic Debugging benchmark (BEA 2023)** — instructors Socratically guiding novice programmers to find and fix their own bugs; explicitly guiding-without-telling with a concrete target. Procedural/code-substrate seed. https://aclanthology.org/2023.bea-1.57/ · https://github.com/taisazero/socratic-debugging-benchmark
- **ArgRewrite V.2** — annotated argumentative-essay revisions. Not probe data, but adjacent signal for the rhetorical substrate. https://arxiv.org/pdf/2206.01677
- **Canonical Socratic texts** (Plato's *Meno*, *Republic*, *Theaetetus*; public domain) — grounding for synthetic textual/rhetorical probes, not raw labels (see strategy above).

## Alternatives considered

**Symmetric error treatment.** Optimize the classifier for plain accuracy. Rejected — it ignores that under- and over-labeling have wildly different costs, and the cheaper-looking symmetric optimum permits the exact failure the project guards against.

**Use MathDial labels as a drop-in gold set.** Skip re-labeling. Rejected — three coarse labels are not four ordinal rungs, and the corpus is math-only; treating its labels as ground truth for our scale would bake in both a granularity error and a substrate bias.

**LLM-judge with no human anchor.** Let a strong model define the gold standard. Rejected for the anchor role — it has no independent ground truth and can drift with the same biases as the classifier. An LLM may *assist* labeling, but the anchor must be human.

**Validate the classifier against student outcomes / delayed re-tests.** Tempting, since we have outcomes. Rejected as the *primary* calibration signal because it conflates "did the classifier read the text right" with "did the student understand," dragging the state-metric failure back in. Outcome data is fine for Gardner's drift *monitoring* (a contradiction signal), but the correctness anchor stays on the text-intrinsic labeling task.

## Consequences

**Commits us to:**
- Maintaining a human-labeled gold set and a weighted-kappa threshold the classifier must clear before it gates anything in production.
- A new Gardner responsibility: logging probe-label / outcome contradictions as a drift signal, distinct from its student-facing analytic work.
- The conservative round-up bias as an explicit training and acceptance criterion, not an emergent property.

**Precludes:**
- Treating classifier accuracy as substrate-uniform — math is seeded from real data, textual/rhetorical from synthetic, and the two will not have equal confidence until stage 3.
- Auto-adopting a recalibration that materially shifts the label distribution.

**Opens:**
- The weighted-kappa threshold value and whether it should differ by substrate (a lower bar where only synthetic data exists, raised as real data arrives).
- Whether the gold set itself becomes part of the institutional efficacy artifact (a record of how the system's judgment was validated over time).

## Status

**Settled.** The gold-set-anchor + Gardner-drift-monitor + conservative-round-up + blast-radius-recalibration approach. Quadratic-weighted kappa as the agreement metric. The three-stage construction order. Human labels (not an LLM) as the correctness anchor. Classifier calibration validated on the text-intrinsic task, never on student outcomes.

**Tentative.** The seed sampling strategy from MathDial and the re-labeling protocol. The synthetic-quadruple generation recipe (drafted in `docs/methods/synthetic-probe-generation.md`). The kappa threshold(s). The per-substrate seeding plan (math → MathDial; language → TSCC/CIMA; code → Socratic-debugging; textual/rhetorical → synthetic grounded in canonical texts) and the precision of the Plato-as-grounding method.

**Open.** Per-substrate thresholds. Whether the validation record is surfaced to institutions. Whether a small CC-licensed interpretive-video sample meaningfully improves real-phrasing coverage for the textual/rhetorical substrates.
