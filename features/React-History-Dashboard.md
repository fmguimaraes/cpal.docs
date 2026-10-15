# Feature: React History Dashboard (cpaltracker.web React rebuild)

> Forward-looking product spec for the **History Dashboard** feature area of the
> KAN-60 React/TypeScript UI rebuild of `cpaltracker.web`. This is a NEW,
> standalone frontend (React 18.3 + TypeScript 5.5 + Vite) consuming the existing
> KAN-116 `/api/v1` REST layer. It makes **no** backend, API-contract, or database
> change, and does **not** touch or disable any `cpaltracker.web` PHP page — the
> PHP monolith keeps running unmodified in parallel.
>
> Single source of truth for the *current* behavior being replicated is the
> reverse-engineering doc
> [`features/technical/History-Dashboard.md`](technical/History-Dashboard.md)
> (CB-n = behavior to replicate, GAP-n = known issues, each explicitly decided
> below). Stack decisions are governed by
> [`architecture/react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md).
> This spec cites those docs by id; it does not copy requirement text from them.

## 1. Feature Name

React History Dashboard — the contract-scoped, date-range historical view: a
map with a time-slider/trail/replay, three metric charts (temperature, humidity,
mass), a lot-hook history, and a per-selection summary, rebuilt in React against
`/api/v1`.

## 2. Problem / Pain Point

c-PAL operators and client users need to review the historical journey and
measurements of tracked devices over a chosen contract and date range: where a
device was, its temperature / humidity / mass over time, and which lot was hooked
when. Today this lives only in the legacy PHP page `dashboard.php`, which is
rendered server-side, styled ad hoc, CDN-dependent (Leaflet, Chart.js, Font
Awesome), and is **more permissive than the system's own documented REST contract**:
it silently swaps reversed date ranges, silently drops malformed dates, and
enforces no maximum span (GAP2–GAP4 in `History-Dashboard.md`). Users get wider,
silently-unfiltered result sets with no signal that their input was wrong, and the
page cannot evolve toward the token-authenticated, documented `/api/v1` world.
Without this feature the historical view stays trapped in the monolith, cannot
share the new design system / theming, and keeps diverging from the REST layer
being built for it.

## 3. Goal / Outcome (Success Definition)

A user opens the History Dashboard in the new React app, picks a contract and a
date range, and sees the device trail on a map they can scrub and replay, three
charts, a lot-hook history, and a selection summary — all matching the legacy
page's historical feature set (CB1–CB28) but delivered through the new design
system (dark/light theming, token-based, desktop-first), with map/chart libraries
bundled rather than CDN-loaded. Invalid date input (reversed range, malformed
date, over-long span) is rejected with a clear, actionable error instead of
silently producing a misleading result. The legacy `dashboard.php` keeps working
untouched the whole time.

## 4. User Stories (Primary Scenarios)

- **US1 — View a contract's history.** As a client user, I want to select one of
  my contracts and a date range and see the devices' measurements on a map and in
  charts, so that I can review what happened to my shipments over a period.
  *(FR1, FR2, FR3, FR9, FR10, FR11, FR12, FR13, FR21, FR22, NFR1, NFR2, NFR6)*
- **US2 — Scrub and replay the trail.** As an operator, I want to drag a
  time-slider, toggle the position trail, and hit "replay" to watch devices move
  over time with live tooltips, so that I can reconstruct a journey instant by
  instant. *(FR12, FR13, FR14, FR15, FR16, FR17, FR18, NFR3)*
- **US3 — Get a clear error on bad date input.** As a user, I want the app to
  reject a reversed range, a malformed date, or a span that is too long with a
  clear message, so that I never silently get a wider or wrong result set without
  knowing. *(FR5, FR6, FR7, FR8, NFR4)*
- **US4 — Narrow down a large result.** As an operator querying a busy period, I
  want to be warned when the result was downsampled and be able to filter which
  devices are shown and read a selection summary, so that I can narrow the range or
  focus on the devices I care about without a page reload.
  *(FR9, FR19, FR20, FR21, FR23, NFR3)*
- **US5 — Work safely within my own data.** As a client user, I want contracts and
  devices that are not mine to be inaccessible and indistinguishable from
  nonexistent ones, and I want a clean message when there is simply no data, so
  that the tool is both secure and unconfusing. *(FR2, FR4, FR24, FR25, NFR5)*

## 5. In Scope (What WILL be delivered)

- A History Dashboard route in the React app (React Router v6) that requires an
  authenticated session via the app's `AuthContext` + thin `/api/v1` REST client.
- Contract selection scoped to the authenticated user; data fetched from the
  KAN-116 `/api/v1` history endpoint (consumed as-is).
- Date-range selection with the **stricter REST-layer validation surfaced in the
  UI**: reversed ranges rejected, malformed dates rejected, a maximum span
  enforced — each with a clear inline error (FR5–FR8). This deliberately departs
  from the legacy page's permissiveness (GAP2–GAP4).
- A default date range when none is supplied, matching the legacy default logic
  (CB5) adapted to the REST response.
- A Leaflet / react-leaflet map with: per-device palette colors (CB19), markers
  colored by mass thresholds with device-colored borders (CB21), a time-slider
  showing each device's last reading at/before the instant (CB20), a tooltip with
  the full measurement + active-lot fields (CB22), an optional position trail up to
  the current instant (CB23), a "replay" animation (CB24), and auto-fit on first
  render plus an on-demand "fit" control (CB25).
- Three Plotly.js / react-plotly.js charts — temperature, humidity, mass — built
  per visible device from the fetched rows, with nulls/NaN filtered, a numeric
  (epoch-ms) x-axis, and no fixed y-bound/clipping (CB26, CB27).
- A device checklist that filters map + charts + summary entirely client-side with
  no refetch; only an explicit "refresh" re-queries the endpoint (CB16, CB17).
- Lot-hook history ("accrochages de lot") rendered from the response's hooks data,
  degrading gracefully to empty when absent (CB14, CB15).
- A downsampling warning banner when the response is sampled (CB10, CB11) and a
  per-selection summary (measurement count, device count, min/max ambient
  temperature) recomputed on every selection change (CB28).
- Graceful empty/error states: "no measurements in this period", fetch-failure
  banner, and an enumeration-safe not-found state (CB-side CB4, GAP5; edge cases in
  §6).
- Full integration with the Core epic's centralized design system: theme tokens,
  dark + light modes, desktop-first responsive layout.
- Unit tests (Vitest + Testing Library) and E2E tests (Playwright) covering every
  FR and the validation rules.

## 6. Out of Scope (What WILL NOT be delivered)

- **Any backend, `/api/v1` contract, or database change.** The KAN-116 REST
  endpoint's validation rules are consumed **as-is**; this feature does not alter,
  relax, or re-specify them. Where the REST contract and the legacy page disagree
  (GAP2–GAP4), the UI surfaces the REST behavior — it does not change either side.
- **Any change to the `cpaltracker.web` PHP monolith** (`dashboard.php`,
  `api_dashboard.php`, `map_timeline.js`, `auth.php`, …). They keep serving traffic
  unmodified; "no disruption to the current monolith" is an acceptance condition.
- Fixing GAP5 (nonexistent vs. not-yours indistinguishability) — it is a
  deliberate anti-enumeration design and is **carried forward as-is** (FR25).
- Resolving GAP1 (`map_timeline.js` live/historical sharing claim) in code — the
  component is built here as reusable, but confirming/correcting `index.php`'s
  wiring belongs to the Live Tracking area (KAN-129). No assumption is made that
  the live view shares this component.
- Mobile-first design. **Desktop is the primary, first-class target** per the
  2026-10-15 product decision; mobile is best-effort and not separately
  acceptance-tested in this epic.
- Any new realtime/polling behavior on the historical view (socket.io is held, not
  adopted — stack decision §2).
- Changing the row-flattening field names or server-side sampling/flattening logic
  (IMP5/IMP6 are legacy-side recommendations, not this feature's work).
- CSV/Excel export of history (candidate future extension, §16).

## 7. User Flow / UX Breakdown

1. User navigates to the History Dashboard route. System checks `AuthContext`; if
   unauthenticated, redirects to login (FR1).
2. System presents the user's accessible contracts. User selects a contract.
   System treats a missing/invalid contract selection as "no contract selected"
   and shows only the selection UI with a message, no data panels (FR2).
3. User optionally sets `date_from` / `date_to`. If left empty, system applies the
   default range (whole contract period capped at now, or last 30 days) (FR3).
4. User triggers load/refresh. System **validates the range client-side before
   calling the API**: if reversed → error "end date must be after start date"
   (FR6); if either date is malformed/unparseable → error naming the bad field
   (FR7); if the span exceeds the maximum → error stating the limit (FR8). No API
   call is made while input is invalid (FR5).
5. On valid input, system calls the `/api/v1` history endpoint with contract + date
   bounds (FR4). On a 401 it routes to login; on an enumeration-safe not-found it
   shows the merged "does not exist or is not accessible" message (FR24, FR25); on
   a network/fetch failure it shows a dismissible error banner (FR23).
6. System renders the map (auto-fitted), the three charts, the lot-hook history,
   the device checklist (all contract devices visible by default), the selection
   summary, and — if the response is flagged sampled — the downsampling banner
   (FR9–FR22). If zero rows are returned, it shows "no measurements in this period"
   instead of empty panels (FR22-adjacent / edge cases).
7. User toggles devices in the checklist → map, charts and summary update
   client-side with no refetch (FR19, FR9).
8. User drags the time-slider → each visible device's marker jumps to its last
   reading at/before that instant; tooltip updates; trail (if enabled) extends to
   the instant (FR12, FR13, FR16, FR17).
9. User clicks "replay" → slider animates from start to end and stops at the end;
   clicks "fit" → map re-fits to visible devices' full series (FR18, FR15).
10. User switches theme (dark/light) → entire dashboard, map chrome and charts
    re-theme via tokens with no layout break (FR26, NFR6).

## 8. Functional Requirements (FRs)

- **FR1.** The dashboard SHALL require an authenticated session (via `AuthContext`
  + `/api/v1`) and redirect unauthenticated users to login before rendering any
  data. *(replicates CB1)*
- **FR2.** The dashboard SHALL scope contract selection to contracts the
  authenticated user may access, and SHALL treat a missing/invalid contract
  selection as "no contract selected", showing selection UI + message and no data
  panels. *(replicates CB2)*
- **FR3.** When the user supplies no date range, the dashboard SHALL apply a default
  range equivalent to the legacy default (whole contract period capped at "now"
  when `DebutContrat`/`FinContrat` are set; otherwise last 30 days; open-ended
  contracts cap the upper bound at now). *(replicates CB5)*
- **FR4.** The dashboard SHALL retrieve history (contract metadata, devices,
  measurement rows, lot hooks, sampling counts/flag) exclusively from the KAN-116
  `/api/v1` history endpoint, consuming its response and validation contract as-is.
- **FR5.** The dashboard SHALL validate the date range on the client BEFORE issuing
  the history request, and SHALL NOT issue the request while the range is invalid.
- **FR6. (GAP3 decision — FIX, do not replicate legacy.)** The dashboard SHALL
  reject a reversed range (`date_from` after `date_to`) with a clear error and
  SHALL NOT silently swap the bounds as the legacy page does (CB7/GAP3).
- **FR7. (GAP4 decision — FIX, do not replicate legacy.)** The dashboard SHALL
  reject a malformed/unparseable date with a clear error naming the offending field
  and SHALL NOT silently drop the bound as the legacy page does (CB6/GAP4).
- **FR8. (GAP2 decision — FIX, do not replicate legacy.)** The dashboard SHALL
  enforce a sane maximum range span, consistent with the `/api/v1` endpoint's
  documented rule (reject a span over the endpoint's limit, e.g. one year), with a
  clear error — the legacy page enforces no span limit (CB7/CB8/GAP2).
- **FR9.** The dashboard SHALL present a device checklist listing all of the
  contract's devices, selected by default, and SHALL filter the map, charts and
  summary by selection entirely client-side with no refetch. *(replicates CB9, CB16, CB17)*
- **FR10.** The dashboard SHALL render a Leaflet/react-leaflet map and assign each
  device a stable color from a fixed palette in first-seen order. *(replicates CB19)*
- **FR11.** The dashboard SHALL color each device marker's fill by measured mass
  thresholds (<200 red, <500 orange, ≥500 green, null/NaN grey) and its border by
  the device's palette color. *(replicates CB21)*
- **FR12.** The dashboard SHALL provide a time-slider whose range spans the
  first/last measurement instant, and for a given slider position SHALL show each
  visible device's last geolocated reading at or before that instant (device not
  drawn if it has none yet). *(replicates CB19 slider bounds, CB20)*
- **FR13.** The dashboard SHALL show a marker tooltip containing measurement
  date/time (UTC), lat/lon (6 decimals), ambient temperature, relative humidity,
  mass, acceleration (G), MPU temperature, and — only when a lot hook is active for
  that device at that instant — client lot number, cPAL lot number, expiry date,
  and quantity. *(replicates CB22)*
- **FR14.** The dashboard SHALL provide a "show trail" toggle that draws, per
  visible device, a polyline through its geolocated points from series start up to
  the currently-selected instant. *(replicates CB23)*
- **FR15.** The dashboard SHALL provide a "fit" control that re-fits the map to each
  visible device's entire position series on demand. *(replicates CB25 "Ajuster")*
- **FR16.** The dashboard SHALL auto-fit the map once on the first render after new
  data loads (to current positions, with padding and a max zoom), and SHALL NOT
  re-fit on subsequent slider moves. *(replicates CB25 premierRendu)*
- **FR17.** Moving the slider SHALL update markers, tooltips and (if enabled) trails
  without refetching data. *(replicates CB16, CB20, CB23)*
- **FR18.** The dashboard SHALL provide a "replay" control that animates the slider
  from start to end and stops automatically at the end, toggling its own label back.
  *(replicates CB24)*
- **FR19.** The dashboard SHALL render three charts (temperature, humidity, mass)
  with Plotly.js/react-plotly.js, one series per visible device, filtering out
  null/NaN values, using a numeric (epoch-ms) x-axis and applying no fixed y-bound /
  clipping. *(replicates CB26, CB27)*
- **FR20.** The dashboard SHALL recompute charts and summary on every device-selection
  change from the already-fetched rows (no refetch). *(replicates CB16, CB26)*
- **FR21.** The dashboard SHALL render the lot-hook history for the contract/devices/range
  from the response's hooks data. *(replicates CB14)*
- **FR22.** When the history response is flagged as sampled, the dashboard SHALL show
  a warning banner naming the pre- and post-sampling counts and suggesting the user
  narrow the range. *(replicates CB10, CB11)*
- **FR23.** The dashboard SHALL show a dismissible error banner on a network/fetch
  failure and a distinct "no measurements in this period" message when zero rows are
  returned. *(replicates CB15-adjacent fetch catch, zero-row message)*
- **FR24.** The dashboard SHALL recompute a per-selection summary (measurement count,
  device count, min/max ambient temperature to one decimal, shown only when at least
  one selected row has a non-null ambient temperature) on every selection change.
  *(replicates CB28)*
- **FR25. (GAP5 decision — CARRY FORWARD as-is.)** The dashboard SHALL present a
  single, merged "does not exist or is not accessible" state for both a nonexistent
  contract and one belonging to another client, preserving the legacy/REST
  anti-enumeration behavior; it SHALL NOT distinguish the two. *(replicates CB3, CB4, GAP5)*
- **FR26.** The dashboard SHALL consume the Core epic's centralized theme tokens and
  render correctly in both dark and light modes, including map chrome and charts.

## 9. Non-Functional Requirements (NFRs)

- **NFR1.** The dashboard SHALL bundle its map and chart libraries (Leaflet,
  Plotly) via the build (Vite), NOT load them from third-party CDNs as the legacy
  page does (removing the CDN/Chart.js fallback dependency of CB-era
  `dashboard.php`). *(US1)*
- **NFR2.** The dashboard SHALL be built on the approved stack — React 18.3 + TS
  5.5 + Vite, React Router v6, Zustand, Tailwind 4, react-leaflet, react-plotly.js
  — per `react-frontend-stack-decision.md`; it SHALL point at local/staging or prod
  `/api/v1` by runtime config without a code change. *(US1)*
- **NFR3.** Device-selection and slider interactions SHALL update the UI
  client-side without a network round-trip; large result sets SHALL remain
  interactive (e.g. virtualize long lists with `@tanstack/react-virtual` where a
  list could exceed a few hundred rows). *(US2, US4)*
- **NFR4.** Date-range validation errors SHALL be surfaced inline, in the user's
  language, and be actionable (state what is wrong and the limit), with no silent
  result-set widening. *(US3)*
- **NFR5.** All data access SHALL go through `/api/v1` with the app's session/cookie
  auth; the UI SHALL never reveal, by message, status, or timing, whether a contract
  exists when it is not accessible. *(US5)*
- **NFR6.** The layout SHALL be desktop-first and responsive via the design tokens;
  theme switching SHALL be a single state change (no per-component conditional) and
  SHALL not break layout. Mobile is best-effort, not separately acceptance-tested.
  *(US1)*
- **NFR7.** The feature SHALL ship unit tests (Vitest + Testing Library, jsdom)
  covering each FR's logic — notably the FR5–FR8 validation rules, FR3 defaulting,
  FR11 mass thresholds, FR12 last-reading-at-or-before lookup, FR24 summary
  recomputation, and FR25 merged not-found state. *(US3, US5)*
- **NFR8.** The feature SHALL ship Playwright E2E tests, run against a running
  local/staging build, covering US1–US5 end to end — including a reversed-range,
  a malformed-date, and an over-span rejection, a scrub/replay interaction, and the
  enumeration-safe not-found path. *(US1–US5)*
- **NFR9.** The feature SHALL cause no change to and no disruption of the existing
  `cpaltracker.web` PHP pages; both run side by side. *(US1)*

## 10. Data Inputs & Outputs

**Inputs (to the React app):**
- User auth session (cookie-based, via `AuthContext` + `/api/v1`).
- Selected `contrat_id` (contract the user may access).
- `date_from` / `date_to` (date-time values; the UI enforces well-formedness,
  ordering and max span before calling the API — FR5–FR8).
- Optional client-side device selection (checklist; never sent to the API — the UI
  requests the full contract device set and filters locally, per CB17).

**Request to `/api/v1` (consumed as specified by KAN-116; not altered here):**
contract id + date bounds; the UI relies on the REST endpoint's documented
validation (reject reversed/malformed/over-span) and mirrors it client-side so the
error is immediate.

**Output (from `/api/v1`, as consumed) → rendered UI:** contract metadata; device
list (ChipID + status); measurement rows (per-device time series: position,
ambient temp, humidity, mass, acceleration, MPU temp, timestamps); lot-hook
records; and sampling counts/flag (pre- and post-sampling totals). The UI maps
these to: map markers/trails, three charts, the lot-hook table, the sampling
banner and the selection summary. The UI does not persist or export data in this
iteration.

## 11. Integration Points

- **KAN-116 `/api/v1` REST layer** — read-only; the sole data source for history,
  contracts and devices. Consumed as-is (no contract change). Auth is
  session/cookie via the app's `AuthContext` thin REST client.
- **Core epic (platform scaffold)** — provides routing, `AuthContext`, the
  centralized design-system theme tokens, and the app shell/navigation this feature
  mounts into.
- **Leaflet / react-leaflet** + an OpenStreetMap tile source for the map (bundled,
  not CDN — NFR1).
- **Plotly.js / react-plotly.js** for the three charts (bundled).
- **Zustand** for dashboard view state (selection, slider position, trail/replay
  toggles); **React Router v6** for the route.
- No integration with the PHP monolith, the TTN ingestion path, or any realtime
  socket (held per stack decision).

## 11a. Software Items (IEC 62304 §5.3)

**N/A.** Per `sdlc/README.md` "Applicability to c-PAL today", software items (IEC
62304) and the W6 IQ/OQ/PQ qualification are *Not applicable* to c-PAL today; these
sections stay empty until c-PAL software enters a regulated scope. No Software Item
Registry UIDs are assigned to this feature.

## 12. Edge Cases & Constraints

- **Contract with zero devices / empty intersection:** render an empty-but-valid
  state (no devices, no rows), not an error. *(replicates CB edge cases)*
- **Open-ended contract (no end date) / no start date:** default range falls back to
  "now" upper bound and 30-days-before lower bound respectively (FR3); UI shows "en
  cours"-equivalent for an open end. *(replicates CB5 / §6 edge cases)*
- **Reversed range / malformed date / over-span:** rejected with a clear error (FR6,
  FR7, FR8) — the explicit departure from the legacy silent-swap / silent-drop /
  no-limit behavior (GAP2–GAP4).
- **Sampled (very large) result:** show the downsampling banner (FR22); keep the UI
  interactive (NFR3). The UI does not attempt to defeat server-side sampling.
- **Device with no reading at/before the slider instant:** that device is simply not
  drawn at that instant, not an error (FR12). *(replicates CB20 / §6)*
- **Missing/unmigrated lot-hook data:** the hooks section degrades to empty without
  failing the view (FR21). *(replicates CB15)*
- **Zero rows in range:** distinct "no measurements in this period" message, not an
  error banner (FR23). *(replicates CB zero-row message)*
- **Network/fetch failure:** dismissible error banner (FR23).
- **Not-your / nonexistent contract:** single merged message; no timing or status
  leak (FR25, NFR5).
- **Theme switch mid-session:** map and charts re-theme via tokens without reload or
  layout break (FR26).

## 13. Metrics / KPIs

- 100% of FRs covered by at least one passing unit or E2E test (NFR7, NFR8).
- 0 silent date-validation acceptances: every reversed / malformed / over-span input
  yields a visible error (measured via E2E — NFR8).
- 0 CDN runtime dependencies for map/charts (bundled — NFR1).
- Device-toggle and slider interactions reflected in < 300 ms with no network call
  on a representative result set (NFR3).
- 0 regressions to the legacy `cpaltracker.web` pages during the rebuild (NFR9).

## 14. Risks & Mitigations

- **Risk: UI validation (FR6–FR8) diverges from the actual `/api/v1` rules.**
  Mitigation: treat the REST endpoint as the source of truth; mirror its documented
  rules and still handle its error responses defensively (FR4, NFR5); do not alter
  the endpoint.
- **Risk: `map_timeline.js` sharing ambiguity (GAP1) leads to wrong component
  boundaries.** Mitigation: build the scrubbable map as a self-contained reusable
  React component driven only by History data; make no assumption that the live view
  shares it; defer GAP1 resolution to the Live Tracking area (KAN-129).
- **Risk: Chart library swap (Chart.js → Plotly) changes axis/scale behavior.**
  Mitigation: preserve the legacy numeric x-axis + no-y-clip decisions explicitly
  (FR19/CB27) and cover with unit tests.
- **Risk: large result sets degrade interactivity.** Mitigation: rely on server-side
  sampling (consumed as-is) plus client virtualization and client-side filtering
  (NFR3).
- **Risk: scope creep into backend "fixes".** Mitigation: §6 forbids any backend/DB
  change; GAP2–GAP4 are surfaced in the UI only, GAP5 carried forward, legacy
  IMP1–IMP6 left out.

## 15. Acceptance Criteria (Definition of Done)

- [ ] Unauthenticated access redirects to login; authenticated access scopes
      contracts to the user. *(FR1, FR2)*
- [ ] Empty date range applies the legacy-equivalent default. *(FR3)*
- [ ] History data is fetched only from `/api/v1`, consumed as-is. *(FR4, NFR2)*
- [ ] Reversed range is rejected with a clear error and NO swap. *(FR5, FR6)*
- [ ] Malformed date is rejected with a clear, field-named error and NO silent
      drop. *(FR5, FR7)*
- [ ] Over-maximum span is rejected with a clear error stating the limit. *(FR5, FR8)*
- [ ] Device checklist filters map/charts/summary client-side with no refetch;
      only explicit refresh re-queries. *(FR9, FR20)*
- [ ] Map renders per-device palette colors, mass-threshold marker fills with
      device-colored borders. *(FR10, FR11)*
- [ ] Time-slider shows each device's last reading at/before the instant; devices
      with none are not drawn. *(FR12)*
- [ ] Tooltip shows all measurement fields plus active-lot fields only when a hook
      is active. *(FR13)*
- [ ] Trail toggle, on-demand fit, first-render auto-fit, and replay all behave per
      spec. *(FR14, FR15, FR16, FR18)*
- [ ] Slider moves update markers/tooltips/trails with no refetch. *(FR17)*
- [ ] Three Plotly charts render per visible device, filter null/NaN, use a numeric
      x-axis and apply no y-clip. *(FR19)*
- [ ] Lot-hook history renders and degrades to empty gracefully. *(FR21)*
- [ ] Sampling banner appears (with both counts) when the response is sampled. *(FR22)*
- [ ] Fetch failure shows a dismissible banner; zero rows show the distinct
      "no measurements" message. *(FR23)*
- [ ] Selection summary (count, device count, temp extrema) recomputes on selection
      change. *(FR24)*
- [ ] Nonexistent and not-yours contracts produce one identical, non-leaking
      message. *(FR25, NFR5)*
- [ ] Dashboard renders correctly in dark and light themes via tokens; theme switch
      is a single state change. *(FR26, NFR6)*
- [ ] Map/chart libraries are bundled, not CDN-loaded. *(NFR1)*
- [ ] Interactions are client-side and responsive on a representative data set. *(NFR3, NFR4)*
- [ ] Unit tests (Vitest + Testing Library) cover each FR's logic. *(NFR7)*
- [ ] Playwright E2E tests cover US1–US5, including all three date-rejection paths,
      a scrub/replay, and the enumeration-safe not-found path. *(NFR8)*
- [ ] Legacy `cpaltracker.web` PHP pages are untouched and keep working. *(NFR9)*
- [ ] All FRs and NFRs validated.

## 16. Future Extensions (Optional, Not Required Now)

- CSV/Excel export of the selected history (PapaParse / `xlsx` are already in the
  stack).
- Comparison view across multiple contracts or date ranges.
- Deep-linkable URLs encoding contract + range + selection for sharing a view.
- A future switch to a realtime/polling live layer (socket.io, currently held)
  should it be adopted by the Live Tracking area.
- Legacy-side alignment (IMP1–IMP4) and REST field-name readability (IMP6) — tracked
  as separate backend work, not this frontend feature.
