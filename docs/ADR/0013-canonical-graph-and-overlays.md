# ADR-0013: Canonical concept graph and curriculum overlays

**Status:** Proposed
**Date:** 2026-05-29

## Context

ADR-0010 introduced `Program` with a per-concept required depth, and the curriculum-authoring discussion (#1) settled on "lazy but smart" sourcing. Two questions remained: how do specific curricula (Vaud, Valais, a Ukrainian curriculum, EPFL first year, a hobbyist topic) relate to the universal space of concepts, and how large is that space?

Two observations force the answer:

1. **Curricula disagree on what is taught, when, and to what depth.** A concrete case: a student schooled in Ukraine learned mathematics and physics at secondary level that the Swiss system teaches only in the first or second year of EPFL. If timing or required depth were properties *of a concept*, this could not be represented — the concept would carry one grade and one depth for everyone. The difference is real and must live somewhere.

2. **The platform's ambition is polymaths** (`VISION.md`). The best students go far beyond any national curriculum. So the universe of concepts the system models cannot be bounded by any one curriculum; it must exceed all of them.

## Decision

**One canonical concept graph, many curriculum overlays.**

**The canonical concept graph** is the single shared source of truth for *what there is to learn*: concepts, prerequisites, understanding ladders (ADR-0010), and cross-substrate links. It carries **no timing and no "required" depth** — only what a concept is, what it depends on, and how deep its ladder goes. It is deliberately a **superset** of every curriculum: the deep ladder rungs and the cross-substrate links that produce polymaths are precisely what curricula omit, so the graph must contain more than their union, not less. It grows lazily (see sourcing).

**A program is an overlay** that *references* the canonical graph and never duplicates it. A program is a selection of concepts plus, per concept, a **required depth** (the floor, ADR-0010) and a **grade/stage timing** (when a typical student on this program reaches it). Vaud's curriculum, Valais's curriculum, a Ukrainian curriculum, "EPFL year 1," and "self-directed polymath" are all overlays on the same concepts. A student may be enrolled in several (the deepest required rung wins where they differ, per ADR-0010).

The consequence that makes this non-negotiable: **timing and required depth live only on the overlay.** That is what lets the same `calculus-fundamental-theorem` node be required at grade 11 in one overlay and at EPFL-year-1 depth in another — the Ukraine/Vaud case from Context, representable only because the concept itself is timing-free.

**Benchmarking across overlays.** Because timing lives on overlays and a student's actual trajectory lives in their subgraph, a student can be positioned against *any* overlay at once: "two years ahead of Vaud," "already at Ukrainian grade 9," "approaching EPFL year 1." Multiple yardsticks from one graph — the efficacy evidence in `STRATEGY.md`, and the way the platform shows a polymath how far ahead they are.

**The minimal canon is an overlay.** `VISION.md`'s minimal set of things every mind needs is not a separate structure; it is the most important overlay onto the open-ended graph. The graph exceeds it for the depth path.

**Sourcing — lazy but smart.** Where an authoritative curriculum exists (the Vaud and Valais cantonal curricula; analogues elsewhere), import it as an overlay and seed any of its concepts not yet present in the canonical graph; then LLM-draft the ladders and probes and human-author the cross-substrate links on top (the authoring split from #1). Where no formal curriculum exists (e.g. a hobbyist learning metal-cutting speeds), author the concepts lazily, on demand, when a student first needs them. The authoritative imports give the skeleton and the timing metadata; Limpide adds the pedagogical apparatus the curricula never contained.

## Alternatives considered

**A separate full graph per curriculum.** Each canton (or country) owns its own complete concept graph. Rejected — massive duplication, no shared concept identity, impossible to benchmark a student across curricula, and the expensive cross-substrate links would be re-authored in every silo.

**Timing and required depth on the concept itself.** Simpler schema. Rejected — it cannot represent that curricula differ in when and how deeply a concept is taught (the Ukraine/Vaud case breaks it outright). This was already decided in ADR-0010; ADR-0013 reaffirms it with the overlay model and the motivating evidence.

**Bound the canonical graph to the national curriculum.** Model only what the Maturité covers. Rejected — it caps the graph at exactly the point the depth path and the polymath ambition begin, defeating the platform's reason for existing.

## Consequences

**Commits us to:**
- The canonical concept graph as the single source of truth for concepts, prerequisites, ladders, and cross-links — timing-free and a deliberate superset.
- `Program` as a referencing overlay: concept selection + required depth + grade timing, never duplicating concepts.
- Multi-overlay enrollment and cross-overlay benchmarking.
- Lazy-but-smart sourcing: authoritative imports where they exist, on-demand authoring for the tail.

**Precludes:**
- Concept duplication across curricula, and curriculum-bound depth.
- A graph whose ceiling is any national curriculum.

**Opens:**
- Pilot scope: which canton and which subject to model first. Both Vaud and Valais curricula are accessible (students available in both); the recommendation is one subject the team knows well, modeled in one canton, then the second canton added as a parallel overlay to prove the cross-overlay benchmarking.
- The representation of timing (grade band vs. typical-age vs. ordinal position) and how to resolve *timing* conflicts across overlays (depth conflicts already resolve to the deepest rung).
- How much of a cantonal curriculum is machine-importable versus needs manual modeling.
- Governance of the canonical graph: who owns and edits the shared superset as multiple institutions contribute overlays, and how that interacts with the data-scope model in `ARCHITECTURE.md`.

## Status

**Settled.** One canonical concept graph as a timing-free superset; programs as referencing overlays carrying required depth and timing; timing/depth never on the concept; multi-overlay benchmarking; the minimal canon as an overlay; lazy-but-smart sourcing.

**Tentative.** The timing representation. The import mechanics for cantonal curricula. The pilot scope (canton + subject).

**Open.** Cross-overlay timing-conflict resolution. Governance of the shared canonical graph across contributing institutions.
