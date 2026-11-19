# Feature: Pulse

> Feature spec (W1, [`FEATURE.md`](../engineering/product/FEATURE.md) template) for the
> **Pulse** page of the React app (`cpaltracker-react`). Epic: KAN-270 (stories KAN-272 to KAN-281), see [`KAN-270-Pulse-Live-Contract-Dashboard.md`](../epics/KAN-270-Pulse-Live-Contract-Dashboard.md).
>
> Sources:
> - Confluence COC « 2026-11-18 - dashboard » §6 (Onglet Pulse), §3 (arbitrages), §8.3–8.4
>   (rights, performance) —
>   https://c-pal.atlassian.net/wiki/spaces/COC/pages/115605505/2026-11-18+-+dashboard
>   (owner: Gaël COQUIO). Mockup (simulated data, layout reference only):
>   https://claude.ai/artifact/S21fVkz7x9471AA4pAgRU8
> - Owner decisions of 2026-11-19 (Felipe), which **override** the Confluence text where
>   they differ: Pulse is a new page, the work is strictly additive, anomalies are a drawer,
>   no profile-menu work, weight colour keeps the existing thresholds.
> - DB objects already built: `cpaltracker.web/docker/mysql/init/03-dashboard.sql`
>   (`V_PULSE`, `V_ANOMALIES_OUVERTES`, `T_DDLOT.Statut`).
>
> Sibling specs: [`BACKLOG-Anomaly-Workflow.md`](BACKLOG-Anomaly-Workflow.md) (the anomaly
> drawer and its read/write endpoints) and [`BACKLOG-Trends.md`](BACKLOG-Trends.md).
> Precedents this spec builds on, cited by ID and never restated:
> [`React-Live-Tracking.md`](React-Live-Tracking.md) (LT-FRn),
> [`React-History-Dashboard.md`](React-History-Dashboard.md) (HD-FRn),
> [`Compact-measurement-charts.md`](Compact-measurement-charts.md) (CMC-FRn, KAN-250/251/252).
>
> Single source of truth: requirement text (FR/NFR/AC/EC IDs, KPIs) lives only here.

---

## 1. Feature Name

**Pulse**: a contract-scoped page that shows the last known position of every device on a
map, lets the user replay a chosen period with a play/pause cursor shared by the map and
four stacked measure charts (weight, temperature, humidity, acceleration), focuses the view
on a customer lot or a set of devices, and hosts the entry point to the anomaly drawer.

## 2. Problem / Pain Point

Two client populations use c-PAL data (Confluence §2): **Quality** (transport conditions,
complaint investigation) and **Planning** (logistics follow-up, lot flow). Today the React
app splits what they need across pages that answer different questions:

- **Suivi en direct** (`/live-tracking`) shows where devices are now, coloured by weight,
  but has no time dimension and no lot information in the tooltip (positions come from
  `GET /devices/positions.php`, which carries no lot).
- **Historique** (`/history`) replays a period, but shows three compact tiles (temperature,
  humidity, mass) that open a modal; acceleration (shocks) is not charted, the charts are not
  visible next to the moving map, and it always loads a series on arrival.
- Neither page lets the user start from **the customer's own lot number**
  (`T_DDLOT.Cust_Lot_Number`), which is how a quality manager receives a complaint
  ("lot 24-117 arrived warm") and how a planner thinks about stock.
- The anomalies computed since V5 (`T_ANOMALIE`, trigger in production since 2026-09-25)
  have no web entry point at all.

So a quality manager investigating a complaint has to guess which device carried the lot,
switch pages, re-pick dates, and still cannot see the shock curve aligned with the route.

## 3. Goal / Outcome (Success Definition)

A logged-in user opens **Pulse**, and for the contract selected in the sidebar:

1. immediately sees every device's last known position, coloured by remaining weight, with
   the lot it carries (customer lot number, product) in the tooltip;
2. picks two dates, presses play, and watches the devices move while a cursor sweeps the
   weight, temperature, humidity and acceleration charts beside the map in step;
3. types a customer lot number and gets the map, the dates and the charts focused on that
   lot's device and hook period, without knowing the device ID;
4. sees at a glance whether the contract has new (red) or in-progress (orange) anomalies,
   and opens the anomaly drawer from there.

Suivi en direct, Historique and Pilotage behave exactly as before, and their API responses
are unchanged.

## 4. User Stories (Primary Scenarios)

- **US1. Fleet at a glance.** As a planner, I want to open Pulse and see where every device
  of my contract last reported, coloured by remaining weight, with the lot each one carries,
  so that I know which stock is where without picking dates.
  *(FR1–FR13, NFR2, NFR3)*
- **US2. Replay a period.** As a quality manager, I want to choose two dates and play the
  period, with one cursor moving across the map and the four charts, so that I can see where
  a device was when the temperature rose or a shock occurred.
  *(FR14–FR22, FR31, FR33, NFR4)*
- **US3. Start from a customer lot.** As a quality manager handling a complaint, I want to
  select the customer lot number and get its device, its hook period and its measures, so
  that I can investigate the lot without knowing the device.
  *(FR7, FR23–FR27, FR32)*
- **US4. Spot and open anomalies.** As a quality manager, I want a red/orange light for the
  contract and an anomaly count per device, and to open the anomaly drawer from them, so that
  I handle new excursions from the page where I see the route.
  *(FR28–FR30)*
- **US5. Nothing else moves.** As the product owner, I want Pulse added without changing
  Suivi en direct, Historique, Pilotage or any existing API response, so that users of
  those pages are not affected. *(NFR1, NFR9)*

## 5. In Scope (What WILL be delivered)

- A new authenticated route `/pulse`, titled "Pulse", with a new sidebar entry (FR1, FR2).
- Contract scope from the existing global contract selector (`ContractProvider`) (FR3).
- A new read endpoint returning, for one authorized contract, the `V_PULSE` snapshot (last
  known position, readings, remaining weight, hooked lot) and the contract's lot hooks for
  the lot filter (FR4–FR8).
- A snapshot map: markers coloured by the existing weight thresholds, lot-aware tooltip,
  manual refresh, last-updated time, fit (FR9–FR13).
- A two-date bar with play/pause and a cursor shared by the map and four stacked charts,
  fed by the existing History endpoint, fetched only once a range is chosen (FR14–FR22).
- Two opt-in parameters on the History endpoint: an acceleration (`g`) summary and a device
  subset; without them the response is unchanged (FR31, FR32).
- A customer-lot filter and a device filter (FR23–FR27).
- The anomaly entry point: contract alert light and per-device anomaly count, opening the
  drawer specified in `BACKLOG-Anomaly-Workflow.md` (FR28–FR30).
- PHPUnit, Vitest and Playwright coverage; OpenAPI update (NFR9, NFR11).

## 6. Out of Scope (What WILL NOT be delivered)

- **Any change to existing pages or responses**: Suivi en direct, Historique, Pilotage,
  account and auth pages, and the current responses of `/devices/positions.php` and
  `/contracts/history.php` (without the new opt-in parameters) stay as they are. No nav
  entry is removed or renamed.
- **Redirecting `/` to Pulse** (Confluence "onglet par défaut: Pulse"): not done here, see
  open question OQ1.
- **Several contracts at once** (Confluence: "devices des contrats actifs du client"): Pulse
  shows the one contract chosen in the sidebar. See §16 and OQ2.
- **The anomaly drawer itself** (detail, map, status change, comment, PDF): specified in
  `BACKLOG-Anomaly-Workflow.md`. Pulse only opens it.
- **Profile menu work**: messages list and badge, "Mes contrats", `T_MESSAGE_LU`
  (owner decision 2026-11-19 §C4).
- **Outgoing notifications** (mail, SMS, webhook), **geofencing, dwell time, ETA**, **CSV/API
  export**, **PDF** (Confluence §3, postponed).
- **Trends** (quadrants, consumption velocity): `BACKLOG-Trends.md`.
- **Realtime push or auto-polling**: refresh is manual (2–3 readings per device per day,
  Confluence §3). No socket transport (LT-NFR10 applies).
- **Closing a lot** (`T_DDLOT.Statut = 'closed'`) from Pulse: belongs to Trends.
- **Display differentiation per profile** (Confluence §2: one dashboard for all users).
- **Mobile-first layout**: desktop first, mobile best effort (same rule as LT-NFR4).

## 7. User Flow / UX Breakdown

1. User clicks **Pulse** in the sidebar. `RequireAuth` lets an authenticated user through;
   otherwise they go to login (FR1).
2. System uses the contract selected in the sidebar (or the remembered/first one) and calls
   the Pulse endpoint for it (FR3, FR4). No series request is made (FR15).
3. System shows the map with one marker per positioned device, coloured by remaining weight,
   auto-fitted, plus the "last updated" time and a Refresh button (FR9–FR13). The four chart
   panels show "Choisissez une période pour afficher les courbes." (FR15). The contract
   alert light shows red, orange or off, with counts (FR28).
4. User hovers a marker: tooltip with readings and, when hooked and owned, customer lot
   number, product, cPAL lot, expiry date (FR11), plus the device's open anomaly count (FR29).
5. User picks a start and an end date in the bar. System validates them (FR14); on a valid
   range it requests the series for the contract (FR16) and shows the sampling banner if the
   response was downsampled (FR19).
6. System draws the four charts (weight, temperature, humidity, acceleration), one series per
   visible device, each with its summary header (FR20–FR22). The cursor sits at the end of
   the range; markers show each device's last reading at or before the cursor (FR17).
7. User presses **play**: the cursor advances from the current instant to the end, moving
   markers (with trail) and the vertical cursor line on all four charts together; **pause**
   holds it; dragging the slider moves everything (FR17).
8. User opens the **lot** filter and selects a customer lot number. System restricts the
   devices to the lot's device, sets the range to the hook period and loads the series for
   that device; rows outside the hook period are not shown (FR24).
9. User changes the **device** filter. In snapshot mode the map filters locally; in range
   mode the series is re-requested for the selected devices so chart headers stay exact
   (FR23).
10. User clicks the contract alert light or a device's anomaly count: the anomaly drawer
    opens, scoped to the contract, or to the device and the selected period (FR28, FR29).
11. User clears the dates: Pulse returns to snapshot mode without a new request (FR18).
    User clicks **Refresh**: the snapshot and the alert light are reloaded (FR12, FR30).

## 8. Functional Requirements (FRs)

### Navigation and page

- **FR1.** The app SHALL expose a new route `/pulse` (constant `PULSE_PATH` in
  `src/auth/paths.ts`), rendered inside `AppShellLayout` behind `RequireAuth`, like the
  existing protected routes. `DocumentTitle` SHALL title it "Pulse".
- **FR2.** The sidebar SHALL gain a "Pulse" entry (`data-testid="nav-pulse"`) in the
  `Navigation` section of `AppShellLayout`. The existing entries (Accueil, Suivi en direct,
  Historique, Pilotage) SHALL keep their labels, paths, icons, test IDs and relative order.
  Pulse is placed directly after Accueil unless OQ3 decides otherwise.
- **FR3.** Pulse SHALL take its contract from `useContracts()` (`ContractProvider`): no
  page-level contract picker. It SHALL show the same states as Live Tracking for loading,
  contract-list error, zero contracts and no selection, and on a 404 from any Pulse request
  for the selected contract it SHALL call `rejectContract()` and show the provider's generic
  `accessError` (same non-disclosure behaviour as LT-FR15 and HD-FR25).

### Backend: Pulse snapshot endpoint

- **FR4.** A new endpoint `GET /api/v1/contracts/pulse.php/{contractId}` (path resolved with
  `cpal_api_resolve_id_from_path_or_query()`, query fallback `?contractId=`) SHALL follow the
  conventions of `contracts/history.php`: CORS/preflight, GET only (405), bearer auth via
  `cpal_api_require_auth()` (403 for `MustChangePassword`), positive-integer id (400
  `invalid_contract_id`), authorization by `cpal_contrat_autorise()` with one identical 404
  for "does not exist" and "not yours", 503 `service_unavailable` on `PDOException`, error
  envelope `{"error":{"code","message"}}`.
- **FR5.** The endpoint SHALL return `positions`: one entry per row of `V_PULSE` for that
  `ContratID`, shaped as `cpal_tracking_position_vers_api()` does (`contractId`, `chipId`,
  `timestamp` ISO 8601 UTC from `EventDate`, `latitude`, `longitude`, `ambientTemperature`,
  `humidity`, `mass`, `mpuTemperature`, `accelG`) plus `remainingWeight` (`PoidsRestant`)
  and `lot`, which is either `null` or `{ cpalLot, custLotNumber, productName, netWeight,
  grossWeight, numberOfProducts, expiryDate, status }`.
- **FR6.** Lot data SHALL reach the client only through the `V_PULSE` ownership rule (lot
  shown only when `T_DDLOT.ClientID` equals the row's `ResponsibleClientID`) and, for FR7,
  the same rule applied to hooks. The endpoint SHALL NOT read lot data through any other
  path.
- **FR7.** The endpoint SHALL return `lots`: the lots hooked under this contract
  (`T_DDLOT_DEVICE_HIST.ContratID`) whose `T_DDLOT.ClientID` is the contract's client, each
  as `{ cpalLot, custLotNumber, productName, status, hooks: [{ chipId, from, to }] }`, with
  `from`/`to` as ISO 8601 UTC and `to = null` for an ongoing hook (sentinel
  `9999-12-31`). Lots of both statuses (`created`, `closed`) are returned.
- **FR8.** The endpoint SHALL return `contract: { contractId, active }`, `active` being true
  when the contract's period contains the server's current time (the `V_PULSE` rule; see
  EC3 for open-ended periods), and `generatedAt` (server time, ISO 8601 UTC). For an
  inactive contract it SHALL return 200 with `positions: []` and `active: false`.

### Snapshot map

- **FR9.** On arrival and on contract change, Pulse SHALL request only the FR4 endpoint and
  render one marker per entry with a valid position; it SHALL NOT request a series until a
  range is chosen (Confluence §8.4).
- **FR10.** Marker colour SHALL use the existing weight thresholds of LT-FR5 / HD-FR11
  (< 200 kg red, < 500 kg orange, ≥ 500 kg green, null or NaN grey) applied to
  `remainingWeight` in snapshot mode and to the row's `m` in range mode, by calling the
  existing `massStatus()` function. Pulse SHALL NOT define its own thresholds or a
  percentage ramp.
- **FR11.** The marker tooltip SHALL show the device ID, the reading date (UTC), position,
  ambient temperature, humidity, remaining weight, acceleration and MPU temperature, and,
  when `lot` is present, the customer lot number, product name, cPAL lot and expiry date.
  All values SHALL be rendered as text, never as HTML. In range mode the lot shown is the
  hook active at the cursor instant (HD-FR13 rule, from the History `hooks`).
- **FR12.** Pulse SHALL provide a manual **Refresh** control and a "last updated" time, with
  the behaviour of LT-FR9/LT-FR10: refresh re-requests the FR4 endpoint and the contract
  alert light (FR30), updates the time only on success, keeps the previous snapshot on
  failure with an inline error, and never polls.
- **FR13.** The map SHALL auto-fit to the displayed markers once after each load and offer a
  "fit" control (HD-FR15/HD-FR16 behaviour).

### Timeline bar and shared cursor

- **FR14.** A horizontal bar SHALL let the user pick a start and an end date (day
  granularity, UTC) and hold the play/pause button and the time slider. The range SHALL be
  validated with the existing `validateDateRangeInputs()` rules (HD-FR5 to HD-FR8: reversed,
  malformed, over 1 year rejected inline, no request while invalid).
- **FR15.** With no range chosen (**snapshot mode**), the slider and play/pause SHALL be
  disabled and each chart panel SHALL show an invitation to choose a period. No default
  range SHALL be applied on arrival.
- **FR16.** When a valid range is chosen (**range mode**), Pulse SHALL make one request to
  `GET /contracts/history.php/{contractId}` with `start`, `end`, `measures=m,ta,hr,g`
  (FR31) and, when a device subset or lot is active, `chipIds` (FR32), and SHALL use its
  `rows`, `hooks`, `devices`, `summaries` and sampling fields for the map, the charts and
  the tooltip. A response arriving for a contract, range or device set that is no longer
  current SHALL be discarded.
- **FR17.** In range mode one instant SHALL drive both the map and the charts:
  - the map shows each visible device's last positioned reading at or before the instant
    (HD-FR12), with the optional trail (HD-FR14);
  - each of the four charts draws a vertical cursor at the instant;
  - play advances the instant to the end of the range and stops there (HD-FR18), pause holds
    it, dragging the slider sets it; play from the end restarts at the start;
  - after a load, the instant is the end of the range and the cursor is shown there.
  Moving the instant SHALL NOT trigger any request.
- **FR18.** Clearing the range SHALL return to snapshot mode, showing the last FR4 snapshot
  again without re-requesting it.
- **FR19.** When the History response has `sampled: true`, Pulse SHALL show the existing
  sampling banner (HD-FR22) with `count` and `total`.

### Stacked charts

- **FR20.** Next to the map, Pulse SHALL show four stacked charts in this order: weight
  (`m`, kg), temperature (`ta`, °C), humidity (`hr`, %), acceleration (`g`, g). Each SHALL
  plot one series per visible device from `rows` (nulls and NaN skipped, HD-FR19 rules),
  share the device colour order with the map (`visibleDeviceOrder`), and draw the
  temperature threshold band the History modal draws.
- **FR21.** Each chart SHALL have a header from the backend `summaries[key]`: label, unit,
  latest value, min and max, and the excursion badge when the measure has thresholds,
  formatted with the existing `measureFormatting` helpers. The front end SHALL NOT compute
  any statistic (CMC-FR15).
- **FR22.** A measure with no reading in the range (`summaries[key] === null` or no non-null
  point) SHALL show "Aucune mesure sur cette période" in its panel; a range with zero rows
  SHALL show one page-level "Aucune mesure sur cette période" message and no markers.

### Filters

- **FR23.** A **device filter** SHALL list the contract's devices (snapshot `positions`
  plus, in range mode, History `devices`), all selected by default, and show the selection
  as a removable chip (pattern of `HistoryReadyPanel`). In snapshot mode a change SHALL
  filter markers locally with no request. In range mode a change SHALL re-request the series
  with `chipIds` set to the selection (debounced, at least 300 ms), so the chart headers
  describe exactly the displayed devices. An empty selection SHALL show a "no device
  selected" message and make no request.
- **FR24.** A **lot filter** SHALL offer a searchable list of the `custLotNumber` values from
  FR7 (with product name; `closed` lots marked). Selecting a lot SHALL:
  set the device selection to the lot's hooked device(s);
  set the range from the first hook's start date to the last hook's end date (today, UTC,
  for an ongoing hook), clipped to the most recent year when longer (EC10, with an inline
  notice);
  load the series as in FR16;
  and hide from map and charts any row outside the lot's hook windows (half-open
  `[from, to)`, the HD-FR13 hook rule).
- **FR25.** Clearing the lot filter SHALL restore "all devices", keep the current range, and
  re-request the series for the whole contract.
- **FR26.** Lot and device filters SHALL combine: while a lot is selected, the device filter
  only offers the lot's device(s).
- **FR27.** A lot whose device has no positioned reading in the range SHALL still show its
  charts, with a message that no position is available for the map.

### Anomaly entry point

- **FR28.** Pulse SHALL show a **contract alert light** from `V_ANOMALIES_OUVERTES` for the
  selected contract: red when `NbDetectees > 0`, orange when only `NbEnCours > 0`, off
  otherwise, always with a text label and the counts (detected / in progress / awaiting
  closure). Activating it SHALL open the anomaly drawer of `BACKLOG-Anomaly-Workflow.md`
  scoped to the contract. The read endpoint and the drawer are owned by that spec.
- **FR29.** Each device SHALL show its anomaly count (in the tooltip and the device filter
  list): in snapshot mode the count of its non-closed anomalies; in range mode the count of
  its anomalies whose `EventDate` falls in the range. Activating the count SHALL open the
  drawer filtered on that device (and on the range in range mode).
- **FR30.** The alert light and the counts SHALL be reloaded on Refresh (FR12), on range
  change, and when the drawer reports a status change.

### History endpoint, opt-in additions

- **FR31.** `GET /contracts/history.php/{contractId}` SHALL accept an optional `measures`
  parameter, a comma-separated subset of `ta,hr,m,g`. Without it, the response SHALL be
  identical to today's (`summaries` with exactly `ta`, `hr`, `m`). With it, `summaries`
  SHALL hold exactly the listed keys; `g` SHALL be computed by a new acceleration entry
  of the enrichment policy (label "Accélération", unit "g", 2 decimals, no thresholds,
  every enricher except `ThresholdExcursion`), applied only when requested. An unknown or
  malformed value SHALL return 400 `invalid_query`. `rows` are unchanged (they already
  carry `g`).
- **FR32.** The same endpoint SHALL accept an optional `chipIds` parameter (comma-separated
  ChipIDs, at most 500). Without it, the response SHALL be identical to today's. With it,
  `devices`, `rows`, `hooks`, `summaries`, `count`, `total` and `sampled` SHALL cover only
  the intersection of `chipIds` with the contract's devices. ChipIDs not on the contract
  SHALL be ignored without any error that reveals whether they exist; an empty intersection
  returns the existing empty-devices shape. A malformed value SHALL return 400
  `invalid_query`.

### Front-end contracts and layout

- **FR33.** The React API layer SHALL extend `fetchContractHistory()` with optional
  `measures` and `chipIds` arguments (existing call sites unchanged) and add a
  `fetchPulse(contractId)` function. Widening `MeasureKey` to include `'g'` SHALL NOT add a
  tile, tab or chart to the History page: `HistoryCharts` and `MeasureChartModal` keep their
  own three-measure lists, and `EMPTY_MEASURE_SUMMARIES` is handled so History never
  renders a `g` entry.
- **FR34.** At desktop width (≥ `lg`) the page SHALL be laid out on `grid-cols-12`: map
  `col-span-8`, the four stacked charts `col-span-4`, the date bar spanning the 12 columns
  above them, filters and the alert light in the page header row. Below `lg` the blocks stack
  (bar, map, charts).
- **FR35.** UI text SHALL be in French, consistent with the existing pages (e.g. "Pulse",
  "Lot client", "Appareils", "Actualiser", "Lecture", "Pause").

## 9. Non-Functional Requirements (NFRs)

- **NFR1. Strictly additive.** Suivi en direct, Historique, Pilotage, account and auth pages
  SHALL behave as before; the responses of every existing endpoint called without the new
  parameters SHALL be byte-identical for the same data; existing PHPUnit, Vitest and
  Playwright suites SHALL pass unmodified.
- **NFR2. Authorization.** Every Pulse read SHALL be scoped server-side to the caller's
  contracts (`cpal_contrat_autorise()`, MAIN/STD visibility of `V_DATA_WITH_CLIENT`); the
  client SHALL never decide access from a role it holds. Lot fields SHALL follow FR6. No
  response, status code or message SHALL reveal whether an inaccessible contract, device or
  lot exists (LT-FR15, HD-NFR5).
- **NFR3. Snapshot performance.** The FR4 endpoint SHALL answer in under 1 s (p95, local
  stack) for a contract of 100 devices, and in under 3 s for the largest Pharma demo
  contract (350 devices, ~1.4 M readings), measured before merge. If querying `V_PULSE`
  directly misses this budget, the endpoint SHALL compute the same result with a
  device-scoped query (the `tracking_logic.php` two-step pattern), `V_PULSE` remaining the
  reference for its semantics.
- **NFR4. Series performance.** For the target volume (100 devices × 30 days ≈ 9,000
  readings, Confluence §8.4) the series SHALL load in one request without sampling and the
  page SHALL render map and charts within 2 s of the response on a typical desktop. During
  playback the 80 ms tick SHALL not block input. For the Pharma stress case the existing
  25,000-point sampling and the 128 MB PHP `memory_limit` (CMC-NFR1) SHALL hold with
  `measures` including `g`.
- **NFR5. SQL portability.** Any new SQL SHALL run on MySQL 8.0 (local) and MariaDB 10.11
  (production).
- **NFR6. Production rights.** Deployment SHALL include `GRANT SELECT` on `V_PULSE` (and on
  the anomaly views used by FR28/FR29) for `cpal_web`, as listed in block 9 of
  `03-dashboard.sql`; the release checklist SHALL record it.
- **NFR7. Accessibility.** Play/pause, slider, date inputs, filters, alert light and anomaly
  counts SHALL be keyboard-operable with visible focus and French `aria-label`s; marker
  state and the alert light SHALL not rely on colour alone (text status, counts); contrast
  SHALL meet 4.5:1 in both themes; charts SHALL have an `aria-label` naming measure, device
  count and range.
- **NFR8. Theming and layout.** Design-system tokens, dark and light themes, desktop first;
  map and libraries bundled, no CDN (HD-NFR1).
- **NFR9. Tests.** PHPUnit SHALL cover the FR4 endpoint (auth, 400/404/405/503, uniform 404,
  lot ownership including an STD contract, inactive contract, ongoing hook) and FR31/FR32
  (absent parameters give an identical response, subset semantics, foreign ChipIDs
  ignored). Vitest SHALL cover the pure logic (lot-to-range mapping, hook-window row
  filter, mode switching, stale-response discard). Playwright SHALL cover the primary flow:
  open Pulse → markers → choose range → play/pause moves the map and the chart cursors →
  select a lot → open the anomaly drawer.
- **NFR10. No realtime.** No WebSocket, server push or background polling (LT-NFR10).
- **NFR11. Documentation.** `api/openapi.yaml` SHALL document the FR4 endpoint and the FR31
  and FR32 parameters; the delivery updates this spec if implementation diverges
  ([docs change with the code](../engineering/documentation.md#docs-change-with-the-code)).
- **NFR12. Time.** All dates SHALL be sent in UTC and displayed in the same zone and format
  as the History page (`formatUtc`).

## 10. Data Inputs & Outputs

**Inputs**

| Input | Source |
|---|---|
| Selected contract | `ContractProvider` (`selectedId`) |
| Snapshot | `V_PULSE` (same rows as `V_LAST_POSITION`, read from `T_DATA` directly, active contracts, lot hooked at the reading via `T_DDLOT_DEVICE_HIST`, owner-only `T_DDLOT` fields) |
| Lots and hook windows | `T_DDLOT` × `T_DDLOT_DEVICE_HIST` for the contract (FR7) |
| Series | `V_DATA_WITH_CLIENT` through `cpal_history_mesures_avec_summaries()` (existing) |
| Anomaly light and counts | `V_ANOMALIES_OUVERTES`, `V_ANOMALIES_STATUT`, via endpoints of `BACKLOG-Anomaly-Workflow.md` |
| User choices | start/end dates, lot, device subset, cursor instant (client state only) |

**Outputs**

- `GET /contracts/pulse.php/{contractId}` →
  `{ contract: { contractId, active }, generatedAt, positions: [...], lots: [...] }` (FR5,
  FR7, FR8).
- `GET /contracts/history.php/{contractId}?start&end[&measures][&chipIds]` → the existing
  shape; `summaries.g` only when requested (FR31); restricted device set when `chipIds`
  (FR32).
- UI only: map, charts, banner, filters, alert light. Pulse writes nothing.

## 11. Integration Points

- **`cpaltracker.web` `/api/v1`**: new `contracts/pulse.php` (read); `contracts/history.php`
  (read, two opt-in parameters); `_lib.php` helpers (auth, CORS, errors, id resolution);
  `auth.php` (`cpal_contrat_autorise()`); `history_logic.php` and `history_enrichment.php`
  (policy entry for `g`, device subset).
- **Database** (read only): `V_PULSE`, `T_DDLOT`, `T_DDLOT_DEVICE_HIST`, `T_CONTRATS`,
  `V_DATA_WITH_CLIENT`; anomaly views through the sibling spec.
- **Anomaly workflow** (`BACKLOG-Anomaly-Workflow.md`): Pulse depends on its contract
  summary and anomaly list endpoints and on its drawer component (open with a contract,
  device and period; emits "status changed").
- **`cpaltracker-react`**: router (`App.tsx`), shell (`AppShellLayout`, `DocumentTitle`,
  `paths.ts`), `ContractProvider`, map and chart kit, History and Live Tracking modules
  (see the reuse map).
- **OpenStreetMap tiles** (third party, read).
- **Trends** (`BACKLOG-Trends.md`): none in this iteration; a future link from a lot to its
  Trends position is §16.

### Reuse map

| Existing piece | Path (under `cpal.global/`) | How Pulse reuses it |
|---|---|---|
| `ContractProvider`, `useContracts`, `rejectContract` | `cpaltracker-react/src/contracts/ContractContext.tsx` | Contract scope and 404 handling (FR3), unchanged |
| Nav sections | `cpaltracker-react/src/chrome/AppShellLayout.tsx`, `src/ui/patterns/app-shell.tsx` | One new `SECTIONS` item (FR2); `app-shell.tsx` unchanged |
| Route constants, titles | `cpaltracker-react/src/auth/paths.ts`, `src/chrome/DocumentTitle.tsx` | New `PULSE_PATH`, new title entry (FR1) |
| Live Tracking states and refresh | `cpaltracker-react/src/routes/LiveTracking.tsx`, `src/live-tracking/ContractPositions.tsx`, `FleetPanel.tsx` | Same state set and refresh/last-updated pattern (FR3, FR12); components reused where they take props, not modified |
| `massStatus()` | `cpaltracker-react/src/live-tracking/markerStyle.ts` (same rule duplicated in `src/history/timelineData.ts`) | Marker colour (FR10); Pulse imports one of them, no third copy |
| Device marker, map kit | `cpaltracker-react/src/ui/map/DeviceMarker.tsx`, `FleetMap.tsx`, `TrackPlayback.tsx` | Marker rendering; `useTrackPlayback` play/pause/step model as reference for the bar (FR17) |
| `TimelineMap` and `timelineData` helpers | `cpaltracker-react/src/history/TimelineMap.tsx`, `timelineData.ts`, `timelineTooltip.ts` | Last-reading-at-instant, trail, replay step, fit, tooltip with active hook (FR11, FR13, FR17). The instant must be controllable from outside (bar + charts); add an optional controlled-instant prop, History's uncontrolled use unchanged |
| Chart cursor sync (KAN-252) | `cpaltracker-react/src/history/HistoryReadyPanel.tsx` (`cursorInstant`), `MeasureChartModal.tsx`, `src/ui/charts/time-series-chart.tsx` (`cursor` prop) | Same lifted-instant pattern feeding four always-visible charts (FR17, FR20) |
| `buildChartTraces`, `visibleDeviceOrder` | `cpaltracker-react/src/history/chartData.ts`, `timelineData.ts` | Series per device, shared colours (FR20); `ChartField` widened with `g` |
| `MeasureTile`, `measureFormatting` | `cpaltracker-react/src/history/MeasureTile.tsx`, `measureFormatting.ts` | Header values and formatting of each chart (FR21) |
| `SamplingBanner`, filter chip pattern, `DeviceChecklist` | `cpaltracker-react/src/history/SamplingBanner.tsx`, `HistoryReadyPanel.tsx`, `DeviceChecklist.tsx` | Sampling notice (FR19), device filter and chips (FR23) |
| `LotHookHistory`, `lotHooks` | `cpaltracker-react/src/history/LotHookHistory.tsx`, `lotHooks.ts` | Hook formatting for the lot filter and tooltip (FR11, FR24) |
| Range validation, picker mapping | `cpaltracker-react/src/history/dateRange.ts`, `timeRange.ts` | Two-date validation (FR14). `useDefaultDateRange` is **not** reused: Pulse applies no default range (FR15) |
| History API client | `cpaltracker-react/src/api/history.ts` | Series fetch with new optional arguments (FR16, FR33) |
| Positions API client | `cpaltracker-react/src/api/positions.ts` | `Position` type as the base of the Pulse position type (FR5, FR33) |
| `contracts/history.php`, `history_logic.php`, `history_enrichment.php` | `cpaltracker.web/api/v1/contracts/history.php`, `cpaltracker.web/history_logic.php`, `cpaltracker.web/history_enrichment.php` | Series, hooks, sampling (25,000 points), summaries; `measures`/`chipIds` added as opt-in (FR31, FR32) |
| `devices/positions.php`, `tracking_logic.php` | `cpaltracker.web/api/v1/devices/positions.php`, `cpaltracker.web/tracking_logic.php` | Endpoint skeleton and JSON shaping (`cpal_tracking_position_vers_api`) for FR4/FR5; fallback query pattern for NFR3. Not modified |
| `V_PULSE`, `V_ANOMALIES_OUVERTES` | `cpaltracker.web/docker/mysql/init/03-dashboard.sql` | Snapshot source (FR5), alert light (FR28) |
| `V_LAST_POSITION`, `V_DATA_WITH_CLIENT`, `T_DDLOT`, `T_DDLOT_DEVICE_HIST` | `cpaltracker.web/docker/mysql/init/01-schema.sql` | Position, visibility, lot and hook semantics (FR5–FR8) |
| OpenAPI | `cpaltracker.web/api/openapi.yaml` | New path and parameters (NFR11) |

## 11a. Software Items (IEC 62304 §5.3)

**N/A.** c-PAL is not in a regulated software scope; per
[`sdlc/README.md`](../sdlc/README.md) the Software Items sections stay empty and no Software
Item Registry exists.

## 12. Edge Cases & Constraints

- **EC1.** User with zero contracts, or contract list error: the Live Tracking states (FR3);
  no map, no request.
- **EC2.** Selected contract not active now (`active: false`): no markers; message
  "Ce contrat n'est pas actif aujourd'hui : choisissez une période pour consulter son
  historique." Range mode still works.
- **EC3.** Contract with `DebutContrat` or `FinContrat` NULL: `V_PULSE` uses
  `NOW() BETWEEN DebutContrat AND FinContrat`, which is never true with a NULL bound, so such
  a contract has no snapshot. Until OQ4 is decided, FR8 reports `active: false` for it and
  EC2 applies.
- **EC4.** Device with no positioned reading: no marker; listed in the device filter as
  "position inconnue".
- **EC5.** Readings after the last positioned one (GPS lost): `V_PULSE` shows the last
  *positioned* reading, so its weight and date may be older than the device's newest
  reading. The tooltip date makes this visible; no merge with unpositioned readings.
- **EC6.** `MASSE` null: grey marker, "—" in the tooltip. Sentinels (position 0,0,
  `RECORD_TIME` 1999-11-30) are already excluded or corrected by `HasPosition` and the
  `EventDate` formula.
- **EC7.** Lot owned by another client (e.g. device seen through an STD contract): no lot
  fields, lot absent from the lot filter (FR6, FR7).
- **EC8.** Lot with several hooks (contradicts the one-lot-one-device arbitrage but the
  schema allows it): every hooked device and window is used (FR24); nothing fails.
- **EC9.** Ongoing hook (`Hook_End_Date = 9999-12-31`): `to = null`, range end = today (UTC).
- **EC10.** Hook period longer than 1 year: range clipped to the last year of the hook
  period, with an inline notice (the History endpoint rejects longer spans).
- **EC11.** Readings dated in the future (the Pharma demo set runs to 2029-05-31): shown as
  recorded; the snapshot may show a future date; tests do not assume the last reading is
  before today.
- **EC12.** Range with zero rows: FR22 page message; markers hidden; alert light unchanged.
- **EC13.** Sampled response: FR19 banner; each device's last point is kept by the existing
  sampler.
- **EC14.** Empty device selection: FR23 message, no request.
- **EC15.** Contract, range or filter changed while a request is in flight: the stale
  response is discarded (FR16).
- **EC16.** Snapshot refresh fails: previous snapshot kept, inline error (FR12).
- **EC17.** Map tiles fail to load: markers, tooltips and charts stay usable (LT behaviour).
- **EC18.** All acceleration values null in range (or `summaries.g` null): FR22 empty state
  for that panel only.
- **EC19.** Anomaly endpoints unavailable (missing production GRANT, 503): the alert light
  shows a neutral "Anomalies indisponibles" state; the rest of Pulse works.
- **EC20.** 404 on the selected contract during the session: FR3 rejection path.
- **EC21.** Range whose rows all share one instant (min = max): slider and play disabled,
  markers shown at that instant.
- **EC22.** A device transferred to an STD contract during the range: rows follow
  `V_DATA_WITH_CLIENT` visibility, so the series may have gaps; no interpolation.

## 13. Metrics / KPIs

- FR4 endpoint p95 latency on the local stack: < 1 s for 100 devices, < 3 s for the Pharma
  stress contract (NFR3), recorded in the delivery notes.
- Time from series response to map and charts rendered: < 2 s at the 9,000-reading target
  (NFR4).
- Peak PHP memory of the History endpoint with `measures=m,ta,hr,g` on the full Pharma
  range: < 128 MB.
- Zero byte-level differences in existing endpoint responses before and after (NFR1), checked
  by tests.
- Adoption, from server access logs (no product analytics exists in c-PAL): share of Pulse
  sessions that request a series, and share that use `chipIds` (lot or device focus), over
  the first month.
- Zero authorization leaks reported (NFR2).

## 14. Risks & Mitigations

| Risk | Mitigation |
|---|---|
| `V_PULSE` latency. The first version (built on `V_LAST_POSITION` → `V_DATA_WITH_CLIENT`) did not finish in 5 min. It was rewritten on 2026-11-19 to read `T_DATA` directly: a `GROUP BY` for the latest EventDate per contract and device, then an index lookup of the reading, with the STD transfer-window rule checked only on candidate readings. Measured locally (MySQL 8.0, 128 MB buffer pool, Pharma set): contract 1: 0.1 s, contract 4 (350 devices): 5.8 s, contract 5 (STD, 103 devices): 1.9 s. Same rows as `V_LAST_POSITION`, checked row for row on contracts 1, 4 and 5. | NFR3 budget to confirm on the VPS; a 5 s server-side cache per contract if 5.8 s is too slow. |
| Contracts with NULL dates silently vanish from Pulse (EC3). | `active` flag and explicit message; OQ4 to amend `V_PULSE` (ours, not yet used elsewhere). |
| Chart headers describe the whole contract while the user looks at a subset. | FR23/FR32: series re-requested with `chipIds`, so summaries match the displayed devices. |
| Widening `MeasureKey` or the policy leaks a 4th tile or a `g` summary into History. | FR31 (policy entry only on request), FR33 (History keeps its own lists), NFR1 response-identity tests. |
| Hook windows are datetimes, History range is whole days. | FR24 client-side `[from, to)` filter on rows. |
| `TimelineMap` owns its instant internally; Pulse needs it controlled from the bar. | Optional controlled-instant prop, History keeps the uncontrolled default (reuse map). |
| Dependency on the anomaly spec's endpoints and drawer. | FR28–FR30 isolated in their own feature area; EC19 degrades gracefully; can ship after the rest. |
| Production rights missing for `cpal_web` on the new views. | NFR6 release checklist item. |
| Two `massStatus()` copies drift. | FR10: Pulse reuses one; merging the copies is optional clean-up, not required here. |
| Overlap with Suivi en direct confuses users. | Out of scope to change it; OQ6 records the question. |

## 15. Acceptance Criteria (Definition of Done)

- [ ] AC1. `/pulse` is reachable from a new "Pulse" sidebar entry, titled "Pulse", login-gated; other sidebar entries unchanged. *(FR1, FR2)*
- [ ] AC2. Pulse uses the sidebar contract; loading, list error, zero contracts, no selection and 404 rejection show the Live Tracking/provider states. *(FR3, EC1, EC20)*
- [ ] AC3. `GET /contracts/pulse.php/{id}` enforces auth, validates the id, returns one identical 404 for unknown and foreign contracts, 405 on non-GET, 503 on DB error. *(FR4, NFR2)*
- [ ] AC4. The response carries `positions` with the FR5 fields, `lots` with hook windows, `contract.active` and `generatedAt`. *(FR5, FR7, FR8)*
- [ ] AC5. Lot fields appear only for the lot owner; an STD-contract viewer gets `lot: null` and no lots. *(FR6, EC7)*
- [ ] AC6. On arrival only the Pulse endpoint is called; markers render for positioned devices and the map auto-fits. *(FR9, FR13, EC4)*
- [ ] AC7. Marker colours follow < 200 red, < 500 orange, ≥ 500 green, null grey, through the existing `massStatus()`. *(FR10, EC6)*
- [ ] AC8. The tooltip shows readings and, when present, customer lot number, product, cPAL lot and expiry, as text. *(FR11)*
- [ ] AC9. Refresh reloads snapshot and alert light, updates "last updated" only on success, keeps the old snapshot on failure; no polling exists. *(FR12, FR30, NFR10, EC16)*
- [ ] AC10. Invalid date pairs are rejected inline with no request; with no range, play and slider are disabled and charts invite to choose a period. *(FR14, FR15)*
- [ ] AC11. A valid range triggers one History request with `measures=m,ta,hr,g`; stale responses are discarded; the sampling banner shows when sampled. *(FR16, FR19, EC13, EC15)*
- [ ] AC12. Play, pause and slider move the map markers and the cursor on all four charts together, with no request. *(FR17, EC21)*
- [ ] AC13. Clearing the range restores the snapshot without a request. *(FR18)*
- [ ] AC14. Four charts are stacked next to the map in the order weight, temperature, humidity, acceleration, one series per visible device, colours matching the markers. *(FR20, FR34)*
- [ ] AC15. Chart headers show backend summaries only; empty measures and empty ranges show "Aucune mesure sur cette période". *(FR21, FR22, EC12, EC18)*
- [ ] AC16. Device filter: local in snapshot mode, re-requests with `chipIds` in range mode, empty selection makes no request. *(FR23, EC14)*
- [ ] AC17. Selecting a customer lot focuses its device, sets the range to its hook period (ongoing hook to today, clipped to 1 year with notice) and hides rows outside the hook windows; clearing it restores all devices. *(FR24, FR25, FR26, FR27, EC8, EC9, EC10)*
- [ ] AC18. The contract alert light shows red/orange/off with text and counts and opens the anomaly drawer; device counts follow the mode and open the drawer filtered; anomaly endpoint failure shows a neutral state. *(FR28, FR29, FR30, EC19)*
- [ ] AC19. `history.php` without `measures`/`chipIds` returns an identical response to before; with them, it returns exactly the requested summaries and device subset, ignores foreign ChipIDs, and rejects malformed values with 400. *(FR31, FR32, NFR1)*
- [ ] AC20. History still shows exactly three tiles and three modal tabs. *(FR33, NFR1)*
- [ ] AC21. Desktop layout follows the 8/4 column split with the full-width bar; narrow screens stack. *(FR34, NFR8)*
- [ ] AC22. UI text is French and consistent with existing pages. *(FR35)*
- [ ] AC23. Existing PHPUnit, Vitest and Playwright suites pass unmodified. *(NFR1)*
- [ ] AC24. Snapshot latency meets the NFR3 budget on 100 devices and on the Pharma stress contract, numbers recorded. *(NFR3)*
- [ ] AC25. Series rendering meets NFR4 at 9,000 readings; the Pharma full range stays under 128 MB with `g`. *(NFR4)*
- [ ] AC26. New SQL runs on MySQL 8.0 and MariaDB 10.11; production GRANTs are on the release checklist. *(NFR5, NFR6)*
- [ ] AC27. Keyboard, labels, non-colour signals and contrast checks pass in both themes. *(NFR7, NFR8)*
- [ ] AC28. PHPUnit, Vitest and Playwright tests listed in NFR9 exist and pass. *(NFR9)*
- [ ] AC29. `openapi.yaml` documents the new endpoint and parameters. *(NFR11)*
- [ ] AC30. Dates are UTC and formatted as on the History page; future-dated readings display as recorded. *(NFR12, EC11)*
- [ ] AC31. Inactive and NULL-dated contracts show the EC2 message and still allow range mode. *(FR8, EC2, EC3)*
- [ ] AC32. Unpositioned latest readings, transfers and tile failures are handled as described. *(EC5, EC17, EC22)*
- [ ] AC33. All FRs and NFRs validated.

## 16. Future Extensions (Optional)

- One map for all active contracts of the user (Confluence §6 wording), with the contract as
  a filter (OQ2).
- Pulse as the landing page (OQ1).
- Acceleration threshold line from the device's active `GH` rule (`T_TRAITEMENT`), and
  thresholds per lot or contract from `T_REGLAGES` (KAN-34).
- Click a chart point to move the cursor there; click an anomaly in the drawer to jump the
  cursor to its `EventDate`.
- Link from a lot to its position in Trends.
- Geofencing, dwell time and ETA; CSV/API export; outgoing notifications (Confluence §3).
- Opt-in client-side auto-refresh timer.

## Open questions

- **OQ1. Landing page.** Confluence makes Pulse the default tab. `/` currently renders the
  "Accueil" connectivity page and is the post-login target (`Login.tsx` → `HOME_PATH`).
  Redirect `/` to `/pulse`, make Pulse the post-login target, or keep `/` as is? This spec
  keeps `/` unchanged.
- **OQ2. All-contracts view.** Should a later iteration show every active contract of the
  user on one map, and how does that interact with the global contract selector?
- **OQ3. Nav order and label.** Pulse first (after Accueil) as proposed, or elsewhere? Where
  does Trends go?
- **OQ4. Open-ended contracts.** Should `V_PULSE` treat a NULL `DebutContrat`/`FinContrat` as
  open (amend the view), or are such contracts really out of the snapshot (EC3)?
- **OQ5. Acceleration display.** Unit shown as "g", 2 decimals, stable band for the trend
  (proposal 0.05 g)? Draw the `GH` threshold when a rule exists?
- **OQ6. Relationship with Suivi en direct.** Pulse's snapshot covers Suivi en direct's map;
  keep both pages, or plan a later retirement (a separate decision, never in this feature)?
- **OQ7. Anomaly counts per device and period.** Confirm that `BACKLOG-Anomaly-Workflow.md`
  provides a list or count endpoint filterable by contract, device and `EventDate` range,
  and a drawer that opens with those filters; FR29 depends on it.
  **Resolved 2026-11-19:** yes. `BACKLOG-Anomaly-Workflow.md` FR24 (`summary.php?contractId=`
  returns per-device counts) and FR1/FR20 (the drawer and `list.php` take `chipId`,
  `custLotNumber` and `period`) cover it, delivered by KAN-261 and KAN-265.
- **OQ8. Closed lots in the lot filter.** Keep `closed` lots selectable (useful for
  complaints on past lots, current proposal) or hide them as Trends does?
- **OQ9. Endpoint naming.** `contracts/pulse.php/{contractId}` (proposed, matches
  `contracts/history.php`) or `devices/pulse-positions.php?contractId=`?
- **OQ10. Behavior tracking.** c-PAL has no product-analytics instrumentation (no SDK in
  `cpaltracker-react`, no funnel doc in `cpal.docs`). Adopt one, and track Pulse events
  (proposed names: `pulse_viewed`, `pulse_range_applied`, `pulse_playback_started`,
  `pulse_lot_selected`, `pulse_device_filter_changed`, `pulse_anomaly_drawer_opened`)? Until
  decided, adoption is measured from access logs (§13).

---

## Requirement inventory

Every ID below must be covered by at least one story.

| Kind | IDs | Count |
|---|---|---|
| Functional | FR1–FR35 | 35 |
| Non-functional | NFR1–NFR12 | 12 |
| Acceptance criteria | AC1–AC33 | 33 |
| Edge cases | EC1–EC22 | 22 |
| User stories (context, not tracked) | US1–US5 | 5 |
| Open questions (to decide before the stories that depend on them) | OQ1–OQ10 | 10 |

Behavior tracking: Pulse is customer-facing, but c-PAL has no analytics or funnel
infrastructure today, so no tracking requirement is set; the proposed events are recorded in
OQ10 and adoption is measured from server logs (§13).

## Proposed feature areas

| # | Feature area | Requirements | Notes |
|---|---|---|---|
| A1 | Backend: Pulse snapshot endpoint | FR4–FR8, NFR2, NFR3, NFR5, NFR6, NFR11 (part), EC2, EC3, EC7–EC9, EC11 | PHP endpoint on `V_PULSE` + lots; latency measurement; OQ4, OQ9 |
| A2 | Backend: History opt-in parameters | FR31, FR32, NFR1 (API part), NFR4 (memory), NFR5, NFR11 (part) | `measures` with `g` policy entry, `chipIds` subset; response-identity tests; OQ5 |
| A3 | Navigation and page shell | FR1–FR3, FR34, FR35, NFR8, EC1, EC20 | Route, nav entry, title, contract states, grid layout; OQ1, OQ3 |
| A4 | Snapshot map | FR9–FR13, NFR10, EC4–EC6, EC16, EC17 | Markers, colours, lot tooltip, refresh, fit |
| A5 | Timeline bar and shared cursor | FR14–FR19, NFR4 (render), NFR12, EC12, EC13, EC15, EC21, EC22 | Two dates, play/pause, controlled instant shared with charts |
| A6 | Stacked measure charts | FR20–FR22, FR33, EC18 | Four charts with summary headers; History left at three measures |
| A7 | Lot and device filters | FR23–FR27, EC8–EC10, EC14 | Lot → device and hook range; `chipIds` refetch; OQ8 |
| A8 | Anomaly entry point | FR28–FR30, EC19 | Depends on `BACKLOG-Anomaly-Workflow.md` (KAN-261, KAN-265); OQ7 resolved |
| A9 | Quality and non-regression | NFR1, NFR7, NFR9, all ACs as verification | Accessibility pass, full test suites, Playwright primary flow |
