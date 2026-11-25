# Epic: Pulse — live contract dashboard

> **Single source of truth:** requirement **text** lives in exactly one place, the
> feature spec. The epic doc and the tracker description reference requirements
> **by ID only**.

## Tracker

| Field | Value |
| --- | --- |
| Key | `KAN-270` |

## Subject
Pulse — live contract dashboard

## Business Case
Quality (transport conditions, complaint investigation) and Planning (logistics
follow-up, lot flow) need one view of where a contract's devices are, how they got
there, and what is wrong with them. Today that answer is split across Suivi en direct,
Historique and Pilotage. Pulse puts the last known position, a replayable period,
four synchronised measure charts, lot focus and the anomaly entry point on one page.

## Description
Add a new `/pulse` page to `cpaltracker-react` backed by a new snapshot endpoint
`GET /api/v1/contracts/pulse.php/{contractId}` in `cpaltracker.web`, and two opt-in
parameters (`measures`, `chipIds`) on `contracts/history.php`. The page shows a map of
last known positions, a date bar with play/pause and a shared cursor driving the map and
four stacked charts (weight, temperature, humidity, acceleration), a lot filter and a
device filter, and the contract alert light that opens the anomaly drawer. The work is
strictly additive. The database objects (`V_PULSE`, `V_ANOMALIES_OUVERTES`,
`T_DDLOT.Statut`, …) are already built in
`cpaltracker.web/docker/mysql/init/03-dashboard.sql` and are not part of any story.

## Requirement References
Feature spec: [`IN-PROGRESS-Pulse.md`](../features/IN-PROGRESS-Pulse.md) (v1 scope: see its "Scope v1" section) — 41 FRs (FR1–FR41),
12 NFRs (NFR1–NFR12), 39 ACs (AC1–AC39), 25 edge cases (EC1–EC25). This epic references
requirements by ID only; requirement text lives solely in the feature spec. Open
questions OQ1–OQ12 are recorded in the spec; the stories that depend on one say so.

## KPIs
- Snapshot endpoint p95: < 1 s for 100 devices, < 3 s for the Pharma stress contract (NFR3).
- Series response to map and charts rendered: < 2 s at the 9,000-reading target (NFR4).
- Peak PHP memory of History with `measures=m,ta,hr,g` on the full Pharma range: < 128 MB.
- Zero byte-level differences in existing endpoint responses (NFR1).
- Adoption from server access logs: share of Pulse sessions that request a series, and
  share that use `chipIds`, over the first month.
- Zero authorization leaks (NFR2).

## Scope v1 (2026-11-19)
Epic trimmed to one story, KAN-272 (5 points): Live Tracking layout plus lot information and a lot filter. Timeline, charts, replay, History parameters and the device filter are deferred to v2; the anomaly drawer opener moved to KAN-261. The Description above is the full (v2) vision.

### Review 2026-11-23
The owner's review of 2026-11-23, and his layout decision of 2026-11-20, are recorded in the spec ("Scope v1" → "Review 2026-11-23"). Changed: FR9, FR12, FR24, FR28, FR34, AC21 (amended in place); FR32 moved from deferred v2 into KAN-272. New: FR36–FR41, AC34–AC39, EC23–EC25, OQ11–OQ12, all mapped to KAN-272. The anomaly list and the anomaly view shown in the FR37 side panel are delivered by KAN-261 and KAN-263 (anomaly spec FR36–FR39). Out of scope, confirmed: messaging; contracts in the user dropdown. Nothing is left without a story.

## Dependencies
- `cpaltracker.web/docker/mysql/init/03-dashboard.sql` (built and pushed, cpaltracker.web commit 173a3b9; shared by Pulse, Trends and Anomaly Workflow).
- Anomaly Workflow feature ([`IN-PROGRESS-Anomaly-Workflow.md`](../features/IN-PROGRESS-Anomaly-Workflow.md)):
  owns `anomalies/summary.php`, the anomaly list endpoint and the drawer; the drawer opener on Pulse is FR28, moved to KAN-261.
- Trends feature ([`IN-PROGRESS-Trends.md`](../features/IN-PROGRESS-Trends.md)): also adds a route and
  nav entry; both touch `App.tsx`, `paths.ts`, `AppShellLayout.tsx` (handled inside KAN-272).
- Production GRANTs for `cpal_web` (block 9 of `03-dashboard.sql`) on the MariaDB 10.11 VPS (rollout is KAN-259).

## Child Stories

Software Items do not apply to c-PAL (spec §11a), so the column is omitted. v1 scope was trimmed on 2026-11-19 (see "Scope v1" in the spec): one vertical slice, backend + UI + its own tests.

| Key | Story | Pts | Depends on | Repos / files owned | Covered Requirements |
| --- | --- | --- | --- | --- | --- |
| KAN-272 | Pulse — Live Tracking with lot information | 5 | — | `cpaltracker.web`: `api/v1/contracts/pulse.php` (reads `V_PULSE`), PHPUnit Pulse tests, `api/openapi.yaml` (pulse path only). `cpaltracker-react`: route `/pulse` and nav entry (`src/App.tsx`, `src/auth/paths.ts`, `src/chrome/AppShellLayout.tsx`, `src/chrome/DocumentTitle.tsx`), `src/routes/Pulse.tsx`, `src/api/pulse.ts`, Pulse page reusing `src/live-tracking` components, lot filter, Vitest tests | FR1–FR13, FR24, FR32, FR34, FR35, FR36–FR41; NFR1, NFR2, NFR5, NFR8, NFR10, NFR11, NFR12 |
| KAN-273 … KAN-281 | [Reporté v2] — backlog (KAN-273, KAN-274, KAN-275, KAN-276, KAN-277, KAN-278, KAN-279, KAN-280, KAN-281) | — | — | Retired; label `reporte-v2` | Deferred requirements, see traceability |

*Total: 5 points across 1 story.*

Moved out of this epic: FR28 (anomalies drawer opener on Pulse), with AC18 and EC19, to KAN-261 (Anomaly epic); NFR6 (production GRANTs) to KAN-259. The DB objects are done (`done (DB, 173a3b9)`); production rollout is KAN-259 for all three epics.

## Requirement Traceability

Every FR, NFR, AC and EC ID of the spec appears exactly once. "deferred v2" requirements stay in the spec.

| Requirement | Story key / status |
| --- | --- |
| FR1 | KAN-272 |
| FR2 | KAN-272 |
| FR3 | KAN-272 |
| FR4 | KAN-272 |
| FR5 | KAN-272 |
| FR6 | KAN-272 |
| FR7 | KAN-272 |
| FR8 | KAN-272 |
| FR9 | KAN-272 |
| FR10 | KAN-272 |
| FR11 | KAN-272 |
| FR12 | KAN-272 |
| FR13 | KAN-272 |
| FR14 | deferred v2 |
| FR15 | deferred v2 |
| FR16 | deferred v2 |
| FR17 | deferred v2 |
| FR18 | deferred v2 |
| FR19 | deferred v2 |
| FR20 | deferred v2 |
| FR21 | deferred v2 |
| FR22 | deferred v2 |
| FR23 | deferred v2 |
| FR24 | KAN-272 |
| FR25 | deferred v2 |
| FR26 | deferred v2 |
| FR27 | deferred v2 |
| FR28 | moved KAN-261 |
| FR29 | deferred v2 |
| FR30 | deferred v2 |
| FR31 | deferred v2 |
| FR32 | KAN-272 (moved from deferred v2, 2026-11-23) |
| FR33 | deferred v2 |
| FR34 | KAN-272 |
| FR35 | KAN-272 |
| FR36 | KAN-272 |
| FR37 | KAN-272 |
| FR38 | KAN-272 |
| FR39 | KAN-272 |
| FR40 | KAN-272 |
| FR41 | KAN-272 |
| NFR1 | KAN-272 |
| NFR2 | KAN-272 |
| NFR3 | deferred v2 |
| NFR4 | deferred v2 |
| NFR5 | KAN-272 |
| NFR6 | moved KAN-259 |
| NFR7 | deferred v2 |
| NFR8 | KAN-272 |
| NFR9 | deferred v2 |
| NFR10 | KAN-272 |
| NFR11 | KAN-272 |
| NFR12 | KAN-272 |
| AC1 | KAN-272 |
| AC2 | KAN-272 |
| AC3 | KAN-272 |
| AC4 | KAN-272 |
| AC5 | KAN-272 |
| AC6 | KAN-272 |
| AC7 | KAN-272 |
| AC8 | KAN-272 |
| AC9 | KAN-272 |
| AC10 | deferred v2 |
| AC11 | deferred v2 |
| AC12 | deferred v2 |
| AC13 | deferred v2 |
| AC14 | deferred v2 |
| AC15 | deferred v2 |
| AC16 | deferred v2 |
| AC17 | KAN-272 |
| AC18 | moved KAN-261 |
| AC19 | deferred v2 |
| AC20 | deferred v2 |
| AC21 | KAN-272 |
| AC22 | KAN-272 |
| AC23 | KAN-272 |
| AC24 | deferred v2 |
| AC25 | deferred v2 |
| AC26 | KAN-272, moved KAN-259 (NFR6 part) |
| AC27 | KAN-272 |
| AC28 | deferred v2 |
| AC29 | KAN-272 |
| AC30 | KAN-272 |
| AC31 | KAN-272 |
| AC32 | KAN-272 |
| AC33 | KAN-272 |
| AC34 | KAN-272 |
| AC35 | KAN-272 |
| AC36 | KAN-272 |
| AC37 | KAN-272 |
| AC38 | KAN-272 |
| AC39 | KAN-272 |
| EC1 | KAN-272 |
| EC2 | KAN-272 |
| EC3 | KAN-272 |
| EC4 | KAN-272 |
| EC5 | KAN-272 |
| EC6 | KAN-272 |
| EC7 | KAN-272 |
| EC8 | KAN-272 |
| EC9 | KAN-272 |
| EC10 | deferred v2 |
| EC11 | KAN-272 |
| EC12 | deferred v2 |
| EC13 | deferred v2 |
| EC14 | deferred v2 |
| EC15 | deferred v2 |
| EC16 | KAN-272 |
| EC17 | KAN-272 |
| EC18 | deferred v2 |
| EC19 | moved KAN-261 |
| EC20 | KAN-272 |
| EC21 | deferred v2 |
| EC22 | deferred v2 |
| EC23 | KAN-272 |
| EC24 | KAN-272 |
| EC25 | KAN-272 |

*Coverage: all 41 FRs, 12 NFRs, 39 ACs and 25 ECs placed exactly once (kept story, moved, or deferred v2). Changed on 2026-11-23: FR9, FR12, FR24, FR28, FR32, FR34, AC21; added: FR36–FR41, AC34–AC39, EC23–EC25.*

## Related Documentation
- Feature spec: [`IN-PROGRESS-Pulse.md`](../features/IN-PROGRESS-Pulse.md)
- Siblings: [`IN-PROGRESS-Anomaly-Workflow.md`](../features/IN-PROGRESS-Anomaly-Workflow.md), [`IN-PROGRESS-Trends.md`](../features/IN-PROGRESS-Trends.md)

## Tracker Link
- Epic: [KAN-270](https://c-pal.atlassian.net/browse/KAN-270) — sprint S4, 2026-11-09 → 2026-11-20 (moved from the expired S3 on 2026-11-19)
