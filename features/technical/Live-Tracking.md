# Technical-Feature: Live Tracking

> Reverse-engineered current behavior. See
> [`TECH-FEATURE.md`](../../sdlc/product/TECH-FEATURE.md) for the authoring
> rules this document follows. Produced for KAN-129 (epic KAN-126, "Technical
> Feature Discovery — cpaltracker.web"), feeding the planned React UI rebuild
> (KAN-60).

## 1. Feature Area

Live Tracking — the logged-in landing page (`index.php`) that lists the
user's accessible contracts, surfaces the client logo and the latest
global/per-contract announcement messages, and is the page from which the
user reaches the map/timeline view.

## 2. Scope (files / pages covered)

- `cpaltracker.web/index.php` — full file (497 lines): login gate, contract
  authorization, logo/message composition, the contract-selection form, the
  client-side message/dashboard-link wiring, the DATA table, and CSV export.
- `cpaltracker.web/map_timeline.js` (301 lines) — the shared Leaflet
  timeline/map module. **Deliberately excluded in depth:** per CB-11/GAP-1
  below, this module is not actually loaded or invoked by `index.php` in
  this checkout — it is wired up exclusively in `dashboard.php`. It is read
  here only to document (a) its own stated "LIVE view" framing and (b) the
  "current position at instant T" rendering logic it would provide if
  `index.php` did use it, since that logic defines what "live" means in this
  codebase. The historical/scrubbable-timeline behavior of this module as
  actually used by `dashboard.php` (slider-driven playback, per-device
  trails, the `charger()`/`api_dashboard.php` fetch flow) is a separate
  feature area and is not documented here.
- `cpaltracker.web/auth.php` — only the five functions `index.php` calls:
  `cpal_contrats_visibles`, `cpal_contrat_autorise`, `cpal_a_un_logo`,
  `cpal_dernier_message_global`, `cpal_derniers_messages_par_contrat`. Other
  `auth.php` internals (login, CSRF, session hardening, password tokens) are
  out of scope for this doc; see `docs/security.md` for their assessment.
- Deliberately excluded (explicit, data-table/export behavior, not live
  tracking): `index.php`'s DATA table rendering (pagination, per-row
  display) and its CSV export branch (`export_csv`, `V_DATA_WITH_CLIENT`
  query) are read for context (CB-19) but belong to a separate "Historical
  Data / Export" feature area, not analyzed in depth here.
- `cpaltracker.web/docs/security.md`, `cpaltracker.web/docs/design-and-standards.md`
  — read for cross-reference only, cited where they document a rule
  relevant to this area; not re-derived at length.
- Not read (out of this story's scope, noted for completeness):
  `client_logo.php` (the image endpoint `index.php` points at),
  `dashboard.php`/`api_dashboard.php` (the historical/timeline page that
  actually uses `map_timeline.js`), `pilotage.php`.

## 3. Current Behavior (CB-n)

- **CB-1.** `index.php` requires a valid session before any rendering:
  `$user = cpal_require_login($pdo)` redirects to `login.php` if there is no
  session user, or to `account.php?premiere=1` if
  `MustChangePassword === 1` (`index.php` line 38; `auth.php`
  `cpal_require_login`, ~lines 220–231).
- **CB-2.** Which contracts the user can even see is role-based:
  `cpal_contrats_visibles($pdo, $user)` returns every row of `T_CONTRATS`
  (joined to `T_CLIENTS`) for an admin, or only rows where
  `ClientID = $u['ClientID']` for a non-admin (`index.php` line 53;
  `auth.php` `cpal_contrats_visibles`, lines 278–297).
- **CB-3.** Any `contrat_id` received from the request is validated twice
  before being trusted: first `ctype_digit()` in `index.php` (else it is
  discarded and treated as "no contract selected", lines 80–88), then
  `cpal_contrat_autorise($pdo, $user, (int) $contratId)`, which re-fetches
  the contract from `T_CONTRATS` and returns it only if the caller is an
  admin or the contract's `ClientID` matches the user's own `ClientID`;
  otherwise it logs `acces_refuse` and returns `null` (`auth.php`
  `cpal_contrat_autorise`, lines 309–332). On `null`, `index.php` sets
  `$error` and resets `$contratId` to `''` (lines 82–85).
- **CB-4.** The client logo is gated server-side, not just by presence of a
  file: `cpal_a_un_logo($pdo, $user)` returns `false` unconditionally for an
  admin or for a user with no `ClientID`, and otherwise checks
  `SELECT 1 FROM T_CLIENTS WHERE ClientID = ? AND Logo IS NOT NULL`
  (`auth.php` lines 345–360). `index.php` only renders the
  `<img src="client_logo.php">` block (with an `onerror` handler that hides
  its parent on load failure) when this flag is true (`index.php` line 61,
  294–299).
- **CB-5.** The global announcement message is fetched and shown
  unconditionally for every logged-in user, independent of which contract
  (if any) is selected: `cpal_dernier_message_global($pdo)` selects the
  single most recent `T_MESSAGES` row where `ContratID IS NULL`
  (`auth.php` lines 364–383); `index.php` renders it once, right under the
  title/logo, whenever non-null (`index.php` line 65, 301–309).
- **CB-6.** Per-contract messages are only ever looked up for contracts the
  user is already allowed to see:
  `cpal_derniers_messages_par_contrat($pdo, array_column($contrats, 'ContratID'))`
  is called with the `$contrats` array that came from `cpal_contrats_visibles`
  (`index.php` lines 53, 66–69). The function's own docstring states it
  performs no access check of its own and requires its input to already be
  filtered by `cpal_contrats_visibles()`/`cpal_contrat_autorise()`
  (`auth.php` lines 390–396) — `index.php`'s call site satisfies that
  precondition.
- **CB-7.** For each contract, only the single latest message is returned:
  the SQL uses `ROW_NUMBER() OVER (PARTITION BY ContratID ORDER BY
  DateMessage DESC)` filtered to `rn = 1` (`auth.php` lines 406–414) — there
  is no message history exposed here, only "last message per contract", as
  the file's own top-of-file changelog states explicitly ("c'est le DERNIER
  message qui est montré, pas un historique", `index.php` lines 20–21).
- **CB-8.** The per-contract messages are serialized once, server-side,
  into a single JS object keyed by `ContratID` as a string (JS object keys
  are strings), with values already HTML-escaped for the date and content,
  using `JSON_HEX_TAG|JSON_HEX_AMP|JSON_HEX_APOS|JSON_HEX_QUOT|JSON_UNESCAPED_UNICODE`
  (`index.php` lines 70–76, 422–430).
- **CB-9.** Selecting a contract in the `<select id="contrat_id">` dropdown
  updates the per-contract message box client-side, without a page reload:
  `maj()` reads `MESSAGES_CONTRAT[selContrat.value]` and, if present, sets
  `msgDate.textContent`/`msgTexte.textContent` (never `innerHTML`) and shows
  the box; otherwise it hides the box (`index.php` lines 444–454, 468–494).
- **CB-10.** The same change handler builds the "Ouvrir le tableau de bord"
  link (`#lienDashboard`) purely client-side: it is hidden until a contract
  is chosen, then its `href` is set to
  `dashboard.php?contrat_id=...&date_from=...&date_to=...` using the
  currently selected contract and date filters (`index.php` lines 468–486).
  This is the only path out of `index.php` toward the map/timeline view.
- **CB-11.** `index.php` loads the Leaflet stylesheet
  (`https://unpkg.com/leaflet@1.9.4/dist/leaflet.css`, line 196) but does
  **not** load `leaflet.js`, does **not** load `map_timeline.js`, and
  declares no map/slider/tooltip DOM elements anywhere in the file
  (confirmed by searching the file for `leaflet`, `CpalTimelineMap`,
  `map_timeline`, `id="map"`, `mapId`, `sliderId`, `timeSlider` — the only
  match is the CSS `<link>` itself). No map of any kind — live or
  historical — is rendered on this page in this checkout.
- **CB-12.** `map_timeline.js` itself states, in its own header comment,
  that it is "Composant partagé entre la vue LIVE (index.php) et le
  dashboard (dashboard.php). Affiche un point par device : sa dernière
  position connue à l'instant T. Déplacer le curseur retrace le chemin
  parcouru." (lines 1–8) — i.e. the module's documented purpose is to be the
  live-tracking map for `index.php`, in addition to the dashboard.
- **CB-13.** `map_timeline.js` is in fact only loaded and instantiated by
  `dashboard.php` (`leaflet.js` line 193, `map_timeline.js` line 202,
  `CpalTimelineMap.create({mapId:'map', sliderId:'timeSlider', ...})` lines
  416–419) — `index.php` is not among its callers.
- **CB-14.** Within `map_timeline.js`, a device's current position at a
  given instant `T` is found by `dernierAvant()`, a binary search over that
  device's rows (sorted by `t`) for the last row with `t <= T`
  (lines 140–148); only rows with `r.p === 1 && r.la !== null && r.lo !==
  null` are kept per device in `parDevice` in the first place — unpositioned
  rows never produce a marker (lines 96–107, 203–204).
- **CB-15.** `setData()` initializes the slider/instant to `tMax`, the
  latest timestamp across all loaded rows, so the map's first render always
  shows each device's latest available position — this is the "current
  position" snapshot referenced in CB-12 (lines 109–126).
- **CB-16.** `setVisibleDevices(list)` filters which devices render,
  entirely client-side against data already fetched — `null` means "show
  all", otherwise only chip IDs in the given list pass `estVisible()`
  (lines 128–135, 198–199, 245–246). No new query is made when the visible
  set changes.
- **CB-17.** Each marker's fill color encodes the device's last measured
  mass via `couleurMasse()` (`< 200` → red, `< 500` → orange, else green,
  `null`/`NaN` → gray); the marker's border color is a fixed per-device
  color assigned from a 10-entry palette in device-first-seen order
  (`map_timeline.js` lines 27–39, 94–102, 215–224).
- **CB-18.** Each marker's tooltip (`etiquette()`) shows date (formatted as
  explicit UTC via `formatUTC()`), position, ambient temperature, humidity,
  mass, acceleration and MPU temperature, plus batch/lot fields if a hook
  covers that instant (`lotA()` matches `h.c === chip && t >= h.from && t <
  h.to`); the HTML is built by string concatenation and escaped per-value
  via `echapper()` before being passed to Leaflet's `bindTooltip()`
  (lines 54–57, 150–184, 226–229) — not assigned through `innerHTML` on
  page-level DOM.
- **CB-19.** (Context, not analyzed in depth — see §2.) `index.php` also
  renders a paginated DATA table and a CSV export (`export_csv` branch)
  for the selected, already-authorized contract, gated behind the same
  CSRF check (`cpal_csrf_ok`) as the table execution, both read from
  `V_DATA_WITH_CLIENT` (`index.php` lines 107–187). This is unrelated to
  live-position rendering and is listed here only because it lives in the
  same file.

## 4. Current Data Model & Contracts

- `T_CONTRATS` (`ContratID`, `ContratRef`, `ContratType`, `ClientID`,
  `DebutContrat`, `FinContrat`) and `T_CLIENTS` (`ClientID`, `Nom`, `Logo`)
  — read via `cpal_contrats_visibles`/`cpal_contrat_autorise`/`cpal_a_un_logo`
  (`auth.php` lines 278–360).
- `T_MESSAGES` (`MessageID`, `ContratID` nullable, `DateMessage`, `Contenu`)
  — read via `cpal_dernier_message_global`/`cpal_derniers_messages_par_contrat`
  (`auth.php` lines 364–414). A `NULL` `ContratID` means "global message".
- Session state consumed: `$user` array from `cpal_require_login` (at least
  `UserID`, `Mail`, `Role`, `ClientID`, `MustChangePassword` are read
  across the functions in scope — `index.php`/`auth.php` throughout).
- Client-side contract: `MESSAGES_CONTRAT` — a JS object
  `{ [ContratID as string]: { date: 'dd/mm/YYYY HH:mm', contenu: string } }`,
  injected server-side as pre-escaped JSON (`index.php` lines 70–76,
  422–430).
- The row/hook shape `map_timeline.js` expects (relevant to "current
  position" rendering even though `index.php` does not currently produce
  it — see CB-11/CB-13): rows `{ c (ChipID), t (epoch ms), p (1 = has
  position), la, lo, ta, hr, m, g, tm }`; hooks
  `{ c, from, to, cust, lot, per, nb }` (`map_timeline.js` lines 78–107,
  150–178). This shape is defined only by this file's inline usage; no
  schema for it is documented elsewhere in the files read for this story.

## 5. Current Integration Points

- Database (read): `T_CONTRATS`, `T_CLIENTS`, `T_MESSAGES` via PDO prepared
  statements, all through `auth.php`'s five functions called from
  `index.php` (see §3/§4).
- `client_logo.php` (read, not analyzed): referenced as the `<img src>`
  for the client logo block whenever `cpal_a_un_logo()` is true
  (`index.php` line 296); its own access control was not verified in this
  story's scope (see §9).
- `dashboard.php` (write via navigation only): `index.php` builds and
  exposes a link to it client-side, passing `contrat_id`/`date_from`/
  `date_to` as query parameters (CB-10); no data flows back from it into
  `index.php`.
- `pilotage.php` (write via navigation only, conditional): a link is shown
  client-side only for a selected contract whose `ContratType === 'MAIN'`;
  `index.php`'s own comment states the real authorization check happens in
  `pilotage.php`, not here (`index.php` lines 28–30, 278–283).
- CDN assets loaded by `index.php`: `unpkg.com/leaflet@1.9.4/dist/leaflet.css`,
  `cdnjs.cloudflare.com` Font Awesome 6.0.0 (`index.php` lines 196–197) —
  see GAP-2 regarding the Leaflet CSS specifically.
- Tile server (reached only via `dashboard.php`, not `index.php`):
  `{s}.tile.openstreetmap.org` (`map_timeline.js` lines 69–72).

## 6. Edge Cases Already Handled

- A `contrat_id` that isn't all-digits is silently treated as "no contract
  selected" rather than erroring (`index.php` lines 80–88).
- A `contrat_id` that is well-formed but not authorized for this user
  produces a user-facing error and clears the selection, rather than
  leaking which contracts exist (`index.php` lines 81–85; `auth.php`
  `cpal_contrat_autorise` lines 309–332).
- A user with zero visible contracts gets an explicit "no contract
  attached" message and the entire selection form is not rendered
  (`index.php` lines 216–221).
- Admin-only `ClientNom` disclosure in the contract dropdown: the client
  name next to a contract is shown only when `cpal_is_admin($user)` is true
  (`index.php` lines 235–237).
- `cpal_derniers_messages_par_contrat` returns an empty array (not an
  error) when given an empty `$contratIds` list, avoiding a malformed `IN
  ()` query (`auth.php` lines 397–399).
- In `map_timeline.js`: a row with `r.p !== 1` or null `la`/`lo` is never
  added to `parDevice`, so it can never produce a marker (lines 104–106).
- In `map_timeline.js`: if no rows are loaded at all, `tMin = tMax =
  Date.now()` and the slider is disabled, rather than operating on
  `undefined` bounds (lines 109–121).
- In `map_timeline.js`: a device whose first positioned row is after the
  current instant (`dernierAvant` returns `-1`) is skipped for that render
  — no marker is drawn for a device with no data yet "at" `T` (lines
  140–148, 203–204).

## 7. Gaps (GAP-n)

- **GAP-1.** `map_timeline.js`'s own header comment documents it as the
  shared component for "la vue LIVE (index.php)" (CB-12), but `index.php`
  never loads `leaflet.js` or `map_timeline.js` and declares no map/slider
  DOM elements (CB-11). The only concrete wiring of this module found in
  this checkout is `dashboard.php`'s historical/timeline mode (CB-13). The
  module's own stated intent — to serve as `index.php`'s live view — is not
  met by the current code.
- **GAP-2.** As a direct consequence of GAP-1, `index.php` loads the
  Leaflet CSS stylesheet from a third-party CDN on every page view
  (`index.php` line 196) without ever using Leaflet on that page — an
  unused dependency fetched unconditionally for every logged-in user. This
  also inherits the no-SRI CDN risk already documented generally in
  `docs/security.md` ("Reliance on three third-party CDNs... with no
  Subresource Integrity... checked in `dashboard.php`/`index.php`/etc."),
  for a load that in `index.php`'s case serves no function at all.
- **GAP-3.** Nothing in the files read for this story — `index.php`,
  `map_timeline.js`, or `dashboard.php`'s wiring of it — contains a
  `setInterval` or any other polling/auto-refresh mechanism (confirmed by
  searching all three files). `map_timeline.js` calls itself and is called
  by `dashboard.php` as the "LIVE" view and documents showing "sa dernière
  position connue à l'instant T" (CB-12), yet the only way `T`'s underlying
  data advances is a user-initiated action (a button click in
  `dashboard.php`, or — per GAP-1 — nothing at all in `index.php`). The
  module's own "LIVE" framing is inconsistent with the one-shot,
  manually-triggered data loading actually implemented.
- **GAP-4.** `docs/design-and-standards.md` documents, as a known deviation
  from `cto-tools`'s Review Gate, that page files violate SRP because
  "every page file does auth-check + data access + HTML rendering + ...
  embedding JS config" with business logic living inline rather than in a
  separate layer. `index.php` is a direct, concrete instance of this
  already-documented pattern: contract-scoping (CB-2/CB-3), logo/message
  composition (CB-4–CB-8) and client-side URL-building for the dashboard
  link (CB-10) are all interleaved with HTML output in one 497-line
  top-level script, with no function decomposition inside `index.php`
  itself (the decomposition that exists — `cpal_contrats_visibles`,
  `cpal_contrat_autorise`, etc. — lives in `auth.php`, not in `index.php`).

## 8. Suggested Improvements (IMP-n)

> The following are this author's recommendations, not current behavior.

- **IMP-1.** (Addresses GAP-1.) Before the React rebuild (KAN-60) carries
  this forward, make an explicit decision about which page owns the
  "live"/current-position map: either build it into `index.php` as the
  module's own comment implies was intended, or update `map_timeline.js`'s
  header comment to stop describing `index.php` as a consumer, so the
  React rebuild isn't designed against a stale claim.
- **IMP-2.** (Addresses GAP-2.) Drop the unconditional Leaflet CSS
  `<link>` from `index.php` unless/until that page actually renders a map,
  removing one unused third-party CDN round-trip from every page view.
- **IMP-3.** (Addresses GAP-3.) If "live" is meant to mean "continuously
  updating" rather than "latest snapshot on demand", add an explicit
  refresh mechanism (e.g. a periodic fetch against the `/api/v1/devices/positions`
  endpoint KAN-116 is extracting) instead of relying on a full page reload
  or a manual re-fetch button. This would also give the React rebuild a
  natural seam to build a true live-polling view on top of.
- **IMP-4.** (Addresses GAP-4.) When extracting this area for the React
  rebuild, pull the contract-scoping/logo/message composition that
  currently lives inline in `index.php` (lines 51–76) into a dedicated
  function/service in the style `auth.php` already uses, consistent with
  the refactor `docs/design-and-standards.md` already recommends generally
  for this codebase.
- **IMP-5.** Document the row/hook JSON shape `map_timeline.js` consumes
  (`r.c/r.t/r.p/r.la/r.lo/r.ta/r.hr/r.m/r.g/r.tm`, `h.c/h.from/h.to/h.cust/h.lot/h.per/h.nb`
  — CB-14/CB-18/§4) as a formal schema, ideally alongside the KAN-116 REST
  endpoint's response shape, rather than leaving it to be inferred from
  this file's inline logic — this is the shape the React rebuild's map
  component will need to reproduce or replace.

## 9. Open Questions

- `client_logo.php` is out of this story's scope; whether it independently
  re-validates the session/`ClientID` or simply trusts being reached from
  a page that already gated on `cpal_a_un_logo()` could not be confirmed by
  reading only the files in scope here.
- Whether `map_timeline.js`'s "vue LIVE (index.php)" framing (GAP-1)
  reflects a feature that existed in an earlier version of `index.php` and
  was later removed, or one that was planned but never wired up, could not
  be determined from the current code — `index.php`'s own changelog header
  only documents schema-v4 (logo/messages) changes, not any map removal.
- KAN-116 ("REST API Extraction") is in progress and is additively
  extracting an `/api/v1/devices/positions` endpoint from `index.php`'s
  underlying data, stated to be with no behavior change to the existing
  page. This doc describes `index.php`'s behavior as checked out for this
  story and does not speculate on how that endpoint will be consumed by
  the eventual React rebuild (KAN-60).
- `dashboard.php`/`api_dashboard.php` (the actual consumers of
  `map_timeline.js`) were read only far enough to confirm CB-13/GAP-1/GAP-3
  (presence of `setInterval`, the `charger()` fetch flow, the
  `CpalTimelineMap.create()` call site); their full behavior belongs to a
  separate feature area's documentation, not this one.

## 10. Coverage Checklist

- [x] Every file in Scope (§2) cited by at least one CB-n.
- [x] Every authz/validation rule found is captured (as CB-n or GAP-n).
