# Feature: React Device Pilotage

> Forward-looking product spec, per
> [`engineering/product/FEATURE.md`](../engineering/product/FEATURE.md). Feeds KAN-60 (React
> UI rebuild of `cpaltracker.web`) as the **Device Pilotage** feature-area child
> epic. This is a new, standalone React frontend consuming the existing
> KAN-116 `/api/v1` REST layer — **no** backend, API-contract or database change.
>
> Ground truth replicated / corrected here:
> - Current behavior: [`technical/Device-Pilotage.md`](technical/Device-Pilotage.md)
>   (CB-n = behavior to replicate; GAP-n = known issues, each decided below).
> - Stack: [`architecture/react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md).
> - Cross-area note: [`technical/CONSOLIDATED-Coverage-and-Gaps.md`](technical/CONSOLIDATED-Coverage-and-Gaps.md) §5.

---

## 1. Feature Name

React Device Pilotage — the MAIN-contract device-pilotage workspace, rebuilt in
React/TypeScript, for reading and changing a device's uplink configuration and
for transferring a device's data visibility to a STD sub-client contract.

## 2. Problem / Pain Point

Today device pilotage lives entirely inside `cpaltracker.web/pilotage.php`: a
single server-rendered PHP page with inline business logic, minimal field hints,
and no separation between UI and gating rules. Operators of MAIN contracts use
it to push sensor settings to devices and to lend device data to STD
sub-clients, but the page has two concrete usability traps documented in the
reverse-engineering pass:

- It presents **10 configuration fields as if they were equivalent, while only
  5 of them ever reach the device** (`Device-Pilotage.md` CB9 / GAP6). An
  operator can carefully set the other 5 and see them "saved," with no
  indication they are inert — a silent trust problem.
- The "MAIN-only" and "contract active today" gating rules exist **only inside
  `pilotage.php`**, with no shared, reusable definition
  (`Device-Pilotage.md` GAP1–GAP5; `CONSOLIDATED-Coverage-and-Gaps.md` §5).

The PHP page is not going away in this iteration — it keeps serving traffic
unmodified. The pain this feature solves is: give operators a modern, themeable,
testable pilotage UI that faithfully replicates the proven behavior, removes the
10-vs-5-field ambiguity, and defers **all** gating authority to `/api/v1` instead
of re-deriving fragile business rules in the browser.

## 3. Goal / Outcome (Success Definition)

A MAIN-contract operator can, in the new React app:

- open a contract's pilotage workspace, see its (non-dead) devices and each
  device's current settings, and immediately tell which settings fields actually
  reach the device;
- submit a configuration change and receive the server's clear accept/reject
  result, with no risk of a page refresh re-submitting it;
- transfer a device's data visibility to one of the contract's STD sub-contracts
  for a date range, and read back the device transfer history;
- and, when the server reports the contract is ineligible (not MAIN) or inactive,
  see exactly why, with submit controls safely disabled — never a client-side
  guess at eligibility.

Success means the new UI reproduces every CB-n behavior that is still correct,
resolves GAP6 explicitly, and introduces **zero** new client-side copies of the
MAIN/active gating logic.

## 4. User Stories (Primary Scenarios)

- **US1.** As a MAIN-contract operator, I want to open a contract's pilotage
  workspace and view its live devices with their current configuration, so that
  I can decide what to pilot.
  *(FR1, FR2, FR3, FR5, FR6, FR7; NFR1, NFR2, NFR3, NFR5)*
- **US2.** As an operator, I want to change a device's configuration and clearly
  see which fields are transmitted to the device versus only stored, so that I
  never set a value that silently does nothing.
  *(FR8, FR9, FR10, FR11, FR12, FR13; NFR4, NFR6, NFR8)*
- **US3.** As an operator, I want to transfer a device's data visibility to one
  of my contract's STD sub-contracts for a period and review the transfer
  history, so that the sub-client can see its measurements during that window.
  *(FR14, FR15, FR16, FR17, FR18; NFR7)*
- **US4.** As an operator whose contract the server reports as ineligible or
  inactive, I want the UI to tell me why and disable submission rather than let
  me send a change that will be refused, so that I understand the restriction.
  *(FR3, FR4, FR11, FR17, FR20; NFR4, NFR9)*
- **US5.** As a platform maintainer, I want the pilotage UI to run as a new
  standalone frontend that only talks to `/api/v1` and uses the shared themeable
  design system, so that the existing PHP monolith keeps serving traffic
  untouched.
  *(FR19; NFR1, NFR2, NFR9, NFR10)*
- **US6.** As a QA engineer, I want the pilotage flows covered by unit and E2E
  tests, so that regressions in validation, field grouping, state-driven
  disabling and history rendering are caught automatically.
  *(FR8, FR20; NFR6, NFR7)*

## 5. In Scope (What WILL be delivered)

- A React Device Pilotage route/screen (React Router v6) rendering a single
  contract's pilotage workspace, resolved from a contract id (CB2).
- A **device list** panel: devices the API returns for the contract (live, not
  dead — CB6), showing per-device identity, current status, and last calibration
  date when the API exposes it (GAP7 decision, below).
- A **settings form** (React Hook Form) exposing the configuration fields, split
  into two visibly labeled groups — "Transmitted to device" (5 fields) and
  "Stored only — not sent to device" (5 fields) — resolving GAP6; with current
  values shown as placeholders/filigrane (CB8), client-side validation mirroring
  server rules (CB13), and blank-means-keep-current messaging (CB14).
- A **transfer form**: pick an STD sub-contract offered by the API (CB22),
  mandatory start date + optional (open-ended) end date (CB23), basic end-after-
  start client check (CB24), submit to the API.
- A **transfer history** panel for the contract's devices, marking open-ended
  transfers "en cours / in progress" (CB29).
- State-driven, server-told UI gating: eligibility (MAIN) and active/inactive are
  consumed from `/api/v1`; the UI only disables/hides controls and shows the
  corresponding messages (CB3, CB4, CB5, CB11, CB20).
- Non-replaying result feedback (flash/toast) so a refresh never re-submits
  (CB19, CB28).
- Centralized design-system tokens + dark/light theme; desktop-first responsive
  layout.
- Vitest + Testing Library unit tests and Playwright E2E tests for the above.

## 6. Out of Scope (What WILL NOT be delivered)

- **Any backend, `/api/v1` contract, or database change.** `/api/v1` (KAN-116)
  is consumed exactly as it is. Any gap found against it is logged as a separate
  backend open question, never worked around client-side (stack decision §5).
- **Any client-side reimplementation or second-guessing of the MAIN-contract or
  "contract active today" gating logic.** The server (`/api/v1`) is the sole
  authority for MAIN-only eligibility (GAP1), active/inactive determination
  (GAP2/GAP3), read-vs-write gating (GAP4), STD-target validity and transfer
  date-window / overlap / parentage / STD-active checks (GAP5, CB22, CB25, CB26).
  The React UI MAY only disable or hide controls and render the messages the API
  returns; it MUST NOT compute or re-derive these rules from contract dates,
  contract type, or any other field.
- Any change to `cpaltracker.web`'s PHP pages (`pilotage.php`,
  `ttn_downlink_reglages.php`, `ttn_webhook.php`, etc.); the monolith runs
  unmodified in parallel.
- Mobile-first design. Desktop is the primary, first-class, acceptance-tested
  target; mobile is best-effort only (stack decision §4).
- The actual device downlink encoding/transport (lives in
  `ttn_downlink_reglages.php`, out of this repo's scope).
- Exposing or editing the binary WiFi credential fields (`WiFi_SSID_CL`,
  `WiFi_PASS_CL`) — never shown in the UI (CB16).

## 7. User Flow / UX Breakdown

1. Operator navigates to the pilotage route for a contract id.
   - System calls `/api/v1`; if unauthenticated, the AuthContext redirects to
     login (FR1).
2. System loads pilotage context from `/api/v1` for that contract.
   - If the API denies access / reports the contract is not eligible (not MAIN),
     the UI shows the access message ("Device pilotage is reserved for MAIN
     contracts") and renders no forms (FR3).
   - If the API reports the contract **inactive**, the UI still renders devices,
     settings and history read-only, shows the informational banner ("Forms are
     viewable but disabled"), and disables both submit buttons (FR4).
3. System renders the device list (live devices only), each with current status
   and last calibration date when available (FR5, FR6).
4. Operator selects a device in the settings form.
   - System shows that device's current config as placeholders, split into the
     "Transmitted to device" group (5) and the "Stored only" group (5), each
     labeled (FR7, FR8).
5. Operator edits one or more fields and submits.
   - UI validates format client-side (FR9); blank fields are shown as
     "keep current value" (FR10); UI POSTs to `/api/v1` and shows the server's
     result (accepted / "no values changed" / "contract not active") as a
     non-replaying flash, then refreshes current config on success (FR11, FR13).
6. Operator switches to the transfer form, picks an STD sub-contract from the
   API-provided list, sets a start date and optional end date, submits.
   - UI checks end-after-start locally (FR16); the API enforces window, overlap,
     parentage and active checks; UI shows the server's accept/reject result
     (FR14, FR15, FR17).
7. System renders the transfer history, marking open-ended rows "en cours"
   (FR18).

## 8. Functional Requirements (FRs)

- **FR1.** The app SHALL require an authenticated session (via the AuthContext
  REST client against `/api/v1`) to reach the Device Pilotage view and SHALL
  redirect unauthenticated users to login. *(replicates CB1)*
- **FR2.** The view SHALL operate in the context of a single contract identified
  by a contract id and SHALL load all pilotage data for only that contract from
  `/api/v1`. *(CB2)*
- **FR3.** The UI SHALL render the pilotage forms only when `/api/v1` returns the
  contract as eligible for pilotage; when the API denies access or reports the
  contract is not eligible (e.g. a STD contract), the UI SHALL display an access
  message equivalent to "Device pilotage is reserved for MAIN contracts" and SHALL
  NOT render the settings or transfer forms. The UI SHALL NOT itself compute
  MAIN-vs-STD eligibility. *(CB3; GAP1 — server is sole authority)*
- **FR4.** The UI SHALL treat the contract's active/inactive state as reported by
  `/api/v1`. When the API reports the contract inactive, the UI SHALL still
  display devices, current settings and history, SHALL disable the settings and
  transfer submit controls, and SHALL show an informational banner equivalent to
  "Forms are viewable but disabled." The UI SHALL NOT compute active/inactive from
  contract dates or any other field. *(CB4, CB5, CB11, CB20; GAP2, GAP3, GAP4 —
  server is sole authority)*
- **FR5.** The UI SHALL display the list of devices the API returns for the
  contract (live devices only; the API applies the dead-device exclusion). *(CB6)*
- **FR6.** The device list SHALL display each device's last-known calibration date
  when the API provides it, as a read-only informational value. *(GAP7 decision:
  fix — surface it rather than silently carry a dead column; shown only if the API
  exposes it, since no backend change is made here)*
- **FR7.** For each selected device the UI SHALL display the device's current
  configuration values (from the API) as placeholder/filigrane hints in the
  settings form. *(CB8)*
- **FR8.** The settings form SHALL present the configuration fields in two
  clearly separated, visibly labeled groups: (a) the 5 fields transmitted to the
  device (`FREQ_MESURE_CL`, `WAKEUP_ESP_CL`, `Seuil_TEMPAMB_CL`, `Seuil_HRAMB_CL`,
  `Seuil_MASSE_CL`) and (b) the 5 fields stored but **not** transmitted
  (`SeuilG_CL`, `Seuil_AccelG_CL`, `Seuil_LONG_CL`, `Seuil_LAT_CL`,
  `Seuil_TEMPMPU_CL`), each group labeled so the operator can tell which values
  reach the device. *(GAP6 decision — see §12; CB9)*
- **FR9.** Each settings field SHALL be validated client-side to mirror the
  server's constraints (uint8 ≤ 255; uint16 ≤ 65535; float accepting a comma as
  decimal separator, normalized to a dot) before submission; the API remains the
  authoritative validator. *(CB13)*
- **FR10.** A field left blank SHALL be submitted with "unchanged / keep current
  value" semantics, and the UI SHALL communicate that a blank field keeps the
  device's last known value rather than clearing it. *(CB14)*
- **FR11.** The UI SHALL submit a settings change only to the `/api/v1` endpoint
  and SHALL surface the API's result — accepted, "no values changed," or
  "contract not active" — as a flash/toast. *(CB10, CB15, CB17, CB18)*
- **FR12.** The settings form SHALL NOT expose the binary WiFi credential fields
  (`WiFi_SSID_CL`, `WiFi_PASS_CL`); they are handled entirely server-side. *(CB16)*
- **FR13.** On a successful settings change the UI SHALL display the API's
  confirmation (including that values will be transmitted on the device's next
  port-3 uplink) and SHALL refresh the displayed current configuration. *(CB17, CB19)*
- **FR14.** The transfer form SHALL let the operator select a target STD
  sub-contract only from the list `/api/v1` returns for this MAIN contract; the
  UI SHALL NOT look up or construct a transfer target independently. *(CB22)*
- **FR15.** The transfer form SHALL require a start date and accept an optional
  end date; when the end date is omitted the UI SHALL indicate the transfer is
  open-ended. *(CB23)*
- **FR16.** The transfer form SHALL perform a basic client-side check that an
  end date (if supplied) is strictly after the start date, and SHALL rely on
  `/api/v1` for date-window containment, overlap, parentage and active-contract
  checks. *(CB24, CB25, CB26; GAP5 — server is sole authority)*
- **FR17.** The UI SHALL submit transfers only to `/api/v1` and SHALL surface the
  API's accept/reject result (e.g. overlap, out-of-window, unknown/unattached
  target, contract not active) as returned. *(CB20–CB28)*
- **FR18.** The UI SHALL display the transfer history for the contract's devices
  as returned by the API, marking open-ended transfers (sentinel end date) as
  "en cours / in progress." *(CB29)*
- **FR19.** The view SHALL be built with the shared centralized design-system
  tokens and SHALL support both dark and light themes with no per-component
  theming. *(stack decision §4)*
- **FR20.** Action results SHALL be shown via non-replaying feedback such that a
  browser refresh after a submit never re-sends the request. *(CB19, CB28)*

## 9. Non-Functional Requirements (NFRs)

- **NFR1.** The feature SHALL be built on the agreed stack: React 18.3 +
  TypeScript 5.5 + Vite, React Router v6, Zustand for state, Tailwind 4, and
  React Hook Form for the settings and transfer forms. *(stack decision §2)*
- **NFR2.** The feature SHALL communicate only with `/api/v1` (KAN-116) over the
  AuthContext REST client; it SHALL NOT access the database, the PHP pages, or
  any other backend directly. *(stack decision §1, §5)*
- **NFR3.** The layout SHALL be desktop-first and desktop is the primary,
  acceptance-tested target; mobile is best-effort and not separately
  acceptance-tested. *(stack decision §4)*
- **NFR4.** All MAIN-only and active/inactive gating authority SHALL remain
  server-side; the client SHALL only disable/hide controls and render
  API-supplied messages, and SHALL contain no logic that re-derives these rules.
  *(GAP1–GAP5)*
- **NFR5.** The pilotage workspace SHALL become interactive within 1 second on a
  warm `/api/v1` on desktop; device and history lists SHALL remain responsive for
  a contract's full device set (virtualize if large).
- **NFR6.** Unit tests (Vitest + Testing Library, jsdom) SHALL cover field-group
  separation (5 vs 5), client-side field validation, blank-keeps-value messaging,
  state-driven disabling of submit controls, and history "en cours" rendering.
  *(stack decision §2)*
- **NFR7.** E2E tests (Playwright, against a running local/staging build) SHALL
  cover the primary flows: view workspace, submit a settings change, submit a
  transfer, and the ineligible/inactive-contract read-only path. *(stack decision §2)*
- **NFR8.** Forms SHALL be accessible: every field labeled and programmatically
  associated with its error and its group, keyboard-navigable, with visible focus.
- **NFR9.** The UI SHALL not render or log secrets, SHALL carry CSRF/session via
  the AuthContext REST client as `/api/v1` requires, and SHALL NOT implement its
  own auth or gating. *(CB16; GAP1–GAP5)*
- **NFR10.** Delivery SHALL cause no disruption to the existing `cpaltracker.web`
  PHP monolith, which continues serving traffic unmodified in parallel. *(stack
  decision §5)*

## 10. Data Inputs & Outputs

All I/O is via `/api/v1` (exact request/response shapes owned by KAN-116; this
feature consumes, never defines, them).

- **Inputs (from API → UI):** contract pilotage context (eligibility + active
  state as the API reports them), device list (identity, status, last calibration
  when exposed), per-device current configuration values (the 10 fields + current
  placeholders), list of attachable STD sub-contracts, transfer history rows.
- **Outputs (from UI → API):**
  - Settings change: target device + the editable configuration fields (blank =
    keep current). WiFi fields never sent (CB16).
  - Transfer: target device, chosen STD sub-contract id, start date, optional end
    date.
- **Rendered outputs:** device list, grouped settings form with current-value
  placeholders, transfer form, transfer history (with "en cours" marking), and
  flash/toast results.
- **Not an input:** contract type and contract dates SHALL NOT be used by the UI
  to decide eligibility or active state — those decisions come only as API flags
  (NFR4).

## 11. Integration Points

- **`/api/v1` (KAN-116, Done)** — the only integration; read for context/devices/
  config/STD list/history, write for settings-change and transfer. Consumed as-is.
- **AuthContext REST client** — session/CSRF handling for all `/api/v1` calls
  (stack decision §3).
- **Shared design-system / theme tokens** (Core epic) — styling and dark/light
  theming.
- **No integration** with `pilotage.php`, `ttn_downlink_reglages.php`,
  `ttn_webhook.php`, the database, or any TTN bridge — explicitly excluded.

## 11a. Software Items (IEC 62304 §5.3)

**N/A.** Per [`sdlc/README.md`](../sdlc/README.md) "Applicability to c-PAL today,"
IEC 62304 software items are *Not applicable* until c-PAL software enters a
regulated scope; this section is intentionally left empty.

## 12. Edge Cases & Constraints

- **GAP1 (MAIN-only rule not centralized) — carry forward as-is, server-side.**
  The UI does not reimplement the MAIN-only check; it renders the forms only when
  the API says the contract is eligible, and shows the reserved-for-MAIN message
  otherwise (FR3). No client-side `ContratType` logic.
- **GAP2 / GAP3 (no canonical "active today" definition; no 409 precedent in the
  PHP page) — carry forward as-is, server-side.** The UI consumes the API's
  active/inactive flag and the API's reject responses for inactive-contract
  writes; it does not compute "active" or assume any status-code semantics
  (FR4, FR11, FR17). Any ambiguity about what "inactive" means is a KAN-116/backend
  open question, not resolved here.
- **GAP4 (reads allowed on inactive MAIN contract) — carry forward as-is.** When
  the API reports the contract inactive, the UI still displays devices/config/
  history read-only and disables only the submit actions (FR4), matching current
  behavior.
- **GAP5 (STD target not checked for "active today"; expired STD not excluded) —
  carry forward as-is, server-side.** The UI offers exactly the STD list the API
  returns and relies on the API to accept/reject the transfer; it does not add or
  remove STD targets based on client-side date logic (FR14, FR16). If product
  later wants expired STD contracts excluded, that is a backend/API change
  (IMP3), out of scope here.
- **GAP6 (10 fields presented as equivalent; only 5 reach the device) — FIX in
  the new UI.** The settings form splits the fields into two visibly labeled
  groups — "Transmitted to device" (the 5 effective fields) and "Stored only —
  not sent to device" (the 5 inert ones) — so the operator always knows which
  values reach the device (FR8). We deliberately **keep** the 5 stored-only
  fields rather than omit them, because whether they are legacy/dead or reserved
  for a future downlink version is unresolved (Device-Pilotage.md §9 open
  question); grouping-with-a-label preserves the data path while removing the
  ambiguity, and avoids a destructive UI decision the backend has not confirmed.
- **GAP7 (`LASTCALIBRATION` fetched but never displayed) — FIX if exposed.** The
  device list surfaces last calibration as a read-only operational signal when
  the API exposes it (FR6); if the API does not return it, the column is simply
  omitted (no backend change is made to expose it).
- **Blank field preserves last value** (CB14) — surfaced explicitly in the UI
  (FR10).
- **No-op settings submission** (CB15) — the API rejects; the UI shows the
  message (FR11).
- **Open-ended transfer** (CB23) — represented as "no end date / en cours" in the
  UI (FR15, FR18), never as a literal sentinel string to the user.
- **Double submission via refresh** (CB19, CB28) — prevented by non-replaying
  result feedback (FR20).
- **Dead devices** (CB6) — excluded by the API; the UI shows only what it returns.

## 13. Metrics / KPIs

- 0 operator reports of "I set a field and it didn't take effect" attributable to
  the 5 inert fields (GAP6 resolved).
- 100% of pilotage gating decisions (eligibility, active/inactive, transfer
  validity) traceable to an `/api/v1` response, with no client-side gating logic
  in the codebase (verified by review + unit tests, NFR4/NFR6).
- 0 double-submissions caused by page refresh (FR20).
- Workspace interactive < 1 s on a warm API (NFR5).
- Unit + E2E suites green in CI for all primary flows (NFR6, NFR7).
- 0 disruptions to the PHP monolith attributable to this feature (NFR10).

## 14. Risks & Mitigations

- **Risk: the team re-implements MAIN/active gating in React for "better UX."**
  Mitigation: NFR4 + Out-of-Scope make server-only authority explicit; review
  gate and unit tests assert the absence of client-side gating logic.
- **Risk: the API does not expose a clean eligibility/active flag, tempting a
  client-side workaround.** Mitigation: log as a KAN-116/backend open question
  (stack decision §5); the UI disables conservatively and shows the API message
  rather than guessing.
- **Risk: omitting the 5 inert fields loses data the backend still expects.**
  Mitigation: we keep and label them (GAP6 decision) instead of omitting, pending
  backend confirmation of their status.
- **Risk: `LASTCALIBRATION` not exposed by the API.** Mitigation: FR6 renders it
  only when present; otherwise it is omitted, with no backend change.
- **Risk: mobile layout expectations creep in.** Mitigation: desktop-first is an
  explicit NFR/Out-of-Scope item (NFR3).

## 15. Acceptance Criteria (Definition of Done)

- [ ] Unauthenticated access to the pilotage view redirects to login (FR1).
- [ ] The view loads pilotage data for exactly one contract id from `/api/v1` (FR2).
- [ ] When the API reports the contract ineligible/not MAIN, the forms are not
  rendered and the reserved-for-MAIN message is shown; no client-side MAIN check
  exists (FR3, NFR4).
- [ ] When the API reports the contract inactive, data is shown read-only, submit
  controls are disabled, and the informational banner appears; no client-side
  active/inactive computation exists (FR4, NFR4).
- [ ] The device list shows only API-returned (live) devices (FR5) and shows last
  calibration when the API exposes it (FR6).
- [ ] Selecting a device shows its current config as placeholders (FR7).
- [ ] The settings form shows two labeled groups: 5 "Transmitted to device"
  fields and 5 "Stored only — not sent to device" fields (FR8; GAP6).
- [ ] Client-side validation enforces uint8/uint16/float(comma→dot) rules (FR9).
- [ ] Blank fields are labeled as "keep current value" and submitted as unchanged
  (FR10).
- [ ] WiFi credential fields are never present in the form (FR12).
- [ ] Settings submit posts only to `/api/v1`, and accepted / no-change /
  inactive results are surfaced; success refreshes current config and shows the
  next-uplink message (FR11, FR13).
- [ ] Transfer form offers only API-provided STD sub-contracts, requires a start
  date, allows an optional (open-ended) end date, and checks end-after-start
  client-side (FR14, FR15, FR16).
- [ ] Transfer submit posts only to `/api/v1`; overlap/out-of-window/unknown-
  target/inactive rejections are surfaced from the API (FR17; GAP5).
- [ ] Transfer history renders with open-ended transfers marked "en cours" (FR18).
- [ ] The view uses design-system tokens and works in both dark and light themes
  (FR19, NFR1).
- [ ] A browser refresh after any submit never re-sends the request (FR20).
- [ ] The feature talks only to `/api/v1`; no DB/PHP/other-backend calls (NFR2,
  NFR9, NFR10).
- [ ] Desktop layout is the acceptance-tested target (NFR3, NFR5).
- [ ] Unit tests cover field grouping, validation, blank-keeps-value, state-driven
  disabling, and history rendering (NFR6).
- [ ] E2E tests cover view / settings-change / transfer / ineligible-inactive
  read-only paths (NFR7).
- [ ] Forms meet the accessibility requirements (NFR8).
- [ ] Every FR and NFR is traced to at least one user story and one acceptance
  criterion.

## 16. Future Extensions (Optional, Not Required Now)

- If the backend (IMP1/IMP2) centralizes the MAIN-only + active-today rules and
  adopts a 409-on-inactive-write convention, the UI can consume those explicit
  signals instead of generic messages.
- If the backend decides the 5 stored-only fields are dead (GAP6 open question),
  the UI can drop them entirely; if they are reserved for a future downlink, the
  "stored only" group can be promoted to "transmitted" without a structural change.
- Exclude expired STD contracts from the transfer target list once the API does so
  (IMP3).
- Render `LASTCALIBRATION`-driven alerts (e.g. overdue calibration) once the API
  exposes richer calibration data (IMP5).
