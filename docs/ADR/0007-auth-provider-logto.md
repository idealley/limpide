# ADR-0007: Logto as the identity provider, inherited from the AYA host

**Status:** Accepted
**Date:** 2026-08-24
**Relates to:** ADR-0014 (Safeguarding and child data protection), ADR-0019, ADR-0020, AYA ADR-0008 (Logto as Identity Provider)

## Context

This ADR was an open placeholder ("Auth provider"); `ARCHITECTURE.md` guessed "likely Clerk or similar, not decided". Since then AYA settled on **Logto** (AYA ADR-0008, accepted 2026-05-01): OIDC, organizations and roles natively, Apache-2.0-compatible, with a deployment progression of Logto Cloud → self-hosted Docker → customer-installed, and a local single-user mode for the embedded/edge profile.

Because Limpide is a plugin against the AYA host (ADR-0019) and uses the host's Surreal token endpoint for the read path (ADR-0020), the identity provider is not Limpide's to choose. What Limpide *does* own is everything ADR-0014 requires on top of identity: who the controller is per deployment, guardian consent, capacity-of-judgment, and the institution-as-consent-gateway model for schools.

## Decision

**Limpide uses Logto through the AYA host. It adds no second identity provider and no Limpide-specific login.**

Limpide's identity-adjacent concerns are layered on Logto's organizations and roles, not on a parallel user store:

- **Entity = deployment controller.** A school deployment is a Logto organization; the school is the data controller (ADR-0014). A consumer deployment is a personal/family entity; the guardian (or the student with capacity of judgment) is the controller.
- **Roles carry the four scopes' read rights.** `student`, `guardian`, `teacher` (cohort-scope reads only), `institution-admin` (institutional scope), and the host's operator role. The mapping from role to Surreal permission clause is the enforcement (ADR-0020 §1); Logto is the source of who holds which role.
- **Consent is a Limpide record, not a Logto attribute.** Guardian consent, its scope, and its date live in Limpide's plugin-private schema with an Event Journal entry, because consent has a lifecycle (given, narrowed, withdrawn) that an identity claim cannot carry, and because it must survive the audit demands of the DPIA (`PILOT-READINESS.md`). Logto tells us who; Limpide records what they agreed to.
- **Minors sign in through the institution or the guardian's entity.** Self-serve sign-up for a student below the digital age of consent is not offered; account creation is a guardian or institution action. This is the Age Appropriate Design Code default from ADR-0014 expressed as an onboarding rule.
- **Edge / single-binary deployments** use AYA's local single-user mode; a school laptop running Limpide offline has one local identity and optional pairing to the institution's Logto.

## Alternatives considered

**Clerk / Auth0 / a hosted consumer IdP for Limpide alone.** Rejected: two IdPs in one process, and none of them self-host cleanly for on-prem schools.

**Rolling a consent-aware identity layer inside Limpide.** Rejected: identity is a solved problem and a compliance liability; consent is the part that is Limpide-specific, and it is modelled as data, not as auth.

## Consequences

**Commits us to:** Logto's org/role model as the shape of Limpide's tenancy; a consent schema and its events as pilot prerequisites (they already are, per `PILOT-READINESS.md`); the same deployment progression as AYA.

**Precludes:** A Limpide sign-in that works without an AYA host.

**Opens:** Enterprise SSO for institutional deployments through Logto's connectors, without Limpide code; the DPIA can cite AYA's identity documentation rather than describing a bespoke system.

## Status

**Settled** on Logto via the host and on consent-as-data.

**Tentative** on the exact role set and on whether `teacher` is one role or a family of roles (subject teacher, class teacher, learning-support) with different cohort views. Decide with the first school pilot.
