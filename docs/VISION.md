# Vision

## The question Limpide exists to answer

In a world where knowledge is always available, what is the minimal set of things a person needs to *understand* — not just have access to — in order to develop a critical mind, the capacity to reason, and the ability to argue clearly?

This is an old question. It has been asked by every serious educational tradition: the Greek paideia, the medieval trivium and quadrivium, the German *Bildung*, the Swiss Maturité Fédérale. They have not agreed on the answer, but they have agreed on the shape of the answer. A person who can think well needs three capabilities, each requiring a specific kind of training:

- **Rigorous reasoning** — mathematics taken far enough that proof becomes second nature, plus enough physics to feel what happens when a model meets reality.
- **Deep reading** — a small number of texts read slowly enough that the reader has internalized minds genuinely different from their own.
- **Clear expression** — rhetoric, the master skill the moderns have abandoned, the thing that makes the other two visible.

These three are not interchangeable. A person with deep reading but no rigor produces beautiful confusion. A person with rigor but no reading produces clean answers to wrong questions. A person with both but no rhetoric is invisible.

Limpide is built to train all three as a single integrated capability.

## What we are pushing against

Most ed-tech, even when started with the right intentions, ends up optimizing for engagement metrics that quietly destroy what they claim to measure. The pattern is recognizable: students complete more lessons, "master" more concepts, score higher on adaptive tests — and emerge unable to explain what they learned, unable to argue, unable to read anything that wasn't designed for them.

The structural failure is in the loop. When a system rewards *delivered* understanding the same as *discovered* understanding, the system optimizes for delivery, because delivery is faster. The student receives an explanation, says they understand, the confidence score goes up, the next concept loads. Nothing was learned. The metric was satisfied.

The failure mode has a name: the Klarna Customer Service problem. An AI agent resolved tickets faster than any human team while quietly destroying customer relationships, because tickets-resolved was measurable and relationship-quality was not. Education has the same shape. Performance is captured cleanly by a *state* metric — a score, a completion, a confidence number. Understanding is not; it shows up only in the *process* of a student explaining themselves and either rebuilding through a gap or collapsing at it. A system that watches only the state cannot tell the two apart, so it optimizes for performance and calls it understanding. The early framing of this — "understanding is unmeasurable" — was imprecise and worth correcting: understanding is measurable, but only through process, and process measurement carries its own discipline. That is the subject of `MEASUREMENT.md`.

Limpide is structurally committed to the difference.

## The principle

> Understanding is what survives the student's attempt to explain it.

This is the platform's first commitment, the one from which everything else follows. A student does not learn by being told. A student learns by being asked to teach what they think they know, hitting the wall in their own explanation, recognizing the wall, and choosing what to do about it.

The Tutor's job is not to explain. The Tutor's job is to ask the student to explain, to listen carefully, and to ask the next question. The student does the work of noticing their own confusion. The system holds the structure that makes noticing possible.

This is sometimes called Socratic, sometimes called Feynman-style, sometimes called maieutic. The names matter less than the substance: *the student must do the active work, or no understanding happens*.

## Learning versus understanding

The conventional verb for what tutoring platforms do is *learn*. Limpide rejects this verb as the framing of its own work, and the rejection is structural rather than rhetorical.

*Learning* in the conventional sense is information transferred, retained, retrievable on a test. The student is the receptacle; the curriculum is the content; success is measured by how much of the content the receptacle holds. This is what most ed-tech optimizes for, because it's what's measurable, and it's what produces students who pass tests and forget the material the following week.

*Understanding* is different. Understanding is information that has become structurally part of how the student thinks. It can be rebuilt from first principles. It connects to other understandings. It changes how the student reads the next thing they encounter. A student who has *understood* something cannot un-understand it without effort, in the way that a student who has *learned* something can passively forget it.

The French distinction between *apprendre* and *comprendre* tracks this difference. The English language has lost the distinction, which is part of why the conventional verb causes confusion. Limpide is in the *comprendre* business — *understanding* in English, with full awareness that the English word is the closest available approximation rather than a perfect translation.

The Greek tradition called this kind of knowledge *gnosis* — knowledge by acquaintance, knowledge that has become part of the knower, distinct from *episteme* (theoretical knowledge) and *techne* (practical skill). Limpide aims at *gnosis*. The student doesn't end a session having learned about springs; they end having understood springs in a way that is now structurally part of how they think about force and displacement.

This distinction is not pedagogical pedantry. It determines what the system optimizes for at every level: which verbs the Tutor uses, which signals Gardner attends to, what counts as a confidence increase, what the Foundation rules protect against. A platform that confuses learning with understanding will optimize for the side of the distinction a state metric can see and quietly destroy the side that lives only in process. Limpide is structurally committed to the harder verb, and to measuring it the harder way (`MEASUREMENT.md`).

## The cross-substrate vision

Mathematics, texts, and rhetoric are not three subjects. They are three *substrates* through which the same underlying capability — clear thinking — is trained.

A student who has derived something from first principles has felt what an argument is supposed to feel like when it works. They will read philosophy differently afterward. A student who has read Plato's *Meno* slave-boy passage closely has seen the same constructive-from-premises move that Euclid uses, in a completely different costume. A student who has had to defend an argument they actually hold, against questions they didn't anticipate, has discovered that "having an argument" and "having a position with reasons attached" are not the same thing.

The connections between substrates are where the deepest learning lives. Limpide makes them explicit. The curriculum graph carries cross-substrate links — Euclid's Proposition 1 connects to Plato's *Meno* on the capability of *constructive reasoning from agreed premises*, which connects to the writing of an argument that builds toward a conclusion the reader didn't start with. When a student moves between these, with the right questions asked at the right moments, they discover that they are practicing the same skill in three forms.

This is the depth path. Most students will not start here. They will arrive with an immediate need — a test on Friday, a homework problem, a concept they couldn't follow in class — and Limpide must serve that need without lecture. The depth path is what the platform *can* do. The urgent-need path is what it *must* do, for everyone.

## The two paths and their relationship

**The urgent-need path** is the entry point for almost every student. They come because they don't understand springs, or they have a probability problem due tomorrow, or their teacher said something about derivatives that didn't make sense. Limpide honors the urgency. It asks the student to teach what they already know about the topic. The teaching attempt exposes the local gap and often a deeper gap underneath. Limpide offers to fork — would you like to spend ten minutes on the deeper thing first, or get the local answer and come back? The student chooses. Whichever they choose, the system delivers what they need by Friday.

**The depth path** is the cross-substrate triad work, where the student is in exploratory mode and has the time to make connections explicit. This is the path that produces thinkers. It is the path that earns the platform its reason for existing. But it is reached only by students who first found the platform useful at the surface, trusted it, and came back when the urgency had passed.

The architectural commitment: Limpide must be unimpeachably good at the surface work in order to earn the right to do the depth work. Any tendency to lecture about depth when the student needs help with springs is the failure mode that kills the platform's credibility. The Foundation rules enforce this. The Tutor can carry the depth work in its head; it must not deploy it inappropriately.

## What success looks like

A student who has used Limpide for a year, working through urgent needs and occasionally going deep, should be able to:

- Explain something they understand to someone who doesn't, in a way that builds the explanation rather than asserting it.
- Read a text they haven't seen before — a primary source, an unfamiliar argument, a piece of writing that wasn't designed for them — and follow its reasoning.
- Notice when their own argument has a gap, before someone else points it out.
- Recognize when a problem in one domain has a structural cousin in another.
- Hold a position under questioning without either collapsing or hardening defensively.

These are not curriculum outcomes. They are person-shaped outcomes. The curriculum is the means; the person is the end.

## Three scales, one principle

Limpide serves the individual student. It also serves institutions — schools that deploy it for their students, organizations that deploy it for their people — and eventually a network of institutions that share anonymized patterns to learn from each other. These three scales — individual, institutional, network — are not separate products. They are the same product seen at three magnifications, governed by the same principle: *understanding emerges from articulation, and articulation is what we make possible.*

The vision's claim is only that these scales exist and share one principle. The same Foundation rules — discovered beats delivered, the student does the noticing, gaps stay internal, never deceptive — apply at every scale; F1 protects the individual student during a session, and the four-scope data architecture (`docs/ARCHITECTURE.md`) protects the individual within their institution and the institution within the network. The rest of this document stays at the individual scale, because that is where the educational thesis lives.

The commercial shape of the three scales — who pays, for what, why it is defensible, how the institutional product is the measurement record of `MEASUREMENT.md` sold to a buyer — has been moved to `STRATEGY.md` so the vision can remain about the student. What matters here is only that the three-scale model is what gives Limpide a coherent path from a tutoring tool to infrastructure for thinking-as-it-actually-develops, in people and in the institutions that house them.

## The honest limits

Limpide does not replace teachers. A skilled human teacher who can sit with a student for an hour and read the room is doing things no system will match in the foreseeable future. What Limpide can offer is *the patient interlocutor at scale* — the thing a student usually doesn't have access to, the thing that asks the next question and waits for the answer and doesn't get tired and doesn't judge. This is a real intervention, but it is one piece of an education, not the whole thing.

Limpide also does not replace reading. Sitting with a primary text — Plato, Euclid, a real piece of literature — and working through it slowly is the activity. Limpide can help the student read more carefully, can ask the questions that make the reading active, can connect the text to other texts and to other substrates. It cannot read for the student.

Finally, Limpide does not protect students from frustration. The pedagogical commitment is that productive failure precedes understanding. A platform that smooths every difficulty is a platform that prevents learning. The Tutor's restraint — its refusal to give the answer when struggling is the right state — is not a bug to be fixed. It is the work.

## Status of this document

**Settled.** The principles in this document are the foundation everything else is built on. Reopening any of them requires a new ADR with serious justification. The existence of the three-scale model (individual, institutional, network) is part of the settled vision and the architecture supports it; its commercial development lives in `STRATEGY.md`. The reframed measurement claim — understanding is measurable only through process, not by state metrics — is settled here and developed in `MEASUREMENT.md`.

**Tentative.** The specific list of capabilities trained by the cross-substrate triads, beyond the first one (constructive reasoning from premises) is a working hypothesis. The first triad is concrete; the next ten will need to be designed and tested as the platform matures.

**Open.** Whether Limpide is eventually owned by a mission-locked company, a foundation, or a steward-ownership structure. The decision can wait until there is something working to govern. See `docs/ADR/0006-licensing.md` and the related governance ADR (when written). The commercial open questions — pricing, go-to-market sequencing, and the corporate structure that codifies the network's data discipline (Aegilo never sells contributed data) — now live in `STRATEGY.md`.
