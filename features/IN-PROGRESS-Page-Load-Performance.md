# Feature: Page Load Performance (Pulse, Alerts, Pilotage)

> Feature spec (W1, [`FEATURE.md`](../engineering/product/FEATURE.md) template) for making the
> data pages of `cpaltracker-react` fast and non-blocking on the real customer volume (the Pharma
> demo dataset). Mostly technical. The `FEATURE.md` template is used, not `TECH-FEATURE.md`,
> because this is new behavior to build. The current-behavior facts it relies on are cited
> inline with file and line.
>
> Sources:
> - Owner ask of 2026-11-26 (Felipe): Pulse, Alerts and Pilotage take 6–11 s to load on the
>   Pharma demo, and Pulse shows a black area while it loads. Make loading fast, and make it
>   non-blocking: on Pulse the map shows at once with a loader over it; the other pages show
>   skeletons instead of blocking the view.
> - Measurements and root-cause analysis of 2026-11-26 on local MySQL 8.0.46, Pharma dataset
>   ([`pharma-demo-dataset.md`](pharma-demo-dataset.md)): contract 4 `MAIN-INNOPHARM` (350
>   devices, ~1.4 M readings in `T_DATA`) and its STD sub-contract 5 `STD-CUST01`.
>
> Sibling specs, cited by ID and never restated: [`IN-PROGRESS-Pulse.md`](IN-PROGRESS-Pulse.md)
> (Pulse FRn), [`IN-PROGRESS-Anomaly-Workflow.md`](IN-PROGRESS-Anomaly-Workflow.md) (AW-FRn),
> [`React-Device-Pilotage.md`](React-Device-Pilotage.md),
> [`React-Live-Tracking.md`](React-Live-Tracking.md) (LT-FRn).
>
> Single source of truth: requirement text (FR/NFR/AC/EC IDs, KPIs) lives only here.

---

## 1. Feature Name

**Page Load Performance**: indexed event date and position flag on `T_DATA`, rewritten
read queries for positions, the Pulse snapshot and the alerts list, a short client-side
cache, gzip on API responses, and loading states that never block the page (a map with a
loader on Pulse, skeletons elsewhere).

## 2. Problem / Pain Point

The Pharma demo is the real customer use case. On it, three pages of the React app wait
6 to 11 seconds for their first useful content:

| Page | Request | Server logic | Rows | Measured |
|---|---|---|---|---|
| Pilotage (and Suivi en direct) | `GET api/v1/devices/positions.php` | `cpal_tracking_positions_courantes` (`cpaltracker.web/tracking_logic.php:102-147`) | 350 | 11.2 s |
| Alerts (and the Pulse side panel) | `GET api/v1/anomalies/list.php` | `cpal_anomalies_lister` over `V_ANOMALIES_STATUT` (`cpaltracker.web/anomalies_logic.php:109-118`) | 13 | 6.7 s |
| Pulse | `GET api/v1/contracts/pulse.php/{id}` | `SELECT * FROM V_PULSE` (`cpaltracker.web/pulse_logic.php:22`; view in `docker/mysql/init/03-dashboard.sql:240-319`) | 350 | 6.3 s |

Pulse calls both the snapshot and the alerts list, so it pays for both.

Root causes (verified):

1. **The event date is recomputed on every reading.** `EventDate` is the `FN_EVENT_DATE` rule
   (`01-schema.sql:1253`): `RECORD_TIME` when it is on or after 2020-01-01 and no later than
   `timestamp_utc` + 1 day, else `timestamp_utc`. The positions query
   (`tracking_logic.php:129-133`), `V_PULSE` (`main_lues`, `lectures`) and `V_DATA_WITH_CLIENT`
   (`01-schema.sql:1547`) all compute it at read time, then take a max or a `ROW_NUMBER` per
   device. No index can serve an expression, so every request scans and sorts every reading of
   the contract. `T_DATA` has only `PRIMARY(DATA_ID)`, `(ChipID, timestamp_utc)` and
   `(RECORD_TIME)` (`01-schema.sql:356-358`).
2. **Alerts compute ownership for all readings to keep 13.** `V_ANOMALIES_STATUT` →
   `V_ANOMALIES_CLIENT` → `V_DATA_WITH_CLIENT`, a `UNION ALL` of a MAIN and an STD branch whose
   STD branch runs a correlated `EXISTS`/`MAX` per reading. MySQL does not push the anomaly
   `DATA_ID` join into the union, so it evaluates ownership on ~1.4 M readings before keeping
   13, and `LIMIT 200` applies afterwards. `T_ANOMALIE` holds 18 rows. The same view filtered
   with a literal `DATA_ID IN (…18 ids…)` answers in 1 ms.
3. **No caching** (secondary). The client refetches on every mount (`src/anomalies/useLoad.ts`,
   `ContractPositions`, `loadPilotageWorkspace`). `cpal_api_emit` (`api/v1/_lib.php:123`) sends
   no `Content-Encoding`.
4. **Loading blocks the view.** Pulse replaces the whole map region with one `Skeleton` until
   positions arrive (`src/live-tracking/ContractPositions.tsx:268-275`, `h-96 md:h-176`); the
   map is not mounted at all, which reads as a black area in the dark theme. Pilotage shows three
   bars until positions and the first device's config have both loaded
   (`src/routes/DevicePilotage.tsx:171-178`, `src/pilotage/loadPilotageWorkspace.ts:38,54`).
   Alerts already shows its shell, filters and a list skeleton (`src/routes/Alerts.tsx:45-55`).
   Its wait is the server time.

The user waits, can't tell whether the page is working, and the demo looks broken in front of
the customer.

## 3. Goal / Outcome (Success Definition)

On the Pharma demo, for contracts 4 and 5:

1. Pulse, Alerts and Pilotage show their content in well under a second once warm.
2. From the first frame, the page shell, header and filters are visible, and on Pulse the map
   with its tiles is visible too, with a loader over it while positions arrive. Nothing is
   black or blank.
3. Going back to a page already visited shows its last data at once and refreshes it quietly.
4. Every page shows exactly the same devices, positions, lots and alerts as before.

## 4. User Stories (Primary Scenarios)

- **US1. Open Pulse during a demo.** As a planner, I want the map to appear at once and the
  pallets to show up on it within a second, so that the product feels live and I can start
  talking about the fleet immediately. *(FR8–FR11, FR15–FR18, NFR1, NFR4)*
- **US2. Check alerts.** As a quality manager, I want the alerts list to load almost
  instantly, so that triage is not slowed by the tool. *(FR12–FR14, FR20, NFR1)*
- **US3. Open Pilotage.** As an operator, I want the device list to appear quickly, with a
  table-shaped placeholder while it loads, so that I know the page is working.
  *(FR8, FR19, NFR1, NFR4)*
- **US4. Move between pages.** As any user, I want a page I just left to show its data at
  once when I come back, refreshed in the background, so that navigation is fluid.
  *(FR24–FR27)*
- **US5. Nothing changes but speed.** As the product owner, I want every page to show exactly
  the same results as before, so that speed is not bought with wrong data.
  *(NFR5, NFR6)*

## 5. In Scope (What WILL be delivered)

- Two STORED generated columns on `T_DATA` (`EventDate`, `HasPosition`) and one index on them
  (FR1–FR3). The init SQL is updated, and an idempotent upgrade script is added for existing
  databases (local Pharma DB, production) (FR4, FR5, FR7).
- Every query that would collide with the new column updated in the same change (FR6).
- Rewritten positions query (FR8, FR9), `V_PULSE` (FR10, FR11) and alerts list (FR12–FR14),
  same results.
- Non-blocking loading. Pulse mounts the map at once with a loader overlay. Pilotage, Alerts,
  the Pulse side panel and the other data pages use content-shaped skeletons. Error and empty
  states keep working (FR15–FR23).
- Short in-memory client cache with stale-while-revalidate for the three reads (FR24–FR27).
- Gzip of JSON responses in `cpal_api_emit` (FR28).
- A repeatable timing script and a before/after equivalence script (FR29, FR30).
- Setup and deployment docs updated (FR31).

## 6. Out of Scope (What WILL NOT be delivered)

- **Redis or any server-side cache.** Not needed once the queries take milliseconds.
- **HTTP caching** (`Cache-Control: max-age`, `ETag`, `304`). The client-side cache (FR24)
  covers navigation; authenticated JSON stays uncached by intermediaries.
- **Fetching only open alert statuses by default.** The status filter shows a count on every
  tab (Ouvertes, Clôturées, Toutes) computed from one fetch of all four statuses
  (`src/routes/Alerts.tsx:24-25,69,102`). Fetching open statuses only would blank or falsify
  those counts. After FR12 the cost of all statuses is the same as one, so the
  current fetch is kept.
- **`V_ANOMALIES_OUVERTES`** (the alert-light aggregate). No API endpoint reads it today (grep of
  `cpaltracker.web/api/`), so it is not on a page-load path. It is left unchanged and listed as
  a follow-up in §16.
- **Playback history** (`history_logic.php` via `V_DATA_WITH_CLIENT`, up to 25,000 rows) and
  the legacy PHP pages (`index.php` CSV, `dashboard.php`). They are follow-ups (§16), but their
  results must not change (NFR5).
- **Trends** (`contracts/trends.php`, `sp_lot_conso_refresh_at`), which is not reported slow. It
  may adopt the stored column later (§16).
- **Changing any API response shape** or the OpenAPI contract. Only `Content-Encoding` and
  `Vary` headers are added (FR28).
- **Changing `FN_EVENT_DATE`** or the anomaly trigger `trg_data_anomalies_ai`. They keep their
  own computation, which must agree with the stored column (NFR6).
- **Product analytics.** c-PAL has no analytics or funnel instrumentation (same as
  `IN-PROGRESS-Pulse.md` OQ10).

## 7. User Flow / UX Breakdown

**Pulse**
1. User clicks Pulse. The shell, sidebar and Pulse header controls render. The map mounts at
   once with its base tiles at the kit's default viewport. A loader card (spinner and
   "Chargement des positions…") sits over the map (FR15). The side panel shows list-shaped
   skeletons (FR17).
2. Positions arrive. The loader goes away, markers are drawn and the map fits them once, as
   today (Pulse FR13) (FR16). The side panel fills when the alerts list arrives.
3. If positions fail, the map stays visible. The loader is replaced by the existing error text
   with a "Réessayer" button (FR22).
4. User goes to Alerts, then back to Pulse. The previous positions and alerts show at once and
   are revalidated in the background (FR24). If the data changed, the view updates in place
   without a skeleton (EC13).

**Alerts**
1. User opens Alerts. Breadcrumb, status filter (with count placeholders) and list skeleton
   show at once (FR20).
2. The list arrives in under the NFR1 budget. Counts, type chips and rows replace the
   placeholders.

**Pilotage**
1. User opens Pilotage. Breadcrumb and page shell show at once. A skeleton shaped like the device
   table (header row and rows, or cards below md) takes the list's place (FR19).
2. Positions and the eligibility check complete. The table, or the existing ineligible or empty
   state, replaces the skeleton.

## 8. Functional Requirements (FRs)

### Data model and upgrade

- **FR1.** `T_DATA` SHALL have a STORED generated column `EventDate` (`datetime`) equal to the
  `FN_EVENT_DATE(RECORD_TIME, timestamp_utc)` rule, written inline as
  `CASE WHEN RECORD_TIME >= '2020-01-01 00:00:00' AND RECORD_TIME <= timestamp_utc + INTERVAL 1 DAY
  THEN RECORD_TIME ELSE timestamp_utc END`. A generated column can't call a stored function.
- **FR2.** `T_DATA` SHALL have a STORED generated column `HasPosition` (`tinyint(1)`), 1 when
  `LAT` and `LONGITUDE` are not null and `ABS(LAT) >= 0.0001 OR ABS(LONGITUDE) >= 0.0001`,
  else 0. This is the same rule as `V_DATA_WITH_CLIENT.HasPosition` and `cpal_anomalies_position`.
- **FR3.** `T_DATA` SHALL have the index `(ChipID, HasPosition, EventDate, DATA_ID)`.
- **FR4.** A fresh database built from `cpaltracker.web/docker/mysql/init/*.sql` SHALL contain
  FR1–FR3: the `T_DATA` definition in `01-schema.sql` carries them. Loading the Pharma set
  afterwards (`make seed-demo`, `LOAD DATA` with an explicit column list) fills them with no
  loader change.
- **FR5.** An upgrade script SHALL bring an existing database (the local Pharma DB, production)
  to the same state as FR4, including every view or query object this feature rewrites
  (FR10). No migrations tool exists (`docs/deployment.md:94`), so it lives in a new folder
  `cpaltracker.web/docker/mysql/upgrade/`. It guards each `ALTER` on `information_schema`, as
  `tmp_dashboard_add_column` does (`03-dashboard.sql:18-48`), and uses `CREATE OR REPLACE` for
  views. It is applied by hand, like the other production schema changes.
- **FR6.** Every statement that reads `T_DATA` with `*` and also computes a column named
  `EventDate` or `HasPosition` SHALL be changed in the same release as FR1/FR2, so that none
  fails with a duplicate-column error. The known site is `tracking_logic.php:129-133`
  (`SELECT d.*, CASE … AS EventDate`), which is replaced by FR8. The story re-runs the grep
  over `cpaltracker.web` (PHP and SQL) and records the result. Explicit column lists
  (`V_DATA_WITH_CLIENT`, `V_PULSE`, `sp_lot_conso_refresh_at`) and `SELECT *` from views are
  not collisions.
- **FR7.** The upgrade script SHALL document its rollback: drop the index, drop the two
  columns, restore the previous definitions of the objects FR10 replaced. The rollback is
  tested once locally.

### Positions (Pilotage, Suivi en direct)

- **FR8.** `cpal_tracking_positions_courantes` SHALL return, for each device of the given
  contracts in `T_CONTRATS_DEVICES`, its reading with `HasPosition = 1` and the greatest
  `(EventDate, DATA_ID)`, found through the FR3 index (one index descent per device, no scan of
  the device's history). Rows, the values of `ContratID` and `EventDate`, order
  (`ContratID, ChipID`) and the API mapping (FR9) SHALL be those of the current function.
  *Amended in KAN-341:* the raw row now also carries `HasPosition` (always 1) and no longer the
  `rn` window column; no consumer reads either.
- **FR9.** The positions response (`cpal_tracking_position_vers_api`) SHALL be unchanged.

### Pulse snapshot

- **FR10.** `V_PULSE` SHALL be redefined so that `derniers` and `lectures` use the stored
  `EventDate` and `HasPosition` through the FR3 index. The rest SHALL stay the same: MAIN rule
  (reading inside the contract dates, contract active now), STD rule (reading inside one of the
  STD client's transfer windows with no later-starting window covering it), tie-break on the
  highest `DATA_ID`, lot join (`EventDate` inside the hook window, lot shown to its owner
  only), and the view's columns and their order.
- **FR11.** The 03-dashboard.sql comment on `V_PULSE` SHALL record the new timings in place of
  the 2026-11-19 ones.

### Alerts

- **FR12.** `cpal_anomalies_lister` SHALL run in two steps:
  (1) select candidate `DATA_ID`s from `T_ANOMALIE` for the devices of the contract's MAIN
  contract (the contract itself when MAIN, its `ParentContratID` when STD). The candidate set
  may include extra rows but must never miss one the final query would return;
  (2) run the joins of `V_ANOMALIES_STATUT` in application SQL, with `ContratID = ?` and
  `DATA_ID IN (…)` (candidates as bound parameters) written on `V_DATA_WITH_CLIENT`. The
  MAIN/STD visibility and owner-at-date rules stay in `V_DATA_WITH_CLIENT`; status, lot
  disclosure, ordering and `LIMIT` (`CPAL_ANOMALIES_LIMITE`) are the same as the view's.
  *Amended in KAN-343:* filtering `V_ANOMALIES_STATUT` itself by `DATA_ID IN (…)` measured
  5–9 s on contract 4, because the predicate cannot reach the `V_DATA_WITH_CLIENT` derived
  table. The view and `cpal_anomalies_from_sql` must be kept in sync; FR30's perf-compare is
  the equivalence check after any change to either.
- **FR13.** When there are more candidates than a constant `CPAL_ANOMALIES_CANDIDATS_MAX`
  (proposal: 1,000), step 2 SHALL run per chunk. The rows are merged, sorted by the current
  order (`EventDate DESC, DetectedAt DESC, DATA_ID DESC, TRAITEMENT_ID DESC`) and cut to the
  limit in PHP. The output is the same as a single query.
- **FR14.** `cpal_anomalies_detail` (one `DATA_ID`) SHALL be covered by the timing script
  (FR29). If it exceeds the NFR1 budget, it gets the same candidate restriction. Otherwise it
  is left unchanged.

### Non-blocking loading

- **FR15.** On Pulse, the map (kit `FleetMap`, base tiles) SHALL mount as soon as the route
  renders, before the contract list and positions are loaded. While positions load, a loader
  overlay SHALL sit over the map: kit `Spinner` and the text "Chargement des positions…",
  `role="status"`, in the style of the map's existing notice card (`FleetMap.tsx:603`). No
  `Skeleton` replaces the map region, so the `ContractPositions.tsx:268-275` branch no longer
  applies to Pulse.
- **FR16.** When positions arrive, the loader SHALL disappear and the markers and the one-time
  auto-fit SHALL behave as Pulse FR9/FR13 do today. Before that, the map shows the kit default
  viewport.
- **FR17.** The Pulse side panel's anomalies list (`src/anomalies/ContractAnomalies.tsx:161`,
  one `h-48` block today) and the pallet panel SHALL show skeletons shaped like their rows
  while loading.
- **FR18.** Suivi en direct (`/live-tracking`) uses the same `ContractPositions`, so it SHALL
  get the FR15/FR16 behavior.
- **FR19.** On Pilotage, the breadcrumb and page shell SHALL render at once. While the
  workspace loads, a skeleton shaped like the device list SHALL replace the three bars of
  `DevicePilotage.tsx:171-178`: the `DataTable` header and rows at md and up, `MobileList`
  cards below md.
- **FR20.** On Alerts (and the device alerts page, which shares `AlertsList`), the breadcrumb,
  status filter and list skeleton SHALL keep rendering at once. The status-count slots SHALL
  show placeholders while loading instead of being absent, so the filter doesn't shift when
  counts arrive.
- **FR21.** Every other data page SHALL follow the same rule: loading is a `Skeleton` shaped
  like the content, and a spinner is never used for a whole page
  (`engineering/ui-guidelines.md` FBK-LOADING and PAT-STATES). The known gap is the History
  page's contract loading text (`src/routes/HistoryDashboard.tsx:100-103`). The story
  re-audits the routes in `App.tsx` and records the result.
- **FR22.** Error and empty states SHALL keep their current texts and test IDs. On Pulse
  and Suivi en direct a positions error keeps the map visible and shows the
  `positions-error` alert, with a "Réessayer" button that reloads, over or above the map.
  Pulse's empty notice (`keepMapWhenEmpty`) is unchanged.
- **FR23.** Each loader and skeleton SHALL carry `role="status"` (or `aria-busy` on its
  region) and French `sr-only` text, and SHALL be removed from the accessibility tree once
  content is shown.

### Client cache and transport

- **FR24.** The React app SHALL keep an in-memory cache of the last successful response of
  `fetchPulse`, `fetchPositions` and `fetchAnomalies`, keyed by function, contract and
  parameters, implemented once and used by Pulse, Suivi en direct, Pilotage, Alerts and the
  Pulse panels. On mount, a cached value younger than its maximum age (proposal: 5 min) SHALL
  be shown at once and revalidated in the background (stale-while-revalidate). An older value
  SHALL NOT be shown; the page loads normally.
- **FR25.** The cache SHALL be invalidated: for the contract's anomalies after
  `postAnomalyTransition` or `postAnomalyComment` succeeds; for the contract's positions after
  `postDeviceTransfer` or `postDeviceConfig` succeeds; and entirely when the session ends
  (logout or session loss, i.e. `setAuthToken(null)`). Pulse's Refresh (Pulse FR12) SHALL
  bypass it.
- **FR26.** The cache SHALL live in memory only: no `localStorage`, `sessionStorage` or
  IndexedDB.
- **FR27.** A failed background revalidation SHALL keep the displayed data, show no full-page
  error state, and report through the existing freshness indicator where there is one (Pulse
  freshness chip).
- **FR28.** `cpal_api_emit` SHALL gzip the JSON body when the request's `Accept-Encoding`
  accepts `gzip` and the body is at least 1 KiB, sending `Content-Encoding: gzip` and
  `Vary: Accept-Encoding`. Otherwise the response is byte-identical to today. It SHALL not
  compress when `zlib.output_compression` is on or a `Content-Encoding` header is already set.

### Tooling and docs

- **FR29.** A timing script SHALL be added in `cpaltracker.web/tools/perf/`, runnable with one
  `make` target. For contracts 1, 4 and 5 it times positions, the Pulse snapshot, the alerts
  list and one alert detail at two levels: the logic function over PDO (one warm-up, then 5
  runs, reporting median and max) and the HTTP endpoint (`curl` total time, with and without
  gzip). It prints a table that can be pasted into the story.
- **FR30.** An equivalence script SHALL be added in `cpaltracker.web/tools/perf/`. It captures,
  for contracts 1, 4 and 5, the decoded JSON of `positions.php`, `pulse.php` (without
  `generatedAt`), `list.php` with all four statuses, and `contracts/history.php` on one fixed
  range of contract 4. It also compares two captures and exits non-zero on any difference.
  The "before" capture is taken on the current schema before any FR1–FR14 change is merged.
- **FR31.** `cpaltracker.web/docs/local-setup.md` (init script table) and `docs/deployment.md`
  (upgrade procedure: when to run, write lock during the rebuild, expected duration, rollback)
  SHALL describe the upgrade script and the perf scripts.

## 9. Non-Functional Requirements (NFRs)

- **NFR1. Warm server time.** Measured with FR29, logic-function median on local docker
  MySQL 8.0 with the Pharma dataset and the default buffer pool, for contracts 1, 4 and 5:
  positions ≤ 300 ms, Pulse snapshot ≤ 300 ms, alerts list ≤ 100 ms, alert detail ≤ 100 ms.
  Endpoint total time (HTTP, local) ≤ 500 ms each.
- **NFR2. Cold server time.** First request after a database restart (empty buffer pool), same
  queries: ≤ 2 s each. Recorded by FR29 with a restart flag.
- **NFR3. Production check.** The same FR29 numbers SHALL be recorded once on the production
  MariaDB 10.11 host (or a MariaDB 10.11 copy of it) after the upgrade. The release is
  accepted when NFR1's endpoint budget holds there for the largest production contract.
- **NFR4. Paint independent of data.** With every data API response delayed by 5 s
  (Playwright route interception), these SHALL be visible within 1.5 s of navigation on the
  local build: the shell (sidebar, breadcrumb or header, filters) on Pulse, Suivi en direct,
  Alerts and Pilotage; and on Pulse and Suivi en direct, the Leaflet map container with its
  tile layer (tile requests issued) and the loader.
- **NFR5. Same results.** The FR30 comparison of before and after captures SHALL report zero
  differences for contracts 1, 4 and 5.
- **NFR6. Column correctness.** On the full Pharma `T_DATA`, zero rows where `EventDate` differs
  from `FN_EVENT_DATE(RECORD_TIME, timestamp_utc)`, or `HasPosition` differs from the
  `V_DATA_WITH_CLIENT` rule (null-safe comparison).
- **NFR7. Idempotent upgrade.** Running the upgrade script twice SHALL succeed, and the second
  run changes nothing. `SHOW CREATE TABLE T_DATA` and `SHOW CREATE VIEW` of every replaced view
  SHALL be identical between an upgraded database and a fresh FR4 database.
- **NFR8. Portability.** All new or changed SQL SHALL run on MySQL 8.0 (local) and MariaDB
  10.11 (production, dump `cPal/db/cpaltracker v0611226.sql`). That means no `LATERAL` (MariaDB
  10.11 lacks it; use a correlated scalar subquery with `ORDER BY … LIMIT 1` or an equivalent),
  no stored function in a generated column, and no MySQL-only syntax. The upgrade script and
  the three queries are run once on a MariaDB 10.11 container.
- **NFR9. Ingestion unaffected.** `insert_data.php` and `trg_data_anomalies_ai` keep inserting
  readings and anomalies. Inserting 1,000 readings takes at most 20% longer than before
  (measured once).
- **NFR10. Upgrade duration.** The upgrade on the ~1.46 M-row `T_DATA` SHALL finish in ≤ 2 min
  locally. The measured time is recorded in `docs/deployment.md` with the note that writes to
  `T_DATA` wait during the rebuild.
- **NFR11. Tests.** Existing PHPUnit, Vitest and Playwright suites pass. New tests cover:
  FR8/FR12/FR13 (PHPUnit: candidate step, empty candidate set, chunking order and limit);
  FR28 (PHPUnit: compressed vs not, by header and size); FR24–FR27 (Vitest: hit, expiry,
  revalidation, invalidation, logout clear, error not cached); FR15–FR23 (Vitest states and
  the NFR4 Playwright test).
- **NFR12. Payload.** With gzip, the transferred size of the contract 4 `pulse.php` and
  `positions.php` responses SHALL be at least 70% smaller than uncompressed.

## 10. Data Inputs & Outputs

- **`T_DATA`** gains `EventDate datetime GENERATED ALWAYS AS (…) STORED` (after `RECORD_TIME`,
  which it reads; a generated column may read an earlier one, as `DIFF_TTN_RECORD_SEC` already
  does in production), `HasPosition tinyint(1) GENERATED ALWAYS AS (…) STORED`, and index
  `(ChipID, HasPosition, EventDate, DATA_ID)`. No writer changes: every `INSERT INTO T_DATA`
  and the Pharma `LOAD DATA` use explicit column lists (`insert_data.php:99`, `02-seed.sql:43`,
  `05-anomalies-demo-contract1.sql`, `06-history-demo-devices.sql`,
  `tools/demo-data/pharma/sql/charger_T_DATA.sql:10-14`).
- **`V_PULSE`**: same columns, same rows (FR10).
- **API responses** of `positions.php`, `pulse.php`, `anomalies/list.php`, `anomalies/detail.php`:
  same JSON. Only `Content-Encoding: gzip` and `Vary: Accept-Encoding` may be added (FR28).
- **Client cache**: in-memory map of key → `{ value, fetchedAt }` (FR24).
- **Perf outputs**: timing table (FR29), JSON captures and a diff report (FR30), written to a
  gitignored folder.

## 11. Integration Points

| Component | Where | Change |
|---|---|---|
| `T_DATA` schema | `cpaltracker.web/docker/mysql/init/01-schema.sql:335-360` | FR1–FR4 |
| Upgrade script (new) | `cpaltracker.web/docker/mysql/upgrade/` | FR5, FR7 |
| Positions query | `cpaltracker.web/tracking_logic.php:102-147` (behind `api/v1/devices/positions.php`) | FR6, FR8, FR9 |
| `V_PULSE` | `cpaltracker.web/docker/mysql/init/03-dashboard.sql:233-319` (read by `pulse_logic.php:22`) | FR10, FR11 |
| Alerts list and detail | `cpaltracker.web/anomalies_logic.php:109-140, 205-220` (behind `api/v1/anomalies/list.php`, `detail.php`) | FR12–FR14 |
| `V_ANOMALIES_STATUT`, `V_ANOMALIES_CLIENT`, `V_DATA_WITH_CLIENT` | `03-dashboard.sql:194-218`, `01-schema.sql:1511,1547` | Read only, unchanged (visibility stays in the views) |
| `cpal_api_emit` | `cpaltracker.web/api/v1/_lib.php:123-138` | FR28 |
| Perf scripts (new) | `cpaltracker.web/tools/perf/`, `Makefile` | FR29, FR30 |
| Docs | `cpaltracker.web/docs/local-setup.md`, `docs/deployment.md` | FR31 |
| `ContractPositions` | `cpaltracker-react/src/live-tracking/ContractPositions.tsx` | FR15, FR16, FR18, FR22, FR24 |
| Pulse page and panels | `src/routes/Pulse.tsx`, `src/anomalies/ContractAnomalies.tsx`, `src/pulse/PalletPanel.tsx` | FR15–FR17, FR24 |
| Pilotage | `src/routes/DevicePilotage.tsx`, `src/pilotage/loadPilotageWorkspace.ts` | FR19, FR24 |
| Alerts | `src/routes/Alerts.tsx`, `src/anomalies/useLoad.ts` | FR20, FR24 |
| History page | `src/routes/HistoryDashboard.tsx:100-103` | FR21 |
| API clients | `src/api/pulse.ts`, `positions.ts`, `anomalies.ts`, `transfer.ts`, `deviceConfig.ts`, `client.ts` | FR24–FR26 |
| UI kit (reused, not changed) | `src/ui/skeleton.tsx`, `src/ui/spinner.tsx`, `src/ui/map/FleetMap.tsx`, `src/ui/data-table.tsx`, `src/ui/mobile-list.tsx`; rules in `cpal.docs/engineering/ui-guidelines.md` (FBK-LOADING, PAT-STATES) | FR15–FR23 |
| Production DB | MariaDB 10.11, applied by hand | FR5, NFR3, NFR8, NFR10 |

## 11a. Software Items (IEC 62304 §5.3)

**N/A.** c-PAL is not in a regulated software scope. Per
[`sdlc/README.md`](../sdlc/README.md) the Software Items sections stay empty, and no Software
Item Registry exists.

## 12. Edge Cases & Constraints

- **EC1.** Device of the contract with no positioned reading: no row in positions or Pulse,
  as today.
- **EC2.** Several readings share the latest `EventDate`: the highest `DATA_ID` wins, in
  positions and Pulse alike.
- **EC3.** `RECORD_TIME` sentinel (before 2020, e.g. 1999-11-30) or NULL (`YEAR` NULL): the
  stored `EventDate` is `timestamp_utc`, like `FN_EVENT_DATE`.
- **EC4.** Readings dated in the future (the Pharma set runs to 2029-05-31): kept exactly as
  today. No `NOW()` filter is added to positions.
- **EC5.** STD device whose newest positioned readings fall outside its STD transfer window:
  the STD branch returns its latest in-window reading, as today. The per-device walk back is
  bounded by that device's own history and stays within NFR1 for contract 5.
- **EC6.** Contract with no devices or no anomaly candidates: empty array, no second query.
- **EC7.** More anomaly candidates than `CPAL_ANOMALIES_CANDIDATS_MAX`: chunked (FR13). Same
  output as one query.
- **EC8.** Anomaly whose reading changed owner (transfer recorded after detection): the
  visibility follows `V_ANOMALIES_CLIENT`, unchanged.
- **EC9.** Upgrade run while readings are being ingested: inserts wait on the table rebuild
  lock. A run interrupted halfway (column added, index missing) completes when re-run.
- **EC10.** Upgrade run on a database without `03-dashboard.sql` objects (no `V_PULSE`): the
  script stops with a clear message before altering anything.
- **EC11.** Contract switched while a request is in flight: the stale response is discarded
  (existing behavior) and never stored under another contract's cache key.
- **EC12.** Cached value older than its maximum age: not shown. Normal loading state.
- **EC13.** Revalidation returns different data: the view updates in place, with no skeleton
  flash, and selection, filters and map viewport kept.
- **EC14.** Logout then login as another user in the same tab: none of the first user's data is
  shown.
- **EC15.** Positions request fails on Pulse or Suivi en direct: map stays visible, error alert
  with retry (FR22).
- **EC16.** Map tiles fail to load (offline): loader, markers and tooltips still work on the
  blank map (Pulse EC17).
- **EC17.** Contract list still loading on Pulse: map and loader shown, no positions request
  until a contract is selected.
- **EC18.** Zero contracts, contract list error, or 404 on the contract: the existing Pulse,
  Pilotage and Alerts states and texts, unchanged.
- **EC19.** Client without `gzip` in `Accept-Encoding`, or body under 1 KiB (errors, empty
  lists): response byte-identical to today.
- **EC20.** Error responses (4xx, 5xx, network) are never cached.
- **EC21.** Very slow response (>10 s, e.g. cold production): the loader stays and the shell
  stays usable. Navigating away drops the response.
- **EC22.** Pilotage ineligible or empty workspace: after the skeleton, the existing
  `pilotage-ineligible` / empty-list states show unchanged.

## 13. Metrics / KPIs

- FR29 numbers for contracts 1, 4, 5 before and after, recorded in the story (target NFR1,
  NFR2; production NFR3).
- Zero FR30 differences (NFR5) and zero NFR6 mismatches.
- NFR4 Playwright check green.
- Gzip ratio on contract 4 responses (NFR12).
- No product analytics: perceived speed is judged in the owner demo review.

## 14. Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Adding the column breaks `tracking_logic.php` at once: its derived table gets two `EventDate` columns. | FR6: the query rewrite (FR8) ships in the same release as the schema change; the upgrade is applied only with that code deployed. |
| `LATERAL` was used in the 0.66 s benchmark but MariaDB 10.11 (production) lacks it. | NFR8: correlated scalar subquery form; benchmark re-run in the chosen form on both engines. |
| Table rebuild blocks ingestion for the upgrade's duration in production. | NFR10, EC9: measured duration, maintenance window, documented in `deployment.md`. TTN uplinks arriving meanwhile wait on the lock. |
| The rewritten `V_PULSE` or alerts query subtly changes visibility (MAIN/STD, lots). | Alerts keep MAIN/STD and owner-at-date visibility in `V_DATA_WITH_CLIENT` (FR12, amended). `V_PULSE` keeps its rules (FR10). The FR30 before/after check on contracts 1, 4, 5 (NFR5) includes the STD contract. |
| The anomaly candidate set misses an anomaly that the view would show. | FR12 requires a superset; PHPUnit case with an STD contract and a transferred device. |
| The cache shows another user's data, or stale data after a change. | Memory only, cleared on session end, invalidated by the mutations, errors not cached (FR25, FR26, EC14, EC20). |
| Gzip applied twice (Apache `mod_deflate` plus PHP). | FR28: skip when `Content-Encoding` is set or `zlib.output_compression` is on; `mod_deflate` leaves already-encoded responses alone. |
| The map mounting before data causes a second viewport jump when positions arrive. | FR16: one auto-fit on arrival, as today. The kit default viewport before that. |
| The "before" baseline is lost once the schema changes. | FR30 capture is taken first (feature area A0 precedes all others). |

## 15. Acceptance Criteria (Definition of Done)

- [ ] AC1. A fresh database from the init scripts has `T_DATA.EventDate`, `T_DATA.HasPosition` (STORED) and the `(ChipID, HasPosition, EventDate, DATA_ID)` index, and the Pharma seed loads unchanged. *(FR1–FR4)*
- [ ] AC2. The upgrade script brings the existing local Pharma DB to the same definitions; a second run is a no-op; it stops cleanly without `03-dashboard.sql` objects; its rollback was run once. *(FR5, FR7, NFR7, EC9, EC10)*
- [ ] AC3. Zero `EventDate`/`HasPosition` mismatches on the full Pharma `T_DATA`. *(NFR6, EC3)*
- [ ] AC4. No PHP or SQL statement fails after the upgrade; the FR6 grep result is recorded in the story. *(FR6)*
- [ ] AC5. Positions for contracts 1, 4, 5 come from the indexed per-device lookup and are identical to before. *(FR8, FR9, NFR5, EC1, EC2, EC4)*
- [ ] AC6. `V_PULSE` returns the same rows as before for contracts 1, 4, 5 (and therefore still the same rows as `V_LAST_POSITION` for active contracts); its comment shows the new timings. *(FR10, FR11, NFR5, EC5)*
- [ ] AC7. The alerts list for contracts 1, 4, 5 is identical to before for every status set; an empty candidate set makes no second query; chunking gives the same output as one query. *(FR12, FR13, NFR5, EC6, EC7, EC8)*
- [ ] AC8. The alert detail meets the NFR1 budget (changed only if it did not). *(FR14)*
- [ ] AC9. The FR29 table shows NFR1 and NFR2 met locally for contracts 1, 4, 5. *(FR29, NFR1, NFR2)*
- [ ] AC10. FR29 numbers are recorded on MariaDB 10.11 after the production upgrade, with the endpoint budget met; the upgrade and queries ran on a MariaDB 10.11 container. *(NFR3, NFR8)*
- [ ] AC11. The FR30 comparison reports zero differences, including `history.php` on the fixed range. *(FR30, NFR5)*
- [ ] AC12. On Pulse and Suivi en direct the map with tiles and the "Chargement des positions…" loader show before positions; markers and one auto-fit follow on arrival; no full-height skeleton or black block. *(FR15, FR16, FR18, EC17)*
- [ ] AC13. The Pulse anomalies list and pallet panel show row-shaped skeletons while loading. *(FR17)*
- [ ] AC14. Pilotage shows its breadcrumb at once and a table-shaped (md+) or card-shaped (below md) skeleton until the workspace loads; ineligible and empty states are unchanged. *(FR19, EC22)*
- [ ] AC15. Alerts shows breadcrumb, filter with count placeholders and list skeleton at once. *(FR20)*
- [ ] AC16. Every data route uses content-shaped skeletons; History's contract loading text is replaced; the audit result is recorded. *(FR21)*
- [ ] AC17. Error, empty, zero-contract and 404 states keep their texts and test IDs; a positions error keeps the map and offers "Réessayer"; tile failure leaves the page usable. *(FR22, EC15, EC16, EC18, EC21)*
- [ ] AC18. Loaders and skeletons are announced (`role="status"` or `aria-busy`, French text) and removed when content shows. *(FR23)*
- [ ] AC19. With API responses delayed 5 s, the shells, and on Pulse and Suivi en direct the map with tiles and the loader, are visible within 1.5 s (Playwright). *(NFR4)*
- [ ] AC20. Returning to Pulse, Pilotage or Alerts within the maximum age shows the last data at once and revalidates in the background; an expired entry is not shown; a failed revalidation keeps the data. *(FR24, FR27, EC12, EC13)*
- [ ] AC21. A transition or comment invalidates the contract's alerts; a transfer or config save invalidates its positions; logout clears everything; Refresh bypasses the cache; errors are never cached; nothing is written to browser storage; a contract switch in flight never fills another contract's entry. *(FR25, FR26, EC11, EC14, EC20)*
- [ ] AC22. JSON responses ≥ 1 KiB to a gzip-accepting client are gzip-encoded with `Vary: Accept-Encoding`; others are byte-identical to before; no double encoding; contract 4 payloads shrink ≥ 70%. *(FR28, NFR12, EC19)*
- [ ] AC23. Inserting readings still evaluates anomalies; 1,000 inserts take ≤ 120% of before. *(NFR9)*
- [ ] AC24. The local upgrade time is ≤ 2 min and recorded in `deployment.md` with the lock note; `local-setup.md` and `deployment.md` describe the upgrade and perf scripts. *(NFR10, FR31)*
- [ ] AC25. Existing suites pass and the NFR11 tests exist and pass. *(NFR11)*
- [ ] AC26. All FRs and NFRs validated.

## 16. Future Extensions (Optional)

- Playback history (`history_logic.php`) and `V_DATA_WITH_CLIENT` on the stored columns, and
  the same candidate-first technique for range reads (up to 25,000 rows).
- `V_ANOMALIES_OUVERTES` (alert light) once an endpoint reads it.
- `sp_lot_conso_refresh_at` (Trends) and `trg_data_anomalies_ai` reading the stored
  `EventDate` instead of `FN_EVENT_DATE`.
- A migrations tool for `cpaltracker.web` (the upgrade folder of FR5 is a stopgap).

## Open questions

None blocking. Non-blocking proposals to confirm during stories: cache maximum age 5 min
(FR24), `CPAL_ANOMALIES_CANDIDATS_MAX` = 1,000 (FR13), gzip threshold 1 KiB (FR28).

---

## Requirement inventory

Every ID below must be covered by at least one story.

| Kind | IDs | Count |
|---|---|---|
| Functional | FR1–FR31 | 31 |
| Non-functional | NFR1–NFR12 | 12 |
| Acceptance criteria | AC1–AC26 | 26 |
| Edge cases | EC1–EC22 | 22 |
| User stories (context, not tracked) | US1–US5 | 5 |

Behavior tracking: the loading UI is customer-facing, but c-PAL has no analytics or funnel
infrastructure (see `IN-PROGRESS-Pulse.md` OQ10), so no tracking requirement is set.
Success is measured by FR29 timings and the demo review (§13).

## Proposed feature areas

| # | Feature area | Requirements | Repo / main files | Depends on |
|---|---|---|---|---|
| A0 | Perf tooling and baseline | FR29, FR30, NFR5 (capture) | `cpaltracker.web/tools/perf/`, `Makefile` | none; **first**: the "before" capture must precede A1–A4 |
| A1 | Schema and upgrade script | FR1–FR7, FR31, NFR6–NFR10, EC3, EC9, EC10 | `docker/mysql/init/01-schema.sql`, `docker/mysql/upgrade/`, `docs/` | A0 |
| A2 | Positions query | FR8, FR9, NFR1/2 (positions), EC1, EC2, EC4 | `tracking_logic.php` | A1; **same release as A1** (FR6 collision) |
| A3 | Pulse view | FR10, FR11, NFR1/2 (Pulse), EC5 | `03-dashboard.sql`, upgrade script | A1 |
| A4 | Alerts query | FR12–FR14, NFR1/2 (alerts), EC6–EC8 | `anomalies_logic.php` | A0 only (no schema dependency) |
| A5 | Non-blocking loading UI | FR15–FR23, NFR4, EC15–EC18, EC21, EC22 | `ContractPositions.tsx`, `Pulse.tsx`, `ContractAnomalies.tsx`, `PalletPanel.tsx`, `DevicePilotage.tsx`, `Alerts.tsx`, `HistoryDashboard.tsx` | none (frontend only) |
| A6 | Client cache | FR24–FR27, EC11–EC14, EC20 | `src/anomalies/useLoad.ts` or a shared hook, `src/api/*`, `ContractPositions.tsx`, `loadPilotageWorkspace.ts`, `AuthContext.tsx` | A5 (same files) |
| A7 | Gzip | FR28, NFR12, EC19 | `api/v1/_lib.php` | none |
| — | Verification | NFR3 (production), NFR11, AC26 | across areas | all; NFR3 after the production upgrade |

File overlaps: A1 and A3 both write the upgrade script. A1 and A2 both concern
`tracking_logic.php` (FR6). A5 and A6 both touch `ContractPositions.tsx`, `Alerts.tsx`,
the Pulse panels and the Pilotage loader, so do them in sequence (A5 then A6) or in one story.
