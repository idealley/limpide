# ADR-0003: Build on agent-core, not from scratch

**Status:** Accepted
**Date:** 2026-04-29

## Context

Limpide needs an agent runtime: tool loop, session state, memory abstraction, policy gating, multi-channel delivery. These are exactly what an agent framework provides.

The author already has `@aegilo/agent-core` — a TypeScript framework with these primitives — and AYA, a broader assistant platform built on agent-core that introduces the Foundation/Playbook/Task/Event/Memory/Dispatch/Policy Gate/Ontology primitives.

Three options: build a Limpide-specific runtime from scratch, adopt a third-party framework (LangChain, LlamaIndex, Mastra, etc.), or extend agent-core and ship Limpide as an AYA Skill.

## Decision

Limpide ships as a Skill on AYA, running on agent-core with extensions. The AYA primitives map cleanly onto the pedagogy:

- **Foundation** holds the pedagogical rules F1–F10 from `PEDAGOGY.md`. They become machine-checkable Policy Gate rules rather than prose in a system prompt.
- **Playbook** holds the six-state session loop, versioned. Encounters log which Playbook version they ran under.
- **Task** is a tutoring session, with the standard declared → delegated → in_progress → review → concluded lifecycle.
- **Event** is the append-only transcript plus state log.
- **Memory** holds curriculum graph, student subgraph, gap records, curiosity signals.
- **Dispatch** with priority lanes runs Tutor (interactive) alongside Gardner (analytic, debounced).
- **Policy Gate** enforces Foundation rules deterministically.
- **Ontology** is the canonical vocabulary: substrates, channels, gap types, outcomes.

agent-core requires extensions to support multi-agent sessions (see ADR-0004). These extensions benefit any future ensemble Skill.

## Alternatives considered

**Build from scratch.** Considered briefly. Rejected because the patterns we'd build are exactly the patterns agent-core already has. Rebuilding would be a months-long detour from the actual work.

**LangChain / LangGraph.** Mature, large ecosystem, lots of integrations. Rejected because LangChain's abstractions are oriented toward pipelines and chains rather than long-lived sessions with persistent state, and because the framework's complexity has grown faster than its coherence. Adapting it to Limpide's pedagogy would require fighting its grain.

**LlamaIndex.** Strong on retrieval-augmented patterns, weak on multi-agent orchestration. Limpide's needs are agent-shaped, not RAG-shaped.

**Mastra / Inngest / similar newer frameworks.** Promising, but adopting them would mean abandoning agent-core and AYA, which already encode the right separation of concerns for what Limpide needs. The fit between agent-core/AYA primitives and Limpide's pedagogy is unusually clean — better to extend than to start over elsewhere.

**Direct LLM API calls without a framework.** Possible for the simplest version. Rejected because the moment we need session persistence, tool loops with policy gating, and multi-agent coordination, we'd be reinventing a framework. The framework abstractions are earning their keep here.

## Consequences

**Commits us to:**
- agent-core as a dependency, with Limpide following its evolution.
- Contributing the multi-agent extensions back to agent-core (ADR-0004), since they benefit any ensemble Skill.
- AYA as the home repository structure. Limpide is a Skill, not a separate project.

**Precludes:**
- Easy adoption by users who want Limpide standalone, without AYA. (Workaround: agent-core + the Limpide Skill can run without the rest of AYA's infrastructure if desired; AYA's orchestration is value-add, not required.)
- Adopting frameworks that come with their own Foundation/Policy/Memory abstractions — the AYA primitives are the canonical source for these in Limpide.

**Opens:**
- Reuse of AYA's Foundation/Playbook/Policy machinery. Limpide doesn't reinvent them.
- The path for additional Skills (research assistant, writing tutor, debate partner) to share the same multi-agent extensions.
- A cleaner story for governance: Foundation rules live in the database, are versioned, are auditable.

## Status

**Settled.** Reopening requires demonstrated mismatch between agent-core / AYA and Limpide's pedagogy that can't be resolved by extension.
