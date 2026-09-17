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

The failure we want to avoid is in the loop: treating receipt of an explanation as equivalent to independently demonstrating understanding. The student receives an explanation, says they understand, the confidence score goes up, the next concept loads. The metric was satisfied without establishing that the student can reason independently.

The failure mode has a name: the Klarna Customer Service problem. An AI agent resolved tickets faster than any human team while quietly destroying customer relationships, because tickets-resolved was measurable and relationship-quality was not. Education has the same shape. Performance is captured cleanly by a *state* metric — a score, a completion, a confidence number. Understanding needs richer evidence, including the process of explaining and the ability to apply an idea independently later. A system relying on a completion or self-report can confuse assisted performance with understanding. The early framing of this — "understanding is unmeasurable" — was imprecise and worth correcting: process evidence can reveal understanding, and it needs validation against independent outcomes. That is the subject of `MEASUREMENT.md`.

Limpide is structurally committed to the difference.

## The principle

> Understanding is what survives the student's attempt to explain it.

This is a guiding principle: invite students to explain what they think they know, notice where their reasoning breaks, and rebuild with appropriate support. A patient interlocutor can make confusion safe to express and remember unfinished work across sessions.

The Tutor listens, questions, demonstrates, explains, and corrects according to what helps the learner. Active thinking matters, but it can follow explicit instruction as well as discovery. Helping a student notice is a preferred move, not a requirement to withhold useful help until they rediscover everything themselves (ADR-0021).

## Learning and understanding

Limpide emphasizes durable conceptual understanding: ideas the learner can explain, apply in unfamiliar situations, and connect to other ideas. The distinction from merely completing a task or recalling a phrase expresses the ambition. Learning is broader than passive information transfer, and receiving an explanation can contribute to understanding.

Whether a particular teaching method produces stronger retention or transfer is an empirical question. Understanding can weaken over time. The system must test its assumptions against delayed, independent demonstrations rather than encode the desired durability into its scores and report that as evidence. Process records and independent outcomes complement one another (`MEASUREMENT.md`).

## The cross-substrate vision

Mathematics, texts, and rhetoric are not three subjects. They are three *substrates* through which the same underlying capability — clear thinking — is trained.

A student who has derived something from first principles has felt what an argument is supposed to feel like when it works. They will read philosophy differently afterward. A student who has read Plato's *Meno* slave-boy passage closely has seen the same constructive-from-premises move that Euclid uses, in a completely different costume. A student who has had to defend an argument they actually hold, against questions they didn't anticipate, has discovered that "having an argument" and "having a position with reasons attached" are not the same thing.

The connections between substrates are where the deepest learning lives. Limpide makes them explicit. The curriculum graph carries cross-substrate links — Euclid's Proposition 1 connects to Plato's *Meno* on the capability of *constructive reasoning from agreed premises*, which connects to the writing of an argument that builds toward a conclusion the reader didn't start with. When a student moves between these, with the right questions asked at the right moments, they discover that they are practicing the same skill in three forms.

This is the depth path. Most students will not start here. They will arrive with an immediate need — a test on Friday, a homework problem, a concept they couldn't follow in class — and Limpide must serve that need without lecture. The depth path is what the platform *can* do. The urgent-need path is what it *must* do, for everyone.

## The two paths and their relationship

**The urgent-need path** is the entry point for almost every student. They come because they don't understand springs, or they have a probability problem due tomorrow, or their teacher said something about derivatives that didn't make sense. Limpide honors the urgency. It asks the student to teach what they already know about the topic. The teaching attempt exposes the local gap and often a deeper gap underneath. Limpide offers to fork — would you like to spend ten minutes on the deeper thing first, or get the local answer and come back? The student chooses. Whichever they choose, the system works toward the immediate need while recording what remains uncertain.

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

## Commercial horizons

The individual learner is the immediate focus. Useful formative information for teachers is an institutional hypothesis to validate alongside tutoring. Cross-institution benchmarks remain a future business goal, with no committed launch year or contribution-based pricing (ADR-0021).

These possible scales share an educational ambition but have different users, buyers, evidence requirements, and operating costs. The first product must earn repeat use and payment through learner and teacher value. The network is not required to justify or implement the pilot. `STRATEGY.md` records the sequencing and conditions for reconsidering it.

Existing privacy and safeguarding boundaries apply at every stage. Preserving those boundaries does not require implementing future network features before the first useful tutoring experience.

## The honest limits

Limpide does not replace teachers. A skilled human teacher who can sit with a student for an hour and read the room is doing things no system will match in the foreseeable future. What Limpide can offer is *the patient interlocutor at scale* — the thing a student usually doesn't have access to, the thing that asks the next question and waits for the answer and doesn't get tired and doesn't judge. This is a real intervention, but it is one piece of an education, not the whole thing.

Limpide also does not replace reading. Sitting with a primary text — Plato, Euclid, a real piece of literature — and working through it slowly is the activity. Limpide can help the student read more carefully, can ask the questions that make the reading active, can connect the text to other texts and to other substrates. It cannot read for the student.

Finally, some difficulty can be productive, but frustration is not the objective. The Tutor must distinguish useful effort from unproductive struggle and offer timely explanation or correction. Whether students feel helped rather than interrogated is a central pilot question.

## Status of this document

**Settled.** The aim of durable understanding, learner dignity and agency, patient support, honest feedback, and the broader reasoning–reading–expression ambition. ADR-0021 establishes tutoring and buyer validation first and defers the network business.

**Tentative.** Which teaching moves work for which learners, how much transfer occurs across subjects, and whether the proposed triads produce the intended capabilities. These are testable hypotheses, not settled effects. Process measurement requires independent validation.

**Open.** Whether Limpide is eventually owned by a mission-locked company, a foundation, or a steward-ownership structure. The decision can wait until there is something working to govern; licensing (ADR-0006) is an open placeholder in the [ADR index](ADR/README.md). Commercial open questions — payer, price, and unit economics — live in `STRATEGY.md`. The contractual structure for any future network's data discipline (Aegilo never sells contributed data) is an open question in `ARCHITECTURE.md`.
