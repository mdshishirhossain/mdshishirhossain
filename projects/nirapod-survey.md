# Nirapod
### Research software designed around consent, continuity and data boundaries

**Context:** Departmental research, NPI University of Bangladesh · **Year:** 2026  
**Technology:** Next.js, TypeScript, PostgreSQL, Prisma  
**Status:** Research survey platform deployed for data collection. **The live study is not a public demonstration.**

[← Portfolio](../README.md)

---

## 01 / Research context

The research explores women's experiences accessing washrooms outside the home. Nirapod is the **survey collection instrument** for that study; it is **not** the proposed future washroom-finder application.

A credible research instrument must do more than render a questionnaire. It must make consent clear, preserve answers through network interruptions, restrict access to sensitive information and maintain a defensible path from a participant's screen to analysis-ready records.

## 02 / User journeys

| Audience | What the application supports |
| --- | --- |
| **Participant** | Bangla-language consent and structured, mobile-first questionnaire |
| **Interviewer / field staff** | Authorized coordination and participant-support workflows without taking ownership of answers |
| **Supervisor** | Oversight of fieldwork, review states and controlled research-data export |

The product deliberately separates the participant-facing experience from the staff-facing workspace. The former prioritizes readability and one-step-at-a-time interaction; the latter emphasizes efficient review and operational visibility.

## 03 / System model

```text
Participant device
     │ consent + questionnaire
     ▼
Validated Next.js application
     │
     ├── Survey answers / progress
     ├── Offline-aware recovery layer
     └── Controlled staff access
              │
              ▼
     PostgreSQL + Prisma
              │
       Supervisor review
              │
       Research export
```

This is an intentionally simplified technical overview; internal security controls and production topology are not published.

## 04 / Engineering considerations

**Consent as a workflow boundary.** Participation is not presented as just another form field. Consent and eligibility are explicit parts of the survey sequence.

**Questionnaire consistency.** The repository includes a structured questionnaire configuration, validation steps and an instrument fixture. These help keep implementation, approved participant wording and analysis outputs aligned. Precise study instrument material is not reproduced here.

**Intermittent connectivity.** An offline-aware layer supports retaining in-progress work and recovering from interruptions. This is particularly important for mobile field collection, where a single lost connection should not imply that a participant must restart.

**Data separation.** Participant contact information is handled separately from survey responses, with protective controls around access and export. Research exports can support analysis without exposing direct contact details as normal response columns.

**Supervisor accountability.** The system includes response review, monitoring and operational activity records, designed to make changes and fieldwork decisions traceable.

**Testing and deployment discipline.** The repository contains questionnaire validation, type checking, application tests, deployment preflight checks and database backup procedures. These are engineering controls, not a claim that all risks are eliminated.

## 05 / Design rationale

The survey and its staff console intentionally use different visual systems:

- **Participant UI:** Bangla-first, mobile-first, large interactive targets and straightforward question progression.
- **Staff UI:** Information-dense tables, operational overview and controlled record review.
- **Shared surfaces:** Components are tested for consistent behavior across contexts instead of relying on accidental style inheritance.

The documented design process also includes automated UI checks, but it would be misleading to equate automated checks with a complete usability or security certification.

## 06 / Evidence and boundaries

The private repository documents the questionnaire, export/codebook tooling, offline support and supervisor features. It contains ongoing study-specific material and is **not** publicly mirrored.

No respondent records, study tokens, contact information, staff entry links, approval documents, internal endpoints or database examples from the production study are included in this case study.

A future preview should be **a separate, approved demonstration using synthetic participants and data**, not the ongoing fieldwork environment.

## 07 / What this work demonstrates

Nirapod shows how product decisions change when software collects sensitive research information. The central challenge is not building a form; it is designing **trustworthy boundaries around consent, recovery, access and research data**.

*Public technical overview; not a clinical, ethics or security certification. Study permissions and research approvals remain the responsibility of the research institution.*