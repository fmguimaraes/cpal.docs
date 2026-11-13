# Epic Template

> **Single source of truth:** requirement **text** lives in exactly one place, the
> feature spec. The epic doc and the tracker description reference requirements
> **by ID only**.

## Tracker

| Field | Value |
| --- | --- |
| Key | `KAN-250` — created via `scripts/jira.py` (the connector in this session was
authenticated to `quietstage.atlassian.net`, not `c-pal.atlassian.net`, so Jira
issues were created with a direct REST client instead). |

## Subject
Compact measure charts with enriched summaries

## Business Case
The History dashboard (`/history`) sends every raw reading to the browser and
renders three full-size Plotly charts with no summary figures. On long ranges
this risks the same memory/payload problem already hit on the device dashboard
(KAN-104 delivery notes), and gives the operator no at-a-glance read of current
value, trend or out-of-range time.

## Description
Add a backend enrichment policy that computes per-measure summary statistics
(latest value, min/max, mean, 24 h change, trend, threshold excursions) and a
downsampled sparkline, server-side, in the History API
(`cpaltracker.web/api/v1/contracts/history.php` →
`history_logic.php::cpal_history_mesures()`). Replace the three full-size
Plotly charts in `cpaltracker-react`'s `HistoryCharts.tsx` (the `/history`
route) with compact tiles fed by that summary, each with an expand button that
opens the existing full chart in a modal.

## Requirement References
Feature spec: [`Compact-measurement-charts.md`](../features/Compact-measurement-charts.md)
— FR1–FR17, NFR1–NFR6. This epic references requirements by ID only;
requirement text lives solely in the feature spec.

Integration note (supersedes the feature spec's references to the legacy
`api_dashboard.php`/`dashboard.php` device dashboard): the `/history` route
this epic targets is served by `cpaltracker-react`'s `HistoryCharts.tsx`,
fed by `cpaltracker.web`'s `GET /api/v1/contracts/history.php/{contractId}`
(`history_logic.php::cpal_history_mesures()`), not by `api_dashboard.php`.
The enrichment policy, enrichers and summary/sparkline shapes (FR1–FR9) apply
there instead.

## KPIs
- History API peak memory and response time over a long contract range,
  before and after.
- History API response size over the default range, before and after.
- Height of the measures section on `/history` at 1440 px width: about 360 px
  before, at most 170 px after (NFR6).

## Dependencies
- Thresholds source (`T_REGLAGES`, KAN-34) — until settled, thresholds are read
  from wherever `HistoryCharts.tsx`'s temperature band currently lives
  (`TEMPERATURE_THRESHOLD` in `HistoryCharts.tsx`), per FR9.
- `cpaltracker-react`'s existing `TimeSeriesChart`/`WidgetCard` kit components
  (KAN-175, KAN-223) for the modal's full chart.

## Child Stories

| Key | Story | Pts | Covered Requirements | Software Items |
| --- | --- | --- | --- | --- |
| KAN-251 | Enrich measure series in the History API | — | FR1–FR9, NFR1–NFR4 | — |
| KAN-252 | Compact measure tiles and expand modal on /history | — | FR10–FR17, NFR5–NFR6 | — |

*Total: 2 stories (points TBD on real Jira creation).*

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |
| FR1 | KAN-251 |
| FR2 | KAN-251 |
| FR3 | KAN-251 |
| FR4 | KAN-251 |
| FR5 | KAN-251 |
| FR6 | KAN-251 |
| FR7 | KAN-251 |
| FR8 | KAN-251 |
| FR9 | KAN-251 |
| FR10 | KAN-252 |
| FR11 | KAN-252 |
| FR12 | KAN-252 |
| FR13 | KAN-252 |
| FR14 | KAN-252 |
| FR15 | KAN-252 |
| FR16 | KAN-252 |
| FR17 | KAN-252 |
| NFR1 | KAN-251 |
| NFR2 | KAN-251 |
| NFR3 | KAN-251 |
| NFR4 | KAN-251 |
| NFR5 | KAN-252 |
| NFR6 | KAN-252 |

Coverage: all 17 FRs and 6 NFRs mapped to at least one story. 100% structural
coverage.

## Related Documentation
- Feature spec: [`Compact-measurement-charts.md`](../features/Compact-measurement-charts.md)

## Tracker Link
- Epic: KAN-250
