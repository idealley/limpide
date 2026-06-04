# ADR-0010: Concept depth — program-required floor, student-desired ceiling

**Status:** Proposed
**Date:** 2026-05-29

## Context

ADR-0009 enforces a terminating floor on the why-recursion at the Policy Gate: the chain bottoms out at the *program-required depth* for a concept, and going deeper requires an active fork. That ADR left the representation of "program-required depth" open, noting only that the curriculum graph has `prereqs` and `capability` but no depth or level field, and that a small addition to the curriculum or student model was likely needed.

"Depth" is one axis with three distinct owners, and the data model currently expresses none of them:

1. How far down a given concept *can* meaningfully be understood — a property of the concept (applying a linear relationship, then explaining why it is proportional, then deriving it from rate-of-change, then the foundations of what a function is). This is the ceiling no one exceeds.
2. How far the student's *program* requires them to go — a property of (program × concept). 9th-grade math may require "can explain why"; a physics track may require "can derive." This is the floor.
3. How far *this student* wants to go — a property of (student × concept). The constraint, stated by the design from the start: a student's desired depth can only ever be **equal to or greater than** the program's required depth.

There is also no representation of a *program* (or curriculum track) at all, nor of a student's enrollment in one. Required depth is program-scoped, so it cannot live on the concept; introducing it forces a minimal program notion into the model.

## Decision

Depth is expressed as an index into a per-concept **understanding ladder**, and the three owners each hold their own index into that shared ladder.

**The concept defines the rungs.** `ConceptNode` gains an `understandingLadder`: an ordered list of named rungs, from shallowest to deepest, ending where the concept bottoms into foundations. The ladder is intrinsic to the concept and is the absolute ceiling — no program or student can target a rung that does not exist. Defining the rungs on the concept is what turns "depth" from a vague integer into a concrete, testable index, which the Policy Gate and the disclosure classifier (ADR-0009) both need.

**The program sets the floor.** A minimal `Program` (curriculum track) entity enters the model, carrying a required rung per concept. A `StudentEnrollment` links a student to one or more programs. The program-required rung is the default terminating floor the gate enforces.

**The student sets the ceiling, never below the floor.** `StudentConcept` gains an optional `desiredLevel`. The invariant, enforced at write time, is `desiredLevel >= requiredLevel(program, concept)`. The student knob only ever raises the ceiling above the floor; it can never lower it.

The gate computes an **effective ceiling** = `desiredLevel ?? requiredLevel`. A probe targeting a rung at or below the effective ceiling is permitted; a probe targeting a deeper rung is denied unless an in-session fork is active. The student's `desiredLevel` therefore behaves as a *standing fork preference* — a durable "this student wants to go deep on this concept" — while an in-session fork is the transient way to exceed even that, in the moment.

Crucially, the floor is non-negotiable and the knob is one-directional by design. A student who only wants to pass ("don't make me understand this") is **not** modeled as a `desiredLevel` below the floor. That case is already handled, honestly, by exam-driven mode (`PEDAGOGY.md`): the system delivers the surface answer, records the outcome as `delivered` or `forced_delivery`, and pins the gap for later. "Wanting less" is a mode and an outcome, not a depth setting. This is why the invariant is the right shape rather than a limitation — reductions have their own honest representation elsewhere, so the depth knob is free to be a pure extension mechanism.

## Alternatives considered

**A single scalar depth on `ConceptNode`.** Simplest. Rejected because required depth is program-scoped — the same concept legitimately requires different depths in different programs — so a single per-concept number cannot express the floor. It would also conflate the concept's intrinsic ceiling with a program's requirement.

**Depth as a free integer recursion bound, no named rungs.** Let "depth 3" mean "three whys deep." Rejected because it is not testable or auditable: the gate and the classifier need to know *which* rung a probe targets, and an unnamed count makes "is this probe within the allowed depth" a judgment rather than a lookup. Named rungs make the gate deterministic, consistent with ADR-0009.

**Allow `desiredLevel` below `requiredLevel`.** Model "I just want to pass" as a low desired depth. Rejected because it would let a student setting silently lower the program's floor — the floor is the institution's commitment, not the student's preference — and because the honest representation already exists (exam mode + delivered outcome + pinned gap). Letting depth express reduction would create two conflicting ways to say "less," one of which hides the gap.

**Per-rung confidence.** Make `confidence` a vector over the ladder rather than a scalar. Tempting and possibly right eventually, but rejected for now as scope creep — it touches the confidence math (ADR-pending) and the entire encounter model. This ADR keeps confidence scalar and treats the ladder as a depth axis, not a mastery vector. Noted as open.

## Consequences

**Commits us to:**
- `understandingLadder` on `ConceptNode`; a `Program` entity and `StudentEnrollment`; `desiredLevel` on `StudentConcept`, with the `desiredLevel >= requiredLevel` invariant enforced at write time.
- The Policy Gate reading an effective ceiling and treating `desiredLevel` as a standing fork preference, in-session forks as transient overrides (ADR-0009).
- Authoring cost: every concept now needs its ladder defined, and every program needs a required rung per concept. This is real curriculum-design work, not just a schema change.

**Precludes:**
- Expressing "less than the program requires" as a depth setting. That stays in urgency mode and the delivered/forced outcomes.
- An unnamed numeric depth — depth is always an index into a defined ladder.

**Opens:**
- Per-rung confidence (whether mastery should eventually be tracked per ladder rung rather than per concept).
- How the program-required rung is authored and by whom (the institution? a default per concept overridable by the institution?), and how a student in multiple programs with conflicting required rungs resolves (presumably the max, but unconfirmed).
- Whether `understandingLadder` rungs map onto `crossSubstrateLinks` — a cross-substrate connection may be reachable only at a particular rung, which would make the depth axis and the triad work interact.

## Status

**Settled.** Depth as an index into a per-concept understanding ladder. The three owners (concept = ladder/ceiling, program = required floor, student = desired ceiling). The `desiredLevel >= requiredLevel` invariant and its one-directional nature. Reductions handled by mode and outcome, not by depth. The gate's effective-ceiling computation and `desiredLevel`-as-standing-fork.

**Tentative.** The exact shape of the `Program` and `StudentEnrollment` entities (minimal here; will firm up when the institutional scope is built per `STRATEGY.md`). Whether the ladder rungs are a flat ordered list or carry their own prereq structure. The default ladder depth for concepts whose authors do not specify one.

**Open.** Multi-program conflict resolution. The interaction between the depth axis and cross-substrate links. Whether students should be able to see and set their own `desiredLevel` directly in the interface, or whether it should be inferred from their fork behavior over time. (Per-rung confidence, previously open here, is resolved in ADR-0018.)
