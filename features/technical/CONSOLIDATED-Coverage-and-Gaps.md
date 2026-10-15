# Consolidated Coverage, Gaps & Improvements — cpaltracker.web

> Produced under **KAN-134** (epic **KAN-126**, "Technical Feature Discovery —
> cpaltracker.web"), rolling up the seven area-level Technical-Feature docs
> into one cross-referenced inventory per FR9/FR10/NFR4 of
> [`IN-PROGRESS-Technical-Feature-Discovery.md`](../IN-PROGRESS-Technical-Feature-Discovery.md).
> Read-only synthesis — no application code and none of the seven source
> docs were modified to produce this document.

## 1. Purpose

This document is the full-coverage inventory of `cpaltracker.web`'s current
behavior, gaps, and suggested improvements, produced to seed **KAN-60**
(the React UI rebuild) story-splitting. It does **not** replace the seven
area-level Technical-Feature docs — `Auth-and-Session.md`,
`Password-Recovery-and-Account-Management.md`, `Live-Tracking.md`,
`History-Dashboard.md`, `Device-Pilotage.md`, `Client-Branding-and-Assets.md`,
`Device-Ingestion.md` — it **indexes and cross-references** them. Every
Current-Behavior (CB-n), Gap (GAP-n) and Suggested-Improvement (IMP-n) cited
below lives in its source doc; this document never re-derives a claim from
the underlying code itself, and a reader who wants the code citation behind
any item should follow the cited doc + id back to that doc's own file/line
citation. Its unique value is the patterns that only become visible once all
seven areas are read together — a security rule applied on four pages but
missing on a fifth, a piece of documentation that is stale in the same way in
two unrelated docs, a permissive-validation posture that shows up twice under
different names — none of which any single area doc, read alone, would
surface.

## 2. File Coverage Traceability

Per FR10/NFR4, every file in `cpaltracker.web`'s web root, as listed in
[`cpaltracker.web/docs/architecture.md`](../../../cpaltracker.web/docs/architecture.md)'s
Components table, must be cited by at least one CB-n in some area doc.
`config.php` and `mailer.php` are mentioned in `architecture.md` immediately
below the Components table, not inside it, and the epic's own FR10 wording
scopes the check to "the Components table" specifically — they are excluded
from this count for that reason (both are, in any case, cited incidentally by
multiple area docs: `config.php`'s `LOGIN_*`/`SESSION_*`/`RESET_*`/`PWD_*`
constants and `requireDeviceAuth()` by Auth-and-Session, Password-Recovery
and Device-Ingestion; `mailer.php` by Password-Recovery).

| Components-table file | Cited by (doc: CB-n range) | Covered? |
|---|---|---|
| `login.php` | Auth-and-Session.md: CB1–CB18 | Yes |
| `logout.php` | Auth-and-Session.md: CB19–CB21 | Yes |
| `auth.php` | Auth-and-Session.md: CB22–CB36 (also cited incidentally throughout Password-Recovery, Live-Tracking, History-Dashboard, Device-Pilotage, Client-Branding) | Yes |
| `forgot_password.php` | Password-Recovery-and-Account-Management.md: CB1–CB7 | Yes |
| `reset_password.php` | Password-Recovery-and-Account-Management.md: CB8–CB16 | Yes |
| `account.php` | Password-Recovery-and-Account-Management.md: CB21–CB31 | Yes |
| `password_policy.php` | Password-Recovery-and-Account-Management.md: CB17–CB20 | Yes |
| `index.php` | Live-Tracking.md: CB-1–CB-19 (also cited by Client-Branding-and-Assets.md: CB14–CB15 for the logo call site) | Yes |
| `dashboard.php` | History-Dashboard.md: CB1–CB3, CB5, CB16–CB17, CB26–CB28 | Yes |
| `api_dashboard.php` | History-Dashboard.md: CB4, CB6–CB15 | Yes |
| `insert_data.php` | Device-Ingestion.md: CB1–CB15 | Yes |
| `pilotage.php` | Device-Pilotage.md: CB1–CB29 | Yes |
| `client_logo.php` | Client-Branding-and-Assets.md: CB1–CB15 | Yes |
| `map_timeline.js` | Live-Tracking.md: CB-12–CB-18 (documented despite not being wired into `index.php`, see §3/§5 below); History-Dashboard.md: CB18–CB25 (actual usage, via `dashboard.php`) | Yes |
| `auth.css` | Client-Branding-and-Assets.md: CB20–CB24 | Yes |
| `styles.css` | Client-Branding-and-Assets.md: CB16–CB19, CB22, CB24 | Yes |

**16 / 16 Components-table files covered — 100%.** No file in the
Components table is uncited. This is a genuine, counted 100%: it is not
rounded up, and the one place a file is cited for a reason other than its own
intended use (`map_timeline.js` is cited in `Live-Tracking.md` even though
CB-11/GAP-1 there establish that `index.php` does not actually load it) is
called out explicitly rather than silently counted as ordinary coverage.

## 3. Consolidated Gaps (cross-area)

31 GAP-n items are recorded across the seven docs (Auth-and-Session: 5,
Password-Recovery: 4, Live-Tracking: 4, History-Dashboard: 5,
Device-Pilotage: 7, Client-Branding: 1, Device-Ingestion: 5). Grouped
thematically below; each item is cited to its source doc and id only — the
underlying code evidence lives in that doc, not here.

### 3.1 Anti-enumeration / timing-channel inconsistencies

The app's anti-enumeration intent is real and stated in multiple files, but
it is not applied uniformly, and two of the docs' examples are a case where
the design is already consistent and intentional rather than a gap — reading
all three together is what makes the pattern (and the one correct
counter-example) visible:

- GAP3, `Auth-and-Session.md` — only the unknown-account branch in
  `login.php` runs a dummy `password_verify()`; soft-deleted, locked, and
  uninitialized-password branches return faster, creating a timing signal.
- GAP4, `Auth-and-Session.md` — the locked-account branch breaks
  `login.php`'s own stated "one generic message for every rejection" rule by
  disclosing remaining lockout minutes.
- GAP1, `Client-Branding-and-Assets.md` — `client_logo.php` returns `403`
  for "not your client" vs. `404` for "no logo", letting a non-admin
  distinguish the two reasons; the `403` branch is also faster (no DB
  round-trip) than the `404` branch, compounding a status-code leak with a
  timing leak — directly relevant because KAN-116's planned
  `/api/v1/clients/{clientId}/logo` endpoint is designed to collapse exactly
  this pair of cases into one `404`.
- GAP5, `History-Dashboard.md` — by contrast, `dashboard.php`/
  `api_dashboard.php` already merge "contract doesn't exist" and "contract
  isn't yours" into one identical message, confirmed deliberate by
  `docs/security.md`'s own comment. Listed here as a gap only in the loose
  sense of "cross-page consistency fact worth carrying forward" — it is the
  one area where the anti-enumeration rule the other three items above
  violate is actually implemented correctly, and should be the pattern
  `client_logo.php`'s REST replacement and `login.php`'s locked-account
  branch converge toward, not diverge from.

### 3.2 Rate-limiting / lockout inconsistencies

- GAP1, `Auth-and-Session.md` — a failed CSRF check or an empty-field POST
  to `login.php` returns before `cpal_log()` runs, so it is never counted by
  `cpal_ip_bloquee()`'s per-IP throttle.
- GAP2, `Auth-and-Session.md` — `FailedLoginCount` is reset only on a
  successful login, never when `LockedUntil` expires, so a single wrong
  attempt after the lock lifts re-locks the account instead of restarting a
  fresh 5-attempt budget.
- GAP1, `Password-Recovery-and-Account-Management.md` — `account.php`'s
  own current-password check (used by all three self-service actions) never
  calls `cpal_ip_bloquee()`, unlike `forgot_password.php` and
  `reset_password.php`, even though a failed attempt there is logged under
  the same `login_ko` event the throttle's own query counts.
- GAP2, `Device-Ingestion.md` — `insert_data.php` has no rate limiting or
  lockout at all; `requireDeviceAuth()` performs a single stateless
  comparison per request with no counter.

Read together, these four show the per-IP/per-account throttle is a real,
working mechanism (`auth.php`'s `cpal_ip_bloquee()`) that is consistently
applied to `login.php`, `forgot_password.php` and `reset_password.php`, but
has three separate holes around its edges: two ways to avoid being counted
by it (Auth GAP1, GAP2) and one authenticated surface that was never wired
to it at all (Password-Recovery GAP1), plus one entirely separate endpoint
with no equivalent mechanism whatsoever (Device-Ingestion GAP2).

### 3.3 Stale/incorrect existing documentation and in-code claims

A pattern that only shows up by reading multiple docs side by side: both
`docs/security.md` and `docs/architecture.md`, plus one module's own header
comment, assert things about the code that the code itself contradicts.

- GAP5, `Auth-and-Session.md` — `docs/security.md` states "no rate
  limiting or lockout was found on `login.php`"; the code has both a
  per-account lockout and a per-IP throttle.
- GAP3, `Password-Recovery-and-Account-Management.md` — `docs/security.md`
  lists the HIBP breach check as an active mitigation without noting
  `PWD_CHECK_HIBP` defaults to `false`, so it is off out of the box.
- GAP4, `Password-Recovery-and-Account-Management.md` — the same
  `docs/security.md` "no rate limiting... on `forgot_password.php`" claim is
  also stale: `forgot_password.php` does call `cpal_ip_bloquee()`.
- GAP-1, `Live-Tracking.md` and GAP1, `History-Dashboard.md` — the same
  discrepancy, confirmed independently from both sides: `map_timeline.js`'s
  own header comment and `docs/architecture.md`'s Components table both
  state the module is shared between `index.php` ("LIVE view") and
  `dashboard.php`; a repo-wide grep (done from the `History-Dashboard.md`
  side) and a file-content check (done from the `Live-Tracking.md` side)
  agree that only `dashboard.php` actually loads/invokes it.
- GAP-3, `Live-Tracking.md` — relatedly, the "LIVE" framing itself is
  inconsistent with the code: no file in scope contains a polling/refresh
  mechanism; "live" means "latest snapshot, loaded once, on a user action,"
  not "continuously updating."

Three independent documentation artifacts (`security.md` twice, on two
different pages' rate-limiting claims; `architecture.md`'s Components table
and `map_timeline.js`'s own header comment, both on the same sharing claim)
are each individually stale, discovered by three different area docs working
independently — strong evidence this is a systemic documentation-drift
problem in this codebase, not an isolated typo, and that the React rebuild
should not take any existing doc's behavioral claim on faith without the
corresponding CB-n citation backing it.

### 3.4 Authorization/business-rule checks not centralized or reusable

- GAP-4, `Live-Tracking.md` — `index.php`'s contract-scoping, logo/message
  composition and dashboard-link building are interleaved with HTML
  rendering in one 497-line file with no internal decomposition, a concrete
  instance of the SRP violation `docs/design-and-standards.md` already
  documents generally.
- GAP1–GAP5, `Device-Pilotage.md` — the "MAIN-contract-only" rule (GAP1)
  and the "is this contract active today" rule (GAP2) both live solely
  inside `pilotage.php`'s own code, bolted on after `cpal_contrat_autorise()`
  returns, with no shared helper either other page or the in-progress
  KAN-116 REST endpoints can inherit; consequently there is no existing
  status-code precedent for a 409-on-inactive-write response (GAP3); reads
  are not gated by contract-active status at all, only writes are (GAP4,
  recorded as a fact to preserve, not itself a defect); and the transfer
  target (STD contract) is checked for date-window containment but never
  for being "active today" the same way the MAIN contract is (GAP5).

This is the single largest gap cluster (6 items) and the clearest signal for
KAN-60/KAN-116: `cpal_contrat_autorise()` is the one shared authorization
primitive in this codebase, and both the "must be MAIN" and "must be active"
refinements that `pilotage.php` layers on top of it were never pushed down
into it or any other shared location — the exact same "logic lives in the
page, not in a reusable layer" finding that `Live-Tracking.md`'s GAP-4
reports independently for `index.php`'s simpler composition logic.

### 3.5 Missing or overly permissive range/date validation

- GAP2, `History-Dashboard.md` — `api_dashboard.php` has no maximum
  date-range span; an arbitrarily large range is accepted and only mitigated
  by point-count sampling, not rejected.
- GAP3, `History-Dashboard.md` — a reversed `date_from > date_to` range is
  silently swapped rather than rejected with a 400.
- GAP4, `History-Dashboard.md` — a malformed (not just out-of-range) date
  parameter is silently dropped — treated identically to an absent one, with
  no error signal.
- GAP3, `Device-Ingestion.md` — `MIN` and `SEC` are required and
  numeric-checked but have no range check, unlike `MONTH`/`DAY`/`HOUR`.
- GAP4, `Device-Ingestion.md` — `DAY`'s range check is a fixed `[1, 31]`
  independent of `MONTH`/`YEAR`, so calendar-invalid dates (e.g. Feb 31) pass
  validation and are written to `T_DATA`.

Two unrelated area docs — one covering a browser-facing JSON endpoint, one
covering a machine-facing ingestion endpoint — independently found the same
underlying posture: this codebase's validation style favors silently
coercing or accepting malformed/out-of-range input over rejecting it with an
error. That is a codebase-wide default, not a coincidence specific to dates,
and is exactly the kind of implicit contract the KAN-116 REST layer (which,
per `History-Dashboard.md`'s GAP2/GAP3, is already specified to reject
out-of-range spans and reversed ranges) is diverging from — intentionally,
per the open question in §6, but the divergence itself is worth the React
team knowing about explicitly rather than discovering by comparing two old
endpoints' behavior against the new one's OpenAPI spec.

### 3.6 Hidden distinctions not surfaced in the current UI

- GAP6, `Device-Pilotage.md` — the settings form renders all 10 `$CHAMPS`
  fields identically, with no indication that 5 of them are persisted but
  never actually transmitted to the device.

### 3.7 Dead/unused code and dependencies

- GAP-2, `Live-Tracking.md` — `index.php` loads the Leaflet CSS stylesheet
  unconditionally on every page view despite never using Leaflet on that
  page (a direct consequence of GAP-1/§3.3 above).
- GAP7, `Device-Pilotage.md` — `d.LASTCALIBRATION` is fetched in the
  device-list query but never read or rendered anywhere in the file.

### 3.8 Session-change notification inconsistency

- GAP2, `Password-Recovery-and-Account-Management.md` — `reset_password.php`'s
  post-change e-mail explicitly states other sessions were closed;
  `account.php`'s post-change e-mail, triggered by the same underlying
  `SessionEpoch` mechanism, never mentions it.

### 3.9 Shared-credential weakness (device ingestion)

- GAP1, `Device-Ingestion.md` — `insert_data.php` authenticates every
  device against one shared `DEVICE_SHARED_KEY`, not a per-device
  credential; already flagged in `docs/security.md`, confirmed directly in
  `requireDeviceAuth()`'s code.

### 3.10 Unverifiable external dependency

- GAP5, `Device-Ingestion.md` — the TTN bridge scripts that actually call
  `insert_data.php` are gitignored and absent from this repo, so whether
  they ever send values that would trigger GAP3/GAP4 above, or how they
  handle webhook retries/dedup, cannot be confirmed from this repo at all.

## 4. Consolidated Suggested Improvements (cross-area)

> **These are the seven docs' authors' own recommendations — not current
> behavior.** Every item below is a proposal to evaluate for KAN-60 or a
> follow-up epic, not a fact about what `cpaltracker.web` does today; each
> is cited to its source doc + IMP-n id. 39 IMP-n items are recorded across
> the seven docs (Auth-and-Session: 6, Password-Recovery: 6, Live-Tracking:
> 5, History-Dashboard: 6, Device-Pilotage: 6, Client-Branding: 5,
> Device-Ingestion: 5).

### 4.1 Close the anti-enumeration/timing gaps (§3.1)

- IMP2, `Auth-and-Session.md` — apply the dummy-hash step uniformly across
  all rejection branches, not only "unknown account."
- IMP4, `Auth-and-Session.md` — move the locked-account disclosure out of
  the synchronous response (e.g. into an out-of-band e-mail).
- IMP1, `Client-Branding-and-Assets.md` — when building the REST logo
  endpoint, verify both unified-404 cases also take comparable response
  time, not just the same status code.

### 4.2 Close the rate-limiting/lockout gaps (§3.2)

- IMP1, `Auth-and-Session.md` — reset `FailedLoginCount` when a lockout
  expires, not only on success.
- IMP3, `Auth-and-Session.md` — log a lightweight event for CSRF-failed/
  empty-field submissions and count it toward the per-IP throttle.
- IMP1, `Password-Recovery-and-Account-Management.md` — add an
  `cpal_ip_bloquee()` (or dedicated counter) check to `account.php`'s
  current-password verification.
- IMP2, `Device-Ingestion.md` — add request throttling to `insert_data.php`.

### 4.3 Centralize authorization/business logic that currently lives on one page (§3.4)

- IMP6, `Auth-and-Session.md` — keep lockout/CSRF/epoch logic in the
  shared `auth.php`-style layer rather than duplicating it in new REST
  handlers; add unit tests around the identified counter/timeout edge cases.
- IMP1, `Device-Pilotage.md` — extract a single shared
  `cpal_contrat_main_actif()`-style helper covering both "MAIN-only" and
  "active" so `pilotage.php` and the KAN-116 REST endpoints share one
  definition instead of two.
- IMP2, `Device-Pilotage.md` — document the 409-on-inactive-write status
  code explicitly as new API surface, not an extraction of existing
  behavior.
- IMP3, `Device-Pilotage.md` — add an explicit "is the target STD contract
  active today" check before accepting a transfer.
- IMP6, `Device-Pilotage.md` — extract `pilotage.php`'s inline query/
  business logic into plain functions ahead of or alongside the REST work.
- IMP4, `Live-Tracking.md` — pull `index.php`'s inline contract-scoping/
  logo/message composition into a dedicated function/service, consistent
  with `docs/design-and-standards.md`'s standing recommendation.
- IMP3, `Client-Branding-and-Assets.md` — consolidate the "does this client
  have a logo" check, currently implemented twice (`auth.php` and
  `client_logo.php`), into one shared function.
- IMP5, `History-Dashboard.md` — extract `api_dashboard.php`'s query/
  shaping logic into HTTP-independent functions so the same code can back
  both the legacy page and a future REST endpoint.

This is the largest improvement cluster (8 items touching 5 of the 7 areas)
and directly mirrors the largest gap cluster (§3.4) — the clearest
single theme in this whole rollup: almost every area's authors independently
arrived at "extract the inline logic into a shared function" as their top
recommendation, which strongly suggests this should be treated as one
cross-cutting refactor effort for KAN-60/KAN-116, not seven separate,
area-scoped extractions that might not converge on the same shape.

### 4.4 Keep documentation and security defaults in sync with code (§3.3)

- IMP5, `Auth-and-Session.md` — update `docs/security.md`'s "Gaps and
  risks" section to reflect the lockout/throttle mechanisms that already
  exist.
- IMP3, `Password-Recovery-and-Account-Management.md` — default
  `PWD_CHECK_HIBP` to `true` (or require an explicit, documented opt-out).
- IMP4, `Password-Recovery-and-Account-Management.md` — update
  `docs/security.md`'s stale "no rate limiting" entry for
  `forgot_password.php`.
- IMP1, `Live-Tracking.md` — before the React rebuild proceeds, make an
  explicit decision about which page owns the "live" map and correct
  whichever of `map_timeline.js`'s header comment / the page's actual
  behavior is wrong, rather than carrying the discrepancy forward.
- IMP4, `History-Dashboard.md` — resolve the same `map_timeline.js`
  sharing claim and correct both the header comment and
  `docs/architecture.md`'s Components table.

### 4.5 Harden range/date validation (§3.5)

- IMP1, `History-Dashboard.md` — add a maximum date-range span check to
  `api_dashboard.php`.
- IMP2, `History-Dashboard.md` — reject a reversed date range with a 400
  instead of silently swapping it.
- IMP3, `History-Dashboard.md` — return a 400 for a malformed date instead
  of silently dropping it.
- IMP3, `Device-Ingestion.md` — add explicit `[0, 59]` range checks for
  `MIN`/`SEC`.
- IMP4, `Device-Ingestion.md` — replace the fixed `DAY` bound with a
  calendar-aware check (`checkdate()`).

### 4.6 Surface currently-hidden distinctions in the eventual React UI (§3.6/§3.7)

- IMP4, `Device-Pilotage.md` — visually distinguish the 5 settings fields
  that reach the device from the 5 that are DB-only.
- IMP5, `Device-Pilotage.md` — either render `LASTCALIBRATION` or stop
  selecting it.
- IMP2, `Live-Tracking.md` — drop the unconditional, unused Leaflet CSS
  load from `index.php`.
- IMP3, `Live-Tracking.md` — if "live" should mean "continuously updating,"
  add an explicit polling/refresh mechanism rather than relying on a manual
  re-fetch, giving the React rebuild a natural seam for a true live view.
- IMP5, `Live-Tracking.md` — formally document the row/hook JSON shape
  `map_timeline.js` consumes, ideally alongside the KAN-116 REST response
  shape.
- IMP4, `Client-Branding-and-Assets.md` — reconcile the two different
  responsive breakpoints (`768px` in `styles.css` vs. `480px` in
  `auth.css`) into one set of design tokens before the rebuild defines its
  own.
- IMP5, `Client-Branding-and-Assets.md` — record/validate an actual MIME
  type for a stored logo instead of hardcoding `image/png`.
- IMP6, `History-Dashboard.md` — re-evaluate the single/double-letter
  response field names (`c`, `t`, `ta`, …) for readability if any part of
  this contract is carried into the documented `/api/v1` OpenAPI spec.

### 4.7 Harden the device-ingestion credential model (§3.9)

- IMP1, `Device-Ingestion.md` — move from one shared `DEVICE_SHARED_KEY`
  to a per-device credential.
- IMP5, `Device-Ingestion.md` — explicitly scope any future auth-model or
  validation change to its own story, since this area is deliberately
  outside both the KAN-116 and KAN-60 epics.

### 4.8 Session-change notification consistency (§3.8)

- IMP2, `Password-Recovery-and-Account-Management.md` — make
  `account.php`'s post-password-change e-mail state explicitly that other
  sessions were closed, matching `reset_password.php`'s wording.

### 4.9 Conditional-request caching for the logo endpoint

- IMP2, `Client-Branding-and-Assets.md` — add `If-Modified-Since`/`ETag`
  support to complement the already-set but currently-unused
  `Last-Modified` header.

### 4.10 Share constants/contracts between the legacy pages and the new REST+React layer

- IMP5, `Password-Recovery-and-Account-Management.md` — keep
  `RESET_TOKEN_MINUTES`/`RESET_MAX_PER_HOUR` as the single source for both
  the new API's responses and any copy shown in the new UI.
- IMP6, `Password-Recovery-and-Account-Management.md` — add a
  `Referrer-Policy` header to `reset_password.php`, which carries a live
  reset token in its URL alongside a third-party stylesheet load.

## 5. Readiness for KAN-60

- **Auth & Session.** The login/lockout/CSRF/session-epoch machinery is
  solid and well-tested conceptually, but has real holes the new UI's error
  states need to account for honestly rather than copy blindly: the
  per-IP throttle can be bypassed by a malformed POST (GAP1), a lockout
  doesn't give a clean 5-attempt reset after it expires (GAP2), and one
  rejection branch (locked account) already leaks state the others don't
  (GAP3/GAP4, `Auth-and-Session.md`). Don't design new copy/UX around
  `docs/security.md`'s current "no rate limiting" claim — it's stale
  (GAP5).
- **Password Recovery & Account Management.** The self-service
  `account.php` flow is the one password-adjacent page with no IP throttle
  on its own password check (GAP1) and the one whose "your other sessions
  were closed" messaging is silently missing compared to
  `reset_password.php` (GAP2) — both worth deciding on before porting the
  account page's UX 1:1. Also note `PWD_CHECK_HIBP` defaults off (GAP3):
  don't assume the HIBP check is "on" in a fresh environment just because
  `docs/security.md` lists it as a strength.
- **Live Tracking.** `index.php` does not actually render a map at all
  today, despite `map_timeline.js`'s own header comment framing itself as
  that page's "LIVE view" (GAP-1, GAP-2, GAP-3, `Live-Tracking.md`) — the
  React rebuild must not assume `index.php`'s historical behavior included
  a working live map; it needs a product decision on whether to build one
  from scratch, drop the Leaflet CSS load entirely, and whether "live"
  should actually mean continuously updating (there is no polling anywhere
  in the current code).
- **History Dashboard.** This is the one area where `map_timeline.js`'s
  full historical/scrubbable feature set (time-slider, trail replay,
  tooltip, auto-fit) is real and actually wired up — treat it as the
  authoritative source for that component's behavior, not
  `Live-Tracking.md`'s read of the same file. Separately, the date-handling
  contract the React rebuild should design against (`api_dashboard.php`) is
  more permissive than the KAN-116 REST endpoint is specified to be
  (reversed ranges swapped not rejected, malformed dates dropped not
  rejected, no max span — GAP2/GAP3/GAP4) — decide explicitly whether the
  new UI should surface the legacy page's permissiveness or the REST
  layer's stricter rules, since they currently disagree.
- **Device Pilotage.** The "MAIN contract only" and "contract is active
  today" rules that gate this whole page exist only inside `pilotage.php`
  itself, with no shared, reusable definition anywhere else in the
  codebase (GAP1–GAP5, `Device-Pilotage.md`) — the React rebuild/KAN-116
  REST work needs to either inherit a definition that doesn't yet formally
  exist or define one as new work, not assume one is already centralized.
  Also: the settings form currently presents 10 fields as if they're
  equivalent, but only 5 actually reach the device (GAP6) — the new UI
  should not repeat that ambiguity.
- **Client Branding & Assets.** `client_logo.php` currently tells a
  non-admin caller, by status code and timing, whether a `client_id` is
  theirs or merely has no logo (GAP1) — this is exactly the distinction the
  planned `/api/v1/clients/{clientId}/logo` endpoint is designed to erase,
  so the React rebuild's error handling for a missing logo should assume
  the new unified-404 behavior, not the current page's two-code behavior.
  The shared CSS also carries two different mobile breakpoints (768px vs.
  480px, CB22) that should be reconciled into the rebuild's own design
  tokens rather than carried forward as two conventions.
- **Device Ingestion.** This area is explicitly non-UI and does not feed
  KAN-60 directly (per its own doc's framing) — nothing here needs
  React-side design work. It is relevant to KAN-60 only indirectly, as a
  reminder that the shared-secret device-auth model (GAP1) and permissive
  date validation (GAP3/GAP4) are backend-only concerns that belong to a
  separate story if ever picked up, not to the UI rebuild.

## 6. Open Questions Rollup

**Server-only / gitignored code that could not be verified from this repo**
(recurring across nearly every area — the single most common type of open
question in this epic):
- Whether `X-Forwarded-Proto` is actually only trusted from a configured
  Traefik hop (reverse-proxy config not in repo) — `Auth-and-Session.md`.
- Where/how `MustChangePassword` is first set to `1` (presumably an
  admin-only script not found in scope) — `Password-Recovery-and-Account-Management.md`.
- `config.php`/`mailer.php`'s actual SMTP credentials/transport — gitignored
  — `Password-Recovery-and-Account-Management.md`.
- The exact firing condition of the `trg_user_role_guard_bu` DB trigger
  (schema not in repo) — `Password-Recovery-and-Account-Management.md`.
- Whether `client_logo.php` independently re-validates access or trusts its
  caller, and the exact SQL type/size of `T_CLIENTS.Logo`/`LogoUpdatedAt`
  (schema not in repo) — `Client-Branding-and-Assets.md`.
- The actual encoding/transport behavior of `ttn_downlink_reglages.php` and
  `ttn_webhook.php` (not in repo; all claims about them are taken from
  `pilotage.php`'s own comments, not verified directly) —
  `Device-Pilotage.md`.
- The TTN bridge scripts that call `insert_data.php`
  (`ttn_webhook.php`, `ttn_downlink_reglages.php`, `ttn_recovery.php`,
  `retrieve_data.php`), their exact payload shape, retry/dedup behavior, and
  how `DEVICE_SHARED_KEY` is provisioned/rotated — all gitignored —
  `Device-Ingestion.md` (two separate open questions).
- Whether a maximum practical row count exists at the `V_DATA_WITH_CLIENT`
  database/view level independent of the 25000-point response sampling (no
  index/EXPLAIN info available) — `History-Dashboard.md`.

**Whether an observed discrepancy is a bug, a deliberate design, or a stale
artifact (needs a human/business-owner call, not further code reading):**
- Whether `FailedLoginCount` never resetting after `LockedUntil` expires
  (GAP2) is an intentional progressive-lockout design or an oversight —
  `Auth-and-Session.md`.
- Whether `map_timeline.js`'s "LIVE view (`index.php`)" framing reflects a
  feature that existed earlier and was removed, or one that was planned but
  never wired up — raised independently in both `Live-Tracking.md` and
  `History-Dashboard.md` (the latter explicitly hands the question to the
  former as the one covering `index.php`'s side).
- Whether an expired STD contract should be rejected outright as a transfer
  target (GAP5/IMP3) — a product call, not yet made —
  `Device-Pilotage.md`.
- Whether the 5 DB-only `$CHAMPS` settings fields (GAP6) are legacy/dead
  columns or deliberately reserved for a future downlink version —
  `Device-Pilotage.md`.
- Whether any upload path validates stored client-logo bytes are actually
  PNG (no upload-handling file was in scope) — `Client-Branding-and-Assets.md`.

**Interaction with the in-progress KAN-116 REST extraction (will need
re-checking once that work lands, not resolvable from this repo today):**
- Whether KAN-119/120's parallel extraction (including "a password
  change/reset revokes all of the account's live API tokens") has landed in
  any of the files read for `Password-Recovery-and-Account-Management.md` —
  not found as of that story's review; flagged for re-check against the
  actual diff once it merges.
- Whether KAN-116's planned `/api/v1/devices/positions` endpoint changes
  how the React rebuild should consume `index.php`'s underlying data —
  `Live-Tracking.md`.
- Whether KAN-116's `/api/v1/contracts/{id}/history` endpoint will replace
  `api_dashboard.php`'s call from `dashboard.php`, run alongside it, or
  expose the same data under a token-authenticated contract — unresolved,
  and it determines whether GAP2–GAP4 (§3.5) should be fixed on the legacy
  endpoint too or only enforced going forward on the new one —
  `History-Dashboard.md`.
- Whether "inactive" for KAN-116's planned 409-on-inactive-write rule is
  meant to be exactly `pilotage.php`'s `$contratActif` date-window
  computation, or some other derived status — there is no stored "active"
  flag on `T_CONTRATS` to confirm against — `Device-Pilotage.md`.
- The in-progress KAN-116 REST implementation of the unified-404 logo
  endpoint, and whether it also addresses the timing asymmetry noted in
  GAP1/IMP1, was not inspected as part of this epic —
  `Client-Branding-and-Assets.md`.

## 7. Final tally

✅ **Epic KAN-126 complete: 7 feature areas documented, 183 total CB-n
items, 31 total GAP-n items, 39 total IMP-n items, file coverage 100%.**
