# Limpide — State of the design and what's needed before pilot

A one-page orientation for a collaborator or counsel joining now. Limpide is a Socratic tutoring platform built on one principle: *understanding is what survives the student's attempt to explain it.* The student teaches back; the system measures the process of that teaching, not a state of "mastery." Full detail in `VISION.md`, `PEDAGOGY.md`, `MEASUREMENT.md`, `ARCHITECTURE.md`, `STRATEGY.md` and the ADRs.

## Where the design stands

The conceptual docs are written and mutually consistent. Twenty ADRs exist (0001–0005 the original foundational stack, of which 0002 and 0003 are now superseded; 0006 and 0008 still open placeholders; 0007 closed on Logto; 0009–0018 the pedagogical design; 0019–0020 the August 2026 realignment onto AYA's actual substrate). The load-bearing decisions:

- **Substrate: Flue inside the AYA envelope, Limpide as a plugin** (ADR-0019, ADR-0020, ADR-0007). AYA retired its bespoke agent framework for Flue, flipped to SurrealDB + Logto, dropped the Workers edge, and became a plugin host. Limpide adds no process of its own; it contributes agents, tools, evaluators, schema, and playbooks to the host's three processes. The pedagogy did not move; the framework under it did.

- **Measurement is process, not state** (`MEASUREMENT.md`). Four signals — iteration delta, recovery-under-probing, self-noticing latency, unprompted transfer — drive everything, kept off the optimisation loop (Goodhart discipline).
- **Two agents** — Tutor (Opus-class, conversational) and Gardner (Haiku-class, analytic) as two Flue sessions on one AYA Task, joined by the Event Journal (ADR-0004 as amended), with the host's deterministic **Policy Gate** running Limpide's evaluators for Foundation rules **F1–F12**.
- **Probe gating** (ADR-0009): a Tutor declares intent, a separate inline classifier labels *disclosure*, the gate decides deterministically, Gardner attributes recovery. Self-learning proposes; it never enacts.
- **Three orthogonal axes**: depth (ADR-0010), expression (ADR-0012), disclosure (ADR-0009) — conflating them is the trap.
- **Canonical concept graph + curriculum overlays** (ADR-0013): one timing-free superset; Vaud, Valais, EPFL-year-1 are overlays that reference it; enables cross-overlay benchmarking.
- **Per-rung, history-derived confidence** (ADR-0018) with decay that encodes the thesis (discovered survives forgetting; delivered fades).
- **Safeguarding (F12) and child-data protection** (ADR-0014): best-interests baseline, per-deployment consent/controller model, children's data excluded from the network scope.
- **Acceptable use** (ADR-0015): institutional evidence is formative, never punitive.
- **System evaluation** (ADR-0016): a four-layer stack (regression → offline replay → simulated students → bounded online) that gates the self-learning loop.
- **Latency/cost budget** (ADR-0017).

## Recommended pilot

**Math, Vaud, one bounded misconception-rich cluster** (proportionality → linear relationships → introductory algebra). Rationale: highest demand and easiest recruitment of struggling students; the only substrate with real seed data (MathDial); cleanest recovery/attribution because answers are objective; most concrete understanding ladder; cleanest curriculum import. Physics and the textual/rhetorical substrates follow once the engine is proven.

## What's needed before pilot

**Needs counsel (treat as gating):**

- A **Data Protection Impact Assessment** under revFADP/GDPR for large-scale processing of children's data (ADR-0014).
- The **cantonal mandatory-reporting mapping** (Vaud, plus Valais if both) defining the F12 human-escalation path and who the designated safeguarding contact is.
- The **consent model** for the pilot: capacity-of-judgment under Swiss law, plus guardian and school/institutional consent; confirm controllership per deployment.

**Decisions for the founder:**

- Confirm pilot subject + canton + the exact concept cluster.
- Consumer-first vs. school-first for the pilot cohort (the Vaud/Valais student access suggests a small supervised school or tutoring setting).
- How much to trust LLM-drafting of curriculum vs. require expert authorship (recommendation: lazy + LLM-drafted + human-owned cross-links).

**Build prerequisites (engineering):**

- The substrate is AYA's and mostly exists: SurrealDB + Logto, the worker on Flue, the refs-only Dispatch adapter, the Policy Gate registry, the Event Journal. What Limpide must build: the plugin package against the host contract (AYA ADR-0017 — gated on the host's plugin extraction, F-042, landing first), the F1–F12 and probe-gating evaluators, the Tutor/Gardner/classifier Flue definitions, the plugin-private schema (ADRs 0019, 0020, 0007).
- **Curriculum import** of the chosen Vaud math cluster as an overlay (skeleton + grade/timing), then LLM-drafted ladders/probes and human-authored cross-substrate links (ADR-0013).
- The **disclosure classifier** and its gold set: re-label MathDial + author-time labels + synthetic quadruples (ADR-0011, `methods/synthetic-probe-generation.md`).
- The **expressive-baseline** initialisation and the **per-rung confidence** computation (ADRs 0012, 0018).
- The **regression suite + offline replay harness** before any student sees the system (ADR-0016).

**Open research (parallel, not blocking the pilot):**

- The attribution function (disclosure + student production → credit). 
- Classifier calibration methodology and drift monitoring.
- Simulated-student fidelity for pre-screening.

## Suggested sequence

1. In parallel from day one: engage **counsel** (DPIA, reporting, consent) and stand up the **Limpide plugin skeleton** on the AYA host — package, manifest, plugin-private schema, evaluator registration — which depends on AYA's plugin extraction (F-042) being far enough along to boot a bare host with one plugin listed.
2. Import the **Vaud math cluster**; draft ladders/probes; author cross-links.
3. Build the **disclosure classifier** from the math gold set; stand up the **regression + replay harness**.
4. **Internal dogfooding** against simulated students and the team; tune confidence and baselines.
5. **Small supervised pilot** with consented students in one canton; measure on the process signals; expand to the second canton to prove cross-overlay benchmarking.

The gating discipline throughout: nothing reaches a real student that has not passed the Foundation-rule regression suite, and the DPIA and reporting path must be settled before the first session.
