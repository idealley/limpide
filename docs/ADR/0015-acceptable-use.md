# ADR-0015: Acceptable use of institutional evidence

**Status:** Proposed
**Date:** 2026-05-29

## Context

`STRATEGY.md` sells institutions longitudinal *process evidence* — how understanding develops, in students' own articulations. The same evidence could be turned against the people it describes: used to rank or grade students on their explanation trajectories, or to evaluate, discipline, or dismiss teachers based on their cohorts' patterns. That would invert the student-first vision (`VISION.md`).

It would also destroy the evidence. The reasoning is the same Goodhart shape the whole project is built around (`MEASUREMENT.md`, Foundation rule F8): the moment students know their explanations are scored punitively, they perform rather than think; the moment teachers know their cohort's process signals are used to judge them, they teach to the signal. Punitive use corrupts the very data that made the institutional product valuable. So acceptable-use is not only an ethical commitment — it protects the truthfulness of the measurement.

## Decision

**The institutional evidence may be used formatively, never summatively-punitively.**

- *Formative, permitted*: helping an individual student progress; a teacher reflecting on and improving their own practice; an institution demonstrating its efficacy; surfacing where a cohort is struggling.
- *Summative-punitive, forbidden*: ranking or grading students on the process signals; high-stakes individual decisions made from them; evaluating, disciplining, or dismissing teachers on their cohorts' patterns; any leaderboard.

This is enforced on three levels, because no single one suffices:

1. **Contractually.** The institutional licence forbids punitive use, alongside the "Aegilo never sells the network's data" commitment already flagged in `STRATEGY.md`. This is the only lever that reaches an institution's *internal* use of exported data.
2. **By design.** Operator views are cohort-level and formative; the system does not expose a per-student process *score* that invites ranking, and no student-ranking or teacher-evaluation surface is built. The cohort scope (`ARCHITECTURE.md`) already structurally prevents drilling to a named individual from aggregate views — this ADR makes that a deliberate acceptable-use boundary, not just a privacy one.
3. **By the invisibility discipline.** F8's logic — keep the signals invisible to those being measured so they cannot be performed — extends from the student to the institutional layer: the evidence is framed as developmental trajectory, not a comparative score.

The validity argument is the load-bearing one: this is what keeps the data worth having. An institution that uses the evidence punitively is not just acting against its students; it is destroying the signal it is paying for.

## Alternatives considered

**Leave acceptable use to the institution.** Ship the data, let buyers decide. Rejected — it abdicates responsibility for foreseeable misuse of a child-derived dataset and invites the corruption that voids the data.

**Technical enforcement only.** Rely on design friction alone. Rejected — once aggregates or reports are exported, design cannot police internal use; the contract is the only reach, and design + invisibility are the reinforcement.

**Ban institutional reporting entirely.** Eliminate the risk by removing the feature. Rejected — it deletes the core of the institutional value proposition (`STRATEGY.md`). The answer is *formative-not-punitive*, not *no-reporting*.

## Consequences

**Commits us to:**
- A no-punitive-use clause in the institutional licence.
- Cohort-level, formative operator views by design; no per-student process-score surface; no ranking or teacher-evaluation features.
- Extending F8's invisibility framing to the institutional layer.

**Precludes:**
- Building or selling student-ranking or teacher-evaluation tooling on the evidence.
- Exposing individual process scores to administrators as a comparative measure.

**Opens:**
- Enforcement when an institution breaches (audit rights? suspension? revocation?).
- The boundary between legitimate formative *individual* support (a teacher helping one student) and punitive individual use — the line is intent and stakes, which is hard to encode.
- Whether students and guardians are told how the evidence may and may not be used, and whether they get any say.

## Status

**Settled.** Formative-permitted / summative-punitive-forbidden. Three-level enforcement (contract, design, invisibility). The validity argument as the primary justification.

**Tentative.** The contract language and the breach-enforcement mechanism. The exact set of operator views considered formative.

**Open.** The formative/punitive individual-use boundary. Student/guardian transparency and voice over institutional use.
