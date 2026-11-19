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
Add a right-side `AnomalyDrawer` to `cpaltracker-react`, opened from Pulse (table of
anomalies with a status filter, detail with status history, status transitions with
comment) and an `anomalies/` API in `cpaltracker.web` (list, detail, transition). v1 scope
is trimmed, see the feature spec "Scope v1"; the map, pagination, summary endpoint and
alert light are deferred to v2. The database part (status, history table, views,
trigger, closing right) is already built and pushed in `docker/mysql/init/03-dashboard.sql` (cpaltracker.web
commit 173a3b9, shared with Pulse and Trends) and is not part of this epic; the
production rollout is KAN-259.

## Requirement References
Feature spec: [`IN-PROGRESS-Anomaly-Workflow.md`](../features/IN-PROGRESS-Anomaly-Workflow.md)
— 35 FRs, 13 NFRs, 22 ECs, 39 ACs (v1 subset: see "Scope v1" in the spec). This epic references requirements by ID only;
requirement text lives solely in the feature spec.

## KPIs
See feature spec §13 (not copied here): time to take in charge, time per transition,
closing backlog, send-back rate, adoption, refused-transition rate, API p95 (NFR5).

## Dependencies
- `cpaltracker.web/docker/mysql/init/03-dashboard.sql` §2–§4 (done, commit 173a3b9).
- Pulse (`IN-PROGRESS-Pulse.md`, KAN-272): hosts the opener button for `AnomalyDrawer`.
- Existing `cpaltracker-react` kit (`Sheet`, `data-table`, `FleetMap`, `ActivityTimeline`, ...).
- Confluence validation of the deviations in feature spec §19 (Gaël).

## Child Stories

Scope v1 trimmed 2026-11-19 (see feature spec "Scope v1"). One vertical slice per story, with its own PHPUnit/Vitest tests. `cpaltracker.web` paths are relative to `api/v1/` unless stated; `cpaltracker-react` paths are under `src/`.

| Key | Story | Pts | Depends on | Repos / files owned |
| --- | --- | --- | --- | --- |
| KAN-261 | Anomalies — table in a drawer, opened from Pulse | 8 | Pulse KAN-272 (for the opener) | cpaltracker.web: `anomalies/list.php`, `anomalies/detail.php`, `anomalies/anomalies_logic.php`, PHPUnit; cpaltracker-react: `anomalies/*` and the opener button on `/pulse`, Vitest |
| KAN-263 | Anomalies — change status with a comment | 3 | KAN-261 | cpaltracker.web: `anomalies/transition.php`, PHPUnit; cpaltracker-react: transition form in the drawer, Vitest |
| KAN-259 | Dashboard DB rollout to production (VPS) | 2 | none | Production SQL: anomaly history table, triggers and views, `V_PULSE`, the hardcoded trends table; GRANTs; collation check |
| KAN-265, KAN-267, KAN-268, KAN-271 | [Reporté v2] — backlog | n/a | n/a | Retired (label `reporte-v2`); their scope is folded into KAN-261 / KAN-263 or deferred |

*Total: 13 points across 3 stories.*

## Requirement Traceability

Every ID of the feature spec maps to a kept story, to "done (DB, 173a3b9)" or to "deferred v2".

| Requirement | Story key(s) / status |
| --- | --- |
| FR1 | KAN-261 |
| FR2 | KAN-261 |
| FR3 | KAN-261 |
| FR4 | KAN-261 |
| FR5 | KAN-261 |
| FR6 | KAN-261 |
| FR7 | deferred v2 |
| FR8 | KAN-261 |
| FR9 | KAN-261 |
| FR10 | deferred v2 |
| FR11 | deferred v2 |
| FR12 | KAN-261 |
| FR13 | KAN-261 |
| FR14 | KAN-263 |
| FR15 | KAN-263 |
| FR16 | KAN-263 |
| FR17 | KAN-263 |
| FR18 | KAN-263 |
| FR19 | KAN-263 |
| FR20 | KAN-261 |
| FR21 | KAN-261 |
| FR22 | KAN-263 |
| FR23 | KAN-263 |
| FR24 | deferred v2 |
| FR25 | KAN-261 |
| FR26 | KAN-261 |
| FR27 | KAN-261 |
| FR28 | deferred v2 |
| FR29 | KAN-261 |
| FR30 | done (DB, 173a3b9) |
| FR31 | deferred v2 |
| FR32 | deferred v2 |
| FR33 | KAN-261 |
| FR34 | KAN-259 |
| FR35 | deferred v2 |
| NFR1 | KAN-261 |
| NFR2 | KAN-261 |
| NFR3 | KAN-263 |
| NFR4 | KAN-263 |
| NFR5 | deferred v2 |
| NFR6 | done (DB, 173a3b9) |
| NFR7 | deferred v2 |
| NFR8 | KAN-261 |
| NFR9 | KAN-261 |
| NFR10 | KAN-261 |
| NFR11 | deferred v2 |
| NFR12 | deferred v2 |
| NFR13 | done (DB, 173a3b9) |
| EC1 | KAN-263 |
| EC2 | KAN-261 |
| EC3 | KAN-261 |
| EC4 | deferred v2 |
| EC5 | KAN-261 |
| EC6 | deferred v2 |
| EC7 | deferred v2 |
| EC8 | KAN-263 |
| EC9 | KAN-263 |
| EC10 | KAN-263 |
| EC11 | KAN-263 |
| EC12 | KAN-261 |
| EC13 | KAN-261 |
| EC14 | KAN-261 |
| EC15 | KAN-263 |
| EC16 | KAN-263 |
| EC17 | KAN-261 |
| EC18 | deferred v2 |
| EC19 | KAN-261 |
| EC20 | KAN-261 |
| EC21 | KAN-263 |
| EC22 | KAN-261 |
| AC1 | KAN-261 |
| AC2 | KAN-261 |
| AC3 | KAN-261 |
| AC4 | KAN-261 |
| AC5 | deferred v2 |
| AC6 | KAN-261 |
| AC7 | KAN-261 |
| AC8 | deferred v2 |
| AC9 | deferred v2 |
| AC10 | KAN-261 |
| AC11 | KAN-263 |
| AC12 | KAN-263 |
| AC13 | KAN-263 |
| AC14 | KAN-263 |
| AC15 | KAN-261 |
| AC16 | KAN-263 |
| AC17 | KAN-263 |
| AC18 | KAN-263 |
| AC19 | done (DB, 173a3b9) |
| AC20 | KAN-263 |
| AC21 | KAN-263 |
| AC22 | KAN-261 |
| AC23 | KAN-261 |
| AC24 | KAN-261 |
| AC25 | KAN-261 |
| AC26 | KAN-261 |
| AC27 | KAN-261 |
| AC28 | deferred v2 |
| AC29 | deferred v2 |
| AC30 | deferred v2 |
| AC31 | KAN-263 |
| AC32 | KAN-261, KAN-263 |
| AC33 | KAN-263 |
| AC34 | deferred v2 |
| AC35 | KAN-261 |
| AC36 | KAN-261 |
| AC37 | KAN-259 |
| AC38 | deferred v2 |
| AC39 | deferred v2 |

*Coverage: all 35 FRs, 13 NFRs, 22 ECs and 39 ACs appear exactly once. Pulse FR28 (the opener) is owned by KAN-261 but belongs to the Pulse spec, so it is not listed here.*

## Related Documentation
- Feature spec: [`IN-PROGRESS-Anomaly-Workflow.md`](../features/IN-PROGRESS-Anomaly-Workflow.md)
- Sibling features: [`IN-PROGRESS-Pulse.md`](../features/IN-PROGRESS-Pulse.md), [`IN-PROGRESS-Trends.md`](../features/IN-PROGRESS-Trends.md)

## Tracker Link
- Epic: [KAN-254](https://c-pal.atlassian.net/browse/KAN-254) — sprint S4, 2026-11-09 → 2026-11-20 (moved from the expired S3 on 2026-11-19)
