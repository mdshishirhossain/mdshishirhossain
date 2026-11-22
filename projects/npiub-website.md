# NPI University of Bangladesh — Website Redevelopment

### Modernizing an institutional publishing system without risking its existing data

**Role:** Website Developer · **Period:** June 2026–present  
**Stack:** Laravel 12 · PHP · Filament 4 · Blade · Tailwind CSS · MySQL/MariaDB · Alpine/Livewire  
**Workstream:** Public-facing institutional website, administration, legacy-data migration and quality engineering  
**Release status:** The private repository documents staging and deployment preparation; **a verified production cutover for this Laravel redevelopment is not established.**

[← Return to portfolio](../README.md) · [Existing official university website](https://npiub.edu.bd/)

> **Link clarification:** The official domain identifies the university's current public website. It is **not** presented here as independent proof that this specific Laravel redevelopment is already live.

---

## Executive overview

A university website is a long-lived institutional record, not simply a set of landing pages. It has to present courses, departments and people accurately; allow staff to publish notices and events; and preserve legacy material during upgrades.

The assignment was to build a modern Laravel-based application with staff-facing content management under practical shared-hosting constraints, while documenting a safe path from the old data model to a new one.

The major challenges were **publishing workflow**, **data continuity**, **hosting limitations** and **interface consistency**.

## 01 / Users and information journeys

| Audience | Need | Application approach |
| --- | --- | --- |
| Prospective students | Find programmes, requirements and academic information | Public academic pages with structured programme/department relationships |
| Current students and visitors | Find notices, events, downloads and contacts | Dedicated publication types and navigation |
| Faculty and university staff | Maintain accurate institutional content | Authorized administration and structured editing tools |
| Website administrators | Manage roles, media, settings and content quality | Filament resource management, permissions and audit support |
| Maintainers | Update safely without losing historical records | Legacy importer, staging workflow and automated verification |

These users have different needs. A general-purpose text editor cannot replace structured academic content modeling, and an attractive public page cannot compensate for unreliable administrative records.

## 02 / Architectural constraints

The intended hosting environment is traditional shared cPanel hosting rather than an always-on application VPS. It does not permit the same long-running Node, queue-worker and Redis assumptions commonly made in cloud-native stacks.

The repository therefore documents a hosting-specific delivery model:

```text
         Browser
            │
            ▼
  Laravel public pages / Filament panel
            │
            ├── MySQL / MariaDB content records
            ├── Role-based content operations
            ├── Media and audit features
            └── Scheduled work via hosting-compatible cron

 Development / GitHub CI
            │
            └── Build frontend assets outside hosting
                   │
                   ▼
                Deployable assets + application
                   │
                   ▼
              Separate staging environment
```

The diagram illustrates repository design, not an independently verified production runtime.

**Technology decisions:** Blade provides server-rendered pages; Filament handles administrative resources; Tailwind and Alpine/Livewire are used where the design benefits from interaction. The documented environment uses database/file/synchronous options instead of assuming Redis or long-running workers.

## 03 / Legacy migration: protecting institutional continuity

The clearest engineering challenge is documented in `docs/DATA-MIGRATION.md`.

A direct destructive replacement of the legacy database would be risky: older faculty, officer, notice, gallery and user records are part of the institution's operational history.

The planned and implemented migration model uses:

1. **A separate restored source copy.** The old production database stays untouched; migration reads from an isolated legacy connection.
2. **Normalized destination tables.** Inconsistent legacy structures and free-text department labels are mapped to a cleaner application model.
3. **Stable upsert anchors.** Legacy identifiers support repeatable imports rather than creating duplicates on every run.
4. **Transactional steps and reconciliation.** Import operations can be checked against expected record mappings.
5. **Bangla-preserving encoding.** The tests explicitly cover multilingual text handling.

### Example transformations, at a conceptual level

| Before | After | Reason |
| --- | --- | --- |
| Faculty and officer records stored under different legacy shapes | Unified people model with defined roles/types | Reduces redundant display logic |
| Departments represented by inconsistent strings | Structured department entities | Improves links between people and academic units |
| Flat gallery media | Album and image relationships | Makes media manageable |
| Existing notices and news with legacy file paths | Structured content with preserved legacy references | Supports later file-aware migration |
| Legacy user permissions | Application role/permission model | Clarifies administration access |

This is a documented implementation and validation strategy. It is not a claim that production data was rewritten during this portfolio review.

## 04 / Content administration

The repository contains Filament resources for academic departments, programmes, faculty members, notices, news, events, downloads, offices, user accounts and other institution-specific entities.

**Why separate resource types?** Notices, academic profiles and programmes have different fields, dates, relationships and approval expectations. Giving all of them a common text editor would make reporting and content accuracy harder.

Role-based access control and audit-logging packages are present in the application design. The exact internal administrative routes and authorization details are not published here.

## 05 / Design-system work

The repository's redesign documentation records concrete consistency problems uncovered across the site:

- Page templates were using conflicting content widths.
- Academic and notice layouts lacked a consistent reading hierarchy.
- Typography, radii and shadows varied excessively across components.
- Mobile contact actions and some keyboard/focus interactions needed improvement.
- Some pages held useful stored fields that never appeared on the public page.

The redesign report describes how shared layout widths, typography rules and reusable content components were introduced.

This illustrates a useful engineering lesson: **a design system is as much about making content trustworthy and maintainable as it is about visual polish.**

### Example: information architecture

The academic area must let a visitor move from a department to its individual programmes, rather than display programme names as unlinked text. That is an information-architecture decision, not merely a visual one.

Similarly, news and research pages should have consistent bylines, document metadata and navigation conventions where the content model supports them.

## 06 / Accessibility and quality checks

The private repository includes `docs/QUALITY-BASELINE.md`, recording an automated axe-core review of **15 representative public page types**, in both light and dark themes.

The report states that its WCAG A/AA automated violations decreased from **95 to 0 flagged issues** after the documented fixes.

**Interpretation is important:** this is a *repository-reported automated audit*, not independent certification. Automated scans do not replace keyboard-only evaluation, screen reader testing or user research; the report explicitly lists manual checks and Lighthouse performance assessment as follow-up work.

The repository also contains CI workflows and tests for:

- Laravel application behavior using Pest.
- Formatting standards using Laravel Pint.
- Static analysis using Larastan.
- Frontend asset compilation.
- Legacy import behavior and content-specific user journeys.

Test files and documented checks provide tangible engineering evidence, but are not proof that every production deployment has passed them.

## 07 / Deployment design and status

The documented deployment path uses a **separate staging environment** and a private repository build process appropriate for cPanel hosting.

The intended safety invariants are:

- Keep the existing university website and its data unchanged during staging work.
- Import from a restored copy, not the live production database.
- Build web assets before deployment when the hosting environment cannot build them.
- Verify staging behavior and content before any eventual production cutover.
- Preserve a rollback path.

**Current reporting boundary:** the university website can be visited publicly, but the repository still documents a separate go-live/cutover phase. A candidate should not present that domain as a verified live demo of the rebuild without checking which version is running and obtaining any necessary institutional permission.

## 08 / Evidence matrix

The following are descriptive paths from a **private** repository, shared to establish the basis for this summary rather than expose source:

| Area | Evidence |
| --- | --- |
| Architecture | `README.md`, Laravel application structure |
| Publishing | `app/Filament/Resources/` and related models |
| Migration | `docs/DATA-MIGRATION.md`, `app/Console/Commands/MigrateLegacy.php` |
| Migration verification | `tests/Feature/LegacyImportTest.php` |
| Quality | `docs/QUALITY-BASELINE.md`, `docs/redesign-report.md` |
| Deployment planning | `DEPLOYMENT.md`, GitHub Actions workflows |
| Accessibility checks | `tests/Feature/AccessiblePassTest.php` and documented audit |

## 09 / What the project demonstrates

The value of the work is not merely that it uses Laravel. It brings together:

- Structured content modeling for an actual institution.
- Staff-facing publishing with access-control considerations.
- Legacy-data transformation with repeatability and safe staging.
- Responsive visual-system improvement based on an explicit audit.
- Testing and deployment strategies adjusted to limited hosting infrastructure.

**Main takeaway:** good institutional software must improve the user experience while respecting the existing data, operations and technical environment.

## 10 / Screenshots and future demo

A useful portfolio presentation would include a small, approved set of images:

1. Public homepage and academic programme pages.
2. A representative administrative content list with dummy records.
3. A before/after layout comparison with any sensitive content removed.
4. A brief walkthrough of a typical notice or programme update.

These materials have **not** been added here because no approved, version-verified demo media was available in the examined repository. The private administration interface and actual university data should not be made public purely for portfolio purposes.

---

*Portfolio case study grounded in inspected project documentation and code structure. Private client data, credentials, hosting account details and administrative endpoints are excluded.*