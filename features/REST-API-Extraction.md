1. Feature Name

REST API Extraction

2. Problem / Pain Point

`cpaltracker.web` has no general-purpose API. `api_dashboard.php` and
`insert_data.php` are the only JSON endpoints, and both are single-purpose:
the first is session+CSRF-gated JSON built only for the page's own
JavaScript (no API key path, no CORS, no versioning), the second is a
device-ingestion webhook authenticated with a shared secret. Every other
capability (login, logout, password reset, account management, live
positions, history/charts, device configuration, client logo) is only
reachable by requesting a server-rendered HTML page.

This blocks the planned UI refactor (KAN-60, "a beautiful UI for the
December salon", including "Prepare React transition"): a React SPA cannot
be fed by page files that mix HTML rendering with business logic and have
no stable, documented JSON contract. `cpaltracker.web/docs/architecture.md`
already names this gap explicitly under "What a React/TypeScript frontend
would need" — stable JSON contracts, token-based auth + CORS, and
separating data access from page rendering.

3. Goal / Outcome (Success Definition)

Every capability the current PHP pages expose is also reachable through a
versioned, documented JSON REST API (`/api/v1/...`), built by extracting
the existing business logic into functions the new endpoints call —
**without changing the behavior, routes, templates or auth model of any
existing page**. When this feature is done, a future frontend (React or
otherwise) can be built entirely against `/api/v1/...` while every current
`.php` page keeps working exactly as it does today, unmodified.

4. User Stories (Primary Scenarios)

- As a future frontend client, I want to authenticate via a REST endpoint
  and receive a session/token I can use on subsequent calls, so that I am
  not dependent on a server-rendered login form.
- As a future frontend client, I want to fetch the current device
  positions and a contract's historical readings as JSON, so that I can
  render the live map and history charts myself.
- As a future frontend client, I want to read and update account details
  and trigger password reset via REST endpoints, so that account
  management does not require the existing HTML forms.
- As a MAIN-contract client integrating their own tooling, I want to read
  and update device configuration and transfer device visibility to a
  sub-client via REST endpoints, so that device pilotage is scriptable.
- As a platform maintainer, I want every extracted endpoint backed by unit
  tests that exercise the shared business-logic functions directly (DB
  mocked at the boundary), so that the new API layer is trustworthy without
  needing a live environment to verify it.
- As a platform maintainer, I want the existing pages left completely
  untouched by this work, so that shipping the API carries zero regression
  risk to the product currently in production.

5. In Scope (What WILL be delivered)

- A versioned JSON API under `/api/v1/` (new PHP files only; a single
  lightweight front controller or one file per endpoint — implementation
  detail for the Design step, not fixed here).
- Extraction of the business logic currently inlined in each page
  (`login.php`, `logout.php`, `forgot_password.php`, `reset_password.php`,
  `account.php`, `index.php`, `dashboard.php`, `api_dashboard.php`,
  `pilotage.php`, `client_logo.php`) into plain functions that take their
  dependencies as parameters (continuing the pattern `auth.php` and
  `password_policy.php` already follow) — called by both the existing page
  and the new endpoint, so there is exactly one implementation of each rule.
- REST endpoints, one group per feature area:
  - **Auth**: login, logout, forgot-password, reset-password.
  - **Account**: read account, update account.
  - **Live tracking**: current device positions (replaces `index.php`'s
    data source).
  - **History**: a contract's historical readings / chart data (replaces
    `dashboard.php`'s data source), scoped exactly like today
    (`cpal_contrat_autorise()`).
  - **Device pilotage**: read/update device configuration, transfer device
    visibility to a sub-client — MAIN-contract only, same authorization
    rule as `pilotage.php` today.
  - **Client assets**: client logo lookup.
- A token-based auth path for the new API (additive: existing session+CSRF
  auth is untouched; the new endpoints accept a bearer token issued by the
  new login endpoint) plus explicit CORS headers scoped to configured
  origins, per the gap named in `architecture.md`.
- An OpenAPI (or equivalent) description of the `/api/v1/` contract.
- Full unit test coverage of every extracted business-logic function
  (DB/mailer/HIBP mocked at the boundary) and of each new endpoint's
  request handling (auth, validation, status codes, response shape).
- A PHPUnit test harness (via Composer, `composer.json`/`vendor/` newly
  introduced — the one accepted new dependency) wired into CI alongside
  the existing syntax lint, with PDO/mailer/HIBP mockable at the boundary.
  Required to meet NFR2; there is no test tooling in the repo today.

6. Out of Scope (What WILL NOT be delivered)

- Any change to the existing `.php` page files' routes, markup, or
  user-visible behavior. They keep rendering HTML and keep working exactly
  as today; this feature only adds a parallel JSON path.
- Any change to `insert_data.php` (device ingestion) or the TTN bridge —
  out of this repo already (see `deployment.md`), and functionally already
  a machine API with its own auth model; not in scope for this extraction.
- The React frontend itself (KAN-60 covers that separately); this feature
  only delivers the API it will consume.
- Rate limiting / lockout, CSP and other security-header hardening,
  Subresource Integrity on CDN assets — pre-existing gaps named in
  `security.md`, not introduced or fixed by this feature.
- Database schema changes.

7. User Flow / UX Breakdown

- A client app calls `POST /api/v1/auth/login` with credentials.
- System validates credentials via the extracted auth logic (same rules as
  `login.php`), issues a token, returns it with the account summary.
- Client app calls `GET /api/v1/devices/positions` with the token.
- System validates the token, applies the same contract-scoping rule as
  `index.php`, returns current positions as JSON.
- Client app calls `GET /api/v1/contracts/{id}/history?start=&end=`.
- System re-validates `cpal_contrat_autorise()` for that contract and
  token, returns the same data `dashboard.php`/`api_dashboard.php` compute
  today, as JSON.
- Client app calls `POST /api/v1/auth/logout`; system invalidates the
  token.
- A MAIN-contract client calls `GET/PUT /api/v1/devices/{id}/config` and
  `POST /api/v1/devices/{id}/transfer`; system applies the same
  MAIN-contract-only authorization `pilotage.php` enforces today.

8. Functional Requirements (FRs)

- FR1. System SHALL expose `POST /api/v1/auth/login` that validates
  credentials using the same rules as `login.php` and returns an API
  token plus account summary on success.
- FR2. System SHALL expose `POST /api/v1/auth/logout` that invalidates the
  presented API token.
- FR3. System SHALL expose `POST /api/v1/auth/forgot-password` that
  triggers the same reset-email flow as `forgot_password.php`.
- FR4. System SHALL expose `POST /api/v1/auth/reset-password` that applies
  the same password-reset rules as `reset_password.php` (including the
  NIST/HIBP password policy).
- FR5. System SHALL expose `GET /api/v1/account` returning the
  authenticated user's account details, equivalent to what `account.php`
  displays.
- FR6. System SHALL expose `PATCH /api/v1/account` applying the same
  validation and update rules `account.php` applies today, across its
  three updatable fields (password, phone, email); every field change
  SHALL require the current password, exactly as `account.php` requires
  today; an email change SHALL send the same old-address and new-address
  notification emails `account.php` sends today.
- FR7. System SHALL expose `GET /api/v1/devices/positions` returning the
  current device positions for the authenticated user's accessible
  contracts, equivalent to the data `index.php` renders, with an optional
  `contractId` query filter restricting the result to one accessible
  contract.
- FR8. System SHALL expose `GET /api/v1/contracts/{contractId}/history`
  returning the historical readings (temperature/humidity/timeline) for
  that contract within a date range, equivalent to the data
  `dashboard.php`/`api_dashboard.php` compute today, enforcing the same
  `cpal_contrat_autorise()` scoping. `start`/`end` query parameters are
  optional ISO 8601 dates (`YYYY-MM-DD`); if `end` precedes `start` the
  request is rejected with 400 (no silent swap, unlike the page); the
  maximum span is 1 year.
- FR9. System SHALL expose `GET /api/v1/devices/{deviceId}/config`
  returning a device's current uplink configuration. The endpoint
  resolves the device's owning contract and applies
  `cpal_contrat_autorise()` plus the existing `ContratType === 'MAIN'`
  check, equivalent to `pilotage.php`'s read path.
- FR10. System SHALL expose `POST /api/v1/devices/{deviceId}/config`
  queuing a configuration change for the next downlink (a command, not an
  idempotent replace — hence `POST`, not `PUT`), under the same
  MAIN-contract check as FR9, refused with 409 if the owning contract is
  inactive (`contratActif`), equivalent to `pilotage.php`'s write path.
- FR11. System SHALL expose `POST /api/v1/devices/{deviceId}/transfer`
  transferring a device's visibility to a sub-client, under the same
  MAIN-contract check as FR9, refused with 409 if the owning contract is
  inactive or the transfer target is not a `STD` contract attached to
  that `MAIN` contract, equivalent to `pilotage.php`'s transfer path.
- FR12. System SHALL expose `GET /api/v1/clients/{clientId}/logo`
  returning the client's logo image (`image/png` body) on success, or the
  FR13 JSON error shape on failure. Unlike `client_logo.php`, both "no
  access" and "no logo for this client" return a uniform 404 — the API
  SHALL NOT reveal whether an out-of-scope client exists.
- FR13. Every FR1-FR12 endpoint SHALL return a consistent JSON error shape
  (status code + machine-readable error code + message) on validation
  failure, auth failure, or authorization failure.
- FR14. System SHALL publish an OpenAPI (or equivalent) description
  covering every endpoint in FR1-FR12.
  **Implementation note (KAN-125):** this app has no router/rewrite rule
  (one PHP file per URL, per `docs/architecture.md`), so every endpoint's
  real, reachable URL has a `.php` suffix (e.g. `/api/v1/auth/login.php`,
  not `/api/v1/auth/login`); the four id-bearing endpoints additionally
  accept the id via a trailing PATH_INFO segment
  (`/api/v1/devices/config.php/{deviceId}`) or, as a permanent fallback,
  a `?xxxId=` query parameter. `api/openapi.yaml` documents the physical
  `.php` form directly so every operation is actually callable; see its
  `info.description` and `docs/architecture.md`'s "Physical URL mapping"
  section for the full rationale.
- FR15. None of FR1-FR14 SHALL change the routes, markup, or observable
  behavior of any existing `.php` page. Extraction SHALL edit a page only
  to make it call a shared function it previously inlined; existing pages
  SHALL be verified against the regression checklist in §15 (Area 9).
  **Exception**: FR4/FR6 password changes additionally revoke all of that
  account's live API tokens (see NFR8) — a deliberate, in-scope behavior
  addition to close the gap in Flagged Issue 3, reachable through the
  existing page too.

9. Non-Functional Requirements (NFRs)

- NFR1. **Additive only**: this feature SHALL NOT alter any existing file's
  externally observable behavior; extraction SHALL move logic into shared
  functions called by *both* the old page and the new endpoint, never
  change what the old page returns.
- NFR2. **Full unit test coverage**: every extracted business-logic
  function SHALL have unit tests (DB, mailer and HIBP calls mocked at the
  boundary) covering its success path and each documented edge case /
  error path; every new endpoint SHALL have unit tests covering auth
  failure, authorization failure, validation failure and the success
  response shape.
- NFR3. New endpoints SHALL enforce the same deny-by-default, server-side
  authorization as the page they extract from (contract scoping,
  MAIN-contract-only checks) — re-checked per request, not cached.
- NFR4. New endpoints SHALL use parameterized queries only (reusing the
  existing PDO layer) and SHALL NOT introduce any new secrets into git.
- NFR5. The API token mechanism introduced for FR1/FR2 SHALL be additive:
  existing session-cookie + CSRF auth SHALL continue to work unchanged for
  every existing page.
- NFR6. CORS headers on `/api/v1/` SHALL be explicit and scoped to a
  configured allow-list of origins read from the `API_CORS_ALLOWED_ORIGINS`
  env var via `cpal_env()` (`config.php`) — never
  `Access-Control-Allow-Origin: *` on an authenticated endpoint. A request
  from an origin not on the allow-list receives no `Access-Control-*`
  headers (browser blocks it client-side); `OPTIONS` preflight is handled
  by the Area 1 front controller.
- NFR7. Response time for any `/api/v1/` endpoint SHALL be within 10x its
  existing page equivalent's response time, measured against the same
  docker-compose seed dataset (`docs/local-setup.md`), with no new N+1
  query pattern introduced during extraction — verified by a query-count
  assertion in that function's unit test plus a timing comparison recorded
  in Area 9 (§6 Future Extensions tracks tightening this bound later).
- NFR8. API tokens SHALL be stored as a new `TokenType = 'api'` row in the
  existing `T_AUTH_TOKEN` table (no schema change — reuses
  `cpal_token_creer`/`valide`/`revoquer` from `auth.php`), with an absolute
  expiry only (no idle/sliding window, since `T_AUTH_TOKEN` has no
  last-seen column). A password change or reset (FR4/FR6, or the existing
  page paths that reach the same shared function) SHALL revoke every live
  `api` token for that account, the same way `SessionEpoch` already
  invalidates sessions today.

10. Data Inputs & Outputs

- Inputs: JSON request bodies (login credentials, password-reset tokens,
  account fields, device config fields, transfer target) and query/path
  parameters (contract id, device id, client id, date range), mirroring
  the fields each existing page/form already collects.
- Outputs: JSON responses mirroring the data each existing page already
  renders (account summary, position list, history/chart series, device
  config, logo binary via a dedicated image response, error shape per
  FR13).
- No new data is introduced; this feature re-exposes existing data through
  a new transport, it does not add fields the DB schema doesn't have.

11. Integration Points

- MySQL via the existing PDO layer (`config.php`) — read/write, same
  tables as today (`T_USER`, `T_DEVICE_HIST`, contract/device tables).
- `mailer.php` (password-reset emails) — reused by the extracted
  forgot/reset-password logic, same as `forgot_password.php` today.
- Have I Been Pwned k-anonymity check (`password_policy.php`) — reused by
  the extracted reset-password logic, same fail-open behavior.
- No new external integrations.

11a. Software Items (IEC 62304 §5.3)

Not applicable — c-PAL has not adopted the Software Item registry (see
`cpal.docs/sdlc/README.md#applicability-to-c-pal-today`). Left empty per
that doc's instruction.

12. Edge Cases & Constraints

- Expired or already-invalidated API token presented on any `/api/v1/`
  call → 401 with the FR13 error shape, same semantics as an expired PHP
  session.
- A non-MAIN-contract token calling a device-pilotage endpoint → 403, same
  rule `pilotage.php` enforces today.
- A token's contract scope not covering the requested `contractId`/
  `deviceId`/`clientId` → 403/404 (do not leak existence of out-of-scope
  resources), mirroring the fixed `dashboard.php`/`api_dashboard.php`
  scoping bug documented in `security.md`.
- HIBP unreachable during `POST /api/v1/auth/reset-password` → fail open
  (allow), identical to `password_policy.php`'s existing documented
  choice.
- Malformed/missing JSON body, missing required field, or `end` before
  `start`/span over 1 year on the history endpoint → 400 with the FR13
  error shape, never a 500.
- Concurrent login/logout or config read/write via both the old page and
  the new endpoint for the same account/device → both paths read/write the
  same tables through the same shared functions, so behavior stays
  consistent; no new race condition is introduced by having two entry
  points.
- An account's password is changed or reset (via FR4/FR6 or the existing
  pages) → every live `api` token for that account is revoked immediately
  (NFR8); a subsequent call with the old token → 401.
- A request whose `Origin` is not on `API_CORS_ALLOWED_ORIGINS` → no
  `Access-Control-*` response headers; the browser blocks the response
  client-side.
- A login for an account with `MustChangePassword=1` → token is issued but
  scoped to the `PATCH /api/v1/account` password-change action only; any
  other endpoint called with that token → 403 with a dedicated
  `must_change_password` error code (mirrors `cpal_require_login`'s
  existing page-level redirect-to-`account.php` behavior).
- Login attempts from an IP already blocked by `cpal_ip_bloquee` →
  rejected the same way `login.php` rejects them today; the throttle
  applies to `POST /api/v1/auth/login` identically.

13. Metrics / KPIs

- 100% of FR1-FR12 endpoints implemented and covered by unit tests
  (NFR2).
- 0 regressions in existing page behavior (NFR1) — verified by the
  existing manual/CI checks on each untouched page still passing.
- 100% of FR1-FR15 and NFR1-NFR8 traced to at least one user story
  (Workflow 3 coverage check).

14. Risks & Mitigations

- Risk: extraction accidentally changes a page's behavior while moving its
  logic into a shared function. Mitigation: extract by moving code
  verbatim into a function with the same inputs/outputs first, then have
  the page call that function — no logic rewrite in the same step; cover
  with unit tests before and after the move.
- Risk: a new token-auth path weakens the deny-by-default posture
  `security.md` credits the current app with. Mitigation: NFR3 requires
  re-checking authorization per request exactly like today; review gate
  (Security lens) blocks on any endpoint missing this check.
- Risk: scope creep into building the React frontend itself. Mitigation:
  Out of Scope §6 explicitly excludes it; this feature ends at a
  documented, tested API.

15. Acceptance Criteria (Definition of Done)

- [ ] `/api/v1/auth/login`, `/logout`, `/forgot-password`, `/reset-password`
      implemented and covered by unit tests (FR1-FR4, NFR2).
- [ ] `/api/v1/account` GET/PATCH implemented and covered by unit tests
      (FR5-FR6, NFR2).
- [ ] `/api/v1/devices/positions` implemented and covered by unit tests
      (FR7, NFR2).
- [ ] `/api/v1/contracts/{id}/history` implemented and covered by unit
      tests (FR8, NFR2).
- [ ] `/api/v1/devices/{id}/config` GET/PUT and `/transfer` implemented and
      covered by unit tests, MAIN-contract-only enforced (FR9-FR11, NFR3).
- [ ] `/api/v1/clients/{id}/logo` implemented and covered by unit tests
      (FR12).
- [ ] Consistent error shape on every endpoint (FR13).
- [ ] OpenAPI description published covering FR1-FR12 (FR14).
- [ ] Every existing `.php` page verified unchanged in observable
      behavior (FR15, NFR1) — no diff to existing page files beyond the
      extraction of logic into shared functions they now call, checked
      against the Area 9 regression checklist.
- [ ] API tokens stored in `T_AUTH_TOKEN` (type `api`) and revoked on
      password change/reset (NFR8).
- [ ] All FRs and NFRs validated.

16. Future Extensions (Optional, Not Required Now)

- Migrating `index.php`/`dashboard.php`/etc. themselves to consume the new
  `/api/v1/` endpoints instead of inlining their own queries (would retire
  the duplication NFR1 temporarily accepts).
- Versioning `insert_data.php` under `/api/v1/` once the TTN bridge code is
  brought into this repo.
- Rate limiting / lockout on the new auth endpoints.
