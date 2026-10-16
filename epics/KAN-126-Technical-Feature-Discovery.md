## Jira

| Field | Value | Note |
| --- | --- | --- |
| Key | `KAN-126` | epic (authoritative — sprint mechanics) |

## Subject

Technical Feature Discovery — cpaltracker.web

## Business Case

The planned React UI rebuild (KAN-60) needs a trustworthy, complete inventory
of what `cpaltracker.web` does today before any frontend design work starts.
Without it, the rebuild risks silently dropping a capability a client relies
on, or carrying forward an existing inconsistency into the new UI. This epic
produces that inventory as a byproduct of reverse-engineering, not guessing.

## Description

Analyze every page in `cpaltracker.web`, grouped into 7 feature areas, and
produce one Technical-Feature doc per area: reverse-engineered current
behavior (cited to file/function), gaps against the codebase's own stated
rules, and suggested improvements. A final consolidation story rolls all 7
into one coverage list, explicitly framed as the requirement-traceability
input for KAN-60. Purely additive/exploratory: no application code changes.

## Requirement References (SSoT = Feature doc)

Feature spec: [`Technical-Feature-Discovery.md`](../features/Technical-Feature-Discovery.md)
— 10 FRs, 4 NFRs, 11 ACs. This epic references requirements by ID only;
requirement text lives solely in the feature doc.

## KPIs

- 100% of the web root's files (per `architecture.md`'s Components table,
  excluding KAN-116's `/api/v1` layer) cited by at least one CB-n.
- 0 Current Behavior claims without a file/function citation.
- 1 consolidated coverage-list doc, explicitly framed as ready for KAN-60.

## Dependencies

- `cpaltracker.web/docs/architecture.md`, `security.md`,
  `design-and-standards.md` — cross-referenced, not re-derived.
- None on KAN-116 (REST API Extraction) — this epic covers the pre-existing
  pages KAN-116 extracts *from*, not the new `/api/v1` layer itself.

## Child Stories

| Jira | Story | Pts | Covered Requirements | Software Items |
| --- | --- | --- | --- | --- |
| KAN-127 | Technical-Feature: Auth & Session | 5 | FR1, NFR1-4 | N/A |
| KAN-128 | Technical-Feature: Password Recovery & Account Management | 5 | FR2, NFR1-4 | N/A |
| KAN-129 | Technical-Feature: Live Tracking | 3 | FR3, NFR1-4 | N/A |
| KAN-130 | Technical-Feature: History Dashboard | 5 | FR4, NFR1-4 | N/A |
| KAN-131 | Technical-Feature: Device Pilotage | 3 | FR5, NFR1-4 | N/A |
| KAN-132 | Technical-Feature: Client Branding & Assets | 2 | FR6, NFR1-4 | N/A |
| KAN-133 | Technical-Feature: Device Ingestion | 2 | FR7, NFR1-4 | N/A |
| KAN-134 | Technical-Feature: Coverage & Gap Consolidation | 5 | FR8, FR9, FR10, NFR1-4 | N/A |

*Total: 30 points across 8 stories. All stories pointed and assigned
to Jira active sprint 3 (S2, 2026-10-11 → 2026-10-22).*

Software Items: N/A — see `cpal.docs/sdlc/README.md#applicability-to-c-pal-today`
(IEC 62304 Software Items are not applicable to this project; this epic
produces documentation, not code).

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |
| FR1 | KAN-127 |
| FR2 | KAN-128 |
| FR3 | KAN-129 |
| FR4 | KAN-130 |
| FR5 | KAN-131 |
| FR6 | KAN-132 |
| FR7 | KAN-133 |
| FR8 | KAN-134 |
| FR9 | KAN-134 |
| FR10 | KAN-134 |
| NFR1 | KAN-127, KAN-128, KAN-129, KAN-130, KAN-131, KAN-132, KAN-133, KAN-134 |
| NFR2 | KAN-127, KAN-128, KAN-129, KAN-130, KAN-131, KAN-132, KAN-133, KAN-134 |
| NFR3 | KAN-127, KAN-128, KAN-129, KAN-130, KAN-131, KAN-132, KAN-133, KAN-134 |
| NFR4 | KAN-127, KAN-128, KAN-129, KAN-130, KAN-131, KAN-132, KAN-133, KAN-134 |

Coverage: all 10 FRs and 4 NFRs mapped to at least one story.
100% structural coverage.

## Related Documentation

- Feature spec: [`Technical-Feature-Discovery.md`](../features/Technical-Feature-Discovery.md)
- Doc-type template: [`engineering/product/TECH-FEATURE.md`](../engineering/product/TECH-FEATURE.md)
- Task-tracking policy: [SDLC — Single source of truth](../sdlc/README.md#single-source-of-truth-cpaldocs)
- Feeds into: KAN-60 (UI Refactor epic)

## Jira Link

- Epic: [KAN-126](https://c-pal.atlassian.net/browse/KAN-126) — sprint S2, 2026-10-11 → 2026-10-22
