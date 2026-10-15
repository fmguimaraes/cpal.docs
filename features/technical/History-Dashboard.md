# Technical-Feature: History Dashboard

> Reverse-engineered from the current `cpaltracker.web` codebase for
> KAN-130 (epic KAN-126, "Technical Feature Discovery — cpaltracker.web"),
> feeding the planned React UI rebuild (KAN-60). Read-only analysis — no
> application code was modified to produce this document.

## 1. Feature Area

History Dashboard — the date-range/contract-scoped historical view
(`dashboard.php`), its JSON data source (`api_dashboard.php`), and the
chart/timeline-rendering parts of the shared map component
(`map_timeline.js`) that it drives.

Live-mode rendering (current-position-only view) is out of scope here and
covered by the sibling story's doc, KAN-129.

## 2. Scope (files / pages covered)

- `cpaltracker.web/dashboard.php` — page: contract/date-range selection,
  device checklist, map + three Chart.js panels.
- `cpaltracker.web/api_dashboard.php` — JSON endpoint: measurements,
  devices, lot-hook history for a contract/date range.
- `cpaltracker.web/map_timeline.js` — shared Leaflet time-slider map
  component. **Entire file read**, but only the parts exercised by
  `dashboard.php` (historical series: `setData`, `setVisibleDevices`,
  time-slider scrubbing, trail/replay, tooltip) are documented as
  "Current Behavior" below. See CB18 for an explicit note on the
  live/historical split inside this file.
- `cpaltracker.web/docs/security.md` — cited, not re-derived, for the
  CSRF/contract-scoping facts already documented there.
- `cpaltracker.web/docs/design-and-standards.md` — cited for the SRP/file
  organization findings already documented there.
- `cpaltracker.web/docs/architecture.md` — cited for the `map_timeline.js`
  "shared by `index.php` and `dashboard.php`" claim checked in GAP1.
- `cpaltracker.web/auth.php` — read only for the specific helper
  functions `dashboard.php`/`api_dashboard.php` call
  (`cpal_require_login`, `cpal_current_user`, `cpal_csrf`/`cpal_csrf_ok`,
  `cpal_contrat_autorise`); not analyzed as its own feature area (that
  belongs to the auth-area stories, e.g. KAN-127/KAN-128).

Excluded: `index.php`/`pilotage.php` (live view, KAN-129), the TTN
ingestion path and `insert_data.php` (not part of this read path), and
the in-progress `/api/v1` REST extraction (KAN-116/KAN-118) — noted only
in Open Questions.

## 3. Current Behavior (CB-n)

**Access & contract scoping**

- CB1. `dashboard.php` requires an authenticated session before rendering
  anything: `cpal_require_login($pdo)` is the first statement after the
  includes, and redirects to `login.php` if there is no valid session
  (`dashboard.php` line 35, `auth.php::cpal_require_login`).
- CB2. The requested contract is accepted from either `$_POST['contrat_id']`
  or `$_GET['contrat_id']`, trimmed, and only used if it is a run of
  digits (`ctype_digit`); any other value (missing, non-numeric) sets
  `$error` to "Aucun contrat sélectionné..." and the page renders only the
  sidebar error, no data panels (`dashboard.php` lines 37–52).
- CB3. Contract ownership is re-checked server-side on every page load via
  `cpal_contrat_autorise($pdo, $user, (int) $contratId)`
  (`dashboard.php` line 46; `auth.php::cpal_contrat_autorise`), which
  returns the contract row only if the caller is an admin
  (`cpal_is_admin`) or the contract's `ClientID` matches the session
  user's `ClientID` (`auth.php` lines 309–329). A contract that doesn't
  exist and a contract that exists but belongs to another client produce
  the identical user-facing message — "Ce contrat n'existe pas ou ne vous
  est pas accessible." (`dashboard.php` line 48) — so the two cases are
  not distinguishable from the response.
- CB4. `api_dashboard.php` independently repeats the same authorization
  check before running any data query: it resolves the current user via
  `cpal_current_user($pdo)` (401 if none), checks the CSRF token via
  `cpal_csrf_ok()` (403 if invalid/missing), then calls
  `cpal_contrat_autorise($pdo, $user, $contratId)` and returns 404
  ("Contrat introuvable.") if it is null — again not distinguishing
  "doesn't exist" from "not yours" (`api_dashboard.php` lines 71–104).

**Date-range handling**

- CB5. If the page is loaded with no `date_from`/`date_to` at all, and the
  contract was resolved, `dashboard.php` defaults the range to the whole
  contract period, capped at "now": `$fin` is
  `min(strtotime(FinContrat), time())` when `FinContrat` is set, else
  `time()`; `$debut` is `strtotime(DebutContrat)` when set, else
  `strtotime('-30 days', $fin)` (`dashboard.php` lines 58–64). If only one
  of the two params is supplied, no default is applied to the other (the
  `if` only fires when both are `''`).
- CB6. `api_dashboard.php` validates each of `date_from`/`date_to`
  independently with `toSqlDate()`, which requires the exact
  `YYYY-MM-DDTHH:MM` shape (`preg_match('/^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}$/')`)
  and otherwise treats the parameter as absent (empty string, no filter
  applied) — there is no error returned for a malformed date, it is
  silently dropped (`api_dashboard.php` lines 54–61, 85–86).
- CB7. If both dates parse and `date_from > date_to`, `api_dashboard.php`
  **swaps them** rather than rejecting the request: `[$dateFromSql,
  $dateToSql] = [$dateToSql, $dateFromSql]` (`api_dashboard.php` lines
  87–89). There is no maximum span check anywhere in the file — an
  arbitrarily large date range (or no range at all) is accepted and
  handled only by the row-count sampling described in CB10–CB11.
- CB8. The SQL filter on `V_DATA_WITH_CLIENT` applies `EventDate >=
  date_from` / `EventDate <= date_to` as independent, optional clauses —
  a request with only one bound filters only that side
  (`api_dashboard.php` lines 143–149), consistent with the file's own
  docstring statement that filtering is on `EventDate` (measurement time)
  and not `timestamp_utc` (DB arrival time) (lines 11–12).

**Device selection**

- CB9. `api_dashboard.php` accepts an optional comma-separated `devices`
  parameter (`ChipID`s). The requested list is intersected with the
  contract's actual device list (`array_intersect($chipsContrat,
  $devicesWanted)`) before querying — a `ChipID` not on the contract is
  silently dropped, never queried (`api_dashboard.php` lines 91–95,
  126–129). An empty/absent `devices` parameter means "all devices on the
  contract" (line 93–95).

**Sampling**

- CB10. Rows are grouped per device (`ChipID`) after the SQL fetch. If the
  total row count across all devices exceeds `MAX_POINTS = 25000`, each
  device's series is independently downsampled to roughly
  `floor(25000 / device_count)` points by fixed-stride selection
  (`floor($i * $n/$quota)`), always keeping that device's last point
  (`api_dashboard.php` lines 43, 160–176).
- CB11. The response reports both the post-sampling row count (`count`)
  and the pre-sampling total (`total`), plus a `sampled: true/false` flag
  (`api_dashboard.php` lines 160–163, 235–244); `dashboard.php` shows a
  French-language warning banner when `sampled` is true, naming both
  numbers and suggesting the user narrow the date range
  (`dashboard.php` lines 262–266).

**Response shape / field flattening**

- CB12. Each returned row uses single/double-letter keys, not the SQL
  column names, to shrink payload size — the file's own comment states
  this cuts payload from "~2 Mo" to "~600 Ko": `c` (ChipID), `t`
  (epoch-ms, computed as `strtotime(EventDate . ' UTC') * 1000`), `ts`
  (raw `EventDate` string), `p` (`HasPosition` as int), `la`/`lo`
  (lat/long or null), `ta` (TEMP_AMB), `hr` (HR_AMB), `m` (MASSE), `g`
  (ACCEL_G), `tm` (TEMP_MPU) (`api_dashboard.php` lines 178–196, 251–263).
- CB13. Rows are globally re-sorted by `t` ascending after per-device
  sampling (`usort($rows, fn($a,$b) => $a['t'] <=> $b['t'])`,
  `api_dashboard.php` line 197), so the client receives one
  chronologically merged stream across devices, not grouped by device.
- CB14. Lot-hook history ("accrochages de lot") for the same contract,
  devices, and date range is queried separately from
  `T_DDLOT_DEVICE_HIST` joined to `T_DDLOT`, filtered so a hook overlaps
  the requested range (`Hook_End_Date >= date_from AND Hook_Start_Date <=
  date_to`), and returned as a `hooks` array with its own short keys
  (`c`, `lot`, `cust`, `from`, `to`, `net`, `gros`, `nb`, `per`)
  (`api_dashboard.php` lines 199–233, 251–263).
- CB15. If `T_DDLOT_DEVICE_HIST`/`T_DDLOT` queries throw a `PDOException`
  (file's own comment: "Les tables DDLOT peuvent ne pas exister sur une
  base non migrée"), the exception is caught, logged via `error_log`, and
  `hooks` is left as an empty array — the rest of the response (`rows`,
  `devices`, etc.) is still returned successfully
  (`api_dashboard.php` lines 201–233).

**Page-side data flow and rendering**

- CB16. `dashboard.php` never performs a full page reload after the
  initial load: toggling a device checkbox or moving the time slider is
  handled entirely client-side against the already-fetched `donnees`
  object; only the "Actualiser" button (`btnCharger`) re-POSTs to
  `api_dashboard.php` with the current `date_from`/`date_to` values
  (`dashboard.php` lines 11–13, 228–272, 421).
- CB17. The request to `api_dashboard.php` is a `POST` with
  `Content-Type: application/x-www-form-urlencoded` carrying
  `csrf_token` (from `cpal_csrf()`, embedded server-side into the page as
  a JS constant `CSRF`), `contrat_id`, `date_from`, `date_to` — no
  `devices` parameter is ever sent by `dashboard.php` itself (it always
  requests the full contract device set and filters client-side via
  `carte.setVisibleDevices()`/per-chart `selection` filtering)
  (`dashboard.php` lines 207–208, 230–242, 306–310, 378–395).
- CB18. `map_timeline.js`'s own header comment describes it as shared
  between "the LIVE view (`index.php`) and the dashboard
  (`dashboard.php`)" (`map_timeline.js` lines 1–10) and
  `docs/architecture.md`'s component table repeats the same claim ("used
  by `index.php` and `dashboard.php`"). The file does not itself branch
  on a live/historical mode — `setData()`/`rendre()`/the time-slider and
  replay logic are the same code path regardless of caller. See GAP1 for
  the discrepancy between this stated sharing and what the current code
  actually wires up.
- CB19. `setData(rows, hooks)` assigns one color per device from a
  10-entry fixed palette in first-seen order (`PALETTE[i %
  PALETTE.length]`, wrapping past 10 distinct devices), stores only the
  geolocated rows (`p === 1 && la !== null && lo !== null`) per device for
  map placement, and sets the time slider's `min`/`max` to the first/last
  row's `t`, with `step` computed as `max(1000, floor((tMax-tMin)/2000))`
  ms (`map_timeline.js` lines 89–126).
- CB20. For a given time-slider position, each visible device's marker
  shows its *last* geolocated reading at or before that instant, found by
  binary search (`dernierAvant`) over that device's sorted series — a
  device with no position at or before the current instant is not drawn
  at all (`map_timeline.js` lines 140–148, 198–232).
- CB21. Marker fill color encodes measured mass via fixed thresholds:
  `< 200` → red, `< 500` → orange, `>= 500` → green, null/NaN → grey
  (`couleurMasse`, `map_timeline.js` lines 34–39), matching the legend
  rendered in `dashboard.php` (lines 162–167). Marker *border* color is
  the device's palette color.
- CB22. The marker tooltip shows, in order: measurement date/time (UTC,
  from `formatUTC`), lat/lon to 6 decimals, ambient temperature, relative
  humidity, mass (via `libelleMasse`), acceleration (G), MPU temperature,
  and — only if a lot hook is active for that device at that instant
  (`lotA()`, half-open interval `t >= h.from && t < h.to`) — client lot
  number, cPAL lot number, expiry date (`per`, if present), and quantity
  (`nb`, if present) (`map_timeline.js` lines 150–184, 226–228).
- CB23. Checking "Afficher la trace" draws, for each visible device, a
  polyline through all of that device's geolocated points from the start
  of its series up to the currently-selected instant (not the full
  series) (`map_timeline.js` lines 195, 207–213).
- CB24. "Rejouer" animates the slider from `tMin` to `tMax` in fixed
  80 ms ticks, advancing by `max(1000, floor((tMax-tMin)/200))` ms of
  simulated time per tick, and stops automatically (toggling the button
  back to "Rejouer") once `instant >= tMax` (`basculerLecture`,
  `map_timeline.js` lines 255–271).
- CB25. On first successful render after `setData()` with at least one
  positioned device, the map auto-fits to the bounding box of all visible
  devices' current positions with 50px padding and `maxZoom: 15`, but
  only once per `setData()` call (`premierRendu` flag) — subsequent
  slider moves do not re-fit the view (`map_timeline.js` lines 110,
  124–125, 234–240). The "Ajuster" button re-triggers the same fit
  on demand, this time over each visible device's *entire* position
  series rather than only the current instant (`ajuster()`,
  `map_timeline.js` lines 243–252).
- CB26. The three Chart.js panels (temperature, humidity, mass) are
  redrawn from scratch (`instances[id].destroy()` then `new Chart(...)`)
  on every selection change, not incrementally updated
  (`dashboard.php` lines 397–413). Each series is built per visible
  device from `donnees.rows`, filtering out `null`/`NaN` values for that
  field and mapping to `{x: r.t, y: r[champ]}` (`dashboard.php` lines
  378–395).
- CB27. Chart x-axes use a `'linear'` scale type (epoch ms as a plain
  number) rather than Chart.js's `'time'` scale, with the file's own
  comment stating this avoids adding a date adapter dependency for "an
  identical result" (`dashboard.php` lines 337–338, 365–372); axis tick
  labels are `fmt(v).substring(0, 16)` (first 16 chars of the UTC
  timestamp string). Y-axes are never fixed-bounded; an 8% `grace` margin
  is applied instead, with the comment noting mass values run up to
  30 000 (grams) so a fixed bound would clip the curve
  (`dashboard.php` lines 340–349). This supersedes a v2.0 behavior,
  per the file's own changelog: v2.1 removed a "Limiter l'axe à
  0–1500 kg" checkbox that had been clipping the mass chart
  (`dashboard.php` lines 21–27).
- CB28. The sidebar "Sur la sélection" summary recomputes from the
  currently selected devices' rows on every selection change: measurement
  count, device count, and min/max ambient temperature (°C, one decimal)
  — only if at least one selected row has a non-null `ta`
  (`dashboard.php` lines 312–325). No humidity or mass aggregate is shown
  here (temperature only, consistent with the v2.1 changelog note that
  the three previous mean values were removed and only temperature
  extrema remain — `dashboard.php` lines 21–24).

## 4. Current Data Model & Contracts

**`api_dashboard.php` request contract** (POST,
`application/x-www-form-urlencoded`, per its own docstring lines 14,
24–31 and the code):

| Field | Required | Format | Behavior on invalid/missing |
|---|---|---|---|
| `csrf_token` | yes | session CSRF token (`cpal_csrf_ok`) | 403 `fail()` |
| `contrat_id` | yes | digits only (`ctype_digit`) | 400 `fail()` |
| `date_from` | no | `YYYY-MM-DDTHH:MM` | silently treated as "no lower bound" |
| `date_to` | no | `YYYY-MM-DDTHH:MM` | silently treated as "no upper bound" |
| `devices` | no | comma-separated `ChipID`s | empty/absent = all contract devices; unknown IDs dropped |

Also requires: `REQUEST_METHOD === 'POST'` (405 otherwise), an
authenticated session (401 otherwise, via `cpal_current_user`)
(`api_dashboard.php` lines 64–77).

**`api_dashboard.php` response contract** (per the docblock at the end of
the file, lines 251–263, cross-checked against the `echo json_encode(...)`
calls):

```
{
  "ok": true,
  "contrat": { ContratID, ContratRef, ContratType, ClientID, DebutContrat, FinContrat },
  "devices": [ { ChipID, STATUT, MORT } ],
  "rows":    [ { c, t, ts, p, la, lo, ta, hr, m, g, tm } ],
  "hooks":   [ { c, lot, cust, from, to, net, gros, nb, per } ],
  "count":   <int>,
  "total":   <int>,
  "sampled": <bool>
}
```

On error: `{ "ok": false, "error": "<message, French>" }` with a non-200
status (`fail()`, `api_dashboard.php` lines 46–52). All known error paths
(405, 401, 403, 400, 404, 500) use this shape (CB4, CB6-adjacent, lines
64–104, 246–249).

**Tables/views touched** (read-only, all via `$pdo->prepare()` with bound
parameters):

- `T_CONTRATS` (via `cpal_contrat_autorise`, `auth.php` lines 312–315) —
  `ContratID, ContratRef, ContratType, ClientID, DebutContrat, FinContrat`.
- `T_CONTRATS_DEVICES` joined to `T_DEVICE` — `ChipID, STATUT, MORT` per
  device on the contract (`api_dashboard.php` lines 107–115).
- `V_DATA_WITH_CLIENT` — the measurement view, filtered by `ContratID`,
  `ChipID IN (...)`, and optionally `EventDate` bounds; columns read:
  `ChipID, EventDate, timestamp_utc, HasPosition, LAT, LONGITUDE,
  TEMP_AMB, HR_AMB, MASSE, ACCEL_G, TEMP_MPU` (`api_dashboard.php` lines
  143–149).
- `T_DDLOT_DEVICE_HIST` joined to `T_DDLOT` — lot-hook history, columns
  `ChipID, cPAL_Lot, Hook_Start_Date, Hook_End_Date, Cust_Lot_Number,
  Net_Weight, Gross_Weight, Number_Of_Products, Date_Peremption`
  (`api_dashboard.php` lines 204–212).

**Session/cookie state used**: the authenticated session established by
`auth.php` (session user id, CSRF token) is read but not modified by
either `dashboard.php` or `api_dashboard.php`; no cookies are set by
either file directly (`api_dashboard.php` only sets `Content-Type` and
`Cache-Control: no-store` response headers, lines 40–41).

## 5. Current Integration Points

- **MySQL via PDO** (read-only for this feature area): `T_CONTRATS`,
  `T_CONTRATS_DEVICES`, `T_DEVICE`, `V_DATA_WITH_CLIENT`,
  `T_DDLOT_DEVICE_HIST`, `T_DDLOT` — all read, no writes, from
  `api_dashboard.php`/`auth.php` as listed in §4.
- **CDN-loaded frontend libraries** (`dashboard.php` lines 73–74,
  193–201): Leaflet 1.9.4 (`unpkg.com`), Font Awesome 6.0.0
  (`cdnjs.cloudflare.com`), Chart.js 4.4.1 (`cdnjs.cloudflare.com`, with a
  documented client-side fallback to `cdn.jsdelivr.net` if `Chart` is
  undefined after the first script tag, lines 196–201).
- **OpenStreetMap tile server** (`map_timeline.js` lines 69–72):
  `{s}.tile.openstreetmap.org` tiles loaded directly by the browser, no
  server-side proxying.
- **No external API/service calls** from `api_dashboard.php` itself
  beyond the DB (no mailer, no HIBP, no TTN — those belong to other
  feature areas per `docs/architecture.md`).

## 6. Edge Cases Already Handled

- Contract with zero devices: `api_dashboard.php` returns `ok: true` with
  empty `devices`/`rows`/`hooks` and `count: 0` immediately after the
  device query, without attempting the measurement query
  (`api_dashboard.php` lines 118–124).
- Requested `devices` list that, after intersecting with the contract's
  actual devices, is empty (e.g. all requested ChipIDs are foreign to the
  contract): same early-return shape as above (`api_dashboard.php` lines
  131–137).
- Open-ended contract (`FinContrat IS NULL`): `dashboard.php`'s default
  date range falls back to `time()` as the upper bound instead of a null
  date (`dashboard.php` lines 59–60), and the UI renders "en cours"
  instead of a date for `FinContrat` (`dashboard.php` line 101).
- Contract with no `DebutContrat`: default lower bound falls back to
  "30 days before the upper bound" rather than an unbounded query
  (`dashboard.php` lines 61, comment on line 59).
- Reversed date range (`date_from > date_to`): handled by silent swap,
  not rejection (CB7).
- Malformed date string in `date_from`/`date_to`: handled by silently
  dropping that bound (CB6), not by a 400 error.
- Missing/unmigrated `T_DDLOT_DEVICE_HIST`/`T_DDLOT` tables: caught
  `PDOException`, logged, `hooks` degrades to `[]` without failing the
  whole request (CB15).
- Result set over `MAX_POINTS` (25000): downsampled per-device rather
  than truncated or rejected, always preserving each device's final point
  (CB10).
- No rows in a time-slider position for a device (not yet "arrived" at
  the start of its series): that device is simply not drawn for that
  instant rather than erroring (`map_timeline.js` line 204,
  `dernierAvant` returning `-1`).
- Chart.js failing to load from both CDNs: `dessinerCourbes()` checks
  `typeof Chart === 'undefined'` and shows a French alert naming both
  CDN hosts instead of throwing (`dashboard.php` lines 398–402).
- Network/fetch failure calling `api_dashboard.php`: caught in a
  `.catch()`, shown via the same `alerter()` banner
  (`dashboard.php` lines 268–271).
- Zero rows returned for the selected range: a specific "Aucune mesure
  sur cette période" message is shown, distinct from a fetch error
  (`dashboard.php` lines 257–261).

## 7. Gaps (GAP-n)

- GAP1. **Stated vs. actual sharing of `map_timeline.js`.** Both the
  file's own header comment ("Composant partagé entre la vue LIVE
  (`index.php`) et le dashboard (`dashboard.php`)", `map_timeline.js`
  lines 5–7) and `docs/architecture.md`'s component table ("Shared
  Leaflet-based time-slider map, used by `index.php` and `dashboard.php`")
  state that `index.php` also uses this component. A repo-wide search
  (`grep -rl "map_timeline.js\|CpalTimelineMap"` over every `.php` file)
  finds only `dashboard.php` referencing it; `index.php` has no `<script
  src="map_timeline.js">` tag and no `CpalTimelineMap` call anywhere in
  the file. This is an inconsistency between the codebase's own
  documentation (in two places) and its current wiring, not a judgment
  call — either `index.php`'s live map was reimplemented separately (and
  this comment/table is stale) or the sharing is aspirational and never
  landed. Left for KAN-129 (which covers `index.php`) to confirm from the
  live-mode side; flagged here because it directly affects how much of
  `map_timeline.js` the React rebuild can treat as a single reusable
  component versus dashboard-only code.
- GAP2. **No maximum date-range span enforced.** `api_dashboard.php`
  accepts any `date_from`/`date_to` combination (including none at all)
  with no upper bound on the span between them (CB7, CB8) — the only
  mitigation for a very large range is point-count sampling (CB10), which
  reduces resolution rather than rejecting the request. This is a gap
  relative to the documented rule for the new `/api/v1/contracts/{id}/history`
  endpoint (KAN-116, per the task context: reject a span over one year) —
  the current page/endpoint pair has no equivalent rule today.
- GAP3. **Reversed date range is corrected, not rejected.** CB7:
  `api_dashboard.php` silently swaps `date_from`/`date_to` when
  `date_from > date_to` rather than returning a 400. This differs from
  the validation rule KAN-116 documents for the new REST endpoint (reject
  a reversed range) — the current behavior is not wrong by its own
  stated rules (it has none for this case), but it is a different
  contract than the one being built for `/api/v1`, which matters for the
  React rebuild if it is meant to assume REST-equivalent semantics on the
  legacy endpoint.
- GAP4. **Malformed date params fail silently rather than erroring.**
  CB6: an unparseable `date_from`/`date_to` (wrong format, not just an
  out-of-range value) is treated identically to an absent parameter — the
  bound is simply dropped. There is no `fail()` call for this case, so a
  client-side typo in the date format (e.g. a stray space, wrong
  separator) produces a wider, silently-unfiltered result set instead of
  a visible error, with no server-side signal that the parameter was
  rejected.
- GAP5. **"Doesn't exist" and "not yours" are indistinguishable**, by
  design, in both `dashboard.php` (CB3) and `api_dashboard.php` (CB4).
  `docs/security.md` documents this merged messaging as a deliberate
  anti-enumeration choice ("Message identique pour « inexistant » et
  « interdit » : distinguer les deux permettrait d'énumérer les contrats
  des autres clients", comment at `api_dashboard.php` lines 99–100) — so
  this is confirmed intentional, not an oversight, and is listed here
  only because it is a cross-page consistency fact worth carrying into
  the React rebuild's own error handling for the new `/api/v1` endpoint
  (which should preserve the same non-distinguishing response per the
  same anti-enumeration rationale, not an inconsistency to fix).

## 8. Suggested Improvements (IMP-n)

- IMP1. *(Recommendation, not current behavior.)* Add an explicit
  maximum-span check to `api_dashboard.php` (e.g. reject with 400 when
  `date_to - date_from` exceeds some bound) ahead of the KAN-116 REST
  endpoint going live, so the legacy page and the new `/api/v1` endpoint
  enforce the same rule rather than diverging — addresses GAP2.
- IMP2. *(Recommendation.)* Reject a reversed date range with a 400 and a
  clear message instead of silently swapping it (addresses GAP3) — this
  would also make `api_dashboard.php`'s date-handling consistent with the
  rule already decided for the new REST endpoint, reducing behavioral
  surprises for anyone comparing the two during the KAN-60 rebuild.
- IMP3. *(Recommendation.)* Return a 400 for a `date_from`/`date_to` value
  that fails the `toSqlDate()` format check, rather than dropping it
  silently (addresses GAP4) — today a malformed date and an intentionally
  omitted one are indistinguishable from the response, which will make it
  harder to port this endpoint's semantics into a documented OpenAPI
  contract under `/api/v1`.
- IMP4. *(Recommendation.)* Resolve GAP1 explicitly as part of (or just
  ahead of) the KAN-60 rebuild: either confirm `index.php`'s live view
  genuinely has its own separate rendering path (and correct the
  `map_timeline.js` header comment and `docs/architecture.md`'s component
  table to stop claiming it's shared), or restore the sharing if it was
  intended — so the React rebuild's component boundaries are based on
  what the code actually does, not on a stale comment.
- IMP5. *(Recommendation, for the React rebuild specifically.)* Per
  `docs/design-and-standards.md`'s own recommendation (cited, not
  re-derived here), extract `api_dashboard.php`'s query/shaping logic
  (contract resolution, device intersection, sampling, row flattening,
  hook-lookup) into plain functions independent of the HTTP layer before
  or during the rebuild, so the same functions can back both the legacy
  page (if kept running during migration) and a future
  `/api/v1/contracts/{id}/history`-style endpoint, avoiding a second,
  independently-maintained implementation of the sampling/flattening
  rules documented in CB10–CB13.
- IMP6. *(Recommendation.)* The row-flattening key scheme (`c`, `t`,
  `ts`, `p`, `la`, `lo`, `ta`, `hr`, `m`, `g`, `tm` — CB12) is undocumented
  outside the trailing comment in `api_dashboard.php` itself. If any part
  of this contract is carried into the documented `/api/v1` OpenAPI spec
  (per `docs/architecture.md`'s "Stable, documented JSON contracts"
  design decision), the field names should be re-evaluated for
  readability (e.g. `temp_ambient` instead of `ta`) rather than carried
  forward purely for payload-size reasons that a versioned, cacheable
  REST API may not share with this legacy endpoint.

## 9. Open Questions

- The relationship between KAN-116's planned
  `/api/v1/contracts/{id}/history` endpoint and this page's
  `api_dashboard.php` is not yet defined in code: it's unclear from
  reading this repo alone whether the new endpoint will *replace*
  `api_dashboard.php`'s call from `dashboard.php`, run alongside it, or
  simply expose the same data under a different, token-authenticated
  contract (per `docs/architecture.md`'s "REST API (/api/v1)" section).
  This affects whether GAP2–GAP4 should be fixed on the legacy endpoint
  or only enforced going forward on the new one.
- GAP1 (the `map_timeline.js`/`index.php` sharing claim) could not be
  fully resolved from this story's scope alone — `index.php` itself is
  sibling story KAN-129's scope. It's possible `index.php` has its own,
  separate Leaflet implementation that supersedes the shared-component
  intent described in the file's header comment and
  `docs/architecture.md`; confirming that is this task's single biggest
  unresolved item and is explicitly handed to KAN-129/the person
  reconciling both docs.
- `config.php` is gitignored and not present in this checkout (per
  `docs/security.md`'s own scope note), so constants it might define
  that affect this area (DB connection details, any contract-level
  constant) could not be verified beyond what's visible in
  `api_dashboard.php`/`auth.php` themselves.
- Whether a maximum practical row count exists at the database/view
  (`V_DATA_WITH_CLIENT`) level — i.e. whether an extremely wide,
  unbounded date range (GAP2) could cause a slow query independent of the
  25000-point response sampling — cannot be determined from this repo
  alone (no index/EXPLAIN information available, view definition not in
  version control per the same `config.php`/DB-schema exclusion noted in
  `docs/security.md`'s scope statement).

## 10. Coverage Checklist

- [x] Every file in Scope (§2) cited by at least one CB-n.
  (`dashboard.php`: CB1–CB3, CB5, CB16–CB17, CB26–CB28;
  `api_dashboard.php`: CB4, CB6–CB15; `map_timeline.js`: CB18–CB25.)
- [x] Every authz/validation rule found is captured (as CB-n or GAP-n).
  (Session/login: CB1. Contract ownership: CB3–CB4. CSRF: CB4. Date
  format/order validation: CB6–CB7, GAP2–GAP4. Device-list scoping: CB9.
  Row-count ceiling: CB10. Enumeration-safe error messaging: CB3–CB4,
  GAP5.)
