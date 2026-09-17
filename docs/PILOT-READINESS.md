# Limpide — State of the design and what's needed before pilot

A one-page orientation for a collaborator or counsel joining now. Limpide is a Socratic tutoring platform built on one principle: *understanding is what survives the student's attempt to explain it.* The student teaches back; the system records that process and checks it against delayed independent evidence. Full detail in `VISION.md`, `PEDAGOGY.md`, `MEASUREMENT.md`, `ARCHITECTURE.md`, `STRATEGY.md` and the ADRs.

## Where the design stands

The conceptual docs record an early design, with key efficacy and commercial assumptions still to validate. ADR-0021 (September 2026) revises the measurement assumptions and defers network business. The [ADR index](ADR/README.md) covers the original stack, the pedagogical design, the August 2026 substrate realignment, and the September validation decision. The load-bearing decisions:

- **Substrate: Flue inside the AYA envelope, Limpide as a plugin** (ADR-0019, ADR-0020, ADR-0007). AYA retired its bespoke agent framework for Flue, flipped to SurrealDB + Logto, dropped the Workers edge, and became a plugin host. Limpide adds no process of its own; it contributes agents, tools, evaluators, schema, and playbooks to the host's three processes. The pedagogy did not move; the framework under it did.

- **Process plus independent outcomes** (`MEASUREMENT.md`, ADR-0021). Four candidate process signals are checked against delayed unaided explanation and unfamiliar application. Internal scores alone cannot establish efficacy; prompt/version selection also needs checks against proxy optimization.
- **Two agents** — Tutor (Opus-class, conversational) and Gardner (Haiku-class, analytic) as two Flue sessions on one AYA Task, joined by the Event Journal (ADR-0004 as amended), with the host's deterministic **Policy Gate** running Limpide's evaluators for Foundation rules **F1–F12**.
- **Probe gating** (ADR-0009): a Tutor declares intent, a separate inline classifier labels *disclosure*, the gate decides deterministically, Gardner attributes recovery. Self-learning proposes; it never enacts.
- **Three orthogonal axes**: depth (ADR-0010), expression (ADR-0012), disclosure (ADR-0009) — conflating them is the trap.
- **Canonical concept graph + curriculum overlays** (ADR-0013): one timing-free superset; Vaud, Valais, EPFL-year-1 are overlays that reference it; later enables cross-overlay comparison, which is not a pilot goal (ADR-0021).
- **Per-rung, history-derived confidence** (ADRs 0018 and 0021). Support attribution is explicit. Teaching-route weights, exam-mode discounts, and differential decay remain hypotheses to calibrate against independent outcomes.
- **Safeguarding (F12) and child-data protection** (ADR-0014): best-interests baseline, per-deployment consent/controller model, children's data excluded from the network scope.
- **Acceptable use** (ADR-0015): institutional evidence is formative, never punitive.
- **System evaluation** (ADR-0016): a four-layer stack (regression → offline replay → simulated students → bounded online) that gates the self-learning loop.
- **Latency/cost budget** (ADR-0017).

## Recommended pilot

**Math, Vaud, one bounded misconception-rich cluster** (proportionality → linear relationships → introductory algebra). Rationale: a concrete starting hypothesis with teacher contacts that may support recruitment; the only substrate with real seed data (MathDial); cleanest recovery/attribution because answers are objective; most concrete understanding ladder; cleanest curriculum import. Physics and the textual/rhetorical substrates follow once the engine is proven.

## Proposed first offer and teacher collaboration

The founder has teacher contacts and proposes an optional placement session followed by short daily mathematics practice. `BOOTSTRAP-PILOT.md` specifies the candidate learner flow: official-curriculum import, school context and recent assignment photos, targeted placement, checked exercise generation, independent assessment, and commercial questions. No teacher or cohort is confirmed.

A candidate trial is 20–30 learners over four to six weeks. Agree the age/course, one concept cluster, teacher workload, assessment access, payer, and continuation criteria before starting. Placement supplies tentative starting points; daily encounters refine individual estimates and suggest human-reviewed content changes. They do not automatically learn a valid shared prerequisite graph.

Mathematics comes first in the proposal. Physics is a possible adjacent trial; philosophy needs a separate reading/argument rubric and audience test. Network benchmarks, contribution tiers, and second-canton comparisons are not pilot success criteria.

## What's needed before pilot

**Needs counsel (treat as gating):**

- A **Data Protection Impact Assessment** under revFADP/GDPR for large-scale processing of children's data (ADR-0014).
- The **cantonal mandatory-reporting mapping** (Vaud, plus Valais if both) defining the F12 human-escalation path and who the designated safeguarding contact is.
- The **consent model** for the pilot: capacity-of-judgment under Swiss law, plus guardian and school/institutional consent; confirm controllership per deployment.

**Decisions for the founder:**

- Confirm subject, canton, learner age/course, concept cluster, teacher participation, and whether the proposed placement/daily format is the first offer.
- Choose the first payer and recruitment route; test a concrete paid pilot or continuation, including review/support cost. Teacher introductions are a route to explore, not confirmed demand.
- How much to trust LLM-drafting of curriculum vs. require expert authorship (recommendation: lazy + LLM-drafted + human-owned cross-links).

**Build prerequisites (engineering):**

- The substrate is AYA's and mostly exists: SurrealDB + Logto, the worker on Flue, the refs-only Dispatch adapter, the Policy Gate registry, the Event Journal. What Limpide must build: the plugin package against the host contract (AYA ADR-0017 — gated on the host's plugin extraction, F-042, landing first), the F1–F12 and probe-gating evaluators, the Tutor/Gardner/classifier Flue definitions, the plugin-private schema (ADRs 0019, 0020, 0007).
- **Curriculum import** from the PER and applicable Vaud material, preserving source references and stage/year expectations (ADR-0013). Use LLM-assisted concept decomposition and inferred prerequisites with review. Import coverage can be broader than the initial validated tutoring slice. Recent assignment images and school context support targeted placement; reviewed task families and checked generation supply exercises, solutions, and explanations.
- The **disclosure classifier** and its gold set: re-label MathDial + author-time labels + synthetic quadruples (ADR-0011, `methods/synthetic-probe-generation.md`).
- The **expressive-baseline** initialisation and **per-rung confidence** computation, explicitly provisional, plus separate teacher-reviewed assessment items/rubrics (ADRs 0012, 0018, 0021).
- The **regression suite + offline replay harness** before any student sees the system (ADR-0016).

**Open research (parallel, not blocking the pilot):**

- The attribution function (disclosure + student production → credit). 
- Classifier calibration methodology and drift monitoring.
- Simulated-student fidelity for pre-screening.

## Suggested sequence

1. Confirm a **teacher/content partner**, the bounded learner offer, and independent assessment/continuation criteria. In parallel: engage **counsel** (DPIA, reporting, consent) and stand up the **Limpide plugin skeleton** on the AYA host — package, manifest, plugin-private schema, evaluator registration — which depends on AYA's plugin extraction (F-042) being far enough along to boot a bare host with one plugin listed.
2. **Import the selected official curriculum structure upfront**, then validate mappings, inferred prerequisites, reference explanations, and generated task families for the first active topics. Add school-context and assignment-photo intake for targeted placement. Reserve independent assessment items; all imported topics need not have validated tutoring content before the first bounded trial.
3. Build the **disclosure classifier** from the math gold set; stand up the **regression + replay harness**.
4. **Internal dogfooding** against simulated students and the team; tune confidence and baselines.
5. **Small supervised pilot** with consented students in one canton. Compare process observations with delayed unaided work; assess voluntary return, actionable teacher feedback, paid continuation, and cost. Decide whether to revise or expand from those observations. A small pilot does not establish causal efficacy; network business remains deferred.

The gating discipline throughout: nothing reaches a real student that has not passed the Foundation-rule regression suite, and the DPIA and reporting path must be settled before the first session.
