# Compact measure charts with enriched summaries

Feature spec (W1, [`FEATURE.md`](../engineering/product/FEATURE.md) template). Epic:
[KAN-??? TBD](https://c-pal.atlassian.net/browse/KAN-???).
Related: [KAN-104](https://c-pal.atlassian.net/browse/KAN-104) (Pharma demo dataset,
used as the test dataset), [KAN-34](https://c-pal.atlassian.net/browse/KAN-34)
(`T_REGLAGES` content, source of thresholds).

Stories:

| Story | Area | Covers |
|---|---|---|
| KAN-??? Enrich measure series in the dashboard API | Backend | FR1–FR9, NFR1–NFR4 |
| KAN-??? Compact measure tiles and expand modal | Front end | FR10–FR17, NFR5–NFR6 |

Screenshots attached to the ticket:

1. **Current**: dashboard with the three full-size charts (Température ambiante,
   Humidité relative, Masse).
2. **Target, compact**: the three compact tiles.
3. **Target, expanded**: the modal open on Température ambiante.

The screenshots are a design mock with simulated data. Where they and this
spec disagree, this spec wins.

Reference images are available at
`cpal.global/cpal.docs/ui/features/compact-measurement-charts`.

This feature is to be implemented on the dashboard page at
`http://localhost:5174/history`.

## 1. Feature name

Compact measure charts: small dashboard tiles showing a measure's latest value,
its key statistics and a sparkline, with an expand button that opens the full
chart in a modal. The statistics are computed in the backend by an enrichment
policy applied to each measure series after it is retrieved.

## 2. Problem

The device dashboard shows one full chart per measure (temperature, humidity,
mass), under the map. Each chart has axes, gridlines, threshold lines, a shaded
zone and a zoom slider.

As it stands:

- **Too much for a dashboard.** Each chart takes about 360 px of height. The
  user has to read the axes to answer simple questions: what is the value now,
  is it going up or down, has it been out of range?
- **No summary figures.** Last value, min, max, 24 h change and out-of-range
  count are not shown anywhere. The right-hand panel shows one temperature range
  only.
- **The front end gets every raw reading.** `api_dashboard.php` loads every
  reading in the requested range, with no row cap. Over a full contract period
  (1.4 M rows on the Pharma dataset) it exceeds PHP's 128 MB `memory_limit`
  (open point from KAN-104 delivery notes). Any summary computed in the browser
  would make this worse.

## 3. Goal

The dashboard shows each measure as a compact tile (about 150 px high) with
the latest value, its 24 h change, min / max, a sparkline and, where thresholds
exist, an out-of-range badge. One click opens the full chart in a modal. Every
figure comes precomputed from the backend, and the dashboard API stays within
memory limits over any range.

## 4. User stories

- As an operator, I want to see each measure's current value and trend at a
  glance, so that I can spot a problem without reading a chart.
- As an operator, I want to see how many readings were out of range, so that I
  know if a lot needs attention.
- As an operator, I want to open the full chart of a measure, so that I can
  inspect when and how long an excursion happened.
- As a developer, I want each statistic to be a separate, tested backend unit
  that I can enable per measure, so that adding a new statistic or a new measure
  type doesn't touch the others.

## 5. In scope

- A backend enrichment policy: an ordered list of enrichers per measure type,
  applied after the series is retrieved, producing a `summary` object.
- Enrichers for: latest value, min, max, mean, count, change over the period,
  change over 24 h, trend, threshold excursions.
- Server-side downsampling for the sparkline and the full chart.
- A dashboard API response that carries `summary` and `sparkline` per measure.
- An endpoint (or API parameter) that returns the full, downsampled series for
  one measure, called when the modal opens.
- Compact tile component and modal component, replacing the current charts in
  the dashboard.

## 6. Out of scope

- **Defining thresholds.** Thresholds come from wherever the current charts read
  them (dashed lines at 2 °C and 8 °C on the temperature chart). Their
  long-term source is `T_REGLAGES`, settled by KAN-34.
- **Anomaly detection** (`T_ANOMALIE`, `sp_anomalies_*`). The excursion count is
  a display statistic, not an anomaly record.
- **Alerting or notifications** on excursions.
- **Recomputing summaries while the map timeline plays.** The tiles follow the
  selected date range, not the playback cursor.
- **The right-hand KPI panel** (mesure(s), appareil(s), temperature range,
  lot history). It stays as is. Removing duplicates is a follow-up.
- **Changing the map.**

## 7. Flow

1. The user opens a device or contract dashboard. The front end calls the
   dashboard API with the selected range, as today.
2. The backend:
   1. Retrieves, for each measure, the aggregates it needs (SQL `MIN`, `MAX`,
      `AVG`, `COUNT`, first and last reading, value at last − 24 h) and a
      downsampled sparkline series (about 60 buckets).
   2. Runs the enrichment policy for that measure type. Each enricher adds its
      fields to `summary`.
   3. Returns `measures[]`, each with `meta`, `summary` and `sparkline`.
3. The front end renders one compact tile per measure from that response. It
   computes nothing except formatting.
4. The user clicks the expand button on a tile. The front end opens the modal
   on that measure and calls the series endpoint for the same range.
5. The backend returns the downsampled full series (up to about 1,000 buckets,
   with min / max per bucket), the thresholds and the same `summary`.
6. The modal shows the stats row and the full chart. The user can switch
   measure with the tabs (one series call per measure, cached for the session's
   range) and close the modal with the close button, the backdrop or Escape.

## 8. Functional requirements

Backend:

- **FR1.** The backend SHALL define an enricher interface. An enricher takes
  the series context (measure type, range, aggregates, sparkline, thresholds)
  and the `summary` built so far, and returns the `summary` with its own fields
  added. It SHALL NOT change fields set by another enricher.
- **FR2.** An enrichment policy SHALL map each measure type to an ordered list
  of enrichers. The policy SHALL be declared in one place (a config array or
  class), not spread across the endpoints. Adding a measure type or an enricher
  SHALL NOT need changes to the API endpoints.
- **FR3.** The following enrichers SHALL exist:

  | Enricher | Output fields | Rule |
  |---|---|---|
  | `LatestValue` | `last`, `lastAt` | Most recent reading in the range. |
  | `MinMax` | `min`, `minAt`, `max`, `maxAt` | Over the range. |
  | `MeanCount` | `mean`, `count` | Over the range. |
  | `PeriodChange` | `periodChange` | `last − first` over the range. |
  | `Change24h` | `change24h`, `change24hRef` | `last − v`, where `v` is the latest reading at or before `lastAt − 24 h`. |
  | `Trend` | `trend` | `rising`, `falling` or `stable`: sign of `change24h`, `stable` when its absolute value is under the measure's `stableBand`. |
  | `ThresholdExcursion` | `excursions` `{count, total, low, high, lastAt}` | Readings below `low` or above `high`. Applied only when the measure has thresholds. |

- **FR4.** The default policy SHALL be:

  | Measure | Enrichers | `decimals` | `stableBand` |
  |---|---|---|---|
  | Température ambiante (°C) | all seven | 1 | 0.2 |
  | Humidité relative (%) | all except `ThresholdExcursion` until thresholds exist | 1 | 1.0 |
  | Masse (kg) | all except `ThresholdExcursion` | 0 | 5 |

  Units, decimals and `stableBand` are part of the policy and sent in `meta`.
- **FR5.** Aggregates SHALL be computed in SQL (`MIN`, `MAX`, `AVG`, `COUNT`,
  and indexed lookups for first, last and the 24 h reference value). The backend
  SHALL NOT load every raw reading into PHP to compute a summary.
- **FR6.** Series SHALL be downsampled in SQL by time bucket: bucket width =
  range / target points, returning per bucket the bucket start, `AVG`, `MIN`,
  `MAX` and the count. Target points: 60 for the sparkline, 1,000 for the full
  chart. When the range holds fewer readings than the target, raw readings SHALL
  be returned.
- **FR7.** The dashboard API SHALL return, per measure:

  ```json
  {
    "key": "temperature",
    "meta": { "label": "Température ambiante", "unit": "°C", "decimals": 1, "stableBand": 0.2 },
    "summary": {
      "last": 20.0, "lastAt": "2026-11-13T05:23:09Z",
      "min": 20.0, "minAt": "…", "max": 24.0, "maxAt": "…",
      "mean": 22.0, "count": 24,
      "periodChange": -3.2,
      "change24h": -3.2, "change24hRef": "2026-11-12T05:23:09Z",
      "trend": "falling",
      "excursions": { "count": 24, "total": 24, "low": 2, "high": 8, "lastAt": "…" }
    },
    "sparkline": [["2026-11-12T06:00:00Z", 23.2], ["…", 22.4]]
  }
  ```

  Fields an enricher couldn't compute SHALL be `null`, not missing. Timestamps
  are ISO 8601 UTC. Values are raw numbers; formatting is the front end's job.
- **FR8.** A series endpoint (new file, or a parameter on `api_dashboard.php`:
  pick what fits the existing routing) SHALL return, for one measure and range:
  `meta`, `summary` (same as FR7), `thresholds` (`[{value, kind: "low"|"high"}]`)
  and `series` as `[[t, avg, min, max, count]]`.
- **FR9.** Thresholds SHALL be read from the same source the current
  temperature chart uses. If that source is hardcoded in the front end, move it
  into the backend policy and mark it with a `TODO(KAN-34)`.

Front end:

- **FR10.** Each measure SHALL render as a compact tile, laid out in a grid of
  equal columns (minimum about 280 px each, wrapping to one column on narrow
  screens). The current full-size charts SHALL be removed from the dashboard.
- **FR11.** A tile SHALL show, top to bottom:
  1. The label and unit, and an icon-only expand button (`aria-label`
     "Agrandir <label>").
  2. The latest value, large, in the accent color, with its unit.
  3. The 24 h change with a sign and a trend arrow (`Δ 24 h −3.2 °C`). When
     `change24h` is `null`, show `Δ 24 h —`.
  4. A sparkline: line and light area fill, no axes, no gridlines, no
     threshold lines, no zoom, no tooltip. Its y-axis scales to the data's own
     min/max.
  5. A footer: `min <min> · max <max>`, and, when `excursions.count > 0`, a
     badge `<count>/<total> hors <low>–<high> <unit>`.

  The mock shows the period change on the tile ("Δ période"). Use the 24 h
  change instead; the period change goes in the modal.
- **FR12.** The expand button SHALL open a modal on that measure. The modal SHALL
  show:
  1. Title `<label> (<unit>)` and the device and range as a subtitle.
  2. Tabs to switch between measures, with `aria-pressed` on the active one.
  3. A stats row: Dernière, Min, Max, Moyenne, Δ période, Δ 24 h, Mesures.
  4. The full chart: time x-axis, value y-axis with gridlines, threshold lines
     (dashed) and the out-of-range zone (shaded), the line, and the bucket
     min/max as a light band when downsampled. Tooltip on hover. The existing
     zoom slider stays here, and only here.
  5. A legend for thresholds and the out-of-range zone, when the measure has
     thresholds.
- **FR13.** The modal SHALL close with the close button, a click on the backdrop
  and the Escape key. Focus SHALL move into the modal when it opens, stay
  inside it while open, and return to the expand button that opened it.
- **FR14.** The modal chart SHALL be resized after the modal finishes opening
  (for example `chart.resize()` with ECharts), so it fills its container.
- **FR15.** The front end SHALL format numbers with `meta.decimals` and the
  `fr-FR` locale, and SHALL NOT compute any statistic.
- **FR16.** When a measure has no readings in the range, its tile SHALL show
  `—` for the value and "Aucune mesure sur cette période" in place of the
  sparkline, with the expand button disabled.
- **FR17.** While the series call is in flight, the modal SHALL show the stats
  row (already known from `summary`) and a loading state for the chart. On
  error, it SHALL show a short message and a retry button.

## 9. Non-functional requirements

- **NFR1.** The dashboard API over the full Pharma contract range
  (MAIN-INNOPHARM, 1.4 M readings) SHALL stay under the 128 MB PHP
  `memory_limit` and SHOULD respond in under 1 s on the local stack.
- **NFR2.** The series endpoint SHALL return at most about 1,000 points per
  measure, whatever the range.
- **NFR3.** Each enricher SHALL be a pure function of its input and covered by
  unit tests with fixed fixtures (empty series, single reading, no reading
  24 h before, values on a threshold, all readings out of range).
- **NFR4.** SQL SHALL run on both MySQL 8.0 (local) and MariaDB 10.11
  (production): no engine-specific syntax. Check the indexes on the
  readings table's device and timestamp columns; add one in the schema if the
  bucketing and 24 h lookup queries need it.
- **NFR5.** Accessibility: real `<button>` elements, visible focus,
  touch targets at least 44 × 44 px, text contrast at least 4.5:1 on the dark
  background, sparkline `aria-hidden`, full chart with an `aria-label`
  summarising the measure, count and range.
- **NFR6.** The tile row SHALL be no taller than about 170 px at desktop width.

## 10. Data inputs and outputs

| Input | Output |
|---|---|
| Readings for the selected devices and range (readings table) | Per measure: `meta`, `summary`, `sparkline` (60 points) in the dashboard API response |
| Enrichment policy (measure → enrichers, unit, decimals, stable band, thresholds) | `summary` fields as listed in FR3 |
| Readings + one measure + range | `series` (≤ 1,000 buckets of `t, avg, min, max, count`), `thresholds`, `summary` |

## 11. Integration points

- **`api_dashboard.php`**: response gains `measures[]`. Keep the fields the
  current front end uses until the old charts are removed in the same change.
- **Readings table**: read only. Aggregation and bucketing queries.
- **Dashboard front end**: chart components replaced by tile and modal
  components. The map, timeline and right-hand panel are untouched.
- **Pharma demo dataset** (`make seed-demo`, KAN-104): the reference dataset for
  testing volume and the injected anomalies.

### Placement decision

| Option | Verdict |
|---|---|
| Compute statistics in the browser from raw readings | Rejected: needs every reading in the browser, which is already what breaks the memory limit. |
| Compute in a SQL view or stored procedure | Rejected: harder to test, engine differences between MySQL and MariaDB, and logic spread between SQL and PHP. |
| SQL aggregates + PHP enrichment policy | **Chosen.** SQL does the heavy reading (aggregates, buckets); each statistic is a small, testable PHP enricher, enabled per measure in one policy. |

## 11a. Software items

Not applicable (see [SDLC, product templates](../sdlc/README.md#product-templates)).

## 12. Edge cases and constraints

- **The range is shorter than 24 h, or there is no reading 24 h before the
  last one.** `change24h`, `change24hRef` and `trend` are `null`.
- **The demo dataset runs to 2029.** The 24 h change is measured from the last
  reading in the range, not from the current time.
- **One reading only.** `min = max = last`, `periodChange = 0`, the sparkline is
  a single point (draw a flat line).
- **Gaps in the data.** Empty buckets are omitted, not zero-filled. The full
  chart breaks the line across gaps longer than three bucket widths.
- **A value exactly on a threshold** is in range (`low ≤ v ≤ high`).
- **Several devices selected.** Keep the current charts' behaviour (one series
  per device or one combined series: check what exists). The policy runs per
  series.
- **Timezones.** Everything is computed and sent in UTC; the front end displays
  in the same zone as the current timeline label.
- **Range changes while the modal is open.** Refetch the series and update the
  summary.

## 13. Metrics

- Dashboard API peak memory over the full MAIN-INNOPHARM range, before and
  after.
- Dashboard API response size and time over the default range, before and
  after.
- Height of the measures section at 1440 px width: about 360 px before, at most
  170 px after.

## 14. Risks and mitigations

| Risk | Mitigation |
|---|---|
| Downsampling hides a short spike | Buckets carry `min` and `max`; the full chart draws them as a band and the excursion count is computed on raw readings in SQL, not on buckets. |
| Thresholds are hardcoded or missing until KAN-34 | Read them in one place in the policy; the excursion enricher is skipped when a measure has none. |
| Enrichers depend on each other's output | FR1 forbids overwriting; order is explicit in the policy; each enricher declares the aggregates it needs. |
| Bucketing queries are slow on 1.4 M rows | Index on device + timestamp (NFR4); measure on the Pharma dataset before merging. |

## 15. Acceptance criteria

- [ ] Enricher interface, policy and the seven enrichers exist, each with unit tests (FR1–FR4, NFR3)
- [ ] Summaries and buckets computed in SQL; no full raw load in PHP (FR5, FR6)
- [ ] Dashboard API returns `measures[]` with `meta`, `summary`, `sparkline` (FR7)
- [ ] Series endpoint returns ≤ 1,000 buckets, thresholds and summary (FR8, FR9, NFR2)
- [ ] Compact tiles replace the full charts, matching screenshot 2 except for FR11's 24 h change (FR10, FR11, FR15, FR16)
- [ ] Modal opens from each tile, switches measure, closes with button, backdrop and Escape, handles focus (FR12–FR14, FR17)
- [ ] Full MAIN-INNOPHARM range loads under the memory limit and in under 1 s locally (NFR1)
- [ ] Works on MySQL 8.0 locally; no MariaDB-incompatible SQL (NFR4)
- [ ] Accessibility checks pass (NFR5)

## 15a. Delivery notes

To be filled on delivery.

## 16. Future extensions

- Excursion duration (time spent out of range), not just a reading count.
- Thresholds per contract or per lot, from `T_REGLAGES` (KAN-34).
- Same tiles for other measures as devices add them, by adding a policy entry.
- Drop the right-hand temperature range from the KPI panel, now redundant with
  the temperature tile.
- Linking an excursion badge to the matching anomaly once anomaly detection runs
  locally.