# Feature: React Password Recovery & Account Management

> Forward-looking product spec for the **new** React/TypeScript implementation of
> `cpaltracker.web`'s forgot-password, reset-password and self-service account
> flows, delivered as a child epic of **KAN-60** (React UI rebuild). Written per
> [`FEATURE.md`](../sdlc/product/FEATURE.md).
>
> This spec is forward-looking: it states what the **new React UI MUST do**. It
> does **not** restate requirement text — the authoritative reverse-engineered
> current behavior lives in
> [`technical/Password-Recovery-and-Account-Management.md`](technical/Password-Recovery-and-Account-Management.md)
> (CB-n = behavior to replicate, GAP-n = known issues). Each FR below cites the
> CB-n it replicates or the GAP-n it resolves. The stack is fixed by
> [`architecture/react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md)
> and is not re-decided here. No backend, API contract or database changes are in
> scope (stack decision §5); `/api/v1` (KAN-116, Done) is consumed as-is.

## 1. Feature Name

React Password Recovery & Account Management — the forgot-password request page,
the reset-password (token-redemption) page, and the authenticated self-service
account page (password / phone / e-mail change, including the forced
"first-connection" password change), rebuilt as React screens consuming `/api/v1`.

## 2. Problem / Pain Point

`cpaltracker.web`'s password-recovery and account-management screens are
server-rendered PHP pages (`forgot_password.php`, `reset_password.php`,
`account.php`) that cannot be evolved without touching the monolith, carry no
shared design system, ship no automated UI tests, and embed UX inconsistencies
that have been formally identified (see GAP1–GAP4 in the Technical-Feature doc).
Users who forget a password, redeem a reset link, or manage their own
credentials experience the oldest, least-maintainable corner of the product.
Without a React rebuild, every other KAN-60 feature area would ship on the new
stack while the security-sensitive credential flows stay stranded on the legacy
pages — forcing users across two visual languages and leaving the known gaps
unaddressed.

## 3. Goal / Outcome (Success Definition)

A user can request a password reset, redeem a reset link, and manage their own
password, phone and e-mail entirely within the new React application, with
behavior that replicates every current safeguard (enumeration-safe responses,
token single-use, password policy, session invalidation) and resolves the known
UX gaps where the frontend can. Success looks like: the three flows feel native
to the new design system (dark/light, desktop-first), every current protective
behavior is preserved, a password change clearly tells the user their other
sessions were closed, and the legacy PHP pages keep serving traffic untouched in
parallel. All behavior is covered by Vitest unit tests and Playwright E2E tests.

## 4. User Stories (Primary Scenarios)

- **US1 — Forgot my password.** As a tracker user who forgot my password, I want
  to submit my e-mail on a forgot-password page and always receive the same clear
  "check your mailbox" confirmation, so that I can start recovery without the page
  revealing whether my account exists. *(covers FR1, FR2, FR3, NFR3)*
- **US2 — Redeem a reset link.** As a user who received a reset e-mail, I want to
  open the link, set a new password with live policy feedback, and see a clear
  confirmation that my password changed and my other sessions were closed, so that
  I regain access and understand the security effect. *(covers FR4, FR5, FR6, FR7,
  FR8, NFR4, NFR7)*
- **US3 — First connection.** As a newly provisioned user forced to change my
  password on first login (`MustChangePassword = 1`), I want the account page to
  show only the password-change form, so that I set a compliant password before
  reaching the rest of the app. *(covers FR9, FR10, FR11, FR12)*
- **US4 — Manage my account.** As an authenticated user, I want to change my
  password, phone number or e-mail from a self-service account page — each gated
  by my current password and clearly validated — so that I keep my credentials up
  to date safely. *(covers FR11, FR12, FR13, FR14, FR15, NFR6)*
- **US5 — Consistent, themeable UI.** As any user, I want these screens to match
  the product's design system in both dark and light mode on desktop, so that the
  credential flows feel like part of the same application. *(covers NFR5)*
- **US6 — Trustworthy, tested delivery.** As the delivery team, I want these
  screens built on the agreed stack, wired only to `/api/v1`, and covered by unit
  and E2E tests, with the legacy PHP pages left running, so that we ship safely
  without disrupting production. *(covers NFR1, NFR2, NFR8, NFR9, NFR10, NFR11)*

## 5. In Scope (What WILL be delivered)

- A React **forgot-password** page: e-mail form, client-side e-mail-format
  validation, enumeration-safe single confirmation state.
- A React **reset-password** page: token read from the URL, new-password +
  confirmation fields, live policy feedback, distinct error states for
  invalid/expired/used/blocked tokens, success confirmation including the
  "other sessions closed" message.
- A React **account** self-service page with three panels — password change,
  phone change, e-mail change — each requiring the current password, plus the
  forced "first-connection" reduced-form mode.
- Client-side validation and normalization that mirrors current rules (e-mail
  format/lowercase/trim, French-national → E.164 phone normalization, new ≠
  current password, confirmation match).
- Faithful rendering of all server-side validation outcomes returned by `/api/v1`
  (password-policy failures, HIBP breach count, duplicate e-mail, throttling).
- Integration with the `AuthContext` thin REST/cookie client against `/api/v1`.
- Design-system compliance (theme tokens, dark + light, desktop-first) and
  React Hook Form for every form.
- Vitest + Testing Library unit tests and Playwright E2E tests for all three
  flows.

## 6. Out of Scope (What WILL NOT be delivered)

- **Any backend, API-contract or database change.** `/api/v1` (KAN-116) is
  consumed exactly as built (stack decision §5). Gaps found against it are logged
  as open questions for a separate backend story, never worked around in the
  frontend.
- **Any change to the PHP monolith.** `forgot_password.php`, `reset_password.php`,
  `account.php`, `password_policy.php` and `auth.php` keep running unmodified in
  parallel; no cutover is performed here.
- **Mobile-first design.** Desktop is the primary, first-class target (product
  decision 2026-10-15, stack decision §4); mobile layout is best-effort and not
  separately acceptance-tested in this epic.
- Server-side rate limiting / IP throttling logic itself (owned by `/api/v1`);
  the UI only reacts to throttle responses (see GAP1 decision).
- Enabling/configuring `PWD_CHECK_HIBP`, SMTP/mailer, or the admin flow that first
  sets `MustChangePassword = 1` (all backend/ops, out of frontend scope).
- E-mail template content and delivery (server-owned; the UI never composes mail).

## 7. User Flow / UX Breakdown

**Forgot password**
1. User opens the forgot-password route and enters their e-mail.
2. UI validates e-mail format client-side; on blur/submit shows inline errors.
3. User submits; UI calls `/api/v1` forgot-password via the AuthContext client.
4. For any account-existence-related result (unknown / inactive / per-account
   rate-limited / send failure / success), the UI shows one identical "check
   your mailbox" confirmation state; a per-IP-throttled request (CB3) or a
   malformed submission instead get their own distinct, non-leaking message
   (stack-decision.md §10).

**Reset password**
1. User opens the reset link; UI reads the `t` token from the query string and
   keeps it internally (never shown as text, retained across resubmission).
2. User enters a new password + confirmation; UI shows live policy hints and a
   match check.
3. User submits; UI calls the `/api/v1` reset endpoint with the token.
4. On success: confirmation screen stating the password changed and all other
   open sessions were closed, with a route to log in.
5. On failure: a distinct, non-leaking message for policy rejection, HIBP breach
   count, invalid/expired/already-used token, or IP-blocked.

**Account (authenticated)**
1. AuthContext gates the route; unauthenticated users are redirected to login
   with a return path.
2. If `MustChangePassword = 1` or `?premiere` is present, only the
   password-change form renders (phone/e-mail panels, nav and logout hidden).
3. For any action, the user enters their current password plus action fields.
4. On submit the UI calls the matching `/api/v1/account` action, validates
   client-side first, and renders the API's result — including, for a password
   change, an explicit "your other sessions were closed" message while keeping
   the acting session logged in.

## 8. Functional Requirements (FRs)

- **FR1.** The forgot-password page SHALL present an e-mail input and SHALL
  validate e-mail format client-side (React Hook Form) before enabling submit.
- **FR2.** The forgot-password page SHALL render a single, identical
  "check your mailbox" confirmation state for every account-existence-related
  outcome — unknown, inactive, per-account rate-limited, or mail-send failure —
  never revealing which occurred; a per-IP-blocked request (CB3) or a
  malformed submission SHALL instead get their own distinct, non-leaking
  message (matching login's own `ipThrottle` precedent), since neither
  reveals anything about whether the submitted e-mail belongs to an account.
  *(replicates CB1, CB3, CB4; §6 edge cases; stack-decision.md §10)*
- **FR3.** The forgot-password submission SHALL be sent through the `AuthContext`
  thin REST/cookie client to the `/api/v1` forgot-password endpoint, including any
  CSRF/session handling the API requires; the UI SHALL NOT mint or validate tokens
  itself. *(replicates CB2; token minting CB5–CB7 stays server-owned; stack §3)*
- **FR4.** The reset-password page SHALL read the reset token from the `t` URL
  query parameter, retain it across resubmission, and SHALL NOT display it as page
  text. *(replicates CB8; mitigates per NFR4 / IMP6)*
- **FR5.** The reset-password page SHALL provide new-password and confirmation
  fields, enforce a client-side match check, and submit the token + new password
  to the `/api/v1` reset endpoint via the AuthContext client. *(token re-validation
  and single-use commit CB11–CB14 stay server-owned)*
- **FR6.** The reset-password page SHALL render password-policy rejection messages
  and the HIBP breach-count rejection exactly as returned by `/api/v1`, without
  re-implementing the policy client-side. *(replicates CB12, CB17, CB18, CB19;
  see GAP3 decision for HIBP)*
- **FR7.** The reset-password page SHALL show distinct, non-leaking error states
  for an invalid, expired, already-used, or IP-blocked token response.
  *(replicates CB9, CB10)*
- **FR8.** On a successful reset the page SHALL show a confirmation stating the
  password was changed and that all other open sessions were closed, and offer a
  route to log in. *(replicates CB15, CB16)*
- **FR9.** The account page SHALL be gated by `AuthContext`; an unauthenticated
  user SHALL be redirected to the login route preserving the intended return path.
  *(replicates CB21)*
- **FR10.** The account page SHALL enter a "first-connection" mode when
  `MustChangePassword = 1` (from the authenticated user) or `?premiere` is
  present, rendering only the password-change form and hiding the phone panel,
  e-mail panel, in-app navigation and logout control. *(replicates CB22)*
- **FR11.** Each of the three account actions (password, phone, e-mail) SHALL
  require the user's current password in a dedicated field before submission.
  *(replicates CB23)*
- **FR12.** The account password-change action SHALL reject (client-side) a new
  password identical to the current one, render API policy/HIBP rejections, and on
  success SHALL (a) display an explicit "your other open sessions were closed"
  message and (b) keep the acting session logged in. *(replicates CB25, CB17, CB18,
  CB26; resolves GAP2 by adding the message CB27 omitted)*
- **FR13.** The account phone-change action SHALL normalize a French national
  number (`0` + 9 digits) to E.164 (`+33…`), accept an already-E.164 number,
  reject other formats inline, and treat an empty value as "clear the number."
  *(replicates CB28)*
- **FR14.** The account e-mail-change action SHALL lowercase and trim the input,
  validate format client-side, reject an address identical to the current one, and
  render the API's duplicate-address rejection; the UI SHALL reflect that the new
  address becomes unverified. *(replicates CB29, CB30)*
- **FR15.** All account actions SHALL handle an HTTP 429 / throttle response from
  `/api/v1` with a clear, consistent "too many attempts, try again later" message.
  *(frontend-side resolution of GAP1; see §12/§14 — the missing server throttle on
  the account password check is logged as an open backend question, CB24)*

## 9. Non-Functional Requirements (NFRs)

- **NFR1.** The feature SHALL be built on the KAN-60 reference stack: React 18.3 +
  TypeScript 5.5 + Vite 5, React Router v6, Zustand, Tailwind CSS 4, React Hook
  Form, per [`react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md).
- **NFR2.** All network I/O SHALL go through the `AuthContext` thin REST/cookie
  client against `/api/v1` only. No Supabase, and no new backend endpoint, API
  contract, or DB schema SHALL be introduced (stack §3, §5).
- **NFR3.** The forgot-password flow SHALL remain enumeration-safe: the frontend
  SHALL NOT introduce any UI, timing, or response difference that reveals account
  existence (reinforces FR2).
- **NFR4.** The reset token SHALL NOT be written to logs, analytics, local/session
  storage, or any persisted state; the reset-password page SHALL set a
  `Referrer-Policy` of `no-referrer` (or `same-origin`) to limit token leakage to
  third-party resources. *(IMP6)*
- **NFR5.** All screens SHALL consume the centralized design-system theme tokens,
  SHALL support dark and light themes, and SHALL be laid out desktop-first
  (stack §4).
- **NFR6.** All forms SHALL be accessible: every field labeled, keyboard
  navigable, with errors programmatically associated to their inputs (WCAG 2.1 AA
  target).
- **NFR7.** User-visible copy SHALL be French, consistent with the existing app;
  policy limits and durations the server owns (e.g. token lifetime, hourly reset
  cap) SHALL be taken from API responses, not re-hardcoded in frontend code.
  *(IMP5)*
- **NFR8.** Vitest + Testing Library (jsdom) unit tests covering every FR's client
  logic SHALL exist and pass before the story is Done.
- **NFR9.** Playwright E2E tests covering the three flows (forgot, reset, account
  incl. first-connection) against a running build SHALL exist and pass before Done.
- **NFR10.** The legacy PHP pages SHALL keep serving traffic unmodified; no
  disruption to the PHP monolith is an acceptance condition (stack §5).
- **NFR11.** The UI SHALL respond to user interactions (validation, field
  normalization, panel toggles) within 300 ms, excluding `/api/v1` network latency.

## 10. Data Inputs & Outputs

Inputs and outputs are HTTP request/response payloads against `/api/v1`
(exact contract owned by KAN-116; not restated here). Shapes the UI reads/writes:

- **Forgot-password:** in — `{ mail }` (+ CSRF/session as the API requires);
  out — a uniform success acknowledgement (no account-state detail by design).
- **Reset-password:** in — `{ t (token), motdepasse, confirmation }`; out —
  success, or a typed error (policy failure with message, HIBP breach count,
  invalid/expired/used token, IP-blocked).
- **Account:** in — `{ action: 'motdepasse'|'telephone'|'mail', actuel,
  + action-specific fields }`; out — success or a typed error (current-password
  wrong, policy failure, HIBP, new === current, invalid phone, invalid/duplicate
  e-mail, throttled). The new-e-mail becomes unverified server-side (CB30).
- **Auth/session:** cookie-based session managed by `AuthContext`; the client
  reads `MustChangePassword` and current user identity to drive FR9/FR10.

## 11. Integration Points

- **`/api/v1` (KAN-116, Done)** — read/write, via the AuthContext REST/cookie
  client. The only backend this feature talks to. Forgot-password, reset-password,
  and account (password/phone/e-mail) endpoints.
- **Browser URL / router (React Router v6)** — read of the `t` reset token and the
  `?premiere` flag; redirect to the login route from the account guard.
- **Design-system theme provider** (Core epic) — consumed for tokens and
  dark/light theming.
- **Have I Been Pwned, SMTP/mailer, DB triggers (`SessionEpoch`)** — reached only
  indirectly through `/api/v1`; the React app has no direct integration with any
  of them.

## 11a. Software Items (IEC 62304 §5.3)

**N/A.** IEC 62304 software items are *Not applicable* for c-PAL today — c-PAL
software has not entered a regulated scope (see "Applicability to c-PAL today" in
[`sdlc/README.md`](../sdlc/README.md); the Software Item Registry row is left empty
while that row is Not applicable). This section will be populated only if/when
c-PAL enters a regulated scope.

## 12. Edge Cases & Constraints

- Forgot-password for an unknown, soft-deleted, inactive, per-account
  rate-limited (CB4), or mail-failed account → identical confirmation (FR2,
  NFR3). A per-IP-blocked request (CB3) is NOT part of this set — it gets its
  own distinct, non-leaking throttle message, matching login's own
  `ipThrottle` precedent (stack-decision.md §10).
- Reset link opened in two tabs → server enforces single-use (CB11); the UI SHALL
  render the "already used / invalid" error on the losing tab (FR7).
- Reset link expired or revoked between open and submit → FR7 error state.
- Self-service password change that invalidates other sessions → the acting
  session is kept alive (CB26) and the UI shows the "others closed" message (FR12).
- Clearing the phone number (empty submit) is a valid action, not an error (FR13).
- E-mail change to an address owned by another account → API rejects; UI renders
  it (FR14).
- **GAP decisions (every GAP addressed):**
  - **GAP1 (account password check has no IP throttle).** *Fix at the frontend
    boundary + log backend follow-up.* The React UI handles a throttle/429 response
    consistently (FR15). The actual missing server-side throttle on the account
    password check is a `/api/v1` concern; per stack §5 it is logged as an open
    backend question (§14), **not** worked around in the frontend and **not** fixed
    in this epic.
  - **GAP2 (account password-change e-mail/UX never says "other sessions
    closed").** *Fixed in the new UI.* This is presentational and frontend-owned:
    FR12 adds the explicit "your other open sessions were closed" message on a
    successful account password change, matching the reset flow (CB16).
  - **GAP3 (`PWD_CHECK_HIBP` defaults off).** *Carried forward as-is at the
    frontend.* HIBP is server-side validation/config the React app does not
    control; the UI renders the HIBP rejection whenever `/api/v1` returns one
    (FR6) and assumes nothing about HIBP being on. Enabling it by default is a
    deployment/backend decision logged as an open question (§14, IMP3).
  - **GAP4 (stale "no rate limiting" claim in `docs/security.md`).** *Carried
    forward / out of frontend scope.* This is a documentation inaccuracy, not a
    behavior; it does not affect the React UI. The doc correction is owned by the
    backend/security docs (IMP4) and is recorded here only so no GAP is left
    unaddressed.

## 13. Metrics / KPIs

- Reset-link redemption success rate (successful resets ÷ valid redemption
  attempts) ≥ legacy baseline.
- Zero account-enumeration regressions: 100% of forgot-password outcomes render
  the identical confirmation (verified by E2E).
- 100% of FRs covered by at least one unit test and the three flows covered by
  E2E (NFR8, NFR9).
- Zero reported PHP-monolith disruptions during rollout (NFR10).
- UI interaction response < 300 ms at the 95th percentile (NFR11).

## 14. Risks & Mitigations

- **Risk: frontend accidentally leaks account existence** (e.g. different error on
  unknown e-mail). *Mitigation:* FR2/NFR3, enforced by an explicit E2E assertion
  that all outcomes render the same state.
- **Risk: reset token leaks via referrer/logs.** *Mitigation:* NFR4 (no
  persistence, `Referrer-Policy`), per IMP6.
- **Risk: GAP1's missing server throttle is mistaken as "fixed" because the UI
  shows a throttle message.** *Mitigation:* FR15 only handles the response; the
  real fix is logged as an **open backend question** for a separate `/api/v1`
  story and explicitly out of scope here.
- **Risk: re-hardcoding server-owned limits (token lifetime, hourly cap) in the
  UI, which then drift.** *Mitigation:* NFR7 — source such values from the API.
- **Risk: divergence from the legacy pages during parallel running.** *Mitigation:*
  FRs cite the CB-n they replicate; E2E covers both happy and error paths.
- **Open backend questions (not resolved in this epic):** (a) add an IP/attempt
  throttle to the `/api/v1` account password check (GAP1/IMP1); (b) default
  `PWD_CHECK_HIBP` on for new deployments (GAP3/IMP3); (c) correct
  `docs/security.md`'s rate-limiting entry (GAP4/IMP4).

## 15. Acceptance Criteria (Definition of Done)

- [ ] Forgot-password page validates e-mail client-side and renders one identical
      confirmation for every account-existence-related outcome, with distinct
      messages for a per-IP throttle and a malformed submission. *(FR1, FR2, NFR3)*
- [ ] Forgot-password submits through the AuthContext client to `/api/v1`, minting
      no tokens itself. *(FR3, NFR2)*
- [ ] Reset page reads the `t` token from the URL, retains it across resubmission,
      and never shows it as text. *(FR4)*
- [ ] Reset page provides new-password + confirmation with a client match check
      and submits to `/api/v1`. *(FR5)*
- [ ] Reset page renders API password-policy and HIBP rejections verbatim. *(FR6)*
- [ ] Reset page shows distinct, non-leaking errors for invalid / expired / used /
      IP-blocked tokens. *(FR7)*
- [ ] Successful reset shows the password-changed + "other sessions closed"
      confirmation with a login route. *(FR8)*
- [ ] Account route is auth-gated and redirects unauthenticated users preserving
      the return path. *(FR9)*
- [ ] First-connection mode shows only the password form, hiding phone/e-mail
      panels, nav and logout. *(FR10)*
- [ ] All three account actions require the current password. *(FR11)*
- [ ] Account password change rejects new === current, renders API policy/HIBP
      errors, shows the "other sessions closed" message, and keeps the acting
      session logged in. *(FR12, resolves GAP2)*
- [ ] Phone change normalizes FR-national → E.164, accepts E.164, rejects other
      formats, and treats empty as "clear." *(FR13)*
- [ ] E-mail change lowercases/trims, validates format, rejects identical, renders
      duplicate-address rejection, and reflects the new address as unverified.
      *(FR14)*
- [ ] Account actions handle a 429/throttle response with a clear message.
      *(FR15, frontend side of GAP1)*
- [ ] Built on the reference stack (React 18.3 + TS 5.5 + Vite, Router v6, Zustand,
      Tailwind 4, React Hook Form). *(NFR1)*
- [ ] All network I/O via AuthContext → `/api/v1` only; no Supabase, no backend/DB
      change. *(NFR2)*
- [ ] Reset token never persisted/logged; `Referrer-Policy` set on the reset page.
      *(NFR4)*
- [ ] Theme tokens applied; dark + light themes work; desktop-first layout. *(NFR5)*
- [ ] Forms are accessible (labels, keyboard, associated errors). *(NFR6)*
- [ ] Copy is French; server-owned limits sourced from the API, not hardcoded.
      *(NFR7)*
- [ ] Vitest + Testing Library unit tests cover every FR and pass. *(NFR8)*
- [ ] Playwright E2E tests cover the three flows and pass. *(NFR9)*
- [ ] Legacy PHP pages verified running unmodified in parallel. *(NFR10)*
- [ ] UI interactions respond < 300 ms excluding network latency. *(NFR11)*
- [ ] Every GAP (GAP1–GAP4) has a recorded carry-forward/fix decision honored by
      the implementation. *(§12)*
- [ ] All FRs (FR1–FR15) and NFRs (NFR1–NFR11) validated.

## 16. Future Extensions (Optional, Not Required Now)

- Backend story to add an IP/attempt throttle on the `/api/v1` account
  password check (closes GAP1 at the source; IMP1).
- Deployment/backend change to default `PWD_CHECK_HIBP` on (GAP3/IMP3).
- Documentation correction to `docs/security.md`'s rate-limiting entry
  (GAP4/IMP4).
- Mobile-first / fully responsive treatment of these screens, if a later epic
  reprioritizes mobile.
- Passwordless / passkey or MFA options for account recovery, if introduced to
  `/api/v1`.
