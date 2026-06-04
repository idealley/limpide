# Method: synthetic probe generation

This is the operational recipe for stage 2 of the gold-set construction in `docs/ADR/0011-classifier-calibration.md` — generating disclosure-labeled probe data, especially for the textual and rhetorical substrates that no public corpus covers. It is a methods document, not a decision record; the decision is in ADR-0011.

## The core idea: a disclosure quadruple

The unit of generation is not a single probe but a **quadruple**: the *same pedagogical intent*, realized at all four disclosure levels. Holding intent constant and varying only how much the probe reveals is what makes the data a controlled contrast — the classifier learns the disclosure axis independent of topic, concept, and intent. The relative ordering (`open < hint < leading < near_answer`) is true *by construction*, so the quadruple is self-labeling for ordinal structure even before a human looks at it.

This mirrors the `Probe` object directly (ADR-0009): one `intent`, four candidate `utterance`s at four `disclosure` levels.

## Operational rung definitions

These definitions are shared between the generator and the disclosure classifier — they must be the same words, or the generator trains the classifier toward a target the classifier doesn't hold.

- **`open`** — a question that directs attention but reveals nothing of the answer. The student must supply the entire content. *"Why does that have to be true?"*
- **`hint`** — surfaces a relevant consideration or direction without stating the relationship the student is reaching for. *"What happens to the force as you stretch it further?"*
- **`leading`** — encodes most of the answer inside the question; the student need only assent. *"So if doubling the stretch doubles the force, that's a proportional relationship, isn't it?"*
- **`near_answer`** — states the answer and asks for confirmation or restatement. *"The force is proportional to displacement — can you say back why?"*

Worked once per substrate so the generator has anchors:

| Rung | Mathematical (Hooke's law) | Textual (*Meno* slave-boy) | Rhetorical (student's draft argument) |
|---|---|---|---|
| open | "Why does that have to be true?" | "What is Socrates actually doing here?" | "What makes this a reason and not just a restatement?" |
| hint | "What happens to the force as you stretch further?" | "Notice who is supplying the answers — what does that tell you?" | "Would a reader who disagrees with you grant this sentence?" |
| leading | "So doubling the stretch doubles the force — proportional, right?" | "Socrates only asks questions, so is he teaching or drawing out, would you say?" | "Since you never define the key term, the argument assumes what it should prove, doesn't it?" |
| near_answer | "Force is proportional to displacement — say back why." | "He's eliciting recollection, not teaching — can you restate that?" | "Your conclusion is unsupported because premise two is missing — can you put that in your words?" |

## Grounding inputs

Disclosure is only meaningful relative to *the answer the student is reaching for*, so generation must be grounded in an explicit target. Supply per request:

- `conceptId` and the targeted **ladder rung** (ADR-0010) — what depth of understanding the probe is working toward.
- the **intent** — the single thing this quadruple is trying to open.
- the **target** — the answer/insight the probe is relative to.
- a simulated **student gap** — the misconception or missing step the probe addresses.
- substrate-specific grounding:
  - *mathematical*: the worked target and the misconception.
  - *textual*: the passage (e.g. *Meno* 82b–86c) and the interpretive move the student should make.
  - *rhetorical*: the student's (simulated) draft argument and the structural weakness being probed.

For textual and rhetorical substrates the grounding texts are the canonical works Limpide already teaches (ADR-0011) — which is what makes synthetic generation trustworthy there: the target is real even though the student is simulated.

## Generation prompt (template)

```
SYSTEM:
You generate tutoring probes for a Socratic tutor. A "probe" is a question that
moves a student toward understanding. You will produce the SAME probe at four
levels of DISCLOSURE — how much of the answer the question reveals — holding the
pedagogical intent fixed. Vary only disclosure, never intent.

Disclosure levels (use exactly these definitions):
- open: directs attention, reveals nothing; student supplies all content.
- hint: surfaces a relevant consideration without stating the relationship.
- leading: encodes most of the answer in the question; student need only assent.
- near_answer: states the answer, asks for confirmation/restatement.

Rules:
- All four must share one intent and target.
- Each must be a plausible thing a warm, patient tutor would actually say.
- Do not name a gap the student has not noticed (Foundation rule F1).
- Then produce 2 HARD NEGATIVES: one probe that LOOKS open but leaks the answer,
  and one that LOOKS leading but is actually open. Label each with its TRUE level.

USER (slots):
  substrate:     {mathematical | textual | rhetorical}
  concept:       {concept name + ladder rung}
  intent:        {the one thing this quadruple opens}
  target:        {the answer/insight the probe is relative to}
  student_gap:   {the misconception or missing step}
  grounding:     {passage / worked target / draft argument}

OUTPUT: JSON only, matching the schema below.
```

## Output schema

Drops directly into training; field names match the `Probe` object where they overlap.

```json
{
  "conceptId": "string",
  "rung": 0,
  "substrate": "mathematical | textual | rhetorical",
  "intent": "string",
  "target": "string",
  "studentGap": "string",
  "grounding": "string",
  "quadruple": [
    { "disclosure": "open",        "utterance": "string" },
    { "disclosure": "hint",        "utterance": "string" },
    { "disclosure": "leading",     "utterance": "string" },
    { "disclosure": "near_answer", "utterance": "string" }
  ],
  "hardNegatives": [
    { "appears": "open",    "actual": "leading",     "utterance": "string" },
    { "appears": "leading", "actual": "open",        "utterance": "string" }
  ],
  "provenance": "synthetic"
}
```

## Quality gates

Synthetic data is cheap to produce and easy to produce *wrong*, so it passes gates before entering training:

1. **Blind re-labeling.** A separate pass (the disclosure classifier itself, or a held-out labeler) labels each utterance with the level stripped. Where the blind label disagrees with the intended level, the item is flagged. This both filters bad items *and* continuously tests the classifier — the same Gardner-style audit loop as ADR-0011.
2. **Ordinal-agreement threshold.** A batch is accepted only if intended-vs-blind agreement (quadratic-weighted kappa) clears the threshold. Low-agreement batches are rejected, not patched.
3. **Human spot-check.** A sample of accepted quadruples is read by an educator to confirm the ordering is real and the probes are natural, before the batch is used.
4. **Hard negatives are mandatory.** The look-open-but-leak and look-leading-but-open cases are exactly where the classifier's conservative round-up bias (ADR-0011) is trained and tested. A batch without them is incomplete.

## The separation guardrail

Synthetic data is for **training and augmentation only**. It never enters the human gold *anchor* set. The anchor is the independent yardstick the classifier is measured against (ADR-0011); if synthetic items the classifier helped label were folded into the anchor, the kappa would be circular and the calibration meaningless. Train on synthetic; validate against human. Keep them in separate stores with `provenance` marking every record.

## Status

**Tentative.** This whole recipe is a first realization of ADR-0011's stage 2 and will be revised once the first batches are generated and a human spot-check reveals where the generator is systematically off (likely: over-producing fluent `leading` probes that read as `hint`). The kappa threshold and the human-sample fraction are unset pending the first runs.
