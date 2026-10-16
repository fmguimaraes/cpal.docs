# Feature: React Live Tracking (cpaltracker.web rebuild)

> Forward-looking product spec for the **Live Tracking** feature-area child epic
> of KAN-60 (React/TypeScript rebuild of `cpaltracker.web`). This is a **new,
> additive frontend** consuming the existing KAN-116 `/api/v1` REST layer; it
> does not modify the PHP monolith, the backend, or the database.
>
> Ground truth and citations:
> - Current-behavior reverse-engineering: [`technical/Live-Tracking.md`](technical/Live-Tracking.md) (CB-n / GAP-n / IMP-n).
> - KAN-60 readiness finding: [`technical/CONSOLIDATED-Coverage-and-Gaps.md`](technical/CONSOLIDATED-Coverage-and-Gaps.md) §5 "Readiness for KAN-60" → *Live Tracking* bullet.
> - Stack: [`../architecture/react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md).
>
> Single Source of Truth: requirement text (FR/NFR/AC IDs, edge cases, KPIs)
> lives only here. Epics and stories reference these IDs; they never copy the
> text.

---

## 1. Feature Name

**React Live Tracking** — a last-known-position device map for a selected
contract, delivered in the new React frontend.

> **Framing note (critical).** Per `CONSOLIDATED-Coverage-and-Gaps.md` §5 and
> `Live-Tracking.md` GAP-1/GAP-2/GAP-3: the current `index.php` **does not render
> a working map at all today**. `map_timeline.js`'s header comment calls itself
> the "vue LIVE (index.php)" (CB-12), but that module is only ever wired into
> `dashboard.php` (CB-13); `index.php` loads only the Leaflet *stylesheet* and no
> map/JS/DOM (CB-11), and there is **no polling or auto-refresh anywhere** in the
> current code (GAP-3). Therefore this feature is a **NEW capability built from
> scratch, not a replication of current behavior** — there is nothing live to
> replicate. Where CB-n items are cited below, they describe reusable *data and
> presentation rules* (contract scoping, marker colouring, tooltip contents) that
> happen to already exist in `map_timeline.js`/`index.php`, not a live map that
> was ever shipped.

## 2. Problem / Pain Point

Operators and client users of c-PAL need to see, at a glance, **where their
monitored devices currently are** for a given contract, together with each
device's latest readings (mass, temperature, humidity, etc.). Today there is no
such view: the legacy `index.php` advertises a "live" map (via
`map_timeline.js`'s comment) but renders none (GAP-1), so the only way to see
device positions at all is to leave `index.php` and open the historical
`dashboard.php` timeline (CB-10, CB-13). Users experience a feature that is
documented and implied by the UI's Leaflet CSS load (CB-11, GAP-2) yet does not
exist. The KAN-60 React rebuild is the opportunity to deliver the current-position
view that the product has always implied but never shipped.

## 3. Goal / Outcome (Success Definition)

A logged-in user selects one of the contracts they are authorized to see and
immediately gets a map showing one marker per device at its **last-known
position**, colour-coded by latest mass, with a tooltip of latest readings. The
snapshot is clearly labelled with the time it was captured, and the user can
**manually refresh** it on demand to pull newer last-known positions from
`/api/v1`. The user understands this is a periodically-refreshed snapshot, not a
continuously-updating real-time feed. From this view the user can jump to the
full History Dashboard for the same contract. The PHP monolith continues to
serve traffic unchanged the entire time.

## 4. User Stories (Primary Scenarios)

- **US-1.** As an **authenticated client user**, I want to pick one of my
  contracts and see all its devices' latest positions on a map, so that I know
  where my monitored shipments/assets currently are without opening the
  historical dashboard. *(FR1, FR2, FR3, FR4, FR15, NFR1, NFR2, NFR4, NFR6)*
- **US-2.** As a **c-PAL admin**, I want the contract selector to show every
  contract (with its client name) and the map to respect the same authorization
  the API enforces, so that I can monitor any client while non-admins still only
  see their own. *(FR2, FR15, NFR2, NFR6)*
- **US-3.** As an **operator reading the map**, I want each device marker
  colour-coded by its latest mass and a tooltip of its latest readings
  (date in UTC, position, ambient temperature, humidity, mass, acceleration, MPU
  temperature, and batch/lot when applicable), so that I can triage device state
  at a glance. *(FR5, FR6, FR7, FR8, NFR6, NFR7)*
- **US-4.** As a **user watching an active contract**, I want a "last updated"
  timestamp and a manual Refresh control, so that I can pull the newest
  last-known positions on demand and always know how fresh the snapshot is.
  *(FR9, FR10, NFR5)*
- **US-5.** As a **client user opening the tracker**, I want to see the global
  announcement, my contract's latest message, and my client logo, and to be able
  to open the History Dashboard for the selected contract, so that the live view
  carries the same context and navigation the legacy landing page did.
  *(FR11, FR12, FR13, FR14, NFR4, NFR6)*
- **US-6.** As the **product/QA team**, I want this feature covered by unit and
  end-to-end tests and shipped in the agreed stack without touching the backend
  or the PHP monolith, so that it is verifiable and non-disruptive.
  *(FR1, FR9, NFR1, NFR2, NFR3, NFR8, NFR9, NFR10)*

## 5. In Scope (What WILL be delivered)

- A new `/live` (working title) route in the React app behind authentication,
  wired through the Core epic's `AuthContext`/REST client (FR1).
- A contract selector populated from `/api/v1`, scoped to the contracts the
  authenticated user may see; admins additionally see the client name (FR2).
- A **new** Leaflet / react-leaflet map that renders one marker per device at its
  last-known position for the selected contract, sourced from `/api/v1` (FR3,
  FR4) — explicitly new functionality, not a port of a working legacy map.
- Marker fill colour from latest mass thresholds and a stable per-device border
  colour (FR5, FR6), replicating the colouring *rules* in CB-17.
- A marker tooltip of latest readings, rendered as text (never raw HTML), with
  UTC date formatting and batch/lot fields when a hook covers the instant (FR7),
  per the field set in CB-18.
- A client-side device-visibility filter (show all / a chosen subset) that does
  not refetch (FR8), per CB-16.
- A manual **Refresh** control plus a visible "last updated" timestamp (FR9,
  FR10). Refresh re-calls the same read endpoint; it is user-initiated.
- Global announcement message, per-contract latest message, and gated client
  logo, carried over from CB-5/CB-6/CB-7/CB-8/CB-9/CB-4 (FR11, FR12, FR13).
- A link to the History Dashboard for the selected contract, passing contract and
  date parameters (FR14), per CB-10.
- Empty / error states: no visible contracts, selected contract with no
  positioned devices, and unauthorized-contract handling (FR15).
- Dark/light theming via the Core design-system tokens; desktop-first responsive
  layout (NFR4).
- Vitest + Testing Library unit tests and Playwright E2E tests for the above
  (NFR8, NFR9).

## 6. Out of Scope (What WILL NOT be delivered)

- **Any backend or database change.** The feature consumes KAN-116 `/api/v1`
  as-is. If a required read (e.g. a last-known-positions endpoint) is missing or
  underspecified, it is logged as an open question / separate backend story — it
  is **never** worked around by the frontend (see Risks R-1; stack decision §5).
- **Adding a realtime or server-side polling endpoint.** That is backend work and
  is out of scope here.
- **True real-time push updates.** There is no realtime backend endpoint;
  `socket.io-client` is held/not adopted per the stack decision. Freshness comes
  only from user-initiated refresh.
- **Any change to the `cpaltracker.web` PHP monolith.** `index.php`,
  `dashboard.php`, `map_timeline.js` and all other PHP pages keep running
  unmodified in parallel; non-disruption is an acceptance condition (NFR3).
- **Mobile-first design.** Desktop is the primary, first-class target; mobile is
  best-effort and not separately acceptance-tested in this epic (NFR4).
- **The historical / scrubbable timeline** (time-slider, trail replay, auto-fit) —
  that is the History Dashboard feature-area's responsibility, not this one.
- **The paginated DATA table and CSV export** that share `index.php` today
  (CB-19) — out of this feature area.
- **Device Pilotage controls** — only a conditional navigation link may be shown;
  pilotage itself is its own feature area.

## 7. User Flow / UX Breakdown

1. User navigates to the Live Tracking route. System checks auth via
   `AuthContext`; if unauthenticated it redirects to login (FR1).
2. System fetches the user's visible contracts from `/api/v1` and renders the
   contract selector (admins also see client name) plus the global announcement
   message (FR2, FR11). If the user has zero visible contracts, the system shows
   an explicit "no contract attached" empty state and renders no map or selector
   (FR15).
3. User selects a contract. System:
   - fetches that contract's devices' last-known positions from `/api/v1` and
     renders markers (FR3, FR4);
   - shows the per-contract latest message, if any (FR12);
   - reveals the "Open History Dashboard" link with contract/date params (FR14);
   - shows the client logo if the API's logo flag is true (FR13);
   - records and displays a "last updated" timestamp (FR10).
4. User hovers/activates a marker. System shows the readings tooltip (FR7).
5. User toggles device visibility. System filters markers client-side with no
   refetch (FR8).
6. User clicks **Refresh**. System re-fetches last-known positions for the
   current contract and updates markers and the "last updated" timestamp (FR9,
   FR10).
7. User clicks "Open History Dashboard". System navigates to the History
   Dashboard route for the selected contract (FR14).
8. If the user's selection resolves to an unauthorized contract (e.g. tampered
   id), the system shows a non-leaking error and clears the selection (FR15).

## 8. Functional Requirements (FRs)

- **FR1.** The system SHALL require an authenticated session (via the Core
  `AuthContext`/REST client against `/api/v1`) before rendering Live Tracking,
  and SHALL redirect unauthenticated users to login. *(replaces CB-1's
  `cpal_require_login` gate, client-side)*
- **FR2.** The system SHALL populate the contract selector only with contracts
  the authenticated user is authorized to see as returned by `/api/v1` (all
  contracts for an admin, own-client contracts otherwise), and SHALL display the
  client name alongside each contract only for admin users. *(CB-2, CB-3; admin
  disclosure per §6 edge case)*
- **FR3.** The system SHALL, when a contract is selected, request that contract's
  devices' last-known positions from `/api/v1` and drive the map from that
  response. *(new; analogous to CB-15's "latest position" snapshot)*
- **FR4.** The system SHALL render a Leaflet / react-leaflet map showing exactly
  one marker per device that has a valid last-known position, and SHALL omit
  devices with no positioned reading. *(new capability; CB-14 positioning rule,
  CB-11/GAP-1 confirm no such map exists today)*
- **FR5.** The system SHALL set each marker's fill colour from the device's
  latest mass: `< 200` → red, `< 500` → orange, otherwise green, and
  `null`/`NaN` → grey. *(CB-17)*
- **FR6.** The system SHALL assign each device a stable border colour from a fixed
  palette in a deterministic order, consistent across refreshes within a session.
  *(CB-17)*
- **FR7.** The system SHALL show, per marker, a tooltip containing: date formatted
  explicitly as UTC, position, ambient temperature, humidity, mass, acceleration
  and MPU temperature, plus batch/lot fields when a hook covers that instant; all
  values SHALL be rendered as text, never as raw/unsanitized HTML. *(CB-18)*
- **FR8.** The system SHALL provide a client-side device-visibility filter where
  `show all` or a chosen subset of devices is rendered, applied against
  already-fetched data with no additional network request. *(CB-16)*
- **FR9.** The system SHALL provide a user-initiated **Refresh** control that
  re-fetches the current contract's last-known positions from `/api/v1` and
  updates the map. There SHALL be no WebSocket, server push, or background
  auto-polling. *(new; addresses GAP-3's absence of any refresh mechanism within
  the no-realtime-backend constraint)*
- **FR10.** The system SHALL display a human-readable "last updated" timestamp
  reflecting when the currently-shown snapshot was fetched, and SHALL update it on
  every successful refresh. *(new)*
- **FR11.** The system SHALL fetch and display the single most recent global
  announcement message (contract-independent) for every logged-in user, when one
  exists. *(CB-5)*
- **FR12.** The system SHALL display the single latest message for the selected
  contract when one exists, and SHALL hide the message box otherwise; message
  values SHALL be rendered as text/sanitized content, never via raw `innerHTML`.
  *(CB-6, CB-7, CB-8, CB-9)*
- **FR13.** The system SHALL render the client logo only when `/api/v1` reports
  the authenticated user's client has a logo, and SHALL gracefully hide the logo
  element on load failure. *(CB-4)*
- **FR14.** The system SHALL expose a navigation link to the History Dashboard for
  the selected contract, passing the contract identifier and the active date
  range, and SHALL hide it until a contract is selected. *(CB-10)*
- **FR15.** The system SHALL handle, with explicit non-leaking UI states: (a) a
  user with zero visible contracts (explicit empty state, no selector/map); (b) a
  selected contract with no positioned devices (empty-map state); (c) a selection
  that resolves to an unauthorized or malformed contract (clear selection, show a
  generic error that does not reveal which contracts exist). *(§6 edge cases of
  `Live-Tracking.md`: CB-3 handling, zero-contracts, GAP-safe rendering)*

## 9. Non-Functional Requirements (NFRs)

- **NFR1.** The feature SHALL be built in the agreed stack — React 18.3 +
  TypeScript 5.5 on Vite 5, React Router v6, Zustand for state, Tailwind CSS 4,
  Leaflet / react-leaflet for the map — per the stack decision doc. *(US-1, US-6)*
- **NFR2.** The feature SHALL consume only KAN-116 `/api/v1` and SHALL introduce
  no backend, API-contract, or database change; a missing/insufficient endpoint is
  logged as an open question, not worked around. *(US-1, US-2, US-6; stack §5)*
- **NFR3.** The feature SHALL cause no disruption to the existing PHP monolith;
  `cpaltracker.web` PHP pages continue to serve traffic unmodified, side by side
  with the new frontend. *(US-6; stack §5)*
- **NFR4.** The feature SHALL use the Core design-system theme tokens, ship in both
  dark and light themes via a single theme switch, and target desktop as the
  first-class responsive layout (mobile best-effort). *(US-1, US-3, US-5; stack §4)*
- **NFR5.** The map SHALL render the initial markers for a selected contract within
  a reasonable interaction budget (target: markers visible < 1.5 s after the
  `/api/v1` response resolves on a typical desktop), and a manual refresh SHALL
  not block the UI thread. *(US-4)*
- **NFR6.** The feature SHALL rely on `/api/v1`'s session/cookie auth and server-
  side authorization (never trusting a client-held role), SHALL not leak the
  existence of contracts a user may not see, and SHALL render all API-sourced text
  (messages, tooltip values) as text/sanitized content (rehype-sanitize for any
  markdown), never as raw HTML. *(US-1, US-2, US-3, US-5; CB-3, CB-9, CB-18)*
- **NFR7.** Marker colours, tooltips and controls SHALL be accessible: keyboard-
  reachable controls, and colour choices that meet contrast requirements in both
  themes (colour SHALL NOT be the only signal for device state). *(US-3)*
- **NFR8.** The feature SHALL ship with Vitest + Testing Library unit tests
  covering the colour-mapping, visibility-filter, tooltip-formatting, message/logo
  gating, and empty/error-state logic. *(US-6)*
- **NFR9.** The feature SHALL ship with Playwright E2E tests covering the primary
  flow (login-gated access → select contract → markers render → refresh updates
  "last updated" → open History Dashboard) against a running build. *(US-6)*
- **NFR10.** The feature SHALL NOT adopt `socket.io-client` or any realtime
  transport; freshness is delivered only by user-initiated refresh. *(US-6; stack
  §2 "Realtime — held, not adopted")*

## 10. Data Inputs & Outputs

**Inputs (read-only, all from `/api/v1`):**

- Authenticated user/session context (role, client scope) via the Core auth
  client — consumed, not stored by this feature. *(CB-1 state: role, client id)*
- Visible contracts list, via `GET /api/v1/contracts/list.php` (KAN-158,
  added for this story — see Risks R-1): `{ contracts: [{ contractId,
  contractRef, contractType, clientId?, clientName? }] }`. `clientId`/
  `clientName` are present only for an admin caller — omitted from every
  row entirely for a non-admin, enforced server-side by the endpoint
  itself. *(CB-2; `T_CONTRATS`/`T_CLIENTS` via `cpal_contrats_visibles()`)*
- Device last-known positions for a contract. The presentation layer expects, per
  device row, the shape established by `map_timeline.js` (CB-14/CB-18, §4 of
  `Live-Tracking.md`): `{ c (ChipID), t (epoch ms), p (1 = has position), la, lo,
  ta, hr, m, g, tm }`, plus batch/lot hooks `{ c, from, to, cust, lot, per, nb }`.
  The exact `/api/v1` response shape is the KAN-116 contract (see Risks R-1).
- Global announcement message and per-contract latest message:
  `GET /api/v1/messages/global.php` and `GET /api/v1/messages/contract.php?contractId=…`
  (KAN-165 backend exception — see
  `architecture/react-frontend-stack-decision.md` §14), each responding
  `{ message: { date, contenu } | null }`, `date` as ISO-8601 UTC.
  *(CB-5, CB-7, CB-8)*
- Client logo availability flag (`GET /api/v1/clients/logo-flag.php`,
  `{ hasLogo: boolean }`, KAN-165 backend exception, §14) + logo image
  endpoint (`GET /api/v1/clients/logo.php`, KAN-124). *(CB-4)*

**Outputs (UI only):**

- A rendered map with coloured, tooltipped device markers (FR4–FR7).
- A "last updated" timestamp (FR10).
- Message boxes (global + per-contract) and the client logo (FR11–FR13).
- A History Dashboard navigation link carrying `contrat_id`/`date_from`/`date_to`
  (FR14).

No data is written back to any backend by this feature.

## 11. Integration Points

- **KAN-116 `/api/v1` (read-only):** contracts, device last-known positions,
  messages (global + per-contract), and client-logo flag/image. All reads; no
  writes. Dependency: KAN-116 (Done).
- **Core epic (consumed):** `AuthContext`/REST client, design-system theme tokens,
  routing shell, shared layout.
- **OpenStreetMap tile server** (`{s}.tile.openstreetmap.org`) via Leaflet, for
  base map tiles — third-party read dependency. *(CB tile usage; GAP-2 notes the
  legacy no-SRI CDN risk — the React app SHALL load Leaflet assets from its own
  bundle, not an unused CDN `<link>`.)*
- **History Dashboard feature area (navigation out):** the "Open History
  Dashboard" link targets that route with contract/date params (FR14). *(CB-10)*
- **Device Pilotage feature area (conditional navigation out, optional):** a link
  may be shown for a selected `MAIN`-type contract; authorization for pilotage is
  enforced by that area/API, not here. *(CB-19 context; `index.php` pilotage link)*

## 11a. Software Items (IEC 62304 §5.3)

**N/A.** Per `sdlc/README.md` → "Applicability to c-PAL today", the Software Items
(IEC 62304) / W6 IQ-OQ-PQ row is **Not applicable** — c-PAL has not entered a
regulated software scope. No Software Item Registry exists yet; this section is
intentionally left empty until that row moves to *Applies*.

## 12. Edge Cases & Constraints

- A malformed or non-numeric contract identifier is treated as "no contract
  selected" rather than erroring. *(CB-3 / §6)*
- A well-formed but unauthorized contract yields a generic error and clears the
  selection, without revealing which contracts exist. *(CB-3 / §6, FR15, NFR6)*
- A user with zero visible contracts gets an explicit empty state and no
  selector/map. *(§6, FR15)*
- A contract whose devices have no positioned readings renders an empty map with a
  clear "no positions to show" state, not a crash. *(CB-14/§6, FR15)*
- Device rows lacking a valid position (`p !== 1` or null `la`/`lo`) never produce
  a marker. *(CB-14, §6)*
- A device whose only readings are all unpositioned is simply absent from the map.
  *(§6)*
- Tooltip and message content from the API is always rendered as text/sanitized;
  no value is assigned via raw `innerHTML`. *(CB-9, CB-18, NFR6)*
- `/api/v1` errors/timeouts on contract load or refresh surface a non-blocking
  error state and leave the previous snapshot (with its older "last updated"
  timestamp) visible where possible. *(FR9, FR10)*
- The map must degrade gracefully if OSM tiles fail to load (markers/tooltips
  remain usable).

## 13. Metrics / KPIs

- Contract-to-markers success rate (selecting an authorized contract renders
  markers or a correct empty state) > 99%.
- Median time from `/api/v1` positions response to markers visible < 1.5 s on a
  typical desktop. *(NFR5)*
- Manual-refresh success rate (refresh updates markers + "last updated") > 99%.
- Zero reported authorization leaks (no user ever sees a contract or device
  outside their scope). *(NFR6)*
- Unit + E2E coverage: all FRs and NFRs exercised; E2E primary-flow suite green in
  CI. *(NFR8, NFR9)*
- Zero regressions/incidents attributable to the PHP monolith during rollout.
  *(NFR3)*

## 14. Risks & Mitigations

- **R-1 — `/api/v1` may not expose a last-known-position endpoint.** Resolved,
  not real: `GET /api/v1/devices/positions.php` (KAN-121) already exists,
  is bearer-authenticated, and is scoped server-side via
  `cpal_contrats_visibles()`/`cpal_contrat_autorise()` — confirmed by
  reading `tracking_logic.php` and the endpoint itself before KAN-158 began
  (see `react-frontend-stack-decision.md` for the full citation trail).
  What *was* genuinely missing, found during KAN-158: no endpoint listed a
  user's authorized **contracts** (as opposed to positions), and no
  endpoint anywhere returned a client name — `positions.php`'s no-filter
  branch only covers contracts that already have at least one positioned
  device, which cannot serve as a general-purpose contract selector.
  *Resolution (deviation from the "log a separate backend story, never
  work around it" default below):* the product owner explicitly
  authorized one new, additive-only endpoint for this specific gap —
  `GET /api/v1/contracts/list.php` (KAN-158) — built by reusing
  `cpal_contrats_visibles()` exactly as `index.php`'s own selector and
  `positions.php` already do, with no change to any existing file's
  behavior. See `react-frontend-stack-decision.md` for the full design
  rationale and the PHPUnit coverage added alongside it. The epic's
  general "no backend change" constraint and this spec's own default
  mitigation (log and defer) still apply to every *other* gap found in
  this feature area — this was a one-off, explicitly authorized exception,
  not a change to that rule.
- **R-2 — Users expecting "real-time".** The legacy "LIVE" naming (CB-12) sets a
  false expectation the product never delivered. *Mitigation:* label the view as a
  refreshable snapshot with an explicit "last updated" time; never call it
  real-time; defer true push to a future epic (NFR10, FR10).
- **R-3 — Scope creep into the historical timeline.** `map_timeline.js` mixes live
  and scrubbable-history behavior. *Mitigation:* this area delivers last-known
  position only; the time-slider/trail is the History Dashboard area (§6).
- **R-4 — Disruption to the PHP monolith.** *Mitigation:* additive new frontend,
  no PHP touched; non-disruption is an acceptance condition (NFR3).
- **R-5 — Third-party tile/CDN dependency.** *Mitigation:* bundle Leaflet assets
  with the app (avoid the legacy unused-CDN pattern of GAP-2); degrade gracefully
  on tile failure (§12).
- **R-6 — Security/authorization in a client app.** *Mitigation:* trust only
  server-side `/api/v1` authorization; never gate on a client-held role; sanitize
  all rendered API content (NFR6).

## 15. Acceptance Criteria (Definition of Done)

- [ ] Unauthenticated access to the Live Tracking route redirects to login. *(FR1)*
- [ ] Contract selector lists only authorized contracts; admins additionally see
      the client name; non-admins never see other clients' contracts. *(FR2, NFR6)*
- [ ] Selecting a contract fetches last-known positions from `/api/v1` and renders
      one marker per positioned device. *(FR3, FR4)*
- [ ] Devices without a valid position produce no marker. *(FR4)*
- [ ] Marker fill colour follows the mass thresholds (<200 red, <500 orange, else
      green, null/NaN grey). *(FR5)*
- [ ] Each device has a stable per-device border colour consistent across
      refreshes in a session. *(FR6)*
- [ ] Marker tooltip shows UTC date, position, ambient temp, humidity, mass,
      acceleration, MPU temp, and batch/lot when applicable, all as text. *(FR7)*
- [ ] Device-visibility filter shows all / a chosen subset with no refetch. *(FR8)*
- [ ] A Refresh control re-fetches positions and updates the map; no WebSocket or
      background auto-polling is present. *(FR9, NFR10)*
- [ ] A "last updated" timestamp is shown and updates on each successful refresh.
      *(FR10)*
- [ ] Global announcement message is shown when present. *(FR11)*
- [ ] Per-contract latest message is shown/hidden correctly and rendered as text.
      *(FR12)*
- [ ] Client logo renders only when the API reports one, and hides on load error.
      *(FR13)*
- [ ] History Dashboard link appears only after contract selection and carries
      contract + date params. *(FR14)*
- [ ] Empty states (no contracts; no positioned devices) and the unauthorized/
      malformed-contract case are handled without leaking contract existence.
      *(FR15, NFR6)*
- [ ] Built in the agreed stack (React 18.3/TS 5.5/Vite, Router v6, Zustand,
      Tailwind 4, Leaflet/react-leaflet). *(NFR1)*
- [ ] No backend, API-contract, or DB change; only `/api/v1` is consumed. *(NFR2)*
- [ ] PHP monolith verified unaffected and still serving. *(NFR3)*
- [ ] Dark and light themes both work via design tokens; desktop layout is
      first-class. *(NFR4)*
- [ ] Performance budget met for initial render and non-blocking refresh. *(NFR5)*
- [ ] Accessibility: controls keyboard-reachable; state not conveyed by colour
      alone; contrast met in both themes. *(NFR7)*
- [ ] Vitest + Testing Library unit tests cover colour mapping, visibility filter,
      tooltip formatting, message/logo gating, and empty/error states. *(NFR8)*
- [ ] Playwright E2E covers the primary flow against a running build. *(NFR9)*
- [ ] All FRs and NFRs validated.

## 16. Future Extensions (Optional, Not Required Now)

- An opt-in client-side periodic auto-refresh timer (repeated `/api/v1` reads) once
  product confirms the UX — still not real-time push.
- True real-time position streaming, once a backend realtime endpoint exists
  (would revisit the held `socket.io-client` decision). *(IMP-3)*
- Clustering/heat visualization for contracts with many devices.
- Cross-linking a marker directly into the History Dashboard timeline at that
  device/instant.
- A formally documented `/api/v1` positions schema to replace the inline
  `map_timeline.js` row/hook shape. *(IMP-5)*
