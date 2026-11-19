# Feature: Trends (expiry risk per lot)

Feature spec (W1, [`FEATURE.md`](../engineering/product/FEATURE.md) template). Epic: [KAN-253](https://c-pal.atlassian.net/browse/KAN-253), [`KAN-253-Trends.md`](../epics/KAN-253-Trends.md).

Sources:

- Confluence COC « 2026-11-18 - dashboard » (owner: Gaël COQUIO),
  https://c-pal.atlassian.net/wiki/spaces/COC/pages/115605505/2026-11-18+-+dashboard —
  §3 (decisions of 18/11/2026), §7 (Trends tab), §8.2 (consumption), §8.3 (rights),
  §8.5 (open points).
- Owner decisions of 2026-11-19 (Felipe): Trends is a **new page**, strictly
  additive, with a new navigation entry. They override the Confluence text where
  they differ (no navigation replacement, no profile menu work).
- Mockup (Gaël): https://claude.ai/artifact/S21fVkz7x9471AA4pAgRU8 — simulated
  data, layout reference only. Where it and this spec disagree, this spec wins.
- Database objects already built locally, which this spec is written against:
  `cpaltracker.web/docker/mysql/init/03-dashboard.sql` (§1 `T_DDLOT.Statut`, §6
  `T_LOT_CONSO_SNAP`, `sp_lot_conso_refresh_at`, `sp_lot_conso_refresh`,
  `ev_lot_conso_weekly`, §7 `V_TRENDS`, §9 production grants) and
  `cpaltracker.web/docker/mysql/init/01-schema.sql` (`T_DDLOT`, `trg_ddlot_bu`,
  `T_DDLOT_DEVICE_HIST`, `FN_EVENT_DATE`, `V_DDLOT_DATA`).

Sibling specs (same dashboard, separate features): `IN-PROGRESS-Pulse.md` (map of last
positions) and `IN-PROGRESS-Anomaly-Workflow.md` (anomaly drawer and status workflow).

**For Gaël's validation:** §10.3 lists every place where the calculation as built
deviates from, or makes precise, the Confluence text. §17 lists the open points.

## Scope v1 (trimmed 2026-11-19)

Owner rule (2026-11-19): Trends v1 is a page that reads a table seeded with fixed (hardcoded) values, one row per lot, with no calculation; the computed pipeline already built in `03-dashboard.sql` §6 (`T_LOT_CONSO_SNAP`, `sp_lot_conso_refresh`, `ev_lot_conso_weekly`, `V_TRENDS`) is kept locally but is v2 and is not rolled out to production in v1 (rollout of the DB part is KAN-259). Requirement text below is unchanged (SSoT); deferred requirements are not deleted or renumbered.

| Story | What the user sees | Requirements |
|---|---|---|
| KAN-256 Trends page — from a hardcoded table (5 pts, no deps) | A new `/trends` page with a nav entry: a quadrant scatter (Plotly used directly) and a table of lots with a risk badge, sortable. cpaltracker.web: a seeded table with fixed rows plus `contracts/trends.php` reading it. cpaltracker-react: route, nav entry, `src/trends/*` with a DataTable. | FR15–FR17, FR25, FR26, FR28–FR33, FR38, FR39; NFR5, NFR7, NFR8 |

Hardcoded table columns (the fields the page displays): lot, customer lot number, product, balance (kg), reference velocity, required velocity, risk score `z`, risk, expiry date, contract. One row per lot, seeded by SQL, read as-is by the endpoint: `z` and the risk are stored values, not computed.

Reductions of kept requirements (FR text not rewritten):

- FR15–FR17: the endpoint reads the hardcoded table, not `V_TRENDS`; the FR17 payload is limited to the columns above (one stored `z`, no periods, no two-grid structure).
- FR28–FR30: the scatter is plotted with Plotly directly, not a kit `ScatterChart` component. Axes as designed: x = days to expiry (from the stored expiry date), y = the stored `z`; nothing is computed server-side. Risk colour and shape, bands, tooltip and click-to-row stay.
- FR29: every row of the table is placed; there is no minimum-history rule (no period counts in the table).
- FR31–FR33: the table shows the columns above; one value per lot, no unit, method or grid selection; sort by risk then days to expiry.
- FR32: the risk badge shows icon and text from the stored risk; the "reason" cells for non-placed rows do not apply.
- FR38: states reduced to loading, error, no contract, STD and no lot (no "no snapshot yet").
- FR39: kit components and tokens only for everything except the Plotly chart; no new kit component.

Deferred to v2:

- FR1–FR14 and D1–D17: the computed pipeline (data model, weekly refresh, `V_TRENDS`, deviations to validate) — built locally, not rolled out.
- FR18: both grids/methods in one call — the hardcoded table has one value per lot.
- FR19: OpenAPI entry for the close endpoint.
- FR20–FR24 and FR37: closing a lot (endpoint, audit, UI).
- FR27: week/month and means/medians toggles (and the unit toggle) — one value per lot.
- FR34, FR35, FR36: product-unit conversion, missing-grid badge, per-row snapshot date.
- FR40: method note popover.
- NFR1–NFR4, NFR6, NFR10, NFR11: refresh duration, MariaDB check, endpoint performance, toggle latency, close-endpoint rules, E2E tests, idempotent refresh.
- NFR9: production script for the computed objects; v1 rollout of the hardcoded table only is KAN-259.

ACs and ECs follow their FR: an AC/EC whose referenced FRs are all deferred is deferred; otherwise it belongs to the story of its FR (KAN-256), reduced as above. Kept in v1: AC8, AC13, AC14, AC16–AC19, AC23, AC25, AC26, AC29; EC1, EC10, EC19, EC21, EC22. ECs on the calculation (EC2–EC9, EC11–EC14), on closing (EC17, EC18), on grids and snapshots (EC15, EC16) and EC20 are deferred. Where an AC mixes kept and deferred parts, only the kept part applies in v1 (for example AC8 without the FR18 part, AC16 without the monthly view, AC19 without the reasons, AC23 without the no-snapshot state, AC25 without the kit component and showcase, AC29 for the kept requirements only). Retired stories are in the Jira backlog with prefix "[Reporté v2]" and label `reporte-v2`.

## 1. Feature Name

Trends: a new page of the c-PAL React app that shows, for every open lot of the
selected contract, the risk that the lot is not consumed before its expiry date, as
a quadrant chart (remaining shelf life × risk score `z`) and a lot detail table, from
a weekly consumption snapshot. The user can close a lot to take it out of the
analysis.

## 2. Problem / Pain Point

A c-PAL client manufactures products and ships them by lot. Each lot carries a
c-PAL device that weighs it; as the lot is used up at the destination, the measured
mass falls. Planning teams (Confluence §2, population "Planification") need to know
which lots will not be used up before they expire, early enough to move stock,
change allocation or push sales.

Today:

- The data exists (mass readings per device, lot hooks, expiry date per lot in
  `T_DDLOT`) but no page turns it into a consumption rate or a risk. The History page
  (`/history`) shows the raw mass curve of one contract over a date range; a planner
  would have to read every lot's curve and do the arithmetic by hand.
- Nothing ranks lots by risk, so the lots that need action are not visible among the
  ones that are fine.
- Lots that are finished or no longer relevant stay "open" forever: there is no lot
  status, so any analysis would keep showing them.

## 3. Goal / Outcome (Success Definition)

A planner opens Trends, sees the snapshot date, and reads at a glance which lots of
the selected contract are at high, medium or low risk of expiring before they are
consumed: each lot is a point placed by its remaining shelf life (x) and its risk
score `z` (y), in coloured bands split at `z = 0` and `z = 1`. They can switch the
reference method (means or medians), the time grid (weekly or monthly) and the unit
(kg or number of products) without reloading. A table under the chart gives every
figure behind each point, sorted by risk. When a lot is finished or no longer
relevant, the planner closes it from the table and it leaves the analysis. Nothing
else in the app changes.

## 4. User Stories (Primary Scenarios)

- **US1 — See which lots are at risk.** As a planner, I want to see every open lot
  of my contract placed by remaining shelf life and risk score, coloured by risk
  level, so that I spot at once the lots that need action.
  *(FR10–FR13, FR25–FR30, FR38)*
- **US2 — Understand a lot's figures.** As a planner, I want a table with the
  balance, reference velocity, dispersion, required velocity, `z` and risk of each
  lot, in kg or in products, so that I can check why a lot is classed as it is and
  quantify the action. *(FR14, FR31–FR36, FR40)*
- **US3 — Compare methods and time grids.** As a planner, I want to switch between
  means and medians and between weekly and monthly consumption, so that one atypical
  week or month does not mislead me. *(FR11, FR27, FR28, FR31)*
- **US4 — Take a lot out of the analysis.** As a planner, I want to close a lot that
  is finished or no longer relevant, so that the chart only shows lots I still have to
  manage. *(FR20–FR24, FR37)*
- **US5 — Stay within my own data.** As a client user, I want to see only the lots
  my company owns under the selected contract, and a clear message when there is
  nothing to show, so that the page is both safe and understandable.
  *(FR15, FR16, FR21, FR38, NFR5)*

## 5. In Scope (What WILL be delivered)

- **Database (already built locally, delivered and verified by this feature):**
  `T_DDLOT.Statut`, `T_LOT_CONSO_SNAP`, `sp_lot_conso_refresh_at(date)`,
  `sp_lot_conso_refresh()`, `ev_lot_conso_weekly`, `V_TRENDS`, as in
  `03-dashboard.sql` §1, §6, §7, with the calculation rules of FR1–FR14, plus their
  production rollout (grants, event scheduler) (NFR9).
- **Backend, read:** a new endpoint `GET /api/v1/contracts/trends.php/{contractId}`
  returning the latest snapshot of the contract's open lots, both grids and both
  methods in one response (FR15–FR19).
- **Backend, write:** a new endpoint `POST /api/v1/lots/close.php/{cpalLot}` that sets
  a lot's status to `closed` (FR20–FR24).
- **Frontend:** a new route `/trends` with a new navigation entry, inside the existing
  authenticated shell and global contract selector (FR25, FR26), containing:
  - a header with the snapshot date and three toggles: method, grid, unit (FR27);
  - the quadrant scatter chart (FR28–FR30), with a new kit `ScatterChart` component;
  - the lot detail table (FR31–FR36);
  - the close-lot action with confirmation (FR37);
  - loading, empty and error states (FR38);
  - a short "how it is calculated" note (FR40).
- OpenAPI documentation of both endpoints (FR19).
- Tests: SQL fixtures for the calculation, PHP unit tests for both endpoints, Vitest
  for the page logic, Playwright E2E (NFR10).

## 6. Out of Scope (What WILL NOT be delivered)

- **Any change to an existing page or API response** (Suivi en direct
  `/live-tracking`, Historique `/history`, Pilotage `/pilotage`, account, auth). No
  existing navigation entry is removed or renamed.
- **On-demand recomputation** of the snapshot from the app. The page reads the latest
  weekly snapshot only.
- **Reopening a closed lot** from the app (open question OQ4).
- **Entering the kg ↔ products coefficient** per lot by the client. It is derived
  from `T_DDLOT` (OQ3).
- **History of snapshots** (how a lot's `z` moved week after week). Snapshots are kept
  in `T_LOT_CONSO_SNAP` but not shown (§16).
- **Lots with no consumption yet** (in transport, flat weight): they have no snapshot
  and are not listed (EC7, OQ8).
- Profile menu, messages, "Mes contrats" panel (owner decision C4).
- Exports (CSV, API), notifications (mail, SMS, webhook).
- Pulse (`IN-PROGRESS-Pulse.md`) and the anomaly workflow (`IN-PROGRESS-Anomaly-Workflow.md`).
- Display roles per population (Confluence §2: one dashboard for everyone).
- Rights per site or region.
- Behavior tracking / product analytics: c-PAL has no analytics or funnel
  instrumentation and its SDLC defines no behavior-tracking step. The only usage
  signal this feature emits is the existing audit log for the close action (FR23).

## 7. User Flow / UX Breakdown

1. The user clicks **Trends** in the sidebar. The app opens `/trends` inside the
   authenticated shell (FR25).
2. The page reads the contract selected in the global contract selector (FR26) and
   calls `GET /api/v1/contracts/trends.php/{contractId}`. While loading, a skeleton of
   the chart and table is shown (FR38).
3. The API checks the user may access the contract (else the same 404 as every other
   endpoint), and returns the contract's open lots from `V_TRENDS`, with the latest
   snapshot date (FR15–FR18).
4. The page shows the header: title, "Instantané du `<date>`" and the three toggles,
   defaulting to Moyennes, Hebdomadaire, kg (FR27).
5. The chart shows one point per lot that can be placed (FR29): x = days to expiry at
   the snapshot date, y = `z` for the selected method and grid. Horizontal bands mark
   low (`z ≤ 0`), medium (`0 < z ≤ 1`) and high (`z > 1`) risk; a vertical line marks
   the quadrant split at 30 days (OQ2). A line under the chart says how many lots are
   not placed and why (FR29).
6. The table lists every lot of the response, high risk first (FR31–FR33).
7. The user switches a toggle. Chart and table update at once, with no network call
   (FR27, NFR4). In "produits" mode, kg figures are converted with the lot's
   coefficient; `z` and risk do not change (FR34).
8. The user hovers a point: tooltip with lot number, product, days to expiry, `z`,
   risk. Clicking it highlights the lot's row in the table and scrolls to it (FR30).
9. The user opens a row's menu and chooses "Clôturer le lot". A confirmation dialog
   names the lot and says it leaves the Trends analysis. On confirm, the app calls
   `POST /api/v1/lots/close.php/{cpalLot}` (FR37, FR20).
10. On success, a toast confirms, and the page refetches: the lot is gone from chart
    and table. On error, the dialog shows what happened and the lot stays (FR37).
11. The user picks another contract in the sidebar: the page refetches for it (FR26).
    If the contract is a STD (shared) contract, the page shows the STD empty state
    (FR38).

## 8. Functional Requirements (FRs)

### 8.1 Data and calculation (database, as built in `03-dashboard.sql`)

- **FR1. Lot status.** `T_DDLOT` SHALL have `Statut enum('created','closed') NOT NULL
  DEFAULT 'created'`. Only `created` lots SHALL be computed by the refresh and listed
  by `V_TRENDS`.
- **FR2. Readings used.** The refresh SHALL use, for each `created` lot, the readings
  of `T_DATA` such that:
  - the device was hooked to the lot (`T_DDLOT_DEVICE_HIST`, any hook of the lot, past
    or current) and the reading's `EventDate` (`FN_EVENT_DATE(RECORD_TIME,
    timestamp_utc)`) is in `[Hook_Start_Date, Hook_End_Date)`;
  - the hook's contract is a `MAIN` contract of the lot's owner client
    (`T_CONTRATS.ClientID = T_DDLOT.ClientID`), the device is attached to it
    (`T_CONTRATS_DEVICES`), and `timestamp_utc` is within the contract's dates;
  - `MASSE` is not NULL and `>= 0`;
  - `EventDate < SnapDate` (the snapshot date, exclusive).

  These are the rows `V_DDLOT_DATA` would return for the lot (its MAIN branch), read
  from `T_DATA` directly for performance (NFR1).
- **FR3. Consumption per reading.** For each reading, consumption SHALL be
  `max(previous MASSE − MASSE, 0)`, where "previous" is the lot's preceding retained
  reading ordered by `EventDate`, then `DATA_ID`, across all hooks of the lot. The
  lot's first reading contributes 0. Increases (reload, weighing artefact) contribute
  0. The consumption is attributed to the period of the later reading.
- **FR4. Periods and velocity.** The refresh SHALL compute, per lot, two grids:
  `semaine` (ISO week, `YEARWEEK(EventDate, 3)`, label `YYYYWW`) and `mois`
  (`DATE_FORMAT(EventDate, '%Y-%m')`). A period's velocity SHALL be the sum of the
  consumption of its readings, in **kg consumed per period**. A period exists only if
  it holds at least one retained reading (no zero-filling of silent periods).
- **FR5. Periods excluded.** The refresh SHALL exclude:
  - the period that contains `SnapDate` (incomplete);
  - every period before the lot's first period with velocity `> 0` (transport phase,
    flat weight).

  A lot with no period of velocity `> 0` SHALL get no snapshot row for that grid.
  Periods with velocity 0 after the first consumption are kept.
- **FR6. Statistics.** Over the retained periods of a lot and grid, the refresh SHALL
  store: `NbPeriodes` (count), `NbMesures` (readings in those periods),
  `VitesseMoyenne` (mean), `EcartType` (sample standard deviation, `n − 1`; NULL when
  `n = 1`), `Mediane`, `Q1`, `Q3` (linear interpolation at rank `h = (n − 1) × p`,
  same result as `PERCENTILE_CONT` and Excel `QUARTILE.INC`), `PremierePeriode`,
  `DernierePeriode`.
- **FR7. Balance.** `SoldeKg` SHALL be the `MASSE` of the lot's last retained reading
  before `SnapDate` (FR2), `SoldeAt` its `EventDate`, and the snapshot's `ContratID`
  the MAIN contract of that reading.
- **FR8. Snapshot storage.** Results SHALL be stored in `T_LOT_CONSO_SNAP`, one row
  per (`cPAL_Lot`, `Maille`, `SnapDate`). Computing a `SnapDate` again SHALL replace
  that date's rows; earlier snapshots SHALL be kept.
- **FR9. Weekly refresh.** `ev_lot_conso_weekly` SHALL run every Monday at 02:00
  server time and call `sp_lot_conso_refresh()`, which computes the snapshot as of
  `CURDATE()`. `sp_lot_conso_refresh_at(date)` SHALL compute the snapshot as of any
  date (backfill, tests). The app SHALL NOT trigger a refresh.
- **FR10. Required velocity.** `V_TRENDS` SHALL return, per `created` lot and grid,
  the latest snapshot, with:
  - `JoursRestants = DATEDIFF(Date_Peremption, SnapDate)` (days to expiry **as of the
    snapshot date**, can be 0 or negative);
  - `JoursParPeriode` = 7 (week) or 365.25 / 12 = 30.4375 (month);
  - `Vreq = SoldeKg / (JoursRestants / JoursParPeriode)` in kg per period when
    `JoursRestants > 0`, else NULL.
- **FR11. Reference and dispersion per method.** Means: `Vref = VitesseMoyenne`,
  dispersion = `EcartType`. Medians: `Vref = Mediane`, dispersion =
  `EcartInterquartile = Q3 − Q1`.
- **FR12. Score z.** `z = (Vreq − Vref) / dispersion`, per method (`z_moyennes`,
  `z_medianes`). `z` SHALL be NULL when the dispersion is NULL or 0, or `Vreq` is NULL.
- **FR13. Risk level.** Per method (`Risque_moyennes`, `Risque_medianes`), evaluated
  in this order, first match wins:

  | # | Condition | Risk |
  |---|---|---|
  | 1 | `SoldeKg` is NULL or `Vref` is NULL | NULL (not evaluated) |
  | 2 | `SoldeKg <= 0` (empty lot) | `low` |
  | 3 | `JoursRestants <= 0` (expired, stock left) | `high` |
  | 4 | `Vreq > Vref + COALESCE(dispersion, 0)` | `high` |
  | 5 | `Vreq > Vref` | `medium` |
  | 6 | otherwise (`Vreq <= Vref`) | `low` |

  Written on `Vreq`, not on `z`, so the level holds when the dispersion is 0 or NULL
  (then `medium` cannot occur). Where `z` is defined, rules 4–6 equal `z > 1`,
  `0 < z <= 1`, `z <= 0`.
- **FR14. Products coefficient.** `V_TRENDS.ProduitsParKg` SHALL be
  `Number_Of_Products / Net_Weight` of the lot, NULL when `Net_Weight = 0`.

### 8.2 Backend API — read

- **FR15. Trends endpoint.** `GET /api/v1/contracts/trends.php/{contractId}` (id by
  `PATH_INFO`, `?contractId=` fallback, `cpal_api_resolve_id_from_path_or_query()`)
  SHALL follow the `history.php` pattern: CORS and preflight from `_lib.php`;
  `405 method_not_allowed` for anything but GET; bearer auth through
  `cpal_api_require_auth()`; `400 invalid_contract_id` if the id is not a positive
  integer; access through `cpal_contrat_autorise()`, with the same
  `404 not_found` for a missing contract and one the caller may not access;
  `503 service_unavailable` on a database error.
- **FR16. Scope.** The endpoint SHALL return the `V_TRENDS` rows with
  `ContratID = {contractId}` and `ClientID` = the contract's `ClientID` (an admin sees
  the contract owner's lots). For a contract of type `STD`, it SHALL return 200 with an
  empty `lots` list (lots belong to the MAIN client).
- **FR17. Response.** The response SHALL be:

  ```json
  {
    "contract": { "contractId": 12, "contractRef": "MAIN-INNOPHARM", "contractType": "MAIN" },
    "snapshotDate": "2027-03-01",
    "latestRefreshDate": "2027-03-01",
    "minPeriods": 4,
    "quadrantSplitDays": 30,
    "lots": [
      {
        "cpalLot": "…", "custLotNumber": "…", "productName": "…",
        "expiryDate": "2027-06-30", "netWeight": 820.0, "numberOfProducts": 4100,
        "productsPerKg": 5.0,
        "week":  { "snapshotDate": "2027-03-01", "balanceKg": 412.5, "balanceAt": "…",
                   "periods": 9, "readings": 180, "firstPeriod": "202652", "lastPeriod": "202708",
                   "meanVelocity": 41.0, "stdDev": 12.3, "medianVelocity": 39.5,
                   "q1": 33.0, "q3": 47.0, "iqr": 14.0,
                   "daysToExpiry": 121, "daysPerPeriod": 7, "requiredVelocity": 23.9,
                   "zMean": -1.39, "riskMean": "low", "zMedian": -1.12, "riskMedian": "low" },
        "month": null
      }
    ]
  }
  ```

  `snapshotDate` = the latest `SnapDate` among the returned rows (null if none);
  `latestRefreshDate` = `MAX(SnapDate)` of `T_LOT_CONSO_SNAP` (null if the table is
  empty; it says whether any refresh has run, and reveals no lot data); `minPeriods`
  and `quadrantSplitDays` = the display constants of OQ1 and OQ2, defined once in the
  backend. A grid with no `V_TRENDS` row for the lot SHALL be `null`. Values are raw
  numbers (no rounding), NULL as `null`, dates ISO 8601. The column mapping is in §10.1.
- **FR18. One call, read-only.** The endpoint SHALL return both grids and both methods
  in one response, take no other parameter, and write nothing.
- **FR19. OpenAPI.** Both endpoints (FR15, FR20) SHALL be documented in
  `api/openapi.yaml` with their request, response and error codes.

### 8.3 Backend API — close a lot

- **FR20. Close endpoint.** `POST /api/v1/lots/close.php/{cpalLot}` (id by
  `PATH_INFO`, `?cpalLot=` fallback) SHALL follow the `devices/transfer.php` pattern:
  preflight/CORS, `405` for anything but POST, bearer auth, `400 invalid_request` if
  the id is empty or longer than 22 characters, `503` on a database error. No request
  body is required; a body, if sent, is ignored.
- **FR21. Ownership.** The lot SHALL exist and belong to the caller's client
  (`T_DDLOT.ClientID = T_USER.ClientID`), or the caller SHALL be admin. Otherwise the
  endpoint SHALL answer the same `404 not_found` for a missing lot and a lot of another
  client. A rejection by `trg_ddlot_bu` (defense in depth) SHALL map to the same 404.
- **FR22. Transition.** The endpoint SHALL run `UPDATE T_DDLOT SET Statut = 'closed',
  UpdatedBy = <caller UserID> WHERE cPAL_Lot = ? AND Statut = 'created'`. If no row is
  updated because the lot is already `closed` (including a concurrent close), it
  SHALL answer `409 lot_already_closed`. On success it SHALL answer 200
  `{ "cpalLot", "status": "closed", "closedAt": <UpdatedAt, ISO 8601>, "closedBy": <UserID> }`.
- **FR23. Audit.** A successful close SHALL be logged with
  `cpal_log($pdo, 'lot_cloture', true, <UserID>, <Mail>, 'lot <cPAL_Lot>')`.
- **FR24. Effect.** Closing SHALL change only `Statut`, `UpdatedBy` and `UpdatedAt` of
  the lot. Snapshots, hooks and readings are untouched. The lot SHALL disappear from
  `V_TRENDS` immediately (the view filters on the current status), with no refresh.

### 8.4 Frontend — page

- **FR25. Route and navigation.** The app SHALL add a route `TRENDS_PATH = '/trends'`
  (`auth/paths.ts`) behind `RequireAuth`, inside `AppShellLayout`, and a navigation
  item "Trends" (lucide `TrendingUp`, `testId: 'nav-trends'`) after "Pilotage". The
  existing items, routes and their order SHALL be unchanged.
- **FR26. Contract.** The page SHALL use the global contract selection
  (`useContracts()`), fetch on arrival and on every change of `selectedId`, and on a
  404 call `rejectContract()` and show `CONTRACT_ACCESS_ERROR`, as History does.
- **FR27. Header and toggles.** The page SHALL start with `PageHeader` (title
  "Trends"), the line "Instantané du `<snapshotDate, dd/MM/yyyy>`" with a hint that
  balances and days to expiry are as of that date, and three single-choice
  `ToggleGroup`s: **Méthode** (Moyennes | Médianes, default Moyennes), **Maille**
  (Hebdomadaire | Mensuelle, default Hebdomadaire), **Unité** (kg | Produits, default
  kg). Toggling SHALL update chart and table from the loaded response, with no
  network call. The choices SHALL persist while navigating within the session
  (sessionStorage), like the contract.
- **FR28. Quadrant chart.** The page SHALL render a scatter chart (new kit component
  `ScatterChart` on top of `ui/charts/plot.tsx`) with:
  - x = `daysToExpiry` of the selected grid, axis title "Durée de vie restante
    (jours)";
  - y = `zMean` or `zMedian` per the method, axis title "Score de risque z";
  - three horizontal bands, low (`z <= 0`), medium (`0 < z <= 1`), high (`z > 1`), with
    lines at `z = 0` and `z = 1`, always in view whatever the data range;
  - a vertical dashed line at `x = quadrantSplitDays`;
  - one point per placed lot (FR29), coloured **and** shaped by its risk (low:
    success/circle, medium: warning/square, high: danger/diamond), with a legend;
  - an accessible `label` stating the contract, grid, method and number of lots shown.
- **FR29. Which lots are placed.** A lot SHALL be placed on the chart for the selected
  grid and method only if: its grid object is not null, `periods >= minPeriods`,
  `z` (for the method) is not null, and `daysToExpiry` is not null. Under the chart
  the page SHALL show "`k` lot(s) non positionné(s)" with the count per reason
  (historique insuffisant, dispersion nulle, lot périmé, aucune période complète),
  when `k > 0`.
- **FR30. Point interaction.** Hovering a point SHALL show a tooltip with the client
  lot number, product, days to expiry, `z` (2 decimals) and risk label. Clicking a
  point SHALL highlight the lot's table row and scroll it into view.
- **FR31. Lot table.** The page SHALL render a `DataTable` (density `compact`,
  sticky header) of every lot in the response, with these columns, for the selected
  method, grid and unit:

  | Column (French label) | Value |
  |---|---|
  | Lot client | `custLotNumber` |
  | Produit | `productName` |
  | Durée de vie restante (j) | `daysToExpiry` |
  | Solde (kg \| produits) | `balanceKg` (converted, FR34) |
  | Vitesse de référence (kg \| produits / semaine \| mois) | `meanVelocity` or `medianVelocity` |
  | Dispersion (same unit) | `stdDev` or `iqr` |
  | Vitesse requise (same unit) | `requiredVelocity` |
  | z | `zMean` or `zMedian`, 2 decimals |
  | Risque | `riskMean` or `riskMedian` as a badge |
  | Péremption | `expiryDate`, dd/MM/yyyy |

  Numbers use the `fr-FR` locale and thousands separators (X-FORMAT); NULL shows "—".
- **FR32. Risk badge and reasons.** The Risque column SHALL show a badge with icon and
  text (Élevé = danger, Moyen = warning, Faible = success; never colour alone). When a
  lot is not placed (FR29), the row SHALL show the reason as a muted badge:
  "Historique insuffisant (`n` période(s))", "Dispersion nulle", "Lot périmé",
  "Aucune période complète". Below `minPeriods`, `z` and Risque SHALL show "—", except
  an expired lot with stock (risk `high`) and an empty lot (risk `low`), whose level
  does not depend on history and is shown.
- **FR33. Sorting.** Default sort: risk high, medium, low, then not evaluated; within
  a level, `daysToExpiry` ascending. Every column SHALL be sortable.
- **FR34. Unit conversion.** In "Produits" mode, balance, reference velocity,
  dispersion and required velocity SHALL be multiplied by `productsPerKg`; `z`, risk
  and days SHALL be unchanged. When `productsPerKg` is null or `<= 0`, these cells SHALL
  show "—" with a tooltip "Coefficient indisponible (poids net ou nombre de produits
  nul)", and the point stays on the chart. Products are shown with no decimals, kg
  with one.
- **FR35. Grid without data.** When a lot's object for the selected grid is null, its
  row SHALL show "—" for every computed column and the badge "Aucune période
  complète".
- **FR36. Older snapshot rows.** When a lot's grid `snapshotDate` is older than the
  response's `snapshotDate`, its row SHALL carry the badge "Instantané du
  `<date>`".
- **FR37. Close a lot.** Each row SHALL have a row menu entry "Clôturer le lot". It
  SHALL open an `AlertDialog` naming the client lot number and product, stating that
  the lot leaves the Trends analysis and cannot be reopened from the app, with
  "Clôturer" (destructive) and "Annuler". On confirm it SHALL call FR20, disable the
  button while pending, and:
  - 200: close the dialog, show a toast "Lot `<n>` clôturé", refetch the page data;
  - 409: show "Ce lot est déjà clôturé." and refetch;
  - 404: show the generic "Ce lot n'est pas disponible." and refetch;
  - network or 503: show "La clôture n'a pas pu être enregistrée. Réessayez." and
    keep the dialog open.
- **FR38. States.** The page SHALL show:
  - loading: `Skeleton` shaped like chart and table;
  - error: inline `Alert variant="danger"` with a retry button;
  - no contract selected: the same message as other pages;
  - STD contract: `Empty` "Les lots sont suivis par le titulaire principal du
    contrat ; ce contrat partagé n'en affiche pas.";
  - no snapshot yet (`latestRefreshDate` null): `Empty` "Aucun instantané n'a encore
    été calculé. Le calcul a lieu chaque lundi à 02:00.";
  - no lot (`lots` empty, a snapshot exists): `Empty` "Aucun lot ouvert de ce contrat
    n'a encore de période de consommation complète (instantané du `<date>`)."
- **FR39. Additivity.** The page SHALL add files and the FR25 entries only; no
  existing component's behaviour, copy or test SHALL change. New kit components
  (`ScatterChart`) SHALL be added to the kit and its showcase first (UI guidelines,
  rule 5).
- **FR40. Method note.** The header SHALL offer an info popover "Comment est calculé le
  risque ?" summarising, in French: velocity = kg consumed per period; required
  velocity = balance ÷ periods left before expiry; `z = (Vreq − Vref) / dispersion`;
  the three bands; values as of the snapshot date. Its text is written once in the
  page and is the user help for this feature.

## 9. Non-Functional Requirements (NFRs)

- **NFR1. Refresh duration.** `sp_lot_conso_refresh_at` on the Pharma demo set (57
  lots, 1,140 hooks, ~1.4 M readings) SHALL complete in under 5 minutes on the local
  stack (MySQL 8.0 docker). It SHALL read `T_DATA` directly with the FR2 rules, not
  through `V_DDLOT_DATA` (a first version through the view did not finish in 10 min).
  The measured time is recorded in the delivery notes.
- **NFR2. Portability.** All SQL SHALL run on MySQL 8.0 (local) and MariaDB 10.11
  (production): no `MEDIAN()`, `PERCENTILE_CONT()` or other engine-specific syntax.
- **NFR3. API time.** The trends endpoint SHALL answer in under 1 s (p95, local stack)
  for a contract with 100 open lots, both grids, with a payload under 200 KB.
- **NFR4. Page time.** With 100 lots, chart and table SHALL render within 2 s of the
  contract being selected (local stack), and every toggle SHALL update both within
  300 ms with no network call.
- **NFR5. Security.** Every read and write SHALL be scoped server-side (FR15, FR16,
  FR21). Missing and not-yours contracts or lots SHALL get identical responses. The UI
  SHALL not gate anything on a client-held role.
- **NFR6. Traceability.** A close SHALL leave `UpdatedBy`, `UpdatedAt` on the lot and
  an auth-log line (FR23). Lots stay undeletable (`trg_ddlot_bd`).
- **NFR7. Additive.** Existing pages, API endpoints and responses SHALL behave as
  before; their existing unit and E2E suites SHALL pass unchanged.
- **NFR8. UI guidelines.** Kit components and tokens only; dark and light themes;
  1440 px and 390 px (table scrolls horizontally, chart full width); X-COLOR (warning is
  yellow; state never by colour alone); X-A11Y (chart label, keyboard-operable row
  menu and dialog, focus restored after the dialog); French copy (X-COPY);
  `date-fns` for dates (X-FORMAT); charts do not animate (X-MOTION).
- **NFR9. Production rollout.** The `03-dashboard.sql` objects used here SHALL be
  applied on the VPS (MariaDB 10.11) with the grants `SELECT` on `V_TRENDS` and
  `UPDATE (Statut, UpdatedAt, UpdatedBy)` on `T_DDLOT` for `cpal_web`, and
  `event_scheduler=ON`. A first snapshot SHALL be computed at install.
- **NFR10. Tests.**
  - SQL: fixtures run through `sp_lot_conso_refresh_at('<date>')` with hand-computed
    expected values for FR3–FR7, FR10–FR14 and EC2–EC9, EC13–EC15. The Pharma demo set is
    future-dated (2026-10 → 2029-05): tests SHALL use `sp_lot_conso_refresh_at` with a
    date inside it, never `sp_lot_conso_refresh()`.
  - PHP: unit tests of both endpoints under `CPAL_API_TESTING` (every status code).
  - Vitest: placement (FR29), conversion (FR34), sort (FR33), reasons (FR32), states
    (FR38).
  - Playwright E2E: US1–US5 against the local stack with a backfilled snapshot.
- **NFR11. Idempotent refresh.** Running `sp_lot_conso_refresh_at(d)` twice on the same
  data SHALL leave identical rows for `d` (apart from `CalculeAt`).

## 10. Data Inputs & Outputs

**Inputs:** `T_DATA` (`MASSE`, `RECORD_TIME`, `timestamp_utc`, `ChipID`, `DATA_ID`),
`T_DDLOT_DEVICE_HIST` (hook windows, contract), `T_CONTRATS`, `T_CONTRATS_DEVICES`,
`T_DDLOT` (`Statut`, `ClientID`, `Cust_Lot_Number`, `Product_Name`,
`Date_Peremption`, `Net_Weight`, `Number_Of_Products`).

**Weekly transformation:** `sp_lot_conso_refresh_at` → `T_LOT_CONSO_SNAP` (FR2–FR9).
**Read model:** `V_TRENDS` (FR10–FR14). **API:** FR17 JSON. **Write:** `T_DDLOT.Statut`
via FR20–FR22.

### 10.1 Data contract (UI value → API field → `V_TRENDS` column)

| UI value | API field (FR17) | `V_TRENDS` column |
|---|---|---|
| Snapshot date ("Instantané du …") | `snapshotDate` (max), `week/month.snapshotDate` | `SnapDate` |
| Any refresh run yet | `latestRefreshDate` | — (`MAX(T_LOT_CONSO_SNAP.SnapDate)`) |
| Lot client | `custLotNumber` | `Cust_Lot_Number` |
| Lot id (close action, key) | `cpalLot` | `cPAL_Lot` |
| Produit | `productName` | `Product_Name` |
| Péremption | `expiryDate` | `Date_Peremption` |
| Durée de vie restante (x axis) | `daysToExpiry` | `JoursRestants` |
| Solde | `balanceKg` (× `productsPerKg`) | `SoldeKg` |
| Last reading of the balance | `balanceAt` | `SoldeAt` |
| Vitesse de référence, means | `meanVelocity` | `VitesseMoyenne` |
| Vitesse de référence, medians | `medianVelocity` | `Mediane` |
| Dispersion, means | `stdDev` | `EcartType` |
| Dispersion, medians | `iqr` (also `q1`, `q3`) | `EcartInterquartile` (`Q1`, `Q3`) |
| Vitesse requise | `requiredVelocity` | `Vreq` |
| z (y axis), means / medians | `zMean` / `zMedian` | `z_moyennes` / `z_medianes` |
| Risque, means / medians | `riskMean` / `riskMedian` | `Risque_moyennes` / `Risque_medianes` |
| Historique insuffisant (n) | `periods` | `NbPeriodes` |
| (tooltip / debug) | `readings`, `firstPeriod`, `lastPeriod`, `daysPerPeriod` | `NbMesures`, `PremierePeriode`, `DernierePeriode`, `JoursParPeriode` |
| Unit coefficient | `productsPerKg` | `ProduitsParKg` |
| (reference) | `netWeight`, `numberOfProducts` | `Net_Weight`, `Number_Of_Products` |
| Grid toggle | `week` / `month` object | `Maille` = `semaine` / `mois` |
| Scope (not shown) | — | `ClientID`, `ContratID` |

The UI computes nothing but placement (FR29), unit conversion (FR34), sort and
formatting. Every statistic, `z` and risk comes from `V_TRENDS`.

### 10.2 Write contract

`T_DDLOT`: `Statut` `created` → `closed`, `UpdatedBy` = caller, `UpdatedAt` set by
`ON UPDATE`. `trg_ddlot_bu` rejects a NULL `UpdatedBy` and an `UpdatedBy` from
another client (admin excepted).

### 10.3 Calculation as built vs Confluence (for validation by Gaël)

| # | Confluence text | As built | Kind |
|---|---|---|---|
| D1 | "Vitesse" (unit not stated) | kg consumed **per period** of the selected grid (per week or per calendar month). `Vreq` uses the same unit. | Precision |
| D2 | §8.2 "device accroché (`Hook_Status='Hooked'`)" | A lot spans several devices (about 20 pallets per lot in the Pharma set). Consumption is computed per device (deltas within one hook window, never across devices) and summed per period. The balance is the sum of the last weight of every device still hooked at SnapDate; an unhooked device's history still counts toward the velocities (EC12). | Deviation |
| D3 | §8.2 sentinels: MASSE NULL, position (0,0), RECORD_TIME 1999-11-30 excluded | MASSE NULL **and negative** excluded; RECORD_TIME handled by `FN_EVENT_DATE` (falls back to `timestamp_utc`); position (0,0) **not** filtered (position plays no part in mass). | Precision |
| D4 | — | Period containing the snapshot date excluded (incomplete). Monday's run therefore uses weeks up to the previous Sunday, and months up to the previous calendar month. | Addition |
| D5 | — | Periods before the first period with consumption excluded (transport, flat weight). Zero periods after it are kept. | Addition |
| D6 | — | Consumption between two readings in different periods counts in the later reading's period. A period with no reading is absent, not 0 (EC14). | Precision |
| D7 | "Écart type" | Sample standard deviation (`n − 1`); NULL with one period. | Precision |
| D8 | "Médiane, quartiles" | Linear interpolation at `h = (n − 1) p` = `PERCENTILE_CONT` = Excel `QUARTILE.INC`. | Precision |
| D9 | "Solde = dernière MASSE valide" | Last valid `MASSE` strictly before the snapshot date, within a hook window. No tare deduction (OQ6). | Precision |
| D10 | "Temps restant avant péremption" | Days from the **snapshot date**, not from today; up to 7 days older than "now" (consistent with the weekly refresh). | Deviation |
| D11 | `Vreq = quantité restante / temps restant` | `SoldeKg / (JoursRestants / JoursParPeriode)`, month = 365.25 / 12 = 30.4375 days. | Precision |
| D12 | Risk table and `z = (Vreq − Vref) / dispersion` | Risk evaluated on `Vreq` (FR13), so it holds when dispersion is 0; `z` is then NULL. With one period (std NULL) or zero dispersion, only high/low are possible. Boundaries: `Vreq = Vref` → low, `Vreq = Vref + disp` → medium (as in Confluence). | Precision |
| D13 | Risk "déjà perdu" not retained | Expired lot with stock (`JoursRestants <= 0`) → high, `z` NULL. Empty lot (`SoldeKg <= 0`) → low. | Precision |
| D14 | §3 "le client fournit le coefficient" | Coefficient derived from `T_DDLOT`: `Number_Of_Products / Net_Weight` (OQ3). | Deviation |
| D15 | — | A lot is assigned to the MAIN contract of its last reading (EC11). | Precision |
| D16 | — | A lot with no consumption yet has no snapshot and is not listed (EC7, OQ8). | Addition |
| D17 | §8.1 `sp_lot_conso_refresh()` | Plus `sp_lot_conso_refresh_at(date)` for backfill and tests; event every Monday 02:00 server time. | Addition |

## 11. Integration Points

- **MariaDB / MySQL `cpaltracker`:** reads `V_TRENDS`, `T_LOT_CONSO_SNAP` (max date),
  `T_CONTRATS` (via `cpal_contrat_autorise`), `T_DDLOT` (close); writes
  `T_DDLOT.Statut`. The event scheduler runs the refresh.
- **`cpaltracker.web` `/api/v1`:** two new files, `api/v1/contracts/trends.php` and
  `api/v1/lots/close.php` (new folder), using `_lib.php`, `auth.php`
  (`cpal_api_require_auth`, `cpal_contrat_autorise`, `cpal_is_admin`, `cpal_log`).
  `api/openapi.yaml` gains two paths. No existing file's behaviour changes.
- **`cpaltracker-react`:** new `api/trends.ts`, `routes/Trends.tsx`, `trends/*`
  (chart, table, logic), `ui/charts/scatter-chart.tsx` (+ showcase); `auth/paths.ts`,
  `App.tsx` and `chrome/AppShellLayout.tsx` gain one entry each.
- **Sibling features:** none at runtime. Pulse reads `V_PULSE.LotStatut`, which this
  feature's close action changes; that is display data only.

### 11.1 Reuse map

| Need | Reuse |
|---|---|
| Endpoint skeleton, CORS, auth, errors, id from path | `api/v1/contracts/history.php`, `api/v1/_lib.php` |
| Contract access, identical 404 | `auth.php` `cpal_contrat_autorise()` |
| Write endpoint shape, audit log | `api/v1/devices/transfer.php`, `cpal_log()` |
| Global contract and 404 handling | `contracts/ContractContext.tsx` (`useContracts`, `rejectContract`, `CONTRACT_ACCESS_ERROR`) |
| Page structure | `routes/HistoryDashboard.tsx`, `ui/patterns/page-header.tsx` |
| Chart theme, Plotly wrapper | `ui/charts/plot.tsx`, `ui/charts/use-chart-theme.ts` (new `ScatterChart` built on them) |
| Table, row menu, sort | `ui/data-table.tsx` |
| Toggles | `ui/toggle-group.tsx` |
| Risk badge | `ui/badge.tsx` (with lucide icon) |
| Confirmation, toast | `ui/alert-dialog.tsx`, `ui/sonner.tsx` |
| States | `ui/skeleton.tsx`, `ui/empty.tsx`, `ui/alert.tsx` |
| Method note | `ui/popover.tsx` |
| Navigation | `chrome/AppShellLayout.tsx` `SECTIONS`, `auth/paths.ts` |
| HTTP client | `api/client.ts` |

## 11a. Software Items

Not applicable (see [SDLC, product templates](../sdlc/README.md#product-templates)).

## 12. Edge Cases & Constraints

- **EC1. No snapshot at all** (fresh install before the first run, or table empty):
  `latestRefreshDate` null; the page shows the "no snapshot yet" state (FR38).
- **EC2. One retained period** (`NbPeriodes = 1`): `EcartType` NULL, `Q1 = Q3` so
  `iqr = 0`; both `z` NULL; risk high or low only (FR13). Not placed (below
  `minPeriods`, and `z` NULL).
- **EC3. Dispersion 0** (all periods equal, `n >= 2`): `z` NULL for that method; risk
  high if `Vreq > Vref`, else low. Row badge "Dispersion nulle"; not placed.
- **EC4. Expired lot with stock** (`JoursRestants <= 0`, `SoldeKg > 0`): `Vreq` and `z`
  NULL, risk high. Row badge "Lot périmé", risk shown; not placed (OQ7).
- **EC5. Expiry on the snapshot date** (`JoursRestants = 0`): treated as expired (EC4).
- **EC6. Empty lot** (`SoldeKg <= 0`): risk low (even if expired); `Vreq = 0` when
  `JoursRestants > 0`, so `z <= 0` and the point sits in the low band. A natural
  candidate for closing.
- **EC7. Lot with no readings, or no consumption yet** (in transport): no snapshot row,
  absent from `V_TRENDS` and from the page (OQ8).
- **EC8. `Net_Weight = 0`:** `ProduitsParKg` NULL; product-mode cells "—" (FR34).
- **EC9. `Number_Of_Products = 0`** (column default): coefficient 0; treated as
  unavailable like EC8 (FR34).
- **EC10. STD contract selected:** 200 with no lots; STD empty state (FR16, FR38).
- **EC11. Lot hooked under two MAIN contracts over time:** its history spans both;
  it is listed under the contract of its last reading only.
- **EC12. Hook ended, lot still `created`:** balance frozen at the last hooked
  reading; the lot keeps being evaluated (its trailing zero periods lower its
  velocity, its risk rises) until the user closes it.
- **EC13. Weight increase** (reload, artefact): contributes 0 to consumption; the
  balance follows the new, higher reading.
- **EC14. Device silent for a whole period:** that period is absent (not 0); the drop
  over the gap is counted in the next period with a reading, which can inflate it.
- **EC15. Lot in the weekly grid but not the monthly one** (consumption started in the
  current month): `month` is null; row shows "Aucune période complète" in monthly view
  (FR35).
- **EC16. Mixed snapshot dates** (a lot's latest row predates the latest run): header
  shows the latest date; the row carries its own date (FR36).
- **EC17. Lot closed concurrently** by another user or tab: 409, message, refetch
  (FR22, FR37).
- **EC18. Lot closed after the snapshot:** absent on the next read (FR24).
- **EC19. Extreme `z`** (very small dispersion): the y axis autoscale includes the data
  and always the 0 and 1 lines (FR28).
- **EC20. Future-dated demo data:** `sp_lot_conso_refresh()` today (2026-11) sees only
  the first weeks of the set; tests use `sp_lot_conso_refresh_at` (NFR10).
- **EC21. Enough weekly periods but too few monthly ones:** placed in weekly view, not
  in monthly view (FR29 is per grid).
- **EC22. Admin user:** sees the lots of the selected contract's owner and may close
  them (FR16, FR21).
- **Constraint.** `Net_Weight <= Gross_Weight`, both 0–30,000 kg; `Number_Of_Products
  <= 99,999`; `cPAL_Lot` is `char(22)`; `T_DDLOT` rows cannot be deleted.

## 13. Metrics / KPIs

- Refresh duration on the Pharma demo set: about 60 s locally for one snapshot, measured 2026-11-19 (NFR1 met).
- Trends API p95 < 1 s and page render < 2 s with 100 lots (NFR3, NFR4).
- 100 % of FR3–FR14 rules covered by SQL fixtures with hand-computed values, and of
  FR/NFR by at least one passing test (NFR10).
- For Gaël's validation: on the Pharma demo set, the risk level of 10 lots picked by
  Gaël matches his own spreadsheet computation (same snapshot date) for both methods.
- In use: number of lots closed per month (from `T_AUTH_LOG` `lot_cloture`), and share
  of open lots with a frozen balance (EC12) — a rising share means closing is
  forgotten.
- 0 regressions on existing pages and endpoints (NFR7).

## 14. Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Users read the risk as "today" while it is as of the snapshot (up to 7 days old) | "Instantané du …" in the header, hint text, row badge for older rows (FR27, FR36) |
| Balance includes tare (pallet, packaging, device) and overstates `Vreq` and the risk | Confirm what `MASSE` measures (OQ6); deltas (velocities) are unaffected |
| Short histories give noisy `z` | Minimum history before placing a lot (OQ1, FR29) |
| A silent device inflates one period (EC14) | Medians method available; shown in the method note |
| Lots never closed accumulate with frozen balances (EC12) | Close action on the page; KPI on frozen-balance share; future auto-suggestion (§16) |
| Close by mistake, no reopen in app | Confirmation dialog naming the lot; audit trail; reopen decision (OQ4) |
| Refresh too slow on production volume | Reads `T_DATA` directly with an index prefilter (NFR1); measured on the 1.4 M-row set |
| Event scheduler off on the VPS: no new snapshot | NFR9 rollout check; header date makes staleness visible |
| MySQL / MariaDB differences | Window-function quantiles, no engine-specific functions (NFR2), tested on both before rollout |
| Adding `T_DDLOT.Statut` changes an existing response | Existing reads select explicit columns (checked: `history_logic.php`); NFR7 regression suites |

## 15. Acceptance Criteria (Definition of Done)

- [ ] AC1. `T_DDLOT.Statut` exists with default `created`; closed lots are absent from the refresh and from `V_TRENDS`. *(FR1)*
- [ ] AC2. On SQL fixtures, `sp_lot_conso_refresh_at` produces the hand-computed velocities, statistics, balance and contract for both grids, including excluded current and leading periods and ignored increases. *(FR2–FR7, EC13, EC14)*
- [ ] AC3. Re-running a snapshot date replaces its rows, keeps older ones and gives identical results. *(FR8, NFR11)*
- [ ] AC4. The weekly event exists (Monday 02:00) and `sp_lot_conso_refresh_at(date)` backfills any date. *(FR9)*
- [ ] AC5. `V_TRENDS` gives the expected `JoursRestants`, `Vreq`, `z` and risk for both methods on fixtures covering normal, `n = 1`, zero dispersion, expired, expiry-day, empty lots and `Net_Weight = 0`. *(FR10–FR14, EC2–EC6, EC8)*
- [ ] AC6. Refresh on the Pharma demo set completes in under 5 min locally; time recorded. *(NFR1)*
- [ ] AC7. All SQL runs unchanged on MySQL 8.0 and MariaDB 10.11. *(NFR2)*
- [ ] AC8. Trends endpoint returns 200 with the FR17 shape for an accessible MAIN contract, 400/404/405/503 as specified, the same 404 for missing and not-yours contracts, and an empty list for a STD contract. *(FR15–FR18, EC10, EC22, NFR5)*
- [ ] AC9. Trends endpoint answers in under 1 s with 100 lots, payload under 200 KB. *(NFR3)*
- [ ] AC10. Close endpoint closes an owned lot (200, `UpdatedBy` set, audit line written), returns the same 404 for missing and other-client lots, 409 for an already closed lot, 400/405/503 as specified. *(FR20–FR23, EC17, NFR6)*
- [ ] AC11. A closed lot disappears from the trends response at once; snapshots and hooks are unchanged. *(FR24, EC18)*
- [ ] AC12. Both endpoints are documented in `openapi.yaml`. *(FR19)*
- [ ] AC13. "Trends" appears in the sidebar after "Pilotage" and opens `/trends`; other entries and routes are unchanged. *(FR25, FR39)*
- [ ] AC14. The page follows the global contract and handles a 404 like History. *(FR26)*
- [ ] AC15. Header shows the snapshot date and the three toggles with their defaults; toggles update chart and table without a network call, within 300 ms on 100 lots. *(FR27, NFR4)*
- [ ] AC16. The chart places eligible lots by days to expiry and `z`, with bands, lines at 0 and 1, the 30-day split, risk colour and shape, legend, accessible label; non-placed lots are counted by reason. *(FR28, FR29, EC19, EC21)*
- [ ] AC17. Point tooltip shows the lot's fields; clicking a point highlights and scrolls to its row. *(FR30)*
- [ ] AC18. The table shows the ten columns for the selected method, grid and unit, fr-FR formats, "—" for nulls, default sort by risk then days. *(FR31, FR33)*
- [ ] AC19. Risk badges show icon and text; non-placed rows show their reason; below the minimum history `z` and risk are hidden except expired and empty lots. *(FR32, EC2–EC4, EC6)*
- [ ] AC20. In products mode the four kg columns are converted; `z`, risk and days unchanged; null or zero coefficient shows "—" with its tooltip. *(FR34, EC8, EC9)*
- [ ] AC21. A lot without data for the selected grid and a row with an older snapshot show their badges. *(FR35, FR36, EC15, EC16)*
- [ ] AC22. Closing from the row menu asks for confirmation, then removes the lot; 409, 404 and network errors show their messages. *(FR37)*
- [ ] AC23. Loading, error, no-contract, STD, no-snapshot and no-lot states display as specified. *(FR38, EC1, EC7, EC10)*
- [ ] AC24. The method note popover explains the calculation in French. *(FR40)*
- [ ] AC25. `ScatterChart` is in the kit and its showcase; the page uses kit components and tokens only, passes lint, and works in dark and light at 1440 and 390 px with keyboard access. *(FR39, NFR8)*
- [ ] AC26. Existing unit and E2E suites pass unchanged; no existing API response changed. *(NFR7)*
- [ ] AC27. Production script applies the objects, grants and event scheduler; a first snapshot exists after install. *(NFR9)*
- [ ] AC28. SQL fixtures, PHP endpoint tests, Vitest and Playwright E2E for US1–US5 exist and pass; tests use `sp_lot_conso_refresh_at`. *(NFR10, EC20)*
- [ ] AC29. All FRs and NFRs validated; Gaël has validated §10.3 and the open questions' defaults.

## 16. Future Extensions (Optional)

- Trend of a lot's `z` over past snapshots (the data is already kept in
  `T_LOT_CONSO_SNAP`).
- Reopen a closed lot, with audit (OQ4).
- Coefficient entered by the client per lot or per product (OQ3).
- Suggest closing empty lots or lots with a frozen balance.
- Listing lots in transport (no consumption yet) with their shelf life.
- On-demand refresh, or a daily refresh.
- Notifications when a lot enters the high band; CSV export (deferred by Confluence §3).
- Filter by product or by lot number shared with Pulse.

## 17. Open Questions

Each has a proposed default, applied unless Gaël decides otherwise.

| ID | Question (source) | Proposed default |
|---|---|---|
| OQ1 | Minimum history to place a lot (Confluence §8.5) | `minPeriods = 4` retained periods per grid. Below it the lot is not plotted and its row says "Historique insuffisant (n période(s))" (FR29, FR32). |
| OQ2 | Remaining shelf life that splits the quadrants (§8.5) | 30 days (`quadrantSplitDays`), drawn as a vertical line. |
| OQ3 | Coefficient kg ↔ products derived or entered by the client (§3, §8.5) | Derived, as built: `Number_Of_Products / Net_Weight`. Client entry is a future extension. |
| OQ4 | Can a closed lot be reopened? | Not from the app in this version; an admin can reset `Statut` in the database (with `UpdatedBy`). |
| OQ5 | Who may close a lot (§8.5 asks about a `manager` role for closing anomalies) | Any active user of the lot's client, and admins, as `trg_ddlot_bu` already allows. `PeutCloturerAnomalies` is not used for lots. |
| OQ6 | Is `MASSE` net of tare (pallet, packaging, device)? If not, the balance and `Vreq` are overstated | Use `MASSE` as measured (as built). If it includes tare, a later change subtracts `Gross_Weight − Net_Weight` or a device tare. |
| OQ7 | How to show lots that cannot be placed (expired with stock, `z` NULL) | Not plotted; listed in the table with their risk and reason, and counted under the chart. |
| OQ8 | Should lots with no consumption yet be listed? | No, in this version (no snapshot row). |
| OQ9 | Month length in `Vreq` (365.25 / 12 days) acceptable vs the actual calendar month | Keep 30.4375 days. |
| OQ10 | Navigation label | "Trends", as in Confluence §5. |
| OQ11 | Default method and grid | Moyennes, Hebdomadaire. |

---

## Requirement inventory

Every item below must be covered by at least one story. Text lives above; this is IDs
only.

| Kind | IDs | Count |
|---|---|---|
| Functional requirements | FR1–FR40 | 40 |
| Non-functional requirements | NFR1–NFR11 | 11 |
| Acceptance criteria | AC1–AC29 | 29 |
| Edge cases | EC1–EC22 | 22 |
| Calculation deviations to validate | D1–D17 | 17 |
| Open questions | OQ1–OQ11 | 11 |

Behavior tracking: none. c-PAL has no analytics or funnel instrumentation and its
SDLC has no behavior-tracking step; the close action's audit-log line (FR23) is the
only usage signal (KPIs in §13).

## Proposed feature areas

> v1 scope was trimmed on 2026-11-19: see "Scope v1" at the top. Only KAN-256 is kept (FR15–FR17, FR25, FR26, FR28–FR33, FR38, FR39); areas A1–A9 below are the original split, the rest is deferred to v2.

| # | Area | Requirements | Notes |
|---|---|---|---|
| A1 | Data model and weekly refresh (DB) | FR1–FR14, NFR1, NFR2, NFR11, NFR9, EC2–EC9, EC11–EC14, EC20, D1–D17; AC1–AC7, AC27 | Objects are built locally: this story writes the SQL fixtures, measures NFR1, checks MariaDB, prepares the VPS script, and gets §10.3 validated by Gaël. |
| A2 | Backend API — trends read | FR15–FR18, FR19 (trends path), NFR3, NFR5, EC10, EC22; AC8, AC9, AC12 | `api/v1/contracts/trends.php` + PHP tests. |
| A3 | Backend API — close lot | FR20–FR24, FR19 (close path), NFR5, NFR6, EC17, EC18; AC10–AC12 | `api/v1/lots/close.php` + PHP tests. |
| A4 | Kit — ScatterChart | FR28 (component part), FR39, NFR8; AC25 | Kit and showcase first (UI rule 5). |
| A5 | Navigation, page shell and states | FR25–FR27, FR38–FR40, NFR4 (toggles), NFR7, NFR8, EC1, EC7, EC10; AC13–AC15, AC23, AC24, AC26 | Route, nav entry, header, toggles, method note, states. |
| A6 | Quadrant chart | FR28–FR30, NFR4, EC19, EC21; AC16, AC17 | Placement rules and interaction. |
| A7 | Lot detail table and units | FR31–FR36, EC2–EC4, EC6, EC8, EC9, EC15, EC16; AC18–AC21 | Columns, reasons, sort, conversion. |
| A8 | Close-lot UI | FR37, EC17; AC22 | Row menu, dialog, outcomes. Depends on A3. |
| A9 | End-to-end tests and test data | NFR10, EC20; AC28, AC29 | Backfilled snapshot on the Pharma set via `sp_lot_conso_refresh_at`; Playwright US1–US5. |

Dependencies: A1 → A2, A3 → A5–A8; A4 → A6; A9 last.

## Notes from the local DB build (2026-11-19)

- `sp_lot_conso_refresh` reads `T_DATA` directly. Going through `V_DDLOT_DATA` ran for more than 10 min and was abandoned.
- On the Pharma set, a snapshot at 2028-06-01 gives 34 lots per time grid. Weekly velocities average 521 kg/week against an average balance of 4,834 kg. Weekly risk: 24 low, 2 medium, 8 high. Monthly risk: 22 low, 2 medium, 10 high.
- The interquartile range is often 0 on lumpy consumption, so `z_medianes` is NULL or extreme (seen: 548 and −1,227). The chart SHALL clamp the displayed z (for example to ±5) and show "dispersion nulle" when the IQR is 0. Risk is computed on Vreq against Vref ± dispersion, so it is unaffected.
