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
Feature spec: [`BACKLOG-Trends.md`](../features/BACKLOG-Trends.md) — FR1–FR40, NFR1–NFR11, AC1–AC29, EC1–EC22, D1–D17. This epic references requirements by ID only.

## KPIs
See feature spec section 13 (refresh duration, API p95 under 1 s, page under 2 s with 100 lots, 100% rule fixtures, Gael's 10-lot check, lots closed per month, 0 regressions).

## Dependencies
- `03-dashboard.sql` objects (built, uncommitted at epic creation).
- Sibling features: Pulse (shares App.tsx and nav files with KAN-260), Anomaly Workflow (no direct dependency).
- Gael COQUIO validation of feature spec section 10.3 and open questions OQ1–OQ11 (defaults apply meanwhile).

## Child Stories

Dependencies and owned files are for parallel scheduling. Schedule: wave 1 = KAN-255, KAN-258; wave 2 = KAN-256, KAN-257; wave 3 = KAN-260; wave 4 = KAN-262, KAN-264; wave 5 = KAN-266; wave 6 = KAN-269.

| Key | Story | Pts | Depends on | Repos / files owned | Covered Requirements |
| --- | --- | --- | --- | --- | --- |
| KAN-255 | Trends data model tests and rollout | 8 | - | cpaltracker.web: docker/mysql/init/03-dashboard.sql (grants section only), tests/sql/trends/*, deploy/migration note (VPS, MariaDB 10.11) | FR1-FR14, NFR1, NFR2, NFR9, NFR11, EC2-EC9, EC11-EC14, EC20, AC1-AC7, AC27, D1-D17 |
| KAN-256 | Trends read endpoint | 8 | KAN-255 | cpaltracker.web: api/v1/contracts/trends.php, trends logic file, OpenAPI entry, tests/php/trends* | FR15-FR19, NFR3, NFR5, EC10, EC22, AC8, AC9, AC12 |
| KAN-257 | Close lot endpoint | 5 | KAN-255 | cpaltracker.web: api/v1/lots/close.php, OpenAPI entry, tests/php/lots_close* | FR19, FR20-FR24, NFR5, NFR6, EC17, EC18, AC10-AC12 |
| KAN-258 | ScatterChart kit component | 8 | - | cpaltracker-react: src/components/kit/ScatterChart/*, kit showcase entry, kit index export | FR28 (component), FR39, NFR8, AC25 |
| KAN-260 | Trends route, nav and page shell | 8 | KAN-256 | cpaltracker-react: src/App.tsx (route), nav config (shared with Pulse), src/pages/Trends/TrendsPage.tsx + header/toggles/states, API client for trends | FR25-FR27, FR38-FR40, NFR4 (toggles), NFR7, NFR8, EC1, EC7, EC10, AC13-AC15, AC23, AC24, AC26 |
| KAN-262 | Trends quadrant chart | 8 | KAN-258, KAN-260 | cpaltracker-react: src/pages/Trends/TrendsQuadrant.tsx (+ tests) | FR28-FR30, NFR4, EC19, EC21, AC16, AC17 |
| KAN-264 | Trends lot table | 8 | KAN-260 | cpaltracker-react: src/pages/Trends/TrendsTable.tsx, units helpers (+ tests) | FR31-FR36, EC2-EC4, EC6, EC8, EC9, EC15, EC16, AC18-AC21 |
| KAN-266 | Close lot from Trends | 5 | KAN-257, KAN-260, KAN-264 | cpaltracker-react: src/pages/Trends/CloseLotDialog.tsx, close API client (+ tests) | FR37, EC17, AC22 |
| KAN-269 | Trends end-to-end tests | 8 | KAN-255, KAN-256, KAN-257, KAN-260, KAN-262, KAN-264, KAN-266 | cpaltracker-react: e2e/trends.spec.ts; cpaltracker.web: test data/backfill script | NFR10, EC20, AC28, AC29 |

*Total: 66 points across 9 stories.*

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |
| FR1 | KAN-255 |
| FR2 | KAN-255 |
| FR3 | KAN-255 |
| FR4 | KAN-255 |
| FR5 | KAN-255 |
| FR6 | KAN-255 |
| FR7 | KAN-255 |
| FR8 | KAN-255 |
| FR9 | KAN-255 |
| FR10 | KAN-255 |
| FR11 | KAN-255 |
| FR12 | KAN-255 |
| FR13 | KAN-255 |
| FR14 | KAN-255 |
| FR15 | KAN-256 |
| FR16 | KAN-256 |
| FR17 | KAN-256 |
| FR18 | KAN-256 |
| FR19 | KAN-256, KAN-257 |
| FR20 | KAN-257 |
| FR21 | KAN-257 |
| FR22 | KAN-257 |
| FR23 | KAN-257 |
| FR24 | KAN-257 |
| FR25 | KAN-260 |
| FR26 | KAN-260 |
| FR27 | KAN-260 |
| FR28 | KAN-258, KAN-262 |
| FR29 | KAN-262 |
| FR30 | KAN-262 |
| FR31 | KAN-264 |
| FR32 | KAN-264 |
| FR33 | KAN-264 |
| FR34 | KAN-264 |
| FR35 | KAN-264 |
| FR36 | KAN-264 |
| FR37 | KAN-266 |
| FR38 | KAN-260 |
| FR39 | KAN-258, KAN-260 |
| FR40 | KAN-260 |
| NFR1 | KAN-255 |
| NFR2 | KAN-255 |
| NFR3 | KAN-256 |
| NFR4 | KAN-260, KAN-262 |
| NFR5 | KAN-256, KAN-257 |
| NFR6 | KAN-257 |
| NFR7 | KAN-260 |
| NFR8 | KAN-258, KAN-260 |
| NFR9 | KAN-255 |
| NFR10 | KAN-269 |
| NFR11 | KAN-255 |
| AC1 | KAN-255 |
| AC2 | KAN-255 |
| AC3 | KAN-255 |
| AC4 | KAN-255 |
| AC5 | KAN-255 |
| AC6 | KAN-255 |
| AC7 | KAN-255 |
| AC8 | KAN-256 |
| AC9 | KAN-256 |
| AC10 | KAN-257 |
| AC11 | KAN-257 |
| AC12 | KAN-256, KAN-257 |
| AC13 | KAN-260 |
| AC14 | KAN-260 |
| AC15 | KAN-260 |
| AC16 | KAN-262 |
| AC17 | KAN-262 |
| AC18 | KAN-264 |
| AC19 | KAN-264 |
| AC20 | KAN-264 |
| AC21 | KAN-264 |
| AC22 | KAN-266 |
| AC23 | KAN-260 |
| AC24 | KAN-260 |
| AC25 | KAN-258 |
| AC26 | KAN-260 |
| AC27 | KAN-255 |
| AC28 | KAN-269 |
| AC29 | KAN-269 |
| EC1 | KAN-260 |
| EC2 | KAN-255, KAN-264 |
| EC3 | KAN-255, KAN-264 |
| EC4 | KAN-255, KAN-264 |
| EC5 | KAN-255 |
| EC6 | KAN-255, KAN-264 |
| EC7 | KAN-255, KAN-260 |
| EC8 | KAN-255, KAN-264 |
| EC9 | KAN-255, KAN-264 |
| EC10 | KAN-256, KAN-260 |
| EC11 | KAN-255 |
| EC12 | KAN-255 |
| EC13 | KAN-255 |
| EC14 | KAN-255 |
| EC15 | KAN-264 |
| EC16 | KAN-264 |
| EC17 | KAN-257, KAN-266 |
| EC18 | KAN-257 |
| EC19 | KAN-262 |
| EC20 | KAN-255, KAN-269 |
| EC21 | KAN-262 |
| EC22 | KAN-256 |
| D1 | KAN-255 |
| D2 | KAN-255 |
| D3 | KAN-255 |
| D4 | KAN-255 |
| D5 | KAN-255 |
| D6 | KAN-255 |
| D7 | KAN-255 |
| D8 | KAN-255 |
| D9 | KAN-255 |
| D10 | KAN-255 |
| D11 | KAN-255 |
| D12 | KAN-255 |
| D13 | KAN-255 |
| D14 | KAN-255 |
| D15 | KAN-255 |
| D16 | KAN-255 |
| D17 | KAN-255 |

*Coverage: all 40 FRs, 11 NFRs, 29 ACs (plus 22 ECs and D1–D17) mapped to at least one story. 100% structural coverage. OQ1–OQ11 are open questions with proposed defaults, applied in the stories above; they carry no story of their own.*

## Related Documentation
- Feature spec: [`BACKLOG-Trends.md`](../features/BACKLOG-Trends.md)

## Tracker Link
- Epic: [KAN-253](https://c-pal.atlassian.net/browse/KAN-253) — sprint S4, 2026-11-09 → 2026-11-20 (moved from the expired S3 on 2026-11-19)
