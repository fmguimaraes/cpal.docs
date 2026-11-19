# Anomaly workflow — status tracking drawer

> **Single source of truth:** requirement **text** lives in exactly one place, the
> feature spec. The epic doc and the tracker description reference requirements
> **by ID only**.

## Tracker

| Field | Value |
| --- | --- |
| Key | `KAN-254` |

## Subject
Anomaly workflow — status tracking drawer

## Business Case
Detected anomalies have no handling status in the app: nobody can say who took one in
charge, whether it is closed or whether it was validated. Gaël's workflow
(detectee, en_cours, fermee, cloturee) makes anomaly handling visible and auditable per
contract, and feeds the Pulse alert light.

## Description
Add a right-side `AnomalyDrawer` to `cpaltracker-react` (list, detail with map and
history, status transitions with comment) and an `anomalies/` API in `cpaltracker.web`
(list, detail, transition, summary). The database part (status, history table, views,
trigger, closing right) is already built in `docker/mysql/init/03-dashboard.sql`
(shared with Pulse and Trends) and is not part of this epic, except production GRANTs,
the VPS migration note and tests. `anomalies/summary.php` is owned here and consumed by
Pulse for its alert light.

## Requirement References
Feature spec: [`BACKLOG-Anomaly-Workflow.md`](../features/BACKLOG-Anomaly-Workflow.md)
— 35 FRs, 13 NFRs, 22 ECs, 39 ACs. This epic references requirements by ID only;
requirement text lives solely in the feature spec.

## KPIs
See feature spec §13 (not copied here): time to take in charge, time per transition,
closing backlog, send-back rate, adoption, refused-transition rate, API p95 (NFR5).

## Dependencies
- `cpaltracker.web/docker/mysql/init/03-dashboard.sql` §2–§4 (built locally, uncommitted).
- Pulse (`BACKLOG-Pulse.md`): opens `AnomalyDrawer` and reads `anomalies/summary.php`.
- Existing `cpaltracker-react` kit (`Sheet`, `data-table`, `FleetMap`, `ActivityTimeline`, ...).
- Confluence validation of the deviations in feature spec §19 (Gaël).

## Child Stories

Dependencies and file ownership drive parallel scheduling. `cpaltracker.web` paths are
relative to `api/v1/` unless stated; `cpaltracker-react` paths are under `src/`.

| Key | Story | Pts | Depends on | Repos / files owned | Covered Requirements |
| --- | --- | --- | --- | --- | --- |
| KAN-259 | Anomaly data model deployment | 3 | none | cpaltracker.web: migration note for VPS (MariaDB 10.11), production GRANTs script, `tools/demo-data/pharma/sql/seed_anomaly_rules.sql`, DB tests of trigger and views | FR30, FR34, NFR6, NFR13, EC1, EC15, AC19, AC37 |
| KAN-261 | Anomalies read API | 8 | KAN-259 (seed data for tests only) | cpaltracker.web: `anomalies/list.php`, `anomalies/detail.php`, `anomalies/summary.php`, `anomalies/anomalies_logic.php`, OpenAPI entries, tests | FR20-22, FR24-27 (reads), FR29, FR33 (unit field), FR35, NFR2, NFR5, EC2, EC3, EC5, EC12, EC13, EC17-20, EC22, AC22-24, AC26, AC27, AC29, AC34 |
| KAN-263 | Anomaly transition API and audit | 8 | KAN-261 (reuses `anomalies_logic.php` helpers) | cpaltracker.web: `anomalies/transition.php`, `anomalies/anomalies_transition_logic.php`, tests | FR23, FR27 (write), FR28, NFR4, NFR10 (server), NFR11, EC4, EC8-11, EC16, EC21, AC11-17, AC25 (API), AC30, AC31 |
| KAN-265 | Anomaly drawer shell and list | 8 | KAN-261 (contract; can start on the documented contract) | cpaltracker-react: `anomalies/AnomalyDrawer.tsx`, `AnomalyList.tsx`, `AnomalyFilters.tsx`, `anomalyFormat.ts`, `api/anomalies.ts` (shared types and client) | FR1-8, NFR7-9, AC1-6, AC35 |
| KAN-267 | Anomaly detail view | 8 | KAN-265 (drawer and `api/anomalies.ts`) | cpaltracker-react: `anomalies/AnomalyDetail.tsx`, `AnomalyHistory.tsx`, `AnomalyMap.tsx`, optional reference-point marker in `ui/map/` | FR9-13, FR31-33, EC6, EC7, EC14, AC7-10, AC26 (UI), AC28 |
| KAN-268 | Anomaly transition workflow UI | 8 | KAN-267 (detail view), KAN-263 (endpoint) | cpaltracker-react: `anomalies/TransitionForm.tsx` and the action area of the detail view | FR14-19, NFR3, NFR10 (UI), NFR12, AC12 (form), AC13, AC18, AC20, AC21, AC25 (UI), AC32, AC33 |
| KAN-271 | Anomaly workflow QA and non-regression | 5 | KAN-263, KAN-268 | tests only: E2E specs, additivity checks, traceability audit | NFR1, NFR12 (E2E), AC36, AC38, AC39 |

*Total: 48 points across 7 stories.*

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |
| FR1 | KAN-265 |
| FR2 | KAN-265 |
| FR3 | KAN-265 |
| FR4 | KAN-265 |
| FR5 | KAN-265 |
| FR6 | KAN-265 |
| FR7 | KAN-265 |
| FR8 | KAN-265 |
| FR9 | KAN-267 |
| FR10 | KAN-267 |
| FR11 | KAN-267 |
| FR12 | KAN-267 |
| FR13 | KAN-267 |
| FR14 | KAN-268 |
| FR15 | KAN-268 |
| FR16 | KAN-268 |
| FR17 | KAN-268 |
| FR18 | KAN-268 |
| FR19 | KAN-268 |
| FR20 | KAN-261 |
| FR21 | KAN-261 |
| FR22 | KAN-261 |
| FR23 | KAN-263 |
| FR24 | KAN-261 |
| FR25 | KAN-261 |
| FR26 | KAN-261 |
| FR27 | KAN-261, KAN-263 |
| FR28 | KAN-263 |
| FR29 | KAN-261 |
| FR30 | KAN-259 |
| FR31 | KAN-267 |
| FR32 | KAN-267 |
| FR33 | KAN-261, KAN-267 |
| FR34 | KAN-259 |
| FR35 | KAN-261 |
| NFR1 | KAN-271 |
| NFR2 | KAN-261 |
| NFR3 | KAN-268 |
| NFR4 | KAN-263 |
| NFR5 | KAN-261 |
| NFR6 | KAN-259 |
| NFR7 | KAN-265 |
| NFR8 | KAN-265 |
| NFR9 | KAN-265 |
| NFR10 | KAN-263, KAN-268 |
| NFR11 | KAN-263 |
| NFR12 | KAN-268, KAN-271 |
| NFR13 | KAN-259 |
| EC1 | KAN-259 |
| EC2 | KAN-261 |
| EC3 | KAN-261 |
| EC4 | KAN-263 |
| EC5 | KAN-261 |
| EC6 | KAN-267 |
| EC7 | KAN-267 |
| EC8 | KAN-263 |
| EC9 | KAN-263 |
| EC10 | KAN-263 |
| EC11 | KAN-263 |
| EC12 | KAN-261 |
| EC13 | KAN-261 |
| EC14 | KAN-267 |
| EC15 | KAN-259 |
| EC16 | KAN-263 |
| EC17 | KAN-261 |
| EC18 | KAN-261 |
| EC19 | KAN-261 |
| EC20 | KAN-261 |
| EC21 | KAN-263 |
| EC22 | KAN-261 |
| AC1 | KAN-265 |
| AC2 | KAN-265 |
| AC3 | KAN-265 |
| AC4 | KAN-265 |
| AC5 | KAN-265 |
| AC6 | KAN-265 |
| AC7 | KAN-267 |
| AC8 | KAN-267 |
| AC9 | KAN-267 |
| AC10 | KAN-267 |
| AC11 | KAN-263 |
| AC12 | KAN-263, KAN-268 |
| AC13 | KAN-263, KAN-268 |
| AC14 | KAN-263 |
| AC15 | KAN-263 |
| AC16 | KAN-263 |
| AC17 | KAN-263 |
| AC18 | KAN-268 |
| AC19 | KAN-259 |
| AC20 | KAN-268 |
| AC21 | KAN-268 |
| AC22 | KAN-261 |
| AC23 | KAN-261 |
| AC24 | KAN-261 |
| AC25 | KAN-263, KAN-268 |
| AC26 | KAN-261, KAN-267 |
| AC27 | KAN-261 |
| AC28 | KAN-267 |
| AC29 | KAN-261 |
| AC30 | KAN-263 |
| AC31 | KAN-263 |
| AC32 | KAN-268 |
| AC33 | KAN-268 |
| AC34 | KAN-261 |
| AC35 | KAN-265 |
| AC36 | KAN-271 |
| AC37 | KAN-259 |
| AC38 | KAN-271 |
| AC39 | KAN-271 |

*Coverage: all 35 FRs, 13 NFRs, 22 ECs, and 39 ACs mapped to at least one story.
100% structural coverage.*

## Related Documentation
- Feature spec: [`BACKLOG-Anomaly-Workflow.md`](../features/BACKLOG-Anomaly-Workflow.md)
- Sibling features: [`BACKLOG-Pulse.md`](../features/BACKLOG-Pulse.md), [`BACKLOG-Trends.md`](../features/BACKLOG-Trends.md)

## Tracker Link
- Epic: [KAN-254](https://c-pal.atlassian.net/browse/KAN-254) — sprint S4, 2026-11-09 → 2026-11-20 (moved from the expired S3 on 2026-11-19)
