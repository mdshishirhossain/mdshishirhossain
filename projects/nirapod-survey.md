# Nirapod — Research Survey & Fieldwork Platform

**Research setting:** NPI University of Bangladesh, Department of CSE · **Year:** 2026  
**Product:** Bangla-language research survey instrument and authorized fieldwork console  
**Technology:** Next.js · TypeScript · PostgreSQL · Prisma · Mobile-first / offline-aware workflows  
**Current use:** Ongoing research data collection; **production is not a public demo**

[← Return to portfolio](../README.md)

---

## Executive overview

Nirapod supports a university research study on women's experiences accessing washrooms outside the home. This application is the **research survey and fieldwork system**—not the future washroom-finder app described in the study's wider motivation.

The project's central engineering question was not “how do we build a form?” It was: **How do we collect sensitive, structured answers from mobile participants without losing consent, losing partially completed work or exposing identity in analytical exports?**

The repository provides evidence of a participant questionnaire, explicit consent, offline-aware progress, staff coordination, supervisor oversight and research export tooling. The live environment and participant records are intentionally excluded from this public case study.

## 01 / Research workflow and audiences

| Person | Task | Design implication |
| --- | --- | --- |
| Participant | Understand the invitation, consent and complete questions on a phone | Bangla-first wording; clear progress; recovery from interrupted connectivity |
| Interviewer / field staff | Coordinate legitimate participation and support the process | Authorized access and operational feedback without viewing private answers as the participant |
| Supervisor | Monitor fieldwork and review authorized research activity | Clear records, controlled privileges, export and auditability |
| Research analyst | Use consistent coded data and supporting documentation | Reproducible columns and a codebook rather than an arbitrary spreadsheet dump |

The system's public participant surface and its staff-facing operational surface intentionally have **different presentation requirements**. A quiet, sequential mobile questionnaire is appropriate for one; a compact operational table is appropriate for the other.

## 02 / System boundaries

```text
Participant (mobile)
    │
    ├── Eligibility / informed consent
    │
    ├── Structured Bangla questionnaire
    │         └── local draft / offline-aware recovery
    │
    ▼
Validated application requests (Next.js / TypeScript)
    │
    ├── Survey response persistence (Prisma + PostgreSQL)
    ├── Protected contact-data handling
    ├── Controlled staff/fieldwork workflows
    └── Audit and review services
                    │
                    ▼
        Analytical export + codebook
        (without identifying contact fields)
```

The diagram is purposely conceptual. Publishing authentication internals, research-only routes or deployment addresses would not improve a recruiter's understanding and could increase exposure.

## 03 / Consent is a workflow boundary

Consent is part of the participant journey rather than a generic checkbox in the last form step.

The repository's `src/offline/sync.ts` documents an ordered synchronization path: consent is recorded before survey sections are submitted; only changed sections need synchronization; any applicable preview state follows. Invalid server responses are reported to the UI rather than endlessly retried.

**Design reasoning:** a background upload must not bypass the same conditions required of an online submission.

This demonstrates the need to align UX state, client caching and server validation instead of treating each as an independent subsystem.

## 04 / Offline continuity and recovery

For a mobile questionnaire, interruptions are normal: the participant may change networks, close the tab temporarily or lose connectivity in the middle of a section.

The private repository includes:

- `src/offline/draft-store.ts` for local draft handling.
- `src/offline/use-draft.ts` and `src/offline/sync.ts` for client coordination.
- `src/offline/idempotency.ts` to help make retries predictable.
- `tests/offline.e2e.mjs` and related recovery tests.

The synchronization code explicitly differentiates *nothing pending*, *currently offline*, *server rejected*, *submitted* and *successfully synced* outcomes. That is more helpful than an ambiguous loading spinner.

**Trade-off:** offline support improves resilience but increases state complexity. The client and server can temporarily disagree, so the design must define what is authoritative and what the user sees when an update is rejected.

## 05 / Privacy by construction in research exports

One of the clearest engineering boundaries appears in `src/server/export-schema.ts`.

Rather than hand-writing arbitrary export columns, the output schema is derived from the questionnaire definition. It models categorical answers, scales, free text, matrix responses, multi-select binary columns and accompanying explanations.

Importantly, the file documents that **participant identity and contact fields are not included in the analytical export schema**. The repository also contains a targeted export test intended to guard against regression.

**Why this matters:** post-processing a sensitive export to delete names is weaker than defining analytical output so those names never enter the export path.

The private code contains separate contact-handling and reveal controls. Their internal implementation is not reproduced publicly.

## 06 / Capacity is enforced before participation

A notable design decision is recorded in `src/server/capacity.ts`.

A study limit cannot be treated merely as a number on the supervisor's dashboard. If an invitation can still be issued after the capacity has effectively been reserved, someone may complete consent and most of a questionnaire only to be rejected at the end.

The documented implementation accounts for completed responses, active drafts and issued-but-unused invitations when evaluating capacity. It addresses stale or revoked invitations when releasing capacity.

**Product principle:** reject new invitations before asking a participant to invest time, instead of refusing a nearly complete submission.

This is a useful example of research ethics translating into operational application logic. The case study intentionally omits the study's current capacity value and fieldwork statistics.

## 07 / Design language and verification

The repository's `docs/design-language.md` describes different visual systems for:

**Participant survey.** Bangla language, large tap targets, calm sequential interaction, restrained surface density.

**Study Control.** English operational workspace, clear status indicators, information-dense tables and supervisory workflows.

These aren't just aesthetic differences; a participant filling in a sensitive survey on a phone has different cognitive and accessibility needs from someone supervising fieldwork at a desk.

Design rules are checked through automated browser tests, while content and questionnaire logic have separate validation and test coverage.

## 08 / Quality and testing evidence

The private repository includes test files covering:

| Area | Example evidence |
| --- | --- |
| Survey data | `answer-validation.test.ts`, `questionnaire-v3.test.ts`, `matrix-walk.e2e.mjs` |
| Offline workflow | `offline.e2e.mjs`, `recover.test.ts`, `come-back.e2e.mts` |
| Exports | `export.test.ts`, `analysis-syntax.test.ts` |
| Staff workflows | `interviewer.e2e.mjs`, `supervisor.e2e.mjs`, `staff-access.e2e.mjs` |
| Mobile interface | `mobile-layout.e2e.mjs`, `design-audit.e2e.mjs` |
| Controlled operations | `retention.test.ts`, `audit-scrub.test.ts`, `capacity.test.ts` |

These files demonstrate test **coverage in the source tree**; they are not a claim that all tests passed on every deployment. Some end-to-end tests require explicit fixtures and environments.

## 09 / Deployment and data safety considerations

The private repository documents a Linux service deployment model, database archival procedures, deployment checks and backup verification steps.

The existence of scripts does **not** itself guarantee disaster recovery. An auditable backup strategy additionally requires off-server copies, protection of associated encryption keys and restoration tests.

A live research survey should never be repurposed as a recruiter playground. A portfolio demonstration requires an independently deployed test instance, synthetic participant identities and approval from the research owner.

## 10 / Constraints, unresolved items and presentation ethics

- Research data and staff access remain private.
- Supervisor materials are not interchangeable with public product documentation.
- A live study link is **not** offered as “Try the demo.”
- This document does not claim a research outcome, number of participants, independent security certification or ethics approval status.
- Any future screenshots must be checked for participant or staff information before publication.

## 11 / What the project demonstrates

Nirapod brings together mobile product design, reliable multi-step data collection, structured data modeling, privacy-aware exports and fieldwork operations.

The most transferable engineering lessons are:

1. Consent and eligibility must shape the data path, not sit at its edges.
2. Offline resilience requires explicit sync outcomes and stable server validation.
3. Analytical exports should exclude identifiers **by design**, not through manual cleanup.
4. Operational constraints, including research capacity, belong in application logic.
5. The sensitivity of a dataset should change what is publicly demonstrated.

**Portfolio evidence:** the original development code, tests and research-specific documentation remain in private repositories. This public case study summarizes implementation patterns without exposing respondents, access methods, credentials or sensitive documents.

---

*This is a software-engineering portfolio case study, not a research publication or certification of compliance.*