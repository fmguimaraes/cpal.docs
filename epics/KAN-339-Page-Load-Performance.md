# Epic: Page Load Performance (Pulse, Alertes, Pilotage)

## Tracker

| Field | Value |
| --- | --- |
| Key | `KAN-339` |

## Subject
Page Load Performance (Pulse, Alertes, Pilotage)

## Business Case
On the pharma demo (contract MAIN-INNOPHARM, 350 devices, ~1.4 M readings), Pulse, Alertes
and Pilotage take 6 to 11 s to load and Pulse shows a black area meanwhile. This is the
customer use case; the pages must open in under a second and never block on data.

## Description
Store `EventDate` and `HasPosition` on `T_DATA` with an index, and read the latest
reading per device through it (positions, `V_PULSE`). Query alerts in two steps so the
ownership view only sees the anomaly readings. Show the page shell and the Pulse map at
once, with loaders and content-shaped skeletons. Add an in-memory client cache with
background revalidation and gzip on API responses. Measure before and after, and prove
the results are identical.

## Requirement References
Feature spec: [`IN-PROGRESS-Page-Load-Performance.md`](../features/IN-PROGRESS-Page-Load-Performance.md)
— 31 FRs, 12 NFRs, 26 ACs, 22 edge cases. This epic references requirements by ID only;
requirement text lives solely in the feature spec.

## KPIs
See the feature spec §13 (Metrics / KPIs): FR29 timings against NFR1/NFR2 budgets, and
the demo review.

## Dependencies
- Production database is MariaDB 10.11 (local is MySQL 8.0): no `LATERAL`; all SQL runs on both (NFR8).
- The production upgrade (KAN-341 script) must run before NFR3 can be recorded (KAN-347).

## Child Stories

| Key | Story | Pts | Covered Requirements | Depends on |
| --- | --- | --- | --- | --- |
| KAN-340 | Perf tooling and baseline | 3 | FR29, FR30, NFR1, NFR2, NFR5; AC9, AC11 | — (first: "before" capture) |
| KAN-341 | T_DATA EventDate index, upgrade and positions query | 8 | FR1–FR9, FR31, NFR1, NFR2, NFR5–NFR10, EC1–EC4, EC9, EC10; AC1–AC5, AC23, AC24 | KAN-340 |
| KAN-342 | Pulse view on indexed EventDate | 5 | FR10, FR11, NFR1, NFR2, NFR5, NFR8, EC5; AC6 | KAN-341 |
| KAN-343 | Alerts list two-step query | 3 | FR12–FR14, NFR1, NFR2, NFR5, NFR8, EC6–EC8; AC7, AC8 | KAN-340 |
| KAN-344 | Non-blocking loading UI | 8 | FR15–FR23, NFR4, EC15–EC18, EC21, EC22; AC12–AC19 | — |
| KAN-345 | Client cache with background revalidation | 5 | FR24–FR27, EC11–EC14, EC20; AC20, AC21 | KAN-344 (same files) |
| KAN-346 | Gzip API responses | 2 | FR28, NFR12, EC19; AC22 | — |
| KAN-347 | Verification and production timings | 2 | NFR3, NFR11; AC9, AC10, AC11, AC25, AC26 | all; production upgrade |

*Total: 36 points across 8 stories.*

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |
| FR1–FR7 | KAN-341 |
| FR8, FR9 | KAN-341 |
| FR10, FR11 | KAN-342 |
| FR12–FR14 | KAN-343 |
| FR15–FR23 | KAN-344 |
| FR24–FR27 | KAN-345 |
| FR28 | KAN-346 |
| FR29, FR30 | KAN-340 |
| FR31 | KAN-341 |
| NFR1, NFR2 | KAN-340, KAN-341, KAN-342, KAN-343 |
| NFR3 | KAN-347 |
| NFR4 | KAN-344 |
| NFR5 | KAN-340, KAN-341, KAN-342, KAN-343 |
| NFR6, NFR7 | KAN-341 |
| NFR8 | KAN-341, KAN-342, KAN-343 |
| NFR9, NFR10 | KAN-341 |
| NFR11 | KAN-347 (each story adds its own tests) |
| NFR12 | KAN-346 |
| AC1–AC5 | KAN-341 |
| AC6 | KAN-342 |
| AC7, AC8 | KAN-343 |
| AC9 | KAN-340, KAN-347 |
| AC10 | KAN-347 |
| AC11 | KAN-340, KAN-347 |
| AC12–AC19 | KAN-344 |
| AC20, AC21 | KAN-345 |
| AC22 | KAN-346 |
| AC23, AC24 | KAN-341 |
| AC25, AC26 | KAN-347 |
| EC1–EC4, EC9, EC10 | KAN-341 |
| EC5 | KAN-342 |
| EC6–EC8 | KAN-343 |
| EC11–EC14, EC20 | KAN-345 |
| EC15–EC18, EC21, EC22 | KAN-344 |
| EC19 | KAN-346 |

*Coverage: all 31 FRs, 12 NFRs, 26 ACs and 22 edge cases mapped to at least one story.
100% structural coverage.*

## Related Documentation
- Feature spec: [`IN-PROGRESS-Page-Load-Performance.md`](../features/IN-PROGRESS-Page-Load-Performance.md)

## Tracker Link
- Epic: [KAN-339](https://c-pal.atlassian.net/browse/KAN-339) — sprint S5, 2026-11-22 → 2026-12-06
