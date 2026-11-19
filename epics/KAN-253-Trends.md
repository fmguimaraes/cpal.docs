# Trends — lot consumption and depletion risk

> **Single source of truth:** requirement text lives in the feature spec. This epic references requirements by ID only.

## Tracker

| Field | Value |
| --- | --- |
| Key | `KAN-253` |

## Subject
Trends — lot consumption and depletion risk

## Business Case
Operators cannot see which lots will not be consumed before they expire, or which will run out early. A weekly consumption snapshot per lot, scored against the remaining shelf life, gives an early risk read per lot and lets finished lots be closed.

## Description
Add a new, strictly additive `/trends` page in `cpaltracker-react` with its own navigation entry: a quadrant chart (new kit `ScatterChart`) and a lot table with risk badges, unit toggle and a close-lot action. The backend adds `contracts/trends.php` (read) and `lots/close.php` (close) in `cpaltracker.web`. The database part (`T_LOT_CONSO_SNAP`, `sp_lot_conso_refresh(_at)`, `ev_lot_conso_weekly`, `V_TRENDS`, `T_DDLOT.Statut`) is already built in `cpaltracker.web/docker/mysql/init/03-dashboard.sql` (shared with Pulse and Anomaly Workflow); no story recreates it. Software Items and Developer Epic Context do not apply to c-PAL.

## Requirement References
Feature spec: [`IN-PROGRESS-Trends.md`](../features/IN-PROGRESS-Trends.md) (v1 subset in its "Scope v1" section) — FR1–FR40, NFR1–NFR11, AC1–AC29, EC1–EC22, D1–D17. This epic references requirements by ID only.

## KPIs
See feature spec section 13 (refresh duration, API p95 under 1 s, page under 2 s with 100 lots, 100% rule fixtures, Gael's 10-lot check, lots closed per month, 0 regressions).

## Dependencies
- `03-dashboard.sql` objects (built, uncommitted at epic creation).
- Sibling features: Pulse (shares App.tsx and nav files with KAN-260), Anomaly Workflow (no direct dependency).
- Gael COQUIO validation of feature spec section 10.3 and open questions OQ1–OQ11 (defaults apply meanwhile).

## Child Stories

v1 scope (trimmed 2026-11-19): Trends is a page reading a hardcoded table, one row per lot, no calculation. See "Scope v1" in the feature spec. The computed pipeline stays local in `03-dashboard.sql` §6 and is v2. Production rollout of the DB part is KAN-259 (all three dashboard epics).

| Key | Story | Pts | Depends on | Repos / files owned | Covered Requirements |
| --- | --- | --- | --- | --- | --- |
| KAN-256 | Trends page — from a hardcoded table | 5 | - | cpaltracker.web: seeded table with fixed rows (SQL), api/v1/contracts/trends.php reading it, tests/php/trends*; cpaltracker-react: route `/trends`, nav entry, src/trends/* (DataTable, Plotly scatter used directly), Vitest tests | FR15-FR17, FR25, FR26, FR28-FR33, FR38, FR39, NFR5, NFR7, NFR8, AC8, AC13, AC14, AC16-AC19, AC23, AC25, AC26, AC29, EC1, EC10, EC19, EC21, EC22 |
| KAN-255, KAN-257, KAN-258, KAN-260, KAN-262, KAN-264, KAN-266, KAN-269 | [Reporté v2] — backlog | - | - | Jira backlog, label `reporte-v2` | deferred v2 |

*Total: 5 points across 1 story.*

## Requirement Traceability

Every ID of the feature spec maps to KAN-256 or "deferred v2" (see "Scope v1" in the spec).

| Requirement | Story key(s) |
| --- | --- |
| FR1 | deferred v2 |
| FR2 | deferred v2 |
| FR3 | deferred v2 |
| FR4 | deferred v2 |
| FR5 | deferred v2 |
| FR6 | deferred v2 |
| FR7 | deferred v2 |
| FR8 | deferred v2 |
| FR9 | deferred v2 |
| FR10 | deferred v2 |
| FR11 | deferred v2 |
| FR12 | deferred v2 |
| FR13 | deferred v2 |
| FR14 | deferred v2 |
| FR15 | KAN-256 |
| FR16 | KAN-256 |
| FR17 | KAN-256 |
| FR18 | deferred v2 |
| FR19 | deferred v2 |
| FR20 | deferred v2 |
| FR21 | deferred v2 |
| FR22 | deferred v2 |
| FR23 | deferred v2 |
| FR24 | deferred v2 |
| FR25 | KAN-256 |
| FR26 | KAN-256 |
| FR27 | deferred v2 |
| FR28 | KAN-256 |
| FR29 | KAN-256 |
| FR30 | KAN-256 |
| FR31 | KAN-256 |
| FR32 | KAN-256 |
| FR33 | KAN-256 |
| FR34 | deferred v2 |
| FR35 | deferred v2 |
| FR36 | deferred v2 |
| FR37 | deferred v2 |
| FR38 | KAN-256 |
| FR39 | KAN-256 |
| FR40 | deferred v2 |
| NFR1 | deferred v2 |
| NFR2 | deferred v2 |
| NFR3 | deferred v2 |
| NFR4 | deferred v2 |
| NFR5 | KAN-256 |
| NFR6 | deferred v2 |
| NFR7 | KAN-256 |
| NFR8 | KAN-256 |
| NFR9 | deferred v2 |
| NFR10 | deferred v2 |
| NFR11 | deferred v2 |
| AC1 | deferred v2 |
| AC2 | deferred v2 |
| AC3 | deferred v2 |
| AC4 | deferred v2 |
| AC5 | deferred v2 |
| AC6 | deferred v2 |
| AC7 | deferred v2 |
| AC8 | KAN-256 |
| AC9 | deferred v2 |
| AC10 | deferred v2 |
| AC11 | deferred v2 |
| AC12 | deferred v2 |
| AC13 | KAN-256 |
| AC14 | KAN-256 |
| AC15 | deferred v2 |
| AC16 | KAN-256 |
| AC17 | KAN-256 |
| AC18 | KAN-256 |
| AC19 | KAN-256 |
| AC20 | deferred v2 |
| AC21 | deferred v2 |
| AC22 | deferred v2 |
| AC23 | KAN-256 |
| AC24 | deferred v2 |
| AC25 | KAN-256 |
| AC26 | KAN-256 |
| AC27 | deferred v2 |
| AC28 | deferred v2 |
| AC29 | KAN-256 |
| EC1 | KAN-256 |
| EC2 | deferred v2 |
| EC3 | deferred v2 |
| EC4 | deferred v2 |
| EC5 | deferred v2 |
| EC6 | deferred v2 |
| EC7 | deferred v2 |
| EC8 | deferred v2 |
| EC9 | deferred v2 |
| EC10 | KAN-256 |
| EC11 | deferred v2 |
| EC12 | deferred v2 |
| EC13 | deferred v2 |
| EC14 | deferred v2 |
| EC15 | deferred v2 |
| EC16 | deferred v2 |
| EC17 | deferred v2 |
| EC18 | deferred v2 |
| EC19 | KAN-256 |
| EC20 | deferred v2 |
| EC21 | KAN-256 |
| EC22 | KAN-256 |
| D1 | deferred v2 |
| D2 | deferred v2 |
| D3 | deferred v2 |
| D4 | deferred v2 |
| D5 | deferred v2 |
| D6 | deferred v2 |
| D7 | deferred v2 |
| D8 | deferred v2 |
| D9 | deferred v2 |
| D10 | deferred v2 |
| D11 | deferred v2 |
| D12 | deferred v2 |
| D13 | deferred v2 |
| D14 | deferred v2 |
| D15 | deferred v2 |
| D16 | deferred v2 |
| D17 | deferred v2 |

*Coverage: 32 IDs mapped to KAN-256, 87 deferred to v2; every FR, NFR, AC, EC and D ID appears exactly once. OQ1–OQ11 are open questions and carry no story; those on the computed pipeline are v2.*

## Related Documentation
- Feature spec: [`IN-PROGRESS-Trends.md`](../features/IN-PROGRESS-Trends.md)

## Tracker Link
- Epic: [KAN-253](https://c-pal.atlassian.net/browse/KAN-253) — sprint S4, 2026-11-09 → 2026-11-20 (moved from the expired S3 on 2026-11-19)
