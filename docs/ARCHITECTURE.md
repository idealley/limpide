# Architecture

This document describes how Limpide is built. It assumes the reader has read `PEDAGOGY.md` and understands the behaviors the architecture exists to support. Justifications that trace back to pedagogical principles are referenced rather than re-argued here.

## The stack at a glance

```
┌──────────────────────────────────────────────────────────────────────┐
│  Frontend (React + Vite)                                              │
│  - Conversation UI                                                    │
│  - Inline visualizations (diagrams, graphs, marked-up text)           │
│  - Optional pinned-gap awareness sidebar                              │
└──────────────────────────────────────────────────────────────────────┘
                            │  WebSocket
                            ▼
┌──────────────────────────────────────────────────────────────────────┐
│  Cloudflare Workers (edge)                                            │
│  - WebSocket fan-out (Durable Objects)                                │
│  - Webhook receivers (auth callbacks, future inbound channels)        │
│  - Scheduled triggers (Gardner background work, decay calculations)   │
│  - Auth + rate limiting                                               │
└──────────────────────────────────────────────────────────────────────┘
                            │  WebSocket / HTTP
                            ▼
┌──────────────────────────────────────────────────────────────────────┐
│  Agent worker (Node process)                                          │
│  - agent-core runtime                                                 │
│  - Tutor agent (Claude Opus, interactive lane)                        │
│  - Gardner agent (Claude Haiku, analytic lane, debounced)             │
│  - Session orchestration, dispatch, Policy Gate                       │
└──────────────────────────────────────────────────────────────────────┘
                            │  SurrealQL / WebSocket live queries
                            ▼
┌──────────────────────────────────────────────────────────────────────┐
│  SurrealDB                                                            │
│  - Curriculum graph (concepts, prereqs, cross-substrate links)        │
│  - Student subgraph (per-student projections)                         │
│  - Session events (append-only)                                       │
│  - Pinned-gap agenda                                                  │
│  - Foundation rules (read by Policy Gate)                             │
└──────────────────────────────────────────────────────────────────────┘
```

The frontend talks to Workers, Workers talk to the agent process and to SurrealDB live queries, the agent process talks to SurrealDB directly and to LLM providers. The split is deliberate: Workers handle short-lived edge concerns, the agent process handles long-running reasoning, and SurrealDB is the single source of truth.

## Why this stack

**Why SurrealDB.** Limpide's data is genuinely multi-model. The curriculum is a graph (prereq edges, cross-substrate links). Concepts and transcripts are documents. Gap records and encounters are relational. Future curiosity classification will need vector search. Running these as four separate systems (Postgres + pgvector + a graph layer + a document store) means maintaining four mental models, four query languages, four operational stories, and synchronization seams between them. SurrealDB collapses all four into one engine with one query language. The multi-model simplification is the entire point.

It also runs embedded — single Rust binary, no separate server — which matters for the desktop and offline-first paths Limpide needs to support eventually. Same engine in development, production, and edge deployments. Same SurrealQL everywhere.

The honest caveat: SurrealDB 3.0 went GA in February 2026. It's stable enough to build on, but expect to file issues. Pin to a specific 3.x version. The new streaming query engine currently covers read-only statements only. Production deployments at Tencent, Volvo, and Walmart provide some confidence; this is not a research project running on something untested.

See `docs/ADR/0001-database-surrealdb.md` for the full decision.

**Why Cloudflare Workers for the edge.** Workers do three things well that we need: WebSocket fan-out via Durable Objects (frontend reactive state without a separate Convex-style backend), webhook receivers (fast cold starts, signature verification, forward to SurrealDB), and scheduled triggers via Cron Triggers (Gardner background work, confidence decay calculations, end-of-day rollups).

Workers do *not* do agent loops well. The CPU time limit per request is too short for a multi-LLM-call session. The agent process stays as a Node service.

See `docs/ADR/0002-edge-cloudflare-workers.md`.

**Why agent-core.** The framework is already built and matches Limpide's needs structurally — Foundation/Playbook/Task/Event primitives map cleanly onto the pedagogy (see *the AYA primitive mapping* below). The alternative would be building from scratch, which is wasteful when an existing framework fits this well, or adopting LangChain / LlamaIndex / similar, which are general-purpose and would require more adaptation than agent-core needs.

See `docs/ADR/0003-framework-agent-core.md`.

**Why two agents on a shared session.** The Tutor and Gardner have fundamentally different jobs (conversational vs. analytic), benefit from different models (strong reasoning vs. structured extraction), need different latency characteristics (interactive vs. debounced), and must not interfere with each other's outputs. A single agent wearing two hats does both jobs worse and conflates concerns that should stay separated.

See `docs/ADR/0004-two-agent-shape.md`.

## The AYA primitive mapping

Limpide is a Skill on the AYA platform. The primitives map as follows:

| AYA primitive | Limpide use |
|---|---|
| **Foundation** | The pedagogical rules F1–F10 from `PEDAGOGY.md`. Encoded as machine-readable Policy Gate rules. Every Tutor and Gardner action is gate-checked against these. |
| **Playbook** | The six-state session loop, versioned. When the loop changes, encounters log which Playbook version they ran under. |
| **Task** | A tutoring session. Lifecycle: declared (student or scheduler initiates) → delegated (Tutor agent picks up) → in_progress (loop running) → review (end-of-session synthesis) → concluded. |
| **Event** | Session transcripts plus state transitions. Append-only. Gardner reads events; the student subgraph is a projection over events. |
| **Memory** | Curriculum graph, student subgraph, gap records, curiosity signals. SurrealDB-backed. |
| **Dispatch** | The lane-based scheduler. Tutor on the interactive lane, Gardner on the analytic lane (debounced), background work on the background lane. |
| **Policy Gate** | Foundation rules as deterministic gates. ALLOW / DENY / REQUIRE_REVIEW on tool calls, outbound events, and state transitions. |
| **Ontology** | Substrate types, channel types, gap types, outcome types. Canonical vocabulary across all primitives. |

This mapping is the structural reason Limpide on AYA fits better than Limpide on a generic agent framework. The framework already encodes the separation between *why* (Foundation), *how* (Playbook), and *what happened* (Event) that Limpide's pedagogy depends on.

## Required extensions to agent-core

agent-core today is single-agent: an `Agent` owns its session. Limpide needs a session that can be joined by multiple agents with separate identities, separate tool sets, separate LLM clients, but a shared event stream and shared memory. This is a real extension and it benefits Limpide and any future ensemble Skill (research-assistant pairs, debate-pair tutors, etc.).

### Session as a first-class object

```typescript
export interface SessionContext {
  sessionId: string;
  traceId: string;
  userId: string;
  orgId: string;
  state: Record<string, unknown>;  // Skill-specific state
}

export interface Session {
  readonly id: string;
  readonly context: SessionContext;
  readonly memory: Memory;
  readonly policy: Policy;
  readonly events: EventStream;
  readonly dispatch: Dispatcher;

  attach(agent: AttachedAgent): AgentHandle;
  detach(handle: AgentHandle): Promise<void>;

  suspend(): Promise<SessionSnapshot>;
  resume(snapshot: SessionSnapshot): Promise<void>;
  conclude(reason: string): Promise<void>;

  // The conversational floor — exactly one agent holds it at a time.
  // Background agents (Gardner) never request the floor.
  requestFloor(handle: AgentHandle): Promise<FloorGrant>;
  releaseFloor(handle: AgentHandle): Promise<void>;
}
```

The floor concept prevents two agents from speaking to the user simultaneously. Tutor holds the floor by default; Gardner never requests it. Future Skills with multiple speaking agents (debate, role-play) can use the same mechanism with explicit handoffs.

### Attached agents

```typescript
export interface AttachedAgent {
  readonly role: string;              // 'tutor', 'gardner'
  readonly systemPrompt: string;
  readonly tools: Tool[];
  readonly llm: LLMClient;
  readonly lane: DispatchLane;        // 'interactive' | 'analytic' | 'background'
  readonly subscriptions: EventSubscription[];

  onEvent(event: SessionEvent, ctx: AgentRunContext): Promise<AgentTurn>;
}

export interface AgentHandle {
  readonly agentId: string;
  readonly role: string;
  readonly attachedAt: Date;
}

export type DispatchLane = 'interactive' | 'analytic' | 'background';
```

Three lanes:

- **Interactive.** Runs immediately. Holds the floor. Latency budget tight (sub-2-second target for human conversation).
- **Analytic.** Runs on debounce. Never holds the floor. Latency budget: "before the next state transition that needs the analysis."
- **Background.** Fire-and-forget. Periodic curriculum maintenance, telemetry rollups, decay calculations. No consumer waiting.

### The event stream

```typescript
export interface SessionEvent {
  readonly id: string;
  readonly sessionId: string;
  readonly at: Date;
  readonly seq: number;               // monotonic per session
  readonly type: string;
  readonly agentRole?: string;        // who emitted; undefined = system
  readonly visibility: EventVisibility;
  readonly payload: unknown;
}

export type EventVisibility =
  | 'public'         // surfaces to the channel (user sees it)
  | 'internal'       // agents see it, user does not
  | 'agent-only';    // only specific agents see it

export interface EventSubscription {
  types?: string[];
  fromRoles?: string[];
  visibility?: EventVisibility[];
  debounce?: {
    minEvents?: number;
    maxLatencyMs?: number;
    coalesce?: boolean;
  };
}

export interface EventStream {
  emit(event: Omit<SessionEvent, 'id' | 'seq' | 'at'>): Promise<SessionEvent>;
  read(opts: { from?: number; to?: number; types?: string[] }): AsyncIterable<SessionEvent>;
  flush(role: string): Promise<void>;
}
```

The visibility flag is the structural enforcement of Foundation rule F3 (Gardner's analysis stays internal). Gardner emits events with `visibility: 'internal'`. The channel adapter only forwards `visibility: 'public'`. This is enforced at dispatch, not in agent prompts.

### The dispatcher

```typescript
export interface Dispatcher {
  route(event: SessionEvent): Promise<void>;
  awaitLane(lane: DispatchLane): Promise<void>;
  metrics(): DispatcherMetrics;
}
```

The `awaitLane` method coordinates the Playbook with Gardner without polling. When the Tutor finishes Teach-back and is ready to evaluate, the Playbook calls `dispatch.awaitLane('analytic')`, which blocks until Gardner has consumed all pending events. Then the Playbook reads the latest gap records from memory and decides the transition.

### Policy Gate extensions

```typescript
export interface Policy {
  evaluateTool(call: ToolCall, ctx: PolicyContext): Promise<PolicyDecision>;
  evaluateEvent(event: SessionEvent, ctx: PolicyContext): Promise<PolicyDecision>;
  evaluateTransition(from: string, to: string, ctx: PolicyContext): Promise<PolicyDecision>;
  evaluateProbe(probe: Probe, ctx: PolicyContext): Promise<PolicyDecision>;
}

export type PolicyDecision =
  | { decision: 'ALLOW' }
  | { decision: 'DENY'; reason: string }
  | { decision: 'REQUIRE_REVIEW'; reason: string; reviewer: string };
```

The transition gate is where Gardner's structural assessment outranks the Tutor's conversational instinct. Concrete example: Foundation rule F10 (confidence cannot rise without teach-back) is encoded as a transition gate that DENIES the Apply→Update transition if Gardner has not emitted a teach-back evaluation event.

The Tutor sees the denial reason and adapts conversationally; advancing is not on the menu.

**Probe gating.** `evaluateProbe` extends the gate from state transitions to the *content of what the Tutor asks the student*. Every diagnostic utterance (the Teach-back probes, "push one step further," mirror-don't-correct — not bridging or conversational turns) is first labeled with a `disclosure` level by the inline classifier below, then gated deterministically against program-required depth, session mode, and student parameters. The gate keys off **structured fields only** — the `disclosure` enum, recursion depth, mode, student params — never the classifier's free-text reason, which is audit metadata. This is what lets the gate stay deterministic and auditable while the semantic judgment ("how much does this reveal") is isolated in the classifier. Program-governed depth is enforced here as a terminating floor on the why-recursion: the gate computes an effective ceiling = `desiredLevel ?? requiredLevel` against the concept's `understandingLadder`, allows probes targeting a rung at or below it, and denies deeper probes unless an active fork is present. Depth is represented as an index into a per-concept understanding ladder; the floor, ceiling, and invariant (`desiredLevel >= requiredLevel`) are specified in `docs/ADR/0010-concept-depth.md`. The full probe-gating design, alternatives, and the four-layer policy/learning stack are in `docs/ADR/0009-probe-gating.md`.

**The gate's own evolution is gated.** Gardner may *propose* new rules or parameter changes from the probe record, but never enacts changes to the rules themselves. Parameter nudges within hard-coded bounds may auto-apply (logged, reversible); changes to a rule's shape or anything touching a Foundation rule resolve to `REQUIRE_REVIEW` for a human. Learning improves the strategy and the bounded parameters; it can never rewrite the constitution or the policy. This is the meta-level discipline that keeps self-learning from optimizing away its own guardrail.

### The disclosure classifier

A third model role, distinct from Tutor and Gardner. It sits on the interactive path (synchronous, before a probe is dispatched) and does exactly one thing: read a probe's `utterance` and assign a structured `disclosure` label plus a free-text reason. It is **program-agnostic** — it judges the intrinsic revealing-power of the words, not their appropriateness for any student or program — which makes it a pure function of the text: testable in isolation, reusable everywhere, and small. It is Haiku-class or smaller (a fine-tuned or partially deterministic classifier is plausible, since the job is narrow), and it must be fast because it is on the hot path, unlike Gardner which is deliberately debounced. Keeping it separate from the Tutor prevents the agent being measured from grading its own leakage; keeping it separate from Gardner keeps the analytic agent off the interactive path.

### Safeguarding gate

Foundation rule F12 (safety overrides pedagogy, ADR-0014) is enforced at the Policy Gate, above all pedagogical gates. When a student signals distress, self-harm, or abuse, the gate pre-empts the session loop: it suspends Socratic withholding, routes the Tutor to a care-and-resources response, and raises an escalation flag to the human path configured for the deployment. Safeguarding escalation is the single defined exception to the otherwise-strict event internality (F3) and scope model — a harm disclosure may leave the session to reach a human — and the exception is explicit, minimal, logged, and limited to the safeguarding purpose. The detection-and-escalation mechanism lives here; the legal reporting judgment (mandatory-reporting duties, jurisdiction- and canton-specific) belongs to the institution and counsel.

## The data model

All entities live in SurrealDB. The TypeScript interfaces below are mirrored as SurrealDB schema definitions in `surrealdb/schema.surql` (to be written).

### Curriculum graph (shared, slow-changing)

```typescript
interface ConceptNode {
  id: string;                    // stable, never reused
  name: string;
  description: string;
  
  substrate: 'mathematical' | 'textual' | 'rhetorical';
  capability: string;            // what underlying skill this trains
  
  prereqs: string[];             // concept ids
  crossSubstrateLinks: CrossLink[];
  
  understandingLadder: LadderRung[];   // ordered, shallow→deep; the depth ceiling (ADR-0010)
  
  problemPrompts: ProblemPrompt[];     // for Pose state
  teachScaffolds: TeachScaffold[];     // for Teach state
  teachBackProbes: string[];           // Socratic follow-ups
  applicationProblems: ProblemPrompt[]; // for Apply state
  
  curiosityHooks?: CuriosityHook[];    // adjacent teasers
  version: number;
}

interface ProblemPrompt {
  id: string;
  text: string;
  exercises: string[];           // concept ids this problem leans on
  gapSurfaces: GapType[];        // what gaps this exposes
}

interface TeachScaffold {
  id: string;
  channel: 'visual' | 'geometric' | 'numeric' | 'narrative' | 'code';
  representation: string;
}

interface CrossLink {
  toConceptId: string;
  capability: string;            // the shared capability
  resonance: string;             // human-readable: what's the same about them
}

interface LadderRung {
  level: number;                 // 0 = shallowest; index into the ladder
  name: string;                  // e.g. "apply it", "explain why", "derive it", "foundations"
  description: string;
}
```

### Program and enrollment (curriculum tracks)

```typescript
interface Program {                // a curriculum overlay onto the canonical graph (ADR-0010, ADR-0013)
  id: string;
  name: string;                    // e.g. "Vaud — Maths", "Valais — Maths", "EPFL year 1"
  conceptPlan: {
    conceptId: string;             // references the canonical graph; never duplicates a concept
    requiredLevel: number;         // the depth floor (ADR-0010)
    expectedBy?: string;           // grade/stage timing — when a typical student reaches it
  }[];
}

interface StudentEnrollment {
  studentId: string;
  programIds: string[];
}

```

The program-required rung is the default terminating floor the Policy Gate enforces on the why-recursion. A student enrolled in several programs takes the deepest required rung among them (provisional; see ADR-0010). Programs are *overlays*: they reference concepts in the shared canonical graph and annotate them with a required depth and timing, never duplicating the concept. The canonical graph is a deliberate superset of every curriculum so the depth path can run past where any program stops; curricula like Vaud and Valais are two overlays on the same concepts (`docs/ADR/0013-canonical-graph-and-overlays.md`). The student's expressive baseline lives on the `Student` profile below.

The `expressiveBaseline` is the *expected fluency of explanation*, distinct from the depth of understanding required (ADR-0010) and from a probe's disclosure (ADR-0009). Gardner judges teach-back relative to it, with recovery-under-probing as the discriminator of grasp; it never caps how deep a curious student may go. See ADR-0012.

### Student subgraph (per student)

```typescript
interface StudentConcept {
  studentId: string;
  conceptId: string;
  confidenceByRung: RungConfidence[];   // per ladder rung, derived from encounters, not a scalar (ADR-0018)
  desiredLevel?: number;         // student's ceiling; invariant: >= program requiredLevel (ADR-0010)
  lastSeenAt: Date;
  
  channelsTried: string[];
  channelSuccess: Record<string, number>;
  
  openGaps: string[];            // gap record ids
  resolvedGaps: string[];
  curiositySignals: string[];
  
  encounters: Encounter[];
}

interface Encounter {
  sessionId: string;
  at: Date;
  rung: number;                  // which ladder rung this encounter worked at (ADR-0010)
  outcome: 'discovered' | 'delivered' | 'forced_delivery' | 'retreated' | 'partial';
  confidenceDelta: number;
  conceptVersion: number;        // for debugging "why was this reteach"
}

interface RungConfidence {       // derived from the encounter log, never written directly (ADR-0018)
  rung: number;                  // ladder index (ADR-0010)
  estimate: number;              // 0..1, the lossy summary
  observations: number;          // credibility weight — distinguishes 0.6-from-1 from 0.6-from-10
  acquisition: 'discovered' | 'delivered' | 'mixed';  // governs decay rate (discovered decays slower)
}
```

The encounter log is the source of truth; `confidenceByRung` is a cached, recomputable summary of it (ADR-0018). It is per ladder rung, observation-weighted, classified relative to the student's expressive baseline (ADR-0012), gated by F10, and decays differentially — discovered understanding slower than delivered, because discovered understanding survives forgetting (`VISION.md`).

### Gap records

```typescript
interface GapRecord {
  id: string;
  studentId: string;
  conceptId: string;
  sessionId: string;
  at: Date;
  
  type: GapType;
  evidence: string;              // verbatim or summary
  severity: 'low' | 'medium' | 'high';
  signal: number;                // -1..+1, negative = gap, positive = cross-substrate recognition
  resolved: boolean;
  resolvedInSessionId?: string;
}

type GapType = 
  | 'glossed_step'
  | 'missing_prereq'
  | 'wrong_intuition'
  | 'surface_only'
  | 'partial'
  | 'cross_substrate_recognition';  // positive signal
```

### Pinned gaps (the agenda)

```typescript
interface PinnedGap extends GapRecord {
  pinnedAt: Date;
  declinedFromConceptId: string;
  
  returnStrategy: ReturnStrategy;
  returnAttempts: ReturnAttempt[];
  
  nextEligibleAt: Date;
  declineCount: number;
  state: 'active' | 'cooling' | 'dormant' | 'resolved';
}

interface ReturnStrategy {
  kind: 'direct_question' | 'context_bridge' | 'orthogonal_concept';
  questionTemplate?: string;
  bridgeConceptIds?: string[];
  orthogonalConceptId?: string;
}

interface ReturnAttempt {
  at: Date;
  sessionId: string;
  strategy: ReturnStrategy['kind'];
  outcome: 'engaged' | 'declined' | 'deferred';
}
```

### Curiosity signals

```typescript
interface CuriositySignal {
  id: string;
  studentId: string;
  sessionId: string;
  at: Date;
  question: string;
  aboutConceptInitial?: string;  // write-time classification (low confidence)
  aboutConceptFinal?: string;    // read-time classification (higher confidence)
  followed: boolean;
  followUpSessionId?: string;
}
```

The dual classification is intentional: write-time for in-session routing (defer / pivot / chase), read-time for long-term knowledge organization. The gap between the two is itself a useful signal about where concept boundaries are fuzzy.

### Probes (the gated diagnostic utterances)

```typescript
interface Probe {
  id: string;
  sessionId: string;
  conceptId: string;
  at: Date;

  utterance: string;             // public — what the student sees/hears
  intent: string;                // internal — declared by Tutor at probe time
  depth: number;                 // the ladder rung this probe targets (ADR-0010)
  mode: 'exam_driven' | 'exploratory';

  // assigned by the disclosure classifier, not the Tutor
  disclosure: 'open' | 'hint' | 'leading' | 'near_answer';
  disclosureReason: string;      // audit metadata only — never a gate input

  gateDecision: 'ALLOW' | 'DENY' | 'REQUIRE_REVIEW';
  reworkOf?: string;             // prior Probe id, if this is a regeneration after DENY

  // assigned by Gardner on the analytic pass (read-time)
  attribution?: 'student' | 'shared' | 'system';  // who owns the recovery, if any
}
```

The three-author pattern mirrors the write-time/read-time split used for curiosity signals and `aboutConcept`: `intent` is the Tutor's write-time annotation, `disclosure` is the classifier's inline label, and `attribution` is Gardner's read-time judgment. Recovery under probing (`MEASUREMENT.md`) is computed from `disclosure` plus the student's subsequent production; `attribution` is what keeps the system's own prompting out of the student's confidence delta. See `docs/ADR/0009-probe-gating.md`.

### Sessions

```typescript
interface Session {
  id: string;
  studentId: string;
  startedAt: Date;
  endedAt?: Date;
  
  mode: 'exam_driven' | 'exploratory';
  
  stack: SessionStack;
  conceptsTouched: ConceptTouch[];
  gapsEmitted: string[];
  curiosityEmitted: string[];
  pinnedThisSession: string[];
  returnedThisSession: string[];
}

interface SessionStack {
  frames: StackFrame[];          // top is current
}

interface StackFrame {
  conceptId: string;
  enteredAt: Date;
  state: SessionState;           // the six-state machine
  
  parentFrame?: number;
  enteredVia: 'session_start' | 'fork_accepted' | 'return_path' | 'cross_substrate_invite';
  
  promisedReturn?: string;       // PinnedGap id if a fork was declined here
}

type SessionState = 
  | 'select' | 'pose' | 'struggle' | 'teach' 
  | 'teach_back' | 'apply' | 'update';
```

### Student profile

```typescript
interface Student {
  id: string;
  name: string;
  age?: number;                  // developmental anchor for the expressive baseline (ADR-0012)
  channelPreferences: Record<string, number>;
  defaultMode: 'exam_driven' | 'exploratory';
  metacognitionLevel: 'low' | 'medium' | 'high';
  expressiveBaseline: {          // expected fluency of explanation, may vary by substrate (ADR-0012)
    substrate: 'mathematical' | 'textual' | 'rhetorical';
    level: number;               // expected articulation, NOT depth of understanding
  }[];
  baselineSource: 'declared' | 'inferred';
}
```

Metacognition level is the dial that determines how strong an external scaffold the Teach-back state needs. Low metacognition → the Tutor explicitly invites cross-checks ("now write that down and read it aloud"). High metacognition → gentler probes that trust the student to catch their own glosses.

## Confidence math

First-cut magnitudes, now interpreted **per ladder rung** and derived from the encounter log rather than written directly — the full model is in `docs/ADR/0018-confidence-model.md`. Tunable as real students use the system.

| Outcome | Base delta |
|---|---|
| Discovered (clean teach-back, applied successfully) | +0.40 |
| Partial discovered (one retry, then applied) | +0.25 |
| Delivered, chosen (student declined fork) | +0.15 |
| Forced delivery (student wanted to fork but couldn't) | +0.05 |
| Retreated to prereq (parent unchanged) | 0.00 to parent, +0.10 to prereq if resolved |
| Untouched concept | -0.02 / week (forgetting) |

In exam-driven mode, all positive deltas multiply by 0.7 (because the mode prioritizes pass-the-test over genuinely-internalized). In exploratory mode, all positive deltas use the base value. The outcome itself is classified relative to the student's expressive baseline (ADR-0012), and confidence decays differentially — discovered slower than delivered (ADR-0018).

The structural commitment: discovered is meaningfully more than delivered, and the system is honest with itself about which kind just happened. The full model — per-rung, observation-weighted, derived-from-history, differentially decaying — is `docs/ADR/0018-confidence-model.md`.

## Cooldown values for pinned gaps

| Event | Cooldown |
|---|---|
| Initial decline | 24 hours |
| First failed re-engagement | 7 days, same strategy |
| Second failed re-engagement | 21 days, **strategy must change** |
| Third failed re-engagement | 60 days, gap moves to `dormant` |
| Successful engagement | gap resolves, increment `learnedAfterNAttempts` counter |

The `learnedAfterNAttempts` counter feeds back into Gardner's strategy selection over time — students whose declined gaps eventually resolve after two attempts have different patience curves than students whose declined gaps tend to resolve immediately or never. Per-student calibration emerges from the data.

## Model selection

Heterogeneous models, intentionally:

- **Tutor: Claude Opus.** The tutoring loop demands strong reasoning — Socratic probing, judging when an explanation is real vs rehearsed, knowing when to bridge vs deliver, reading the student's affect from text. This is not a small-model task.
- **Gardner: Claude Haiku.** Structured extraction from text. Smaller, faster, cheaper. Specialized prompt with strict output schema.
- **Background work: Claude Haiku or smaller.** Periodic curriculum maintenance, agenda audits, decay calculations.

agent-core's `setLLMClient` is per-agent-friendly. No shared model.

## Data scopes and consent boundaries

Limpide's data is organized into four nested scopes. The boundaries between scopes are enforced at the storage layer and at the Policy Gate. Data flows *up* through the scopes only with explicit consent and only as anonymized aggregates; data never flows *down* without authorization.

This four-scope model is shared across the AYA platform — Limpide, Aegilo (geopolitical intelligence), and any future Skills use the same architecture. Building it correctly once means every Skill inherits sovereign-by-default storage and federated-with-consent network features.

### The four scopes

**Personal scope.** Owned by the individual user (student, employee, analyst). Includes the personal subgraph, individual gap records, encounter history with full discovered/delivered annotations, the conversation transcripts, and the curiosity signals. The user can request deletion at any time. The institution sees only what is operationally necessary to do its job (a teacher needs to know roughly where a student stands; an employer needs to know that an employee completed required training); the institution does not see the raw transcripts or fine-grained gap records.

**Cohort scope.** Anonymized, aggregated patterns across a group: a class, a project team, a department. The cohort is the natural unit at which a teacher operates, a manager operates, a team lead operates. Cohort data tells the operator how the group is doing, where the median struggle points are, what channels work for this group. Individual identification is structurally prevented by the aggregation; the cohort scope does not allow drilling down to a named individual.

**Institutional scope.** Longitudinal patterns across the whole school, company, or organization. This is the scope that produces evidence of institutional efficacy — how does pedagogy at this school develop students over four years, how does work actually get done at this company, what cultural patterns emerge. The institution owns this data. Aegilo cannot read it without the institution's explicit grant. For a hosted Limpide consumer deployment, the institution is the user themselves; the institutional scope collapses into the personal scope.

**Network scope.** Anonymized aggregations contributed across institutions, only with consent at the institutional level, only through privacy-preserving aggregation (differential privacy, secure aggregation, or both). The network scope produces comparative benchmarks: "students at your school make X cross-substrate connections per quarter; the network median is Y." For Aegilo: "your sector's procurement velocity median is X; you are 2.3 standard deviations below." Institutions opt in to the network and receive network benchmarks in exchange for their contributions. They can opt out at any time; their existing contributions remain in the network as historical aggregates that cannot be reversed (the privacy mechanism prevents extraction even by the contributor).

### Child data and controllership

Because most users are minors, the controller of personal-scope data is named explicitly per deployment (ADR-0014). In school deployments the institution is the controller and consent gateway (in the US, FERPA's school-official exception, which the sovereignty layout fits provided Aegilo never becomes a controller and never reuses or unilaterally deletes records in a way that breaks the school's direct control). In consumer deployments the guardian/user is the controller, with verifiable parental consent below the digital age of consent (COPPA in the US, GDPR Art. 8 in the EU) and a capacity-of-judgment model in Switzerland. Children's personal data never enters the network scope — only anonymised aggregates do, which is what keeps the federated-benchmark feature compatible with child-data law by construction. The platform's design baseline is best-interests-of-the-child / Age Appropriate Design Code: high privacy by default, data minimisation, no profiling or advertising of children.

### The architectural commitments

Three commitments make the scope model trustworthy:

**Sovereign storage by default.** Personal and institutional data live in the institution's own SurrealDB instance — embedded for desktop, dedicated cloud account for SaaS, on-premise for enterprise. Aegilo, as the platform operator, does not have read access to this data. This is enforced by the storage layout (each institution's database is logically and ideally physically separate) and by SurrealDB's permission model (Aegilo's operational role cannot grant itself read access to institutional data without the institution's authorization).

**Aggregation happens at the source.** When an institution opts into the network, the aggregation computation runs in the institution's environment. Raw data does not leave the institution. Only the differentially-private aggregates are pushed to the network service. This is technically more demanding than centralized aggregation, but it's the only architecture compatible with the data sovereignty commitment.

**Network value flows back to contributors.** The benchmarks computed from the network are visible only to institutions that contribute. Non-contributing institutions can use Limpide for personal and institutional purposes but do not receive network benchmarks. This is the incentive design: contributing makes the network more valuable, and the network's value is structurally returned to those who contribute. Pricing reflects this — the standard tier includes network participation; an opt-out tier exists for institutions that cannot share aggregated data (defense, certain healthcare and finance contexts) and they pay a premium for the additional isolation.

The cross-Skill consequence: once this is built for Limpide, it serves Aegilo's geopolitical intelligence (companies opt into network risk benchmarks while keeping their proprietary supply-chain data sovereign) and any future Skill the same way. The federated-with-consent architecture is the platform's most defensible long-term position.

### Phasing

Phase 1 (MVP): single-tenant deployments only. Personal and institutional scopes are implemented; cohort scope is implemented as institutional aggregations; network scope does not yet exist. The schema distinguishes the scopes from day one so that retrofitting is not required.

Phase 2 (post-launch): the cohort scope is exposed to operators (teachers, team leads) with the appropriate read views. Network features are not yet active.

Phase 3 (when there are enough institutional deployments to make benchmarks meaningful, probably year 2–3): network features are introduced. Differential privacy parameters are tuned, the aggregation infrastructure is deployed, the consent flows are added to the institutional admin interface, the first benchmarks are computed and shared back.

The discipline that protects this phasing: **the Phase 3 features must be designed in advance and must not require structural changes to the Phase 1 schema.** A schema that distinguishes `personal | cohort | institutional | network` scopes from the first migration, even when only `personal` is being written, preserves the option to build the rest. Adding scope distinctions to a schema that didn't have them is the hardest refactor in software; we are paying the schema-design cost up front to avoid it.

## Deployment topologies

**Local-first / desktop.** SurrealDB embedded as a library in the agent process. Single binary deployment. No edge layer. Frontend talks directly to the agent process. Useful for development, for highly privacy-sensitive deployments (a school running its own instance), and eventually for the offline-capable mode.

**Hosted / multi-tenant.** SurrealDB in server mode (single-node initially, distributed cluster eventually). Cloudflare Workers as the edge. Agent process running on a dedicated host or container. Frontend served via Cloudflare Pages. This is the production deployment for the public service.

**Hybrid.** Agent process running in the cloud, but the student's session data remains in a local SurrealDB instance that syncs selectively. This is the path for institutions that want hosted reasoning but local data sovereignty. Not in scope for MVP; the data model should not preclude it.

## The migration path from AYA's Convex to SurrealDB

Documented in `docs/ADR/0005-aya-migration-to-surrealdb.md`. Order, briefly:

1. New SurrealDB-backed Memory implementation in agent-core. Existing skills continue using Convex/InMemory/Falkor backends unchanged.
2. Limpide ships on the SurrealDB Memory backend, with its own SurrealDB instance. Patterns are proven in production.
3. AYA's other primitives (Tasks, Events, Foundation) port to SurrealDB in one cohesive pass — the boundaries between primitives are too tight for piecemeal migration.
4. Convex deprecated for AYA. Existing data migrated. WebSocket subscriptions move from Convex's reactive queries to Cloudflare Workers proxying SurrealDB live queries.

Step 4 is months out. Steps 1–2 are the immediate work.

## Status of this document

**Settled.** The overall stack (SurrealDB + Workers + Node agent process + agent-core). The two-agent-on-shared-session shape. The Session/AttachedAgent/Dispatcher interface signatures. The data model entities. The AYA primitive mapping. The four-scope data model (personal, cohort, institutional, network) and the phased introduction of federated network features. Probe gating: the `Probe` entity, `evaluateProbe`, the inline disclosure classifier as a third model role, and the four-layer policy/learning stack with propose-vs-enact discipline on rule evolution (`docs/ADR/0009-probe-gating.md`). Concept depth as an index into a per-concept understanding ladder, with the program-required floor and student-desired ceiling and the `desiredLevel >= requiredLevel` invariant (`docs/ADR/0010-concept-depth.md`). The `expressiveBaseline` on the student model and the three orthogonal axes — depth, expression, disclosure (`docs/ADR/0012-expressive-baseline.md`). The canonical concept graph as a timing-free superset with programs as referencing overlays (`docs/ADR/0013-canonical-graph-and-overlays.md`). The safeguarding gate (F12) and the per-deployment child-data controller model (`docs/ADR/0014-safeguarding-and-child-data.md`). The four-layer system-evaluation stack as the evidence gate for the self-learning loop (`docs/ADR/0016-system-evaluation.md`), and the interactive-path latency/cost budget (`docs/ADR/0017-latency-cost-budget.md`). Per-rung, history-derived, differentially-decaying confidence (`docs/ADR/0018-confidence-model.md`).

**Tentative.** The specific confidence math values (the statistical form and decay constants are in `docs/ADR/0018-confidence-model.md`). The exact cooldown durations. The choice of Claude Opus vs other strong-reasoning models for Tutor. The disclosure enum values and the classifier's model (prompted, fine-tuned, or partially deterministic). The blast-radius thresholds separating auto-applicable parameter nudges from human-review rule changes. The exact shape of the `Program` and `StudentEnrollment` entities and the authoring of per-concept ladders and required rungs (`docs/ADR/0010-concept-depth.md`). The Workers Durable Objects design for WebSocket fan-out (probably right, not yet built). The specific differential privacy parameters and aggregation library choice for Phase 3 network features.

**Open.** The frontend architecture in detail (we have constraints but no concrete spec). The auth flow (likely Clerk or similar, not decided). The licensing of the agent-core extensions (must coordinate with existing agent-core license). The attribution function and its external validation, and classifier calibration methodology (`MEASUREMENT.md`, ADR-0009). The contractual structure that codifies "Aegilo never sells the network's data" — corporate articles, customer contracts, third-party attestation, or all three.
