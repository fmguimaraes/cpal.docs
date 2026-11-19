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
Feature spec: [`BACKLOG-Pulse.md`](../features/BACKLOG-Pulse.md) — 35 FRs (FR1–FR35),
12 NFRs (NFR1–NFR12), 33 ACs (AC1–AC33), 22 edge cases (EC1–EC22). This epic references
requirements by ID only; requirement text lives solely in the feature spec. Open
questions OQ1–OQ10 are recorded in the spec; the stories that depend on one say so.

## KPIs
- Snapshot endpoint p95: < 1 s for 100 devices, < 3 s for the Pharma stress contract (NFR3).
- Series response to map and charts rendered: < 2 s at the 9,000-reading target (NFR4).
- Peak PHP memory of History with `measures=m,ta,hr,g` on the full Pharma range: < 128 MB.
- Zero byte-level differences in existing endpoint responses (NFR1).
- Adoption from server access logs: share of Pulse sessions that request a series, and
  share that use `chipIds`, over the first month.
- Zero authorization leaks (NFR2).

## Dependencies
- `cpaltracker.web/docker/mysql/init/03-dashboard.sql` (built, uncommitted at the time of
  writing; shared by Pulse, Trends and Anomaly Workflow).
- Anomaly Workflow feature ([`BACKLOG-Anomaly-Workflow.md`](../features/BACKLOG-Anomaly-Workflow.md)):
  owns `anomalies/summary.php`, the anomaly list endpoint and the drawer that KAN-280 consumes.
- Trends feature ([`BACKLOG-Trends.md`](../features/BACKLOG-Trends.md)): also adds a route and
  nav entry; both touch `App.tsx`, `paths.ts`, `AppShellLayout.tsx` (see KAN-274).
- Production GRANTs for `cpal_web` (block 9 of `03-dashboard.sql`) on the MariaDB 10.11 VPS.

## Child Stories

Software Items do not apply to c-PAL (spec §11a), so the column is omitted. The
Depends-on and Owns columns drive parallel scheduling: stories with no unmet dependency
can run at once; stories sharing a file are noted.

| Key | Story | Pts | Depends on | Repos / files owned | Covered Requirements |
| --- | --- | --- | --- | --- | --- |
| KAN-272 | Pulse snapshot endpoint | 8 | — | `cpaltracker.web`: `api/v1/contracts/pulse.php` and new `pulse_logic.php` (new), `tests/Api` Pulse tests (new), `api/openapi.yaml` (pulse path only), VPS migration/GRANT release note | FR4–FR8, NFR2, NFR3, NFR5, NFR6, NFR11 (part) |
| KAN-273 | History opt-in measures and chipIds | 8 | — | `cpaltracker.web`: `api/v1/contracts/history.php`, `history_logic.php`, `history_enrichment.php`, `tests/History` (new tests), `api/openapi.yaml` (history parameters only) | FR31, FR32, NFR1 (API), NFR4 (memory), NFR11 (part) |
| KAN-274 | Pulse route and nav entry | 2 | — | `cpaltracker-react`: `src/App.tsx`, `src/auth/paths.ts`, `src/chrome/AppShellLayout.tsx`, `src/chrome/DocumentTitle.tsx`, placeholder `src/routes/Pulse.tsx` (shared files with Trends: merge or sequence with its route story) | FR1, FR2 |
| KAN-275 | Pulse page shell and API client | 5 | KAN-274 | `cpaltracker-react`: `src/routes/Pulse.tsx`, new `src/pulse/PulsePage.tsx` (layout, states), new `src/api/pulse.ts`, `src/api/history.ts` (optional `measures`/`chipIds`), `MeasureKey` widening | FR3, FR33, FR34, FR35, NFR8 |
| KAN-276 | Pulse snapshot map | 8 | KAN-275, KAN-272 | `cpaltracker-react`: new `src/pulse/SnapshotMap*`, `usePulseSnapshot`, refresh and last-updated | FR9–FR13, NFR10 |
| KAN-277 | Pulse timeline bar and shared cursor | 13 | KAN-275, KAN-276, KAN-273 | `cpaltracker-react`: new `src/pulse/TimelineBar*`, range state and cursor hook, `src/history/TimelineMap.tsx` (optional controlled-instant prop only) | FR14–FR19, NFR4 (render), NFR12 |
| KAN-278 | Pulse stacked measure charts | 8 | KAN-275, KAN-277 | `cpaltracker-react`: new `src/pulse/PulseCharts*`, `src/history/chartData.ts` (`ChartField` widened with `g`), History kept at three measures | FR20–FR22, FR33 (History non-regression) |
| KAN-279 | Pulse lot and device filters | 13 | KAN-277, KAN-272, KAN-273 | `cpaltracker-react`: new `src/pulse/filters/*` (lot search, device filter, lot-to-range and hook-window logic), Vitest for the pure logic | FR23–FR27 |
| KAN-280 | Pulse anomaly entry point | 5 | KAN-276, KAN-279, KAN-261 (anomalies summary + list endpoints), KAN-265 (anomaly drawer) | `cpaltracker-react`: new `src/pulse/AnomalyLight*`, per-device counts in tooltip and device filter | FR28–FR30 |
| KAN-281 | Pulse quality and non-regression | 8 | KAN-272 … KAN-280 | `cpaltracker-react`: `e2e/pulse.spec.ts` (new), accessibility fixes inside `src/pulse/*`; full PHPUnit, Vitest, Playwright runs | NFR1, NFR7, NFR9 |

*Total: 78 points across 10 stories.*

Suggested schedule: wave 1 KAN-272, KAN-273, KAN-274 in parallel; wave 2 KAN-275;
wave 3 KAN-276; wave 4 KAN-277; wave 5 KAN-278 and KAN-279 in parallel (both compose into
`PulsePage.tsx`; coordinate that file); wave 6 KAN-280; wave 7 KAN-281.

## Requirement Traceability

Per-story tests are part of each story; NFR7 and NFR9 are implemented per story and
verified in KAN-281.

| Requirement | Story key(s) |
| --- | --- |
| FR1 | KAN-274 |
| FR2 | KAN-274 |
| FR3 | KAN-275 |
| FR4 | KAN-272 |
| FR5 | KAN-272 |
| FR6 | KAN-272 |
| FR7 | KAN-272 |
| FR8 | KAN-272 |
| FR9 | KAN-276 |
| FR10 | KAN-276 |
| FR11 | KAN-276 |
| FR12 | KAN-276 |
| FR13 | KAN-276 |
| FR14 | KAN-277 |
| FR15 | KAN-277 |
| FR16 | KAN-277 |
| FR17 | KAN-277 |
| FR18 | KAN-277 |
| FR19 | KAN-277 |
| FR20 | KAN-278 |
| FR21 | KAN-278 |
| FR22 | KAN-278 |
| FR23 | KAN-279 |
| FR24 | KAN-279 |
| FR25 | KAN-279 |
| FR26 | KAN-279 |
| FR27 | KAN-279 |
| FR28 | KAN-280 |
| FR29 | KAN-280 |
| FR30 | KAN-280 |
| FR31 | KAN-273 |
| FR32 | KAN-273 |
| FR33 | KAN-275, KAN-278 |
| FR34 | KAN-275 |
| FR35 | KAN-275 |
| NFR1 | KAN-273, KAN-281 |
| NFR2 | KAN-272, KAN-273 |
| NFR3 | KAN-272 |
| NFR4 | KAN-273, KAN-277 |
| NFR5 | KAN-272, KAN-273 |
| NFR6 | KAN-272 |
| NFR7 | KAN-277, KAN-279, KAN-280, KAN-281 |
| NFR8 | KAN-275, KAN-278 |
| NFR9 | KAN-272, KAN-273, KAN-279, KAN-281 |
| NFR10 | KAN-276 |
| NFR11 | KAN-272, KAN-273 |
| NFR12 | KAN-277 |
| AC1 | KAN-274 |
| AC2 | KAN-275 |
| AC3 | KAN-272 |
| AC4 | KAN-272 |
| AC5 | KAN-272 |
| AC6 | KAN-276 |
| AC7 | KAN-276 |
| AC8 | KAN-276 |
| AC9 | KAN-276, KAN-280 |
| AC10 | KAN-277 |
| AC11 | KAN-277 |
| AC12 | KAN-277 |
| AC13 | KAN-277 |
| AC14 | KAN-278 |
| AC15 | KAN-278 |
| AC16 | KAN-279 |
| AC17 | KAN-279 |
| AC18 | KAN-280 |
| AC19 | KAN-273 |
| AC20 | KAN-275, KAN-278 |
| AC21 | KAN-275 |
| AC22 | KAN-275, KAN-281 |
| AC23 | KAN-281 |
| AC24 | KAN-272 |
| AC25 | KAN-273, KAN-277 |
| AC26 | KAN-272, KAN-273 |
| AC27 | KAN-281 |
| AC28 | KAN-281 |
| AC29 | KAN-272, KAN-273 |
| AC30 | KAN-277 |
| AC31 | KAN-272, KAN-276 |
| AC32 | KAN-276, KAN-277 |
| AC33 | KAN-281 |
| EC1 | KAN-275 |
| EC2 | KAN-272, KAN-276 |
| EC3 | KAN-272, KAN-276 |
| EC4 | KAN-276 |
| EC5 | KAN-276 |
| EC6 | KAN-276 |
| EC7 | KAN-272 |
| EC8 | KAN-272, KAN-279 |
| EC9 | KAN-272, KAN-279 |
| EC10 | KAN-279 |
| EC11 | KAN-272, KAN-277 |
| EC12 | KAN-278 |
| EC13 | KAN-277 |
| EC14 | KAN-279 |
| EC15 | KAN-277 |
| EC16 | KAN-276 |
| EC17 | KAN-276 |
| EC18 | KAN-278 |
| EC19 | KAN-280 |
| EC20 | KAN-275 |
| EC21 | KAN-277 |
| EC22 | KAN-277 |

*Coverage: all 35 FRs, 12 NFRs, 33 ACs and 22 ECs mapped to at least one story.
100% structural coverage.*

## Related Documentation
- Feature spec: [`BACKLOG-Pulse.md`](../features/BACKLOG-Pulse.md)
- Siblings: [`BACKLOG-Anomaly-Workflow.md`](../features/BACKLOG-Anomaly-Workflow.md), [`BACKLOG-Trends.md`](../features/BACKLOG-Trends.md)

## Tracker Link
- Epic: [KAN-270](https://c-pal.atlassian.net/browse/KAN-270) — sprint S4, 2026-11-09 → 2026-11-20 (moved from the expired S3 on 2026-11-19)
