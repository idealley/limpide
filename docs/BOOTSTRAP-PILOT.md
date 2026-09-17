# Proposed first offer: daily mathematics practice

**Status:** Proposed, 2026-09-10. An experiment following [ADR-0021](ADR/0021-validate-tutoring-first.md), not a launch commitment. The founder has teacher contacts; participation and a cohort are not confirmed.

## The learner's reason to use it

> A short mathematics problem chosen for where you are, help when you need it, and another chance to see whether it has stuck.

Combine an optional starting-point session with an invitation to spend roughly 5–10 minutes on mathematics each day. Each session must provide useful practice. Data collection is a byproduct, not the learner's reason to subscribe.

Daily cadence is a hypothesis. Let learners choose a realistic schedule, skip days, and resume without a backlog or broken streak. Allow an urgent school question to replace the scheduled exercise. Any reminders are optional and separately enabled.

## Build the curriculum structure before the first learner

Import the chosen official curriculum before collecting learner data. This is the sourcing model already described in ADR-0013. For Vaud compulsory schooling, start with the Plan d’études romand (PER) and applicable cantonal material. Official objectives, progressions, and expectations supply the initial structure and the curriculum overlay; teachers do not need to write these from scratch.

Use extraction and LLM-assisted structuring to map objectives to stable concept identifiers, decompose broad objectives into teachable concepts, and draft prerequisites, understanding ladders, explanations, and exercise specifications. Preserve the distinction between an explicit source statement and an inferred decomposition or prerequisite. Keep the source URL, objective identifier, version/date, location in the source, original stage/year granularity, and review status. A curriculum's teaching order is useful evidence but does not by itself prove a strict prerequisite.

The imported graph can cover a broader subject/year band than the first practice trial. Separate **curriculum coverage** from **validated tutoring coverage**: importing an objective does not mean every associated generated exercise is ready to serve. Review coverage mappings and inferred relationships, with deeper educator review on the first active topics. Preserve gaps in the source rather than inventing precise year-by-year expectations where it specifies only a cycle.

## Placement using school context and recent work

Begin with age, canton/school system, actual school year, subject, and stream or subject level where applicable. Age provides a rough starting assumption; the actual program and year select the curriculum overlay. Ask what the learner is working on now and where they want help. If school details are unknown, use tentative defaults and refine them through the interaction.

Invite photos of the last three teacher-assigned exercises, ideally including the learner's attempts and any corrections when available. This is optional; a topic, textbook reference, or typed exercise can substitute. Read the notation and diagrams, ask for clarification when extraction is uncertain, and map the exercises to curriculum objectives, concepts, and likely prerequisites. Assigned questions show classroom context; attempted solutions add evidence about the learner. Neither alone establishes independent understanding. Keep uploaded work within the authorized learner context and avoid retaining unrelated identifying details.

Then ask a few targeted questions near that location in the graph: a representative current-topic problem, a reasoning or transfer check, and prerequisite questions only where a response indicates uncertainty. Approximately 4–6 short tasks over 10–15 minutes is an initial interaction budget, not a promise of full placement. Branch rather than administer a broad exam. Use calculation, brief explanation, prediction, or diagrams as appropriate.

Produce a provisional local placement: current curriculum topic, concepts with supporting evidence, likely prerequisite difficulties, and unobserved areas. Keep **expected by the curriculum**, **apparently assigned in class**, and **demonstrated by the learner** separate. Allow the learner to correct the inferred topic, adjust difficulty, skip placement, or proceed directly to their immediate question.

Record an unaided attempt before offering help, then teach usefully. Revisit uncertainty over subsequent sessions. A wrong answer can reflect wording, arithmetic, expression, or inattention; a correct answer can reflect recall or guessing. Later independent attempts refine placement without postponing useful tutoring.

## Generate targeted material from the graph and learner evidence

Select the curriculum objective and concept, required depth, prerequisite readiness, recent difficulty, desired representation, and available time. Use these to generate an original exercise, solution, acceptable reasoning paths, hints, explanation, and follow-up. Match the teacher's notation and task style where helpful. A full handwritten bank for the whole curriculum is not a prerequisite.

For the first active topics, build reviewed exercise families and reference explanations, then generate constrained variations. Check that each generated task is well-posed, that its answer and explanation agree, that arithmetic/algebra and units are valid where applicable, and that its difficulty and tags fit the intended objective. Use deterministic or symbolic checks where suitable; a second model's agreement alone is not a guarantee. New task types and uncertain outputs need educator review or a fallback to validated material. Independent assessment tasks remain separate from the generation context.

The resulting loop is: official curriculum → contextual placement → targeted generation and support → observed attempts → revised learner estimates and reviewed content improvements. Initial learner data personalizes and improves an existing structure; it is not needed to create the official curriculum map.

## The daily interaction

1. **Attempt:** one bounded problem, answered through calculation, a short explanation, or a diagram as appropriate.
2. **Understand the response:** one useful follow-up where needed—why the method works, what changes at a boundary, or which example fits. Correctness, reasoning, and expression remain separate observations.
3. **Help:** offer a hint, prerequisite detour, worked example, or explanation. Time-box unproductive questioning; direct help is available.
4. **Check:** if time permits, invite a small new attempt. Record it as a supported-session check, not delayed retention evidence.
5. **Return later:** mix current needs, uncertain concepts, and delayed revisits with different surface details. Reserve unfamiliar tasks for independent assessment.

A session ends at a useful stopping point. A five-minute exercise may need only one follow-up. Deeper prerequisite work is an optional extension, not a compulsory recursive stack.

## What the data can bootstrap

| Layer | Initial source | What encounters add | Limit |
|---|---|---|---|
| Shared curriculum graph | Official curriculum objectives and progression, structured with source references; LLM-drafted decompositions and prerequisites reviewed where inferred | Calibration, recurring misconceptions, and candidate refinements to concepts or links | Keep official statements distinct from inferred links; response correlations alone do not establish prerequisites |
| Individual learner map | School context, recent assignment photos/attempts, and targeted diagnostic questions | Evidence by concept, task, date, support, and delayed revisit | Unknown is not weak; supported success is not independent demonstration |
| Exercise bank | Curriculum-grounded generation from reviewed task families, with checked solutions and rubrics | Observed difficulty, clarity, useful hints, and alternative solutions | Generation needs correctness and alignment checks; related variants are not fully independent items |
| Tutoring approach | Educator-informed initial policies | Helpful questions, unproductive questioning, effective explanations | Tutor behavior and task difficulty affect the trajectory |

Import the relevant curriculum structure upfront, potentially across the whole selected subject/year band. The first validated tutoring slice can still be modest—perhaps 8–12 concepts around proportionality → linear relationships → introductory algebra, with needed prerequisites. A small reviewed reference bank (perhaps 30–50 items) and exercise families can support generated variations; the whole curriculum does not need a manually authored bank. Keep independent assessment items separate.

Each item needs concept/rung tags, prerequisite assumptions, expected reasoning, acceptable alternatives, misconception hypotheses, hints, a solution, source, and version. Preserve variant relationships and the reason an item was selected, including learner overrides. Otherwise adaptive selection may look like improvement simply because tasks became easier.

Per encounter, keep only observations needed for the stated purpose: item/version, necessary response evidence, support received, rubric judgment and uncertainty, and immediate versus delayed status. Use stable pseudonymous learner identifiers where appropriate, with identity separate; pseudonymized longitudinal records remain personal data.

The shared graph must not silently absorb private transcripts or student-specific records. Reusable curriculum changes require educator review and applicable authorization. Ordinary use is not blanket permission for research, model training, or cross-institution aggregation. The network remains deferred.

## Independent checks of benefit

Define outcomes before starting. Use matched but distinct baseline and later tasks, including an unaided revisit about a week later and an unfamiliar application of the same principle. Assess reasoning and correctness using a fixed teacher-reviewed rubric. Where feasible, an educator reviews responses without knowing the tutor's score or teaching route.

Keep assessment items out of ordinary tutoring and generation prompts. Record hints or outside help so supported attempts are not labeled unaided. Consider item familiarity/difficulty, classroom instruction outside Limpide, and changing attendance when interpreting progress.

Process signals help explain the result; they do not replace it. A small before-and-after pilot cannot isolate Limpide's causal effect. A later comparison with ordinary practice or an existing tutoring option needs a separately designed study. The first cohort is a feasibility and demand test, not proof of broad efficacy.

## Teachers and the commercial test

Invite one or two teachers to confirm curriculum mappings and classroom progression, review inferred prerequisites and initial task families/rubrics, observe sessions, and independently assess samples. Their role is to validate and enrich the official-material import, not manually create the whole graph. Agree a bounded workload and compensation or another clear arrangement before relying on continuing contributions. Teacher access to individual work must be explicitly scoped; having a contact does not authorize sharing learner records.

A candidate cohort is 20–30 learners over four to six weeks in one supervised setting. Existing consent and safeguarding prerequisites apply (`PILOT-READINESS.md`). An adult mathematics cohort is an alternative if adults are the intended market; it would not establish suitability or demand for schoolchildren.

Teacher relationships can help recruitment and content validation. Identify the payer separately: parent, tutoring provider, or school. A free introduction may reduce friction, but test continuing willingness to pay with an offer at a stated price. A paid pilot or continuation is stronger evidence than a waitlist. Measure inference cost, educator review, and support time before settling pricing.

Agree practical continuation criteria before starting, covering:

- Unaided benefit that remains after the session.
- Voluntary return and reasons for skipping, quitting, or requesting answers.
- Teacher observations that lead to useful instructional action.
- Actual payment or a concrete paid commitment from the chosen buyer.
- Delivery cost and review workload compatible with the price.

## Subject expansion

**Mathematics first.** It matches the existing pilot recommendation and offers bounded material with comparatively clear solutions. Correct numbers still need reasoning checks.

**Physics next, conditional on mathematics working and a suitable teacher participating.** Prediction → explanation → calculation is a promising short format. Reuse mathematics concepts, but distinguish mathematical difficulty from physical misconceptions. Start with one cluster such as proportional relationships in motion or forces. Transfer is something to test, not infer from shared graph links.

**Philosophy as a separate experiment.** A short primary-text passage, an argument question, and an objection or counterexample could make a useful recurring reading practice. Diagnose familiarity and reasoning within the task, not a universal philosophy level. Expert-reviewed rubrics should assess faithful interpretation, premises, coherence, evidence, and engagement with objections. Allow defensible disagreement; do not reward a preferred philosophical position. Audience, reading time, and payer need separate validation.

Launching all three together would divide content and recruitment effort and obscure which assumption failed. Preserve the broader vision and earn each subject through its own small trial.

## Research rationale and its limit

[The role of variable retrieval in effective learning (PNAS, 2024)](https://www.pnas.org/doi/10.1073/pnas.2413511121) reports benefits from varying retrieval cues across spaced practice in vocabulary experiments, with a further experiment on lecture content. This supports exploring spaced, varied revisits; it does not establish the effectiveness of daily mathematics exercises, a particular cadence, or Limpide's placement method. Those remain pilot questions.

## Official starting sources

- [État de Vaud: Plan d’études romand (PER)](https://www.vd.ch/formation/enseignement-obligatoire-et-pedagogie-specialisee/deroulement-de-lecole-obligatoire-dans-le-canton-de-vaud/plan-detudes-romand-per), checked 2026-09-10: describes the compulsory-school curriculum and links to the official platform, downloadable brochures, cantonal supplements, and teaching materials. Applicable supplements must be selected for the learner's course; they are not automatically the standard program for every pupil.
- [État de Vaud: school structure](https://www.vd.ch/formation/enseignement-obligatoire-et-pedagogie-specialisee/deroulement-de-lecole-obligatoire-dans-le-canton-de-vaud), checked 2026-09-10: identifies school years/cycles and secondary pathways, supporting placement by actual program rather than age alone.

- [CIIP: mathematics objectives](https://portail.ciip.ch/per/disciplines/5) and [the official MSN 33 mathematics brochure](https://bdper.plandetudes.ch/uploads/per_pdf/Mathematiques_et_sciences_de_la_nature/Mathematiques/PER_print_MSN_33.pdf), checked 2026-09-10: the source already identifies objectives and provides learning progressions, fundamental expectations, and pedagogical guidance. MSN 33 covers numerical/algebraic problems, including proportionality and functions. Preserve the original year/cycle and level annotations during extraction; validate table structure rather than trusting flattened PDF text.

These sources establish the import route. A complete curriculum extraction, field mapping, and validated exercise generator have not yet been built.
