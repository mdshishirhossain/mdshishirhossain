# NPI University of Bangladesh
### Institutional website redevelopment under real hosting and data-migration constraints

**Work:** Website Developer · **Period:** June 2026–present  
**Application:** Laravel 12, Filament, MySQL/MariaDB, Blade, Tailwind CSS  
**Status:** Redevelopment and staging/deployment work documented; production cutover of the rebuilt Laravel site has **not** been independently verified.

[← Portfolio](../README.md)

---

## 01 / The assignment

An institutional website has multiple responsibilities: present academic information publicly, allow authorized staff to update content and preserve years of existing records. Replacing the pages alone is not enough if notices, faculty information or media disappear in the process.

This project approaches the university website as both a **public information system** and an **administrative publishing workflow**.

## 02 / Product scope

| Surface | Main responsibilities |
| --- | --- |
| **Public website** | Academic departments and programmes, announcements, notices, events, people profiles and information pages |
| **Staff administration** | Content management, permissions, media and institutional settings |
| **Legacy transition** | Import existing content into a normalized schema without overwriting the original source |
| **Delivery and quality** | Responsive styling, testing, accessibility checks, asset builds and deployment preparation |

## 03 / Architecture shaped by constraints

The target hosting environment is shared cPanel hosting without long-running Node services or Redis. That ruled out copying assumptions from a typical VPS-based stack.

\`\`\`text
Public site / staff panel
           │
     Laravel + Filament
           │
     MySQL / MariaDB
           │
  Prepared assets (built in CI)
           │
   cPanel-compatible deployment
\`\`\`

The application uses server-rendered Blade templates with Tailwind and interactive components where required. Database/file/synchronous options replace services unavailable on the target host.

**Why it matters:** Technical architecture should reflect the infrastructure a client actually operates, not only a developer's preferred stack.

## 04 / The legacy-data problem

The repository documents a repeatable migration path:

1. Preserve the original database and restore a separate read-only copy for inspection.
2. Map old tables and inconsistent department strings to a normalized new schema.
3. Import data through stable legacy identifiers and repeatable upserts.
4. Keep legacy source data unchanged; verify reconciliation and Bangla text handling.

The migration tests cover representative mappings, role conversions, preserved relationships and the ability to rerun imports without generating duplicate records.

This is a documented safety design and test strategy; it does not imply production data was modified during this portfolio review.

## 05 / Administrative and design decisions

**Publishing instead of hard-coded updates.** Staff-facing Filament resources cover institutional content types, reducing dependence on code edits for routine updates.

**Permissions and accountability.** Role and permission management and audit logging support separation of administrative responsibilities.

**Design-system consistency.** The repository's redesign audit identified uneven content widths, typography scales, cards and responsive behaviors, followed by shared layout and component changes across page templates.

**Accessibility as a measurable check.** A recorded automated audit covered 15 representative page types in light and dark modes. The repository reports zero flagged WCAG A/AA issues *under that automated test* after fixes; this does not establish full accessibility compliance, which also requires manual testing.

**Deployment discipline.** The documented process builds frontend assets outside the shared host and deploys to staging with separate data and configuration before any cutover.

## 06 / Quality and project status

The repository documents CI checks (Pest, Pint, Larastan and asset compilation), staging preparation, migration tooling and design/accessibility review.

The university's public website may be online, but that alone does not prove this specific Laravel rebuild is the production version. This case study intentionally avoids presenting a public URL as a verified demo of the rebuild.

Screenshots should be added only when confirmed to depict the intended version and approved for public sharing.

## 07 / What this work demonstrates

This project combines Laravel development with content modeling, access control, data migration, design-system discipline and deployment planning. A key lesson is that **safe institutional modernization means preserving the client's data and publishing workflows while improving the interface**.

*The source repository remains private. This case study excludes credentials, private operational paths, client records and internal administration URLs.*