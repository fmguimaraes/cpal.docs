# Feature: React Auth & Session (cpaltracker.web UI rebuild)

> Forward-looking product spec for the **Auth & Session** feature-area child
> epic of KAN-60 (React 18.3 + TypeScript 5.5 / Vite rebuild of
> `cpaltracker.web`'s UI as a new, standalone frontend consuming the existing
> `/api/v1` REST layer, KAN-116). This document describes what the **new React
> UI MUST do**. It is not a reverse-engineering of the PHP monolith — that is
> [`features/technical/Auth-and-Session.md`](./technical/Auth-and-Session.md)
> (CB-n = current behavior to replicate, GAP-n = known issues). Every FR below
> cites the CB-n it replicates and/or the GAP-n it resolves or carries forward.
>
> Stack, auth-client deviation (AuthContext over a thin REST/cookie client
> against `/api/v1`, **not** Supabase) and design-system requirements are fixed
> by [`architecture/react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md)
> and referenced, not re-decided, here.

## 1. Feature Name

React Auth & Session — login, session handling, CSRF, lockout UX, and logout
for the new `cpaltracker.web` React frontend.

## 2. Problem / Pain Point

The current authentication surface lives inside the PHP monolith
(`login.php`, `logout.php`, `auth.php`): server-rendered pages with no
component model, no design system, no automated UI tests, and user-facing
error copy and lockout behavior that are hard to evolve. KAN-60 rebuilds the
UI in React against `/api/v1`. Auth & Session is the gating surface — nothing
else in the new app is reachable until a user can log in, hold a session, be
guarded on protected routes, and log out — so it must be rebuilt first, and it
must replicate the monolith's security-critical behavior exactly while not
blindly copying its known defects (the reverse-engineering review found real
holes in lockout reset, enumeration leakage, and timing, per
`Auth-and-Session.md` GAP1–GAP5 and
[`CONSOLIDATED-Coverage-and-Gaps.md`](./technical/CONSOLIDATED-Coverage-and-Gaps.md) §5).
Today there is no React auth at all; without it the rebuild cannot start.

## 3. Goal / Outcome (Success Definition)

A user can log in to the new React app with their existing credentials, is
kept logged in across navigation with the same session rules as the monolith
(idle + absolute timeout, epoch invalidation, per-request revalidation), is
guided correctly when a password change is mandatory, is protected from
cross-site logout, and is given honest, non-enumerating feedback when their
account is locked. The experience is themed (dark/light), desktop-first,
keyboard-accessible, and fully covered by Vitest/RTL unit tests and Playwright
E2E before it ships. The existing PHP login keeps running, untouched, in
parallel.

## 4. User Stories (Primary Scenarios)

- **US1 — Log in.** As a tracker user, I want to log in with my email and
  password on a clear, themed form, so that I can reach the application.
  *(covers FR1, FR3, FR4, FR5, FR6, FR7; NFR3, NFR4, NFR5, NFR6, NFR9)*
- **US2 — Already authenticated.** As a user who already has a valid session,
  I want the app to skip the login form and take me straight in, so that I am
  not asked to re-authenticate needlessly. *(covers FR2, FR11, FR12)*
- **US3 — Locked out gracefully.** As a user who mistypes my password several
  times, I want clear feedback that I must wait, without the app confirming
  whether the address has an account or revealing an exact countdown, so that
  I understand what happened without the system leaking account state.
  *(covers FR7, FR8; NFR8, NFR9)*
- **US4 — Log out safely.** As a user, I want to log out explicitly and be
  protected against a third-party page forcing my logout, so that my session
  ends only when I intend it to. *(covers FR9, FR10)*
- **US5 — Session ends, I come back.** As a user whose session expires, is
  invalidated, or who hits a protected page while logged out, I want to be
  returned to login and then back to where I was headed, so that I lose no
  context. *(covers FR3, FR11, FR12, FR13)*
- **US6 — Admin & forced password change.** As an admin (or any user flagged
  `MustChangePassword`), I want my role recognized for in-app gating and to be
  forced to change a temporary password before using any other screen, so that
  access control and first-login hygiene match the current system.
  *(covers FR14, FR15)*
- **US7 — Quality & no disruption.** As the engineering/QA team, I want the
  auth UI covered by unit and E2E tests and shipped without touching the PHP
  monolith, so that we can release safely alongside the running app.
  *(covers NFR1, NFR2, NFR7)*

## 5. In Scope (What WILL be delivered)

- A React login screen (email, password, submit) built on the shared design
  system (tokens, dark/light), using React Hook Form, desktop-first.
- An `AuthContext` + thin REST client against `/api/v1` auth endpoints
  (`fetch`, credentials included), per the stack decision §3 — no Supabase.
  Superseded: `/api/v1` is bearer-token only and never relies on cookies —
  `credentials: 'omit'` always, token attached via `Authorization: Bearer`.
  See stack decision §3 (KAN-142) and §6 (KAN-151).
- Client-side required-field validation and submit/loading/disabled states.
- Sanitization of the `suite` return-target so only same-origin relative paths
  survive (open-redirect prevention).
- Full mapping of server auth outcomes to user-facing copy: generic
  credential-rejection message, session-expired/CSRF message, service-
  unavailable message, per-IP throttle message, and a non-enumerating lockout
  message.
- CSRF token handling on all state-changing auth requests (login, logout).
- Explicit, action-only logout (never triggered by a bare GET/markup load),
  clearing `AuthContext` and returning to login.
- Session lifecycle handling: idle/absolute timeout and epoch-invalidation
  responses transition the app to logged-out with correct messaging; a 401 /
  invalidation from any `/api/v1` call is treated as session loss.
- Route guards: protected-route redirect to login with `suite`, and a
  `MustChangePassword` guard blocking every route except the password-change
  route.
- Role exposure from `AuthContext` for RBAC gating (admin flag).
- Vitest + Testing Library unit tests and Playwright E2E tests covering all
  FRs, required before Done.

## 6. Out of Scope (What WILL NOT be delivered)

- **Any backend, API-contract, or database change.** `/api/v1` (KAN-116) is
  consumed as-is. Server-owned behaviors are replicated at the UI layer only;
  they are not re-implemented in React: timing-equalization dummy hash (CB8),
  per-IP throttle counting (CB5–CB7), per-account `FailedLoginCount`/
  `LockedUntil` writes (CB13–CB15), session cookie flags and strict/only-
  cookies (CB22–CB23), and session-id regeneration at auth (CB36) remain
  server responsibilities.
- **Any change to or disruption of the PHP monolith.** `login.php`,
  `logout.php`, `auth.php` keep running unmodified in parallel.
- **Fixing GAP1, GAP2, GAP3, GAP5 in code** — these are backend-/docs-owned
  (see §12 and §9 Open Questions). This epic resolves only GAP4 (a UI-copy
  decision) and ensures the UI does not depend on the stale GAP5 claim.
- **Mobile-first design.** Desktop is the primary, acceptance-tested target
  per the 2026-10-15 product decision; mobile is best-effort only.
- Password recovery, self-service account management, HIBP/Argon2id policy,
  registration, SSO/MFA — other feature areas / future.

## 7. User Flow / UX Breakdown

1. User navigates to the app. `AuthContext` bootstraps against `/api/v1`.
   - If a valid session exists → redirect to the intended route / home (no
     form shown).
   - Else → render the login form (with CSRF token obtained from bootstrap).
2. User enters email + password and submits.
   - Empty field → inline field error, no request sent.
   - Otherwise the client POSTs credentials + CSRF token with cookies.
3. System responds; UI maps the outcome:
   - Success + `MustChangePassword` → redirect to the password-change route.
   - Success → redirect to the validated `suite` target, else home.
   - Wrong/unknown/soft-deleted/uninitialized credentials → one generic
     "Adresse ou mot de passe incorrect." message.
   - Account locked / too many account attempts → neutral throttle message
     (no exact countdown, no account-existence confirmation).
   - Per-IP throttle → the IP-throttle message.
   - CSRF/expired → "Votre session a expiré…" and a refreshed token.
   - 5xx / DB outage → "Le service est momentanément indisponible…".
4. While authenticated, every `/api/v1` response is watched: a 401 /
   invalidation (idle timeout, absolute timeout, epoch change, account
   disabled) drops the user to logged-out and routes to login with `suite`
   preserved.
5. User clicks Logout (an explicit in-app action) → client POSTs logout with
   CSRF token → `AuthContext` cleared → redirect to login with a "logged out"
   indicator.

## 8. Functional Requirements (FRs)

- **FR1.** The app SHALL render a login form (email, password, submit) built
  on the shared design system with dark/light theming. *(replicates CB18)*
- **FR2.** On load, if `AuthContext` bootstrap reports a valid session, the app
  SHALL skip the login form and redirect to the intended route / home.
  *(replicates CB1)*
- **FR3.** The app SHALL accept a `suite` return-target only if it is a
  same-origin relative path beginning with `/` and containing no `://`,
  dropping it to empty otherwise. *(replicates CB2)*
- **FR4.** The app SHALL submit credentials and the CSRF token to the `/api/v1`
  auth endpoint via the thin REST/cookie client (`fetch`, credentials
  included), per stack decision §3. *(replicates CB18, CB24; integrates
  `/api/v1`)* — superseded by stack decision §3/§6: `/api/v1` is bearer-token
  only, `credentials: 'omit'` always, no CSRF token in the contract (KAN-151).
- **FR5.** The app SHALL block submission and show an inline message when email
  or password is empty, without issuing a request. *(replicates CB4)*
- **FR6.** On authentication success the app SHALL redirect to the
  password-change route when the API reports `MustChangePassword`; otherwise to
  the validated `suite` target, else home. *(replicates CB16)*
- **FR7.** The app SHALL map server auth outcomes to user-facing copy: a single
  generic message for unknown / soft-deleted / uninitialized-password / wrong-
  password rejections; the session-expired message on CSRF failure; the
  service-unavailable message on 5xx/DB outage; and the per-IP throttle
  message. *(replicates CB9–CB12, CB14, CB3, CB17, CB5)*
- **FR8.** On an account-lockout signal (newly locked or still-locked), the app
  SHALL show a neutral throttle message ("too many attempts, try again later")
  that does NOT reveal the exact remaining minutes and is presented identically
  whether or not the account exists. *(replicates CB13/CB11 lockout concept;
  **resolves GAP4** at the UI layer)* — superseded by stack decision §7
  (KAN-155): `/api/v1` already collapses a locked account into the same 401
  `invalid_credentials` response as any other rejection (GAP4 closed
  server-side, not just at the UI), so there is no distinct lockout signal
  for the client to key a separate branch on; the generic-rejection copy
  (FR7) serves both buckets via one code path. The ipThrottle copy also
  drops CB5's minutes figure, since the real 429 response carries none
  (NFR8) — see §7 for both adaptations.
- **FR9.** The app SHALL obtain the CSRF token at bootstrap, send it on every
  state-changing auth request (login, logout), and on a CSRF failure show the
  session-expired message and refresh the token before retry.
  *(replicates CB3, CB18, CB24)*
- **FR10.** Logout SHALL occur only via an explicit authenticated in-app action
  that POSTs the CSRF token; it SHALL never be triggerable by a bare GET or
  third-party markup load. On logout the app clears `AuthContext` and redirects
  to login with a "logged out" indicator. *(replicates CB21, CB35; replicates
  and hardens the CB19/CB20 anti-CSRF-logout intent by removing GET-triggered
  logout entirely)*
- **FR11.** `AuthContext` SHALL treat idle-timeout, absolute-timeout, account-
  disabled, and epoch-mismatch responses from `/api/v1` as session loss,
  transitioning to logged-out with the session-expired message.
  *(replicates CB27, CB28, CB30, CB31)*
- **FR12.** The app SHALL treat a 401 / session-invalidation response from any
  `/api/v1` call as session loss and route to login with `suite` set to the
  current location. *(replicates CB29, CB30, CB31, CB32)*
- **FR13.** A protected-route guard SHALL redirect unauthenticated access to
  login with `suite` set to the attempted URL. *(replicates CB32)*
- **FR14.** A `MustChangePassword` guard SHALL block every app route except the
  password-change route until the flag is cleared. *(replicates CB33)*
- **FR15.** `AuthContext` SHALL expose the user's role (including an admin
  flag) so that in-app RBAC gating can consume it. *(replicates CB34, CB29
  role re-read)*

## 9. Non-Functional Requirements (NFRs)

- **NFR1.** All FR UI logic (validation, `suite` sanitization, outcome→copy
  mapping, lockout copy, guards, logout action) SHALL be covered by Vitest 4 +
  Testing Library unit tests; passing unit tests are a Done condition.
  *(US1, US7; stack decision §2)*
- **NFR2.** Playwright E2E SHALL cover login success, wrong password, lockout,
  explicit logout, protected-route redirect, and session expiry/invalidation
  against a running local/staging build; passing E2E is a Done condition.
  *(US7; stack decision §2)*
- **NFR3.** The UI SHALL rely on the backend-set httponly session cookie and
  never store credentials or session secrets in `localStorage`/`sessionStorage`
  or app logs; it SHALL not weaken any server cookie flag (CB23). *(US1)* —
  the httponly-cookie clause is superseded by stack decision §3 (bearer token,
  in-memory only); the no-persistent-storage/no-logging clauses still apply
  and are regression-tested (KAN-151). See stack decision §6 for the full
  reconciliation, including the leftover XSS-exposure tradeoff this
  supersession carries.
- **NFR4.** The login form SHALL be interactive within the app's standard
  first-paint budget and the auth bootstrap SHALL resolve without blocking the
  shell or causing layout shift. *(US1)*
- **NFR5.** The login form SHALL be keyboard-navigable with labeled fields and
  errors announced via `aria-live`; desktop-first. *(US1)*
- **NFR6.** All auth-surface styling SHALL come from design tokens (dark/light)
  with no hardcoded raw values a token covers (stack decision §4). *(US1)*
- **NFR7.** The frontend SHALL run alongside the unmodified PHP monolith and
  SHALL target its `/api/v1` base via runtime config (no code change to switch
  local/staging/prod). *(US7; stack decision §1, §5)*
- **NFR8.** The UI SHALL NOT implement or depend on client-side rate limiting
  and SHALL NOT design error copy around `docs/security.md`'s stale "no rate
  limiting" claim. *(US3; **carries-forward awareness of GAP5**)*
- **NFR9.** User-facing auth messages SHALL preserve the existing French copy
  where replicating a CB-n behavior, unless superseded by FR8's neutral lockout
  copy. *(US1, US3)*

## 10. Data Inputs & Outputs

- **Inputs:** login form fields `mail`/email, `motdepasse`/password,
  `csrf_token`, `suite` (GET param / hidden); logout `csrf_token`. CSRF token
  and current-session state from the `/api/v1` bootstrap.
- **Outputs:** authenticated requests to `/api/v1`; `AuthContext` state
  (authenticated flag, user identity, role/admin flag, `MustChangePassword`
  flag); navigation side effects (redirects to home / `suite` / password-change
  / login); rendered error/status messages.
- No credential or session value is persisted client-side beyond the
  backend-managed httponly cookie (NFR3).

## 11. Integration Points

- **`/api/v1` (KAN-116, Done)** — auth endpoints for login, logout, current-
  user/session bootstrap and CSRF token issuance; read/write over
  `fetch` with cookies. The exact endpoint set is the one exposed by the
  KAN-118 `/api/v1/auth/*` extraction; any missing capability is logged as a
  backend open question (§9 of the stack decision), never worked around.
- **Browser cookie store** — backend-set `CPALSESSID` httponly session cookie
  (read/written by the browser, never by app JS).
- **React Router v6** — redirects and route guards.
- **Zustand / AuthContext** — session + role state shared across the app.
- Consumes the shared design system / theme tokens (Core epic).

## 11a. Software Items (IEC 62304 §5.3)

**N/A.** Per [`sdlc/README.md`](../sdlc/README.md) "Applicability to c-PAL
today", IEC 62304 and the Software Item Registry are not in force for c-PAL at
this time. No software-item UIDs are declared.

## 12. Edge Cases & Constraints

- Open-redirect attempt via `suite` → neutralized (FR3 / CB2).
- Already-authenticated visit to `/login` → redirected away (FR2 / CB1).
- CSRF/expired-token POST → generic session-expired message, token refreshed
  (FR9 / CB3).
- DB/5xx outage during login → "service momentanément indisponible", no stack
  trace (FR7 / CB17).
- Cross-site forced logout (`<img src=logout>`-style) → impossible: logout is
  an authenticated POST action only (FR10 / CB19–CB21).
- Session expiry / epoch change / account disabled mid-use → graceful drop to
  login with `suite` preserved (FR11, FR12 / CB27–CB32).
- **GAP1 (per-IP throttle bypassed by malformed/CSRF-less POST)** —
  **carry forward as-is**: server-owned, no backend change in this epic. The
  React client always sends a valid CSRF token and never issues malformed
  POSTs, so it neither triggers nor papers over the gap; logged as a backend
  open question.
- **GAP2 (`FailedLoginCount` not reset after lockout expires)** —
  **carry forward as-is**: server-owned counter logic, out of scope for a
  frontend-only epic; logged as a backend open question. FR8's neutral copy
  avoids promising a "fresh 5 attempts" the backend may not grant.
- **GAP3 (timing side channel across rejection branches)** —
  **carry forward as-is**: server-side response timing, not addressable from
  the UI; logged as a backend open question.
- **GAP4 (locked-account response discloses account existence/state)** —
  **fix in the new UI**: FR8 renders a single neutral throttle message with no
  exact countdown and identical presentation regardless of account existence,
  restoring the monolith's own stated single-message anti-enumeration goal at
  the UI layer.
- **GAP5 (`docs/security.md` stale "no rate limiting" claim)** —
  **carry forward / flag**: NFR8 forbids designing UI copy around the stale
  claim; the doc correction itself is a backend/docs task, out of scope here.

## 13. Metrics / KPIs

- Login success rate for valid credentials ≈ parity with the monolith.
- 100% of FRs covered by at least one unit test and the critical paths by E2E.
- Zero regressions reported against the PHP monolith during parallel run.
- Zero distinct lockout-vs-generic message divergence observable in the React
  UI (GAP4 closed) — verified by E2E.

## 14. Risks & Mitigations

- **`/api/v1` auth surface may not expose every signal the UI needs** (e.g.
  distinguishing lockout from generic rejection, or epoch invalidation) →
  mitigate by confirming the KAN-118 contract early; log any gap as a backend
  open question rather than working around it (stack decision §5).
- **Over-hardening GAP4 hides genuine lockout from users** → mitigate with a
  neutral but informative "too many attempts, try again later" that guides the
  user without a countdown or enumeration signal.
- **Cookie/CSRF handling differences between fetch client and PHP session** →
  mitigate with E2E against a real build and explicit credential-include
  configuration.
- **Scope creep into backend fixes (GAP1–3/5)** → mitigate by the explicit
  Out-of-Scope statement and open-question logging.

## 15. Acceptance Criteria (Definition of Done)

- [ ] Themed login form renders (dark/light), keyboard-accessible (FR1, NFR5,
      NFR6).
- [ ] Valid existing session skips the form and redirects (FR2).
- [ ] `suite` is accepted only as a same-origin relative path; open-redirect
      attempts are dropped (FR3).
- [ ] Credentials + CSRF token are submitted to `/api/v1` via the cookie client
      (FR4, NFR3).
- [ ] Empty email/password blocks submit with an inline message (FR5).
- [ ] Success honors `MustChangePassword`, else `suite`, else home (FR6).
- [ ] Generic / session-expired / service-unavailable / per-IP throttle copy is
      mapped correctly (FR7).
- [ ] Lockout shows the neutral, no-countdown, non-enumerating message — GAP4
      closed (FR8).
- [ ] CSRF token is sent on login & logout; CSRF failure refreshes the token
      (FR9).
- [ ] Logout works only via explicit authenticated POST action; no GET/markup
      can trigger it (FR10).
- [ ] Idle/absolute timeout, account-disabled, and epoch mismatch drop to login
      with session-expired copy (FR11).
- [ ] Any 401/invalidation routes to login with `suite` preserved (FR12).
- [ ] Protected-route guard redirects unauthenticated access with `suite`
      (FR13).
- [ ] `MustChangePassword` guard blocks all routes but the password-change route
      (FR14).
- [ ] `AuthContext` exposes role/admin flag for RBAC gating (FR15).
- [ ] Vitest/RTL unit tests pass for all FRs (NFR1).
- [ ] Playwright E2E passes for login/lockout/logout/guard/expiry (NFR2).
- [ ] UI stores no credentials/session secrets client-side and does not weaken
      cookie flags (NFR3).
- [ ] Login interactive within budget; no blocking/layout shift at bootstrap
      (NFR4).
- [ ] API base is runtime-configurable; PHP monolith runs unmodified in
      parallel (NFR7).
- [ ] No client-side rate limiting and no copy built around the stale GAP5
      claim (NFR8).
- [ ] French user-facing copy preserved except where FR8 supersedes it (NFR9).
- [ ] GAP1, GAP2, GAP3, GAP5 explicitly logged as backend/docs open questions,
      not implemented in the frontend.
- [ ] All FRs and NFRs validated.

## 16. Future Extensions (Optional, Not Required Now)

- MFA / SSO.
- Backend fixes for GAP1–GAP3 (throttle on malformed POST, lockout reset after
  expiry, uniform timing) once a backend story is opened.
- `docs/security.md` correction (GAP5).
- First-class mobile layout with its own acceptance tests.
- "Remember this device" / persistent sessions (currently session-cookie only,
  CB23).
