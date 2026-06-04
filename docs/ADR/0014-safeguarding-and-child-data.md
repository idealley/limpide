# ADR-0014: Safeguarding and child data protection

**Status:** Proposed
**Date:** 2026-05-29

> This ADR is a design-and-compliance map, not legal advice. The cross-jurisdiction specifics — and especially the Swiss cantonal rules and mandatory-reporting duties — must be reviewed by qualified counsel before any deployment touching real students.

## Context

Limpide's users are largely minors, including young children (the platform is designed to serve a learner from early childhood through the polymath depth path). That creates two distinct obligations the project has not yet addressed:

1. **Child data protection** — lawful, minimal, consented handling of children's personal data.
2. **In-session safeguarding / duty of care** — what the system does when a child in a session signals distress, self-harm, or discloses abuse.

Both are table stakes for the institutional sale in `STRATEGY.md` (no school will deploy without them) and, more importantly, both protect the actual children using the product. The four-scope data-sovereignty architecture (`ARCHITECTURE.md`) is a strong foundation but does not by itself satisfy either obligation.

The landscape, confirmed by research (sources below):

- **COPPA (US)** — for under-13s. The amended Rule is in force (effective 23 June 2025; compliance required from 22 April 2026). Expanded definition of personal information (incl. biometric and government identifiers), **separate** parental consent for third-party disclosure, stricter retention/security, prescribed parental notice.
- **FERPA (US)** — student education records. The **school official exception** lets a vendor receive student PII without separate parental consent only if it performs an institutional function, has a legitimate educational interest, is **under the school's direct control** for use and maintenance of records, and **does not reuse or redisclose** the data. Notably, a vendor that can delete records unilaterally may break the "direct control" requirement.
- **GDPR Article 8 (EU)** — digital age of consent is 13–16 depending on member state; below it requires consent from the holder of parental responsibility, with reasonable verification efforts.
- **Switzerland (revFADP, in force Sept 2023)** — the pilot jurisdiction (Vaud, Valais). GDPR-aligned, but for minors there is **no bright-line age**: validity of a minor's consent turns on their **capacity of judgment** (Urteilsfähigkeit) under the Civil Code. Cantonal public-sector data-protection laws additionally govern school-held data, so Vaud and Valais each add a layer.
- **UK Age Appropriate Design Code** — not binding here, but the clearest operational standard: best interests of the child first (per the UNCRC), high-privacy by default, data minimisation, no nudge techniques to extract data or weaken privacy, profiling/geolocation off by default.
- **Emerging AI-chatbot safety norms** — crisis detection with referral to support, human review/escalation of severe disclosures (self-harm, abuse), clear AI-not-human disclosure to minors, and an evolving duty-of-care/foreseeability standard for emotionally responsive systems used by minors.

## Decision

Two pillars.

### Pillar 1 — Child data protection: strictest-superset, best-interests baseline

Limpide adopts a single high baseline rather than per-jurisdiction minimums: the **best-interests-of-the-child** posture operationalised through the **Age Appropriate Design Code** principles — high privacy by default, data minimisation, no profiling or advertising of children, no nudge techniques, purpose limitation. Meeting the strictest design standard generally satisfies the others and avoids a brittle per-user rules matrix. This *extends* the existing sovereignty commitments rather than competing with them.

**Consent and controllership are deployment-specific**, and the deployment determines who the data controller is:

- **School / institutional deployments.** The institution is the controller and the consent gateway. In the US this is FERPA's school-official exception — which Limpide's sovereignty architecture fits well, *provided* it honours the exception's conditions: institutional function, school direct control, no reuse, no redisclosure. The architecture must not let Aegilo become a controller of child data, and must not delete records in a way that strips the school's "direct control."
- **Direct-to-consumer (personal scope).** Verifiable parental/guardian consent below the digital age of consent — COPPA's verifiable-consent methods for under-13 (US), GDPR Art. 8 parental consent with reasonable verification (EU).
- **Switzerland (pilot).** Capacity-of-judgment model, not an age gate: guardian consent plus an assessment of the minor's capacity, under revFADP and the Civil Code, with the relevant **cantonal** public-sector law applying to school-held data in Vaud and Valais.

**Data discipline:** collect only what teach-back requires; treat transcripts as sensitive child PII; no secondary use; no advertising; no third-party disclosure without separate, specific consent (COPPA 2025). **Children's personal data never enters the network scope** — only anonymised aggregates do (already the architecture's rule), which keeps the federated-benchmark feature compatible with child-data law by construction.

**Retention and deletion** are controller-mediated: in institutional deployments the institution (as controller) governs deletion, reconciling the FERPA direct-control requirement with the personal-scope "delete anytime" principle; in consumer deployments the guardian/user controls deletion directly. The controller is named explicitly per deployment.

### Pillar 2 — In-session safeguarding: safety overrides pedagogy

A new Foundation rule, **F12: Safety overrides pedagogy.** When a student signals distress, self-harm, or suicidal ideation, or discloses abuse, the safeguarding response **pre-empts the session loop**. The Tutor does not treat it as a tutoring matter, does not apply Socratic withholding, and does not prioritise the lesson over the child's welfare. Encoded at the Policy Gate so no Tutor or Gardner prompt can override it.

What F12 entails operationally:

- **AI transparency.** The Tutor always discloses it is an AI, not a human — explicitly for minors — with periodic reminders in long sessions. Limpide is a study tool, not a companion or therapist (`PEDAGOGY.md` already forbids fostering emotional dependence); F12 reinforces that it surfaces and escalates rather than counsels.
- **Crisis response.** On distress or harm signals, the system responds with care, stops the pedagogical loop, and surfaces appropriate support resources. It does not run safety-assessment interrogations.
- **Human escalation.** Severe disclosures (self-harm, abuse) raise an escalation flag to a designated human, configured **per deployment before launch**: the school's designated safeguarding lead in institutional deployments; the guardian and/or crisis resources in consumer deployments. Limpide provides the *detection and escalation mechanism*; the *legal reporting judgment* (mandatory-reporting duties, which vary by jurisdiction and by Swiss canton) belongs to counsel and the institution.
- **A narrow, bounded data exception.** Safeguarding escalation is the one defined exception to the otherwise-strict internal/sovereign data flow (F3, the scope model): a harm disclosure may have to leave the session to reach a human. This exception is explicit, minimal, logged, and limited to the safeguarding purpose.

## Alternatives considered

**Per-jurisdiction minimum compliance.** Apply exactly what each law requires, per user. Rejected — a brittle rules matrix that varies by user location and races to the floor, against the best-interests principle. The strictest-superset baseline is simpler and safer.

**Safeguarding as a Tutor prompt instruction only.** Rejected — it must be Policy-Gate-enforced (F12), so no prompt, jailbreak, or model drift can override it. Same logic as F1 and F10.

**A terms-of-service disclaimer instead of in-session handling.** Rejected — duty-of-care and foreseeability standards (and emerging AI-minor law) make "we disclaimed it" untenable for a child-facing product, and it is ethically wrong.

**Age-gate the platform to 13+/16+ to avoid child rules entirely.** Rejected — Limpide deliberately serves young children and the early-curiosity-to-polymath arc. It must handle child data and safeguarding properly, not exclude the youngest learners.

## Consequences

**Commits us to:**
- An AADC / best-interests design baseline applied platform-wide.
- A deployment-specific consent-and-controller model (FERPA school-official in US schools; verifiable parental consent for consumer; capacity-of-judgment for Swiss minors), with the controller named per deployment.
- Data minimisation, purpose limitation, no advertising or profiling of children, no third-party disclosure without separate consent, children's data excluded from the network scope.
- Foundation rule **F12** with crisis detection, AI-disclosure, a configured human-escalation path, and a narrow logged safeguarding data exception.

**Precludes:**
- A single global consent flow; silent third-party sharing; treating safety as soft Tutor behaviour; advertising or profiling of children; unilateral deletion that breaks a school's FERPA direct control.

**Opens:**
- The exact escalation contacts and the **mandatory-reporting mapping per canton/jurisdiction** (counsel + institutional configuration).
- How capacity-of-judgment is assessed in the Swiss consumer case.
- **Age verification without over-collection** — the inherent tension between verifying age/consent and AADC data-minimisation.
- Whether a **Data Protection Impact Assessment** is required (likely yes under GDPR/revFADP for large-scale processing of children's data) before the pilot.
- How the safeguarding data exception is audited and bounded over time.

## Status

**Settled.** The strictest-superset / best-interests baseline. Deployment-specific consent and controllership. Children's data excluded from the network scope. Foundation rule F12 (safety overrides pedagogy) with crisis detection, AI-disclosure, configured human escalation, and a narrow bounded data exception.

**Tentative.** The precise consent mechanics per deployment. The retention schedule. Whether F12 needs its own Policy-Gate encoding distinct from the existing gates or extends them.

**Open.** Cantonal mandatory-reporting mapping. Capacity-of-judgment assessment for Swiss consumer use. Age-verification method under minimisation constraints. DPIA scope and timing.

## Sources

- COPPA amended Rule (2025): https://www.federalregister.gov/documents/2025/04/22/2025-05904/childrens-online-privacy-protection-rule · https://www.ecfr.gov/current/title-16/chapter-I/subchapter-C/part-312
- FERPA school-official exception (US Dept. of Education, vendor responsibilities): https://studentprivacy.ed.gov/sites/default/files/resource_document/file/Vendor%20FAQ.pdf
- GDPR Article 8 (child consent / information society services): https://gdpr-info.eu/art-8-gdpr/
- Swiss revFADP (in force Sept 2023): https://www.kmu.admin.ch/kmu/en/home/facts-and-trends/digitization/data-protection/new-federal-act-on-data-protection-nfadp.html
- UK Age Appropriate Design Code (ICO): https://ico.org.uk/for-organisations/uk-gdpr-guidance-and-resources/childrens-information/childrens-code-guidance-and-resources/age-appropriate-design-a-code-of-practice-for-online-services/
