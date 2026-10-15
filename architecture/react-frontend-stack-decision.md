# React Frontend Stack Decision — KAN-60

Reference stack record for the `cpaltracker.web` React/TypeScript UI rebuild
(KAN-60 and its child epics). Recorded once, here, so every epic/story
references this doc by id instead of restating or re-deciding the stack.
Supersedes nothing in `cpaltracker.web/docs/architecture.md` — the PHP
monolith is unchanged; this is a new, additive frontend consuming KAN-116's
`/api/v1`.

## 1. Scope of this decision

A new, standalone frontend application (new repo/package, not bound yet —
see [SDLC project bindings](../sdlc/README.md#project-bindings-c-pal)) that:

- Talks only to `/api/v1` (KAN-116) — no new backend or DB work, ever, under
  this stack decision.
- Can point at either a local/staging `/api/v1` or the prod one via runtime
  config (env var / build-time target), without a code change.
- Does not touch, replace, or disable any `cpaltracker.web` PHP page. Both
  run side by side until a separate, future cutover decision is made.

## 2. Stack (as given, with one deviation)

| Layer | Choice |
|---|---|
| Core | React 18.3 + TypeScript 5.5, built with Vite 5 |
| Routing | React Router v6 |
| State | Zustand |
| Auth | **Deviation** — see §3 |
| Styling | Tailwind CSS 4 (`@tailwindcss/vite`), Lucide React (icons) |
| Forms | React Hook Form |
| Content | React Markdown (+ remark-gfm, rehype-sanitize) |
| Onboarding | React Joyride |
| File input | React Dropzone |
| Maps | Leaflet / React-Leaflet |
| Charts | Plotly.js + react-plotly.js |
| Graphs/flows | `@xyflow/react` + `@dagrejs/dagre` + `elkjs` |
| Large lists | `@tanstack/react-virtual` |
| CSV/Excel | PapaParse, `xlsx` |
| Client-side search | MiniSearch |
| Realtime | `socket.io-client` — **held, not adopted** until an epic needs it; `/api/v1` has no realtime endpoint today ([`Live-Tracking.md`](../features/technical/Live-Tracking.md) GAP1–3: today's "live" view has no working map and no polling) |
| Lint/types | ESLint 9 + typescript-eslint, `tsc --noEmit` via `npm run typecheck` (two tsconfigs; root config intentionally checks nothing, per project convention) |
| Unit testing | Vitest 2.1 + Testing Library (jsdom) — the originally pasted reference stack said "Vitest 4"; `package.json` installed 2.1.1, the latest available at scaffolding time (KAN-60), and nothing since has needed a bump (see §9) |
| E2E testing | **Added** — Playwright, against a running local/staging build, per `cto-tools/SDLC.md`'s "scenario first, automate against the running app" E2E strategy |

## 3. Deviation: auth client

The pasted reference stack specifies "Supabase JS client (auth, via
AuthContext)". This system has no Supabase project anywhere.
`cpaltracker.web`'s own HTML pages use PHP session/cookie auth, re-checked
against the DB on every request, but **`/api/v1` (KAN-116) is separate and
bearer-token authenticated, not cookie-based** — `api/v1/_lib.php` sends no
`Access-Control-Allow-Credentials` header specifically because the REST
layer never relies on cookies; a client-issued token comes from
`POST /auth/login.php` and is sent back as `Authorization: Bearer <token>`
([`architecture.md`](../../cpaltracker.web/docs/architecture.md),
[`Auth-and-Session.md`](../features/technical/Auth-and-Session.md),
[`openapi.yaml`](../../cpaltracker.web/api/openapi.yaml)). An earlier draft
of this record assumed the REST layer shared the HTML pages' cookie session
— it does not (caught in KAN-142's review gate).

**Decision:** keep the `AuthContext` pattern (a single React context wrapping
auth state and actions), but back it with a thin REST client calling
`/api/v1`'s auth endpoints over `fetch`, holding the bearer token in memory
and attaching `Authorization: Bearer <token>` to every request —
`credentials: 'omit'`, never `'include'`. No Supabase package is added to
the stack. Token storage is deliberately in-memory only (cleared on reload)
until Auth & Session (KAN-136) decides whether/how to persist a session
across page reloads; that is a security tradeoff for that epic to make
explicitly, not a default to inherit silently from this scaffold.

## 4. Design system foundation (added requirement)

A centralized, theme-based design system is a Core-epic requirement, not an
incidental styling choice:

- Design tokens (color, spacing, typography, radii, shadows) defined once,
  consumed everywhere — no component hardcodes a raw value the token set
  already covers.
- Dark and light themes both ship from day one; switching theme is a single
  state change (e.g. a `data-theme` attribute + CSS variables, or Tailwind 4
  theme variables), not a per-component conditional.
- Responsive layout exists (the token set and layout primitives support
  mobile breakpoints), but **desktop is the primary, first-class target**
  for this rebuild per product decision 2026-10-15; mobile is best-effort,
  not separately acceptance-tested unless a later epic says otherwise.

## 5. What does not change

- No backend code, API contract, or database schema changes under this
  decision or any epic it seeds. `/api/v1` (KAN-116) is consumed as-is; a
  gap found against it (e.g. §5 of
  [`CONSOLIDATED-Coverage-and-Gaps.md`](../features/technical/CONSOLIDATED-Coverage-and-Gaps.md))
  is logged as an open question for a separate backend story, never worked
  around by the frontend calling something else.
- `cpaltracker.web`'s PHP pages keep serving traffic unmodified throughout
  the rebuild. No disruption to the current monolith is an acceptance
  condition on every story in every child epic.

## 6. CSRF / client-side security constraints reconciliation (KAN-151)

`React-Auth-and-Session.md`'s FR9 ("obtain a CSRF token at bootstrap, send it
on every state-changing auth request, refresh it on a CSRF failure") and its
FR4/NFR3 cross-references read `/api/v1` against the wrong model. They were
written against `cpaltracker.web`'s PHP pages (CB3, CB18, CB19–CB21, CB23,
CB24 in
[`Auth-and-Session.md`](../features/technical/Auth-and-Session.md)), where
`cpal_session_start()` seeds a `$_SESSION['csrf_token']`, the browser
auto-attaches the `CPALSESSID` cookie on every request (including ones a
malicious third-party page can trigger), and the CSRF token is the only thing
stopping that ambient cookie from being replayed cross-site. §3 above already
established that `/api/v1` is bearer-token only and never relies on cookies
(`api/v1/_lib.php` sends no `Access-Control-Allow-Credentials` header; this
repo's client always calls `fetch` with `credentials: 'omit'`). Re-reading
`openapi.yaml`'s login/logout schemas for KAN-151 confirms there is no CSRF
field anywhere in the contract — the backend has nothing to issue, and
nothing to validate.

**Decision: FR9's literal "CSRF token" mechanism does not apply and is not
implemented.** CSRF is an attack that relies on the browser *automatically*
attaching an ambient credential (a cookie) to a cross-origin request the
victim didn't knowingly make. This client never has an ambient credential for
a forged request to ride on: the bearer token lives only in an in-memory
module variable (`client.ts`), is attached explicitly via
`Authorization: Bearer <token>` by this app's own code, and no browser
mechanism will attach it to a request this app didn't itself issue.
Implementing a double-submit-cookie/CSRF-token scheme here would validate
against nothing the real backend checks — dead code that would mislead a
future reader into believing a cookie session exists where there is none.
FR9's acceptance criterion ("CSRF token is sent on login & logout") is
therefore **reclassified as N/A for this architecture**, not implemented, and
logged here rather than silently dropped.

What *is* implemented, as the architecturally-correct translation of FR9's
and NFR3's underlying security intent for a bearer-token client (KAN-151):

- **No ambient-credential regression.** `credentials: 'omit'` is sent on
  every single `/api/v1` request — it's set unconditionally in `client.ts`'s
  `request()`, after `...init`, so no caller can override it — and is
  regression-tested in `src/api/client.test.ts`. Also regression-tested:
  no request ever carries a CSRF-shaped header/field the backend doesn't
  validate (an allowlist of the exact three headers this client ever sends).
- **In-memory-only token (NFR3).** The bearer token lives in `client.ts`'s
  module-scoped `authToken` variable only — never written to
  `localStorage`/`sessionStorage`, and cleared via `setAuthToken(null)` on
  logout and on a failed `refresh()` (KAN-151 review round 3: an earlier
  version only cleared it on logout, leaving a stale token live — and sent —
  after an expired-token 401). Not independently tested against
  `document.cookie` or IndexedDB, since nothing in this client ever writes to
  either. Regression-tested in `client.test.ts` and `AuthContext.test.tsx`
  (spies on `Storage.prototype.setItem` across a full login→logout cycle, and
  on every `console.*` method, to catch the token being persisted or logged
  — within the limits of a substring check: an encoded form, e.g.
  `btoa(token)`, would not be caught).
- **Server cookie flags.** NFR3's "SHALL not weaken any server cookie flag
  (CB23)" is vacuously satisfied: this client never sets, reads, or
  participates in any cookie at all, so there is no flag it could weaken.
  `auth.php`'s `cpal_session_start()` still runs on every `/api/v1` request
  and still emits a `Set-Cookie: CPALSESSID`, a known backend-side artifact of
  reusing the legacy session bootstrap — harmless here only because nothing in
  `/api/v1` ever reads that session and `credentials: 'omit'` means this
  client ignores the `Set-Cookie` it receives. It is the one place a cookie
  touches this API at all and is out of scope for this frontend-only
  reconciliation (backend-owned, not a regression to fix here).
- **Leftover risk NFR3's cookie clause would have covered.** NFR3's first
  clause ("rely on the backend-set `httponly` session cookie") is superseded
  by §3's bearer-in-memory decision, not satisfied by it: an `httpOnly` cookie
  is unreadable to injected script, while an in-memory JS variable is not — a
  successful XSS against this app can read the live token (or simply call
  `apiClient` directly; either way the attacker doesn't need the token
  itself). Moving to a cookie would remove that exposure but reintroduce the
  cookie-borne CSRF surface this section argues doesn't currently exist, so it
  is not proposed here. What limits the exposure today: the token lives only
  in `client.ts`'s module variable, is cleared on reload/logout, and every
  `/api/v1` request re-validates it server-side (role, `mustChangePassword`)
  on each call rather than trusting a long-lived client claim. Mitigating XSS
  itself (CSP, output encoding, dependency hygiene) is tracked as its own
  story, KAN-177 (CSP & Security-Header Hardening for the React Frontend,
  filed under KAN-136), not owned implicitly by this record or the epic at
  large.
- **NFR8** (no client-side rate limiting; no copy built around
  `docs/security.md`'s stale "no rate limiting" claim) is unaffected by this
  reconciliation and remains satisfied as-is — no rate-limiting code exists
  anywhere in this app (lockout/throttle copy is FR7/FR8's scope, KAN-155).

This reconciliation was flagged as a judgment call for review: if a future
story finds a literal backend CSRF mechanism was added to `/api/v1`, or that
any `/api/v1` endpoint started accepting session-cookie auth (it is not
present as of KAN-151 per `openapi.yaml`), this section must be revisited
before any FR9-shaped work is marked done against it.

## 7. Lockout signal reconciliation (KAN-155)

`React-Auth-and-Session.md`'s FR8 ("on an account-lockout signal... show a
neutral throttle message... that does NOT reveal the exact remaining
minutes and is presented identically whether or not the account exists")
was written against `Auth-and-Session.md`'s GAP4: the legacy `login.php`
page's still-locked branch returns a *distinct* message disclosing the
remaining lockout minutes (`login.php` lines 80-84), unlike every other
rejection branch, which is an exception to the page's own stated
single-message anti-enumeration rule.

**That gap is already closed at the `/api/v1` layer, before this frontend
ever sees a response.** `cpal_login_verifier()` (`auth.php` lines 534-607,
shared by `login.php` and `POST /api/v1/auth/login`) collapses unknown
account, soft-deleted, uninitialized-password, wrong-password, *and*
still-locked/newly-locked into one `raison: 'identifiants'`, and
`api/v1/auth/login.php` (lines 100-117) responds to all of them with the
exact same `401 invalid_credentials` body, word for word — there is no
status code, error code, or any other field in the real response this
client could key a distinct "locked" branch on (confirmed against
`api/openapi.yaml`'s `/auth/login.php` operation, which documents only
`400`/`401`/`429`/`405`/`503`).

**Decision:** `src/auth/authOutcome.ts`'s `mapAuthOutcome()` does not
implement a separate `accountLockout` outcome. The single `genericRejection`
outcome/message is used for both the ordinary-rejection case and the
locked-account case, because the real backend already makes them
indistinguishable — the same one branch (not two sharing a string) handles
both, which is a stronger guarantee against UI-structure leakage than FR8's
literal wording asks for. This mapping keeps `LOGIN_MAX_FAILS`/
`LOGIN_LOCK_MINUTES` (`config.php`) entirely server-side; the frontend never
hardcodes or guesses either value (NFR8).

**Open question for a future backend story, not worked around here:** if a
product need ever arises for the UI to show a dedicated "your account is
temporarily locked" message (distinct from "wrong email or password"),
`POST /api/v1/auth/login` would need a new, deliberately-still-neutral
distinguishing signal (e.g. a dedicated `account_locked` error code that
itself omits the countdown) — that is a backend contract change and is out
of scope for KAN-155 and this frontend-only epic.

## 8. Logout & session-loss signal reconciliation (KAN-156)

`React-Auth-and-Session.md`'s FR10 ("logout SHALL occur only via an explicit
authenticated in-app action that POSTs the CSRF token... never triggerable
by a bare GET or third-party markup load") inherits the same stale "CSRF
token" phrasing §6 above already reconciled for FR9 — `POST
/api/v1/auth/logout.php` requires no CSRF token to send or validate (there
is none in the contract, same as §6). The rest of FR10's intent is already
true of this architecture **by construction, not by added code**:
`api/v1/auth/logout.php` only ever handles `POST` (`cpal_api_auth_logout_handle()`
replies `405 method_not_allowed` to anything else — `logout.php` lines
50-53), there is no server-side route that performs it on `GET`, and this
frontend's only call site is `AuthContext.logout()`'s
`apiClient.post('/auth/logout.php')` — a POST issued solely from an explicit
in-app button click (`HelloWorld`'s "Sign out"), never a link, `<img>`, or
other GET-triggerable markup. CB19/CB20's bespoke "logged-in + bad CSRF token
renders a confirmation page instead of logging out" branch (the legacy
`logout.php` page's own anti-`<img src="logout.php">` defense) has no bearer-
token equivalent to replicate: a cross-site request simply cannot attach this
app's `Authorization: Bearer` header at all (§6's "no ambient credential"
argument applies identically to logout), so there is nothing an attacker's
page could forge in the first place — not a narrower defense than CB19/CB20,
a structurally stronger one (FR10's own wording: "removing GET-triggered
logout entirely").

**FR11/FR12 — session-loss signal collapsing.** FR11 asks `AuthContext` to
distinguish idle-timeout, absolute-timeout, account-disabled and
epoch-mismatch responses (CB27, CB28, CB30, CB31 — all legacy
`$_SESSION`-based checks inside `cpal_current_user()`, `auth.php` lines
180-212) and FR12 asks it to treat "any 401/session-invalidation response"
as session loss. Re-reading the real `/api/v1` bearer-token path
(`cpal_api_require_auth()`, `auth.php` lines 670-715, and its error mapping
`cpal_api_require_auth_error()`, lines 726-735) shows every one of those four
legacy conditions — along with a simply missing/malformed/expired/already-
invalidated token — collapses into the exact same response before this
frontend ever sees it: `401 unauthorized`, `"Missing, invalid or expired
token."`, word for word, for every case
(`api/v1/account.php` line 139 is one caller of that same shared mapping).
Confirmed against `api/openapi.yaml`, which documents only one `Unauthorized`
response component reused by every authenticated operation — there is no
status code, error code, or body field left for this client to key a
four-way (or even two-way) branch on. This is the same shape of finding as
§7's GAP4 lockout-signal collapse (KAN-155): the gap is already closed at the
`/api/v1` layer, not something the frontend needs to work around or invent
distinguishing UI branches for.

Review gate round 1 (KAN-156): "collapses" is accurate for the *response
the client sees*, but three of the four legacy conditions do have a real,
server-side bearer-token analogue causing that collapse, while one does
not — worth stating precisely rather than implying all four are equally
"handled":
- **Epoch-mismatch** → a real analogue: API tokens are revoked on a
  password change/reset (`account_logic.php:123`,
  `password_recovery_logic.php:245`).
- **Account-disabled** → a real analogue: the token-validity check includes
  the account's `SuppressionDate`.
- **Absolute-timeout** → a real analogue: a token's `ExpiresAt`.
- **Idle-timeout** → **no analogue at all.** `cpal_token_valide` has no
  sliding/last-seen check and nothing refreshes a token's expiry on
  activity (no counterpart to CB28's idle-refresh) — a bearer token that's
  merely gone unused, but isn't yet past `ExpiresAt`, stays valid
  indefinitely. This is a genuine security-posture gap against CB27's
  legacy 1800s idle window, not something this story's client-side
  reconciliation closes — logged here as a backend open question (see
  below), not silently treated as covered.

**Decision:** a single shared mechanism (`client.ts`'s `request()`, the one
place every `/api/v1` call passes through) detects this one real signal and
notifies `AuthContext` via a registered callback
(`setSessionLossHandler()` — the same module-variable registration pattern
`setAuthToken()` already uses, since a plain `fetch` wrapper can't reach into
React context directly). It fires only when the failing request **was
carrying a bearer token** (i.e., this client believed it was authenticated)
and got back `401` with code `unauthorized` specifically — not merely any
401. That second condition is what keeps a plain not-yet-signed-in visitor's
very first bootstrap `GET /account.php` (also a `401 unauthorized`, since
`cpal_api_require_auth()` rejects a missing `Authorization` header the same
way) from being mislabeled "your session expired": nothing was ever
authenticated to lose in that case. `POST /auth/login.php` can never
trigger it either way — that endpoint requires no bearer token
(`security: []`) and always replies `invalid_credentials` on rejection,
never `unauthorized`; a rejected login is FR7/FR8's job
(`src/auth/authOutcome.ts`, KAN-155), not a session-loss event. Once the
interceptor fires, `AuthContext` drops to `anonymous` and reuses FR13's
already-existing `RequireAuth` anonymous-redirect (`buildLoginRedirectPath`)
unmodified for FR12's "routes to login with suite set to the current
location" — no second redirect/open-redirect-protected code path was added.
FR11's "session-expired copy" requirement reuses
`src/auth/authOutcome.ts`'s existing `sessionExpired` message (KAN-155) via a
new `?notice=` query indicator (`src/auth/loginNotice.ts`), the same
query-param-driven style `suite` already uses, rather than inventing new
copy or a new redirect-targeting mechanism.

**Open question for a future backend story, not worked around here:**
two distinct gaps, not one:
1. If a product need ever arises for the UI to distinguish *why* a session
   ended (e.g. "your account was disabled" vs "you were idle too long"),
   `/api/v1` would need to start returning a distinguishing code per case —
   a backend contract change, out of scope for KAN-156 and this
   frontend-only epic.
2. Separately, and regardless of (1): bearer tokens have no idle-timeout
   enforcement at all today (see the "Idle-timeout" point above) — a
   security-posture gap against the legacy session model's behavior, not
   merely a UI-copy limitation. Flagged here for a future backend story to
   decide whether/how to close it; this frontend cannot enforce an
   idle-timeout the server itself doesn't track.

## 9. Test coverage & CI gate (KAN-157)

By the time this story was picked up, every prior Auth & Session story
(KAN-145, KAN-149, KAN-151, KAN-155, KAN-156) had already added its own
Vitest/Testing Library and Playwright coverage as part of its own Done
condition. KAN-157's work was therefore an audit against this epic's ACs
plus the one genuine gap: neither suite was wired into CI yet.

Audit result — every required area already has passing coverage, no new
feature tests were needed:
- Unit (Vitest 2.1 + Testing Library — the §2 table's "Vitest 4" was never
  actually installed; `package.json` pins 2.1.1): empty-field validation
  (`Login.test.tsx`), suite sanitizer (`suite.test.ts`), outcome→copy
  mapper (`authOutcome.test.ts`), lockout copy (`authOutcome.test.ts` —
  the neutral, no-digit copy for the collapsed 401/429 buckets), route
  guards (`RequireAuth.test.tsx`), logout
  handler (`AuthContext.test.tsx` + `HelloWorld.test.tsx`).
- E2E (Playwright): login success via the actual form submission
  (`logout-session.spec.ts` + `csrf-security.spec.ts` — `hello-world.spec.ts`
  only fakes an already-authenticated session by mocking `/account.php`, it
  never submits the login form), wrong password / lockout — per §7, a
  locked account gets the *same* `401 invalid_credentials` response as a
  wrong password (anti-enumeration: the UI must not reveal a locked account
  exists), both mapped to the identical genericRejection copy and both
  covered by the same test (`login.spec.ts`'s invalid-credentials case; the
  429 `rate_limited` case in the same file is the separate per-IP throttle,
  not lockout), explicit logout (`logout-session.spec.ts`), protected-route
  redirect (`protected-routes.spec.ts`), session expiry/invalidation
  (`logout-session.spec.ts`).

The actual deliverable: `.github/workflows/ci.yml`, running on every push
and pull request to `main` (this repo currently commits straight to `main`
per its delivery workflow, so a PR-only trigger would never fire) —
typecheck + lint + the Vitest suite in one job, the Playwright suite
(against a built-and-previewed app, per `playwright.config.ts`'s own
`webServer`) in a second — both required to pass, making this epic's own
stated Done condition ("both suites are wired into CI and treated as a Done
condition gate") real rather than only a documented intent. There is no
branch protection rule enforcing this yet (commits land on `main` directly,
`pr:false`), so today the gate is procedural — a story is not closed out
until its CI run for that commit is checked green — rather than one GitHub
itself blocks on.

### 9.1 NFR2's "running local/staging build" means a built app with a mocked `/api/v1`, not a live PHP/DB backend

`React-Auth-and-Session.md`'s NFR2 asks for E2E "against a running
local/staging build"; §13 says GAP4 is "verified by E2E". Every Playwright
spec added across this whole epic (KAN-145 through KAN-156) already
satisfies this the same way: `playwright.config.ts`'s `webServer` builds
the real production bundle and serves it with `vite preview`, and each spec
mocks `/api/v1` responses with `page.route` rather than pointing at a live
`cpaltracker.web` instance. KAN-157 keeps that pattern rather than changing
it — no PHP/MySQL backend is available in this CI environment (and this
epic's constraint is no backend/DB change or disruption to the running
monolith, which a CI job hitting it would risk), and every acceptance
criterion this epic has needed so far has been verifiable against the
built frontend's own request/response handling without one. GAP4 is a
partial exception, split across two different kinds of evidence rather
than fully closed by mocked E2E: its UI half (a locked account gets the
same neutral, no-digit copy as a wrong password, never a distinguishing
message) is what E2E actually verifies; its server half (that a real
locked account is collapsed to the same `401 invalid_credentials` the
wrong-password case gets, rather than some other signal) rests on §7's
code-level reconciliation against `auth.php`/`openapi.yaml`, not on any
test run here — and would be a reasonable scope item for an optional
live-backend smoke story if that server-side behavior is ever meant to be
test-verified rather than reconciled by reading.

Three spec file headers (`e2e/login.spec.ts`, `e2e/protected-routes.spec.ts`,
`e2e/logout-session.spec.ts`) still had comments saying a "live `/api/v1`
backend" run "remains KAN-157's job" — stale from when this story was
scoped before the audit above found no new flows were actually needed;
removed as part of this story, since KAN-157 did not add one.

If a *live*-backend E2E run (hitting a real local `cpaltracker.web` +
MySQL) is wanted in addition to this — e.g. as a periodic smoke test rather
than a per-commit gate — that is a new, explicitly-scoped story, not
something this one silently left undone.

## 10. Forgot-password 429/400 handling (KAN-146)

`POST /auth/forgot-password.php` (`api/openapi.yaml`) issues a distinct
`429` when the caller's IP is already throttled (`cpal_ip_bloquee`, CB3),
matching `POST /auth/login.php`'s own `429 rate_limited` (§7's
`authOutcome.ts`, `ipThrottle`). The legacy page confirms this directly:
`forgot_password.php` checks `cpal_ip_bloquee($pdo)` in an `elseif` branch
that sets its own distinct "Trop de tentatives..." message, and only its
final `else` branch — reached solely when that check has already passed —
sets `$envoye = true`. `api/v1/auth/forgot-password.php`'s own header
comment says the same thing explicitly: the two cases that get a distinct
response are a malformed body (`400`) and an IP already blocked by
`cpal_ip_bloquee()` (`429`, CB3) — neither reveals anything about a
specific account, so showing either distinctly does not violate
anti-enumeration (an IP block and a malformed request are facts about the
request, not about whether any account exists for the submitted e-mail).

Review gate round 1 on KAN-146 (B1) caught an earlier version of
`forgotPasswordOutcome.ts` that collapsed `429` into the "check your
mailbox" confirmation anyway, on the mistaken premise that the legacy page
hides IP-throttling from the visitor unconditionally (it doesn't — see
above). That collapse was not an anti-enumeration improvement; it was a UX
regression — a legitimately-throttled visitor would be told an e-mail was
on its way when none ever would be, with no indication to retry later,
where both the legacy page and `/api/v1` already tell them exactly that.
The corrected mapper has four buckets: a genuine success, and any other
account-existence-related outcome — unknown/soft-deleted/inactive account,
an account over its own per-account hourly cap (`cpal_demander_reinitialisation`'s
CB4), or a mail-send failure, all of which `cpal_demander_reinitialisation()`
(`password_recovery_logic.php`) collapses identically via a silent `return`
with nothing signalled back either way (CB1, proper) — map to `'confirmed'`;
`429` maps to its own `ipThrottle`, reusing login's exact copy; `400`
(malformed body — defensive only, FR1 already blocks this client-side)
maps to its own `invalidRequest`; and a genuine `5xx`/transport failure
maps to `serviceUnavailable`.

## 11. Account self-service token revocation (KAN-152, GAP5)

`PATCH /api/v1/account.php`'s password action (`account_logic.php`'s
`cpal_compte_changer_motdepasse`) calls the same `cpal_token_revoquer($pdo,
$userId, 'api')` the reset-password flow uses (NFR8) — but with no
exemption for the bearer token that authenticated the very PATCH request
that just succeeded. The legacy `account.php` page does not have this
problem: its own "other sessions closed" effect is the `SessionEpoch`
trigger plus `session_regenerate_id()`, which only invalidates *other*
browser sessions, never the one making the change. The real `/api/v1`
behavior is stricter than that: every live `api`-type token for the
account is revoked, including the one this request is using right now —
the next call with that same token gets a `401 unauthorized`, which
`client.ts`'s session-loss interceptor (§8) would otherwise treat as an
ordinary dropped session and log the user out.

FR12(b) requires the acting session to stay logged in through a
self-service password change, so `AuthContext.updateAccount()` resolves
this at the frontend boundary: immediately after a successful password
PATCH, it silently re-authenticates (`POST /auth/login.php`) with the
e-mail on the account and the password the user just set, installing the
freshly issued token before the caller's own stale one can ever be used
again — the user never sees an interruption. This deliberately does not
reuse `login()`'s own state update (which only has `login.php`'s smaller
`AccountSummary` to work with) — the richer `Account` object the PATCH
response itself returned is kept instead, and only the token is replaced.
If the re-authentication call itself fails (a narrow, rare window — the
password change already succeeded by this point), `updateAccount()`
swallows that failure rather than reporting the whole action as failed;
the next real API call then simply hits the already-revoked token's `401`
and falls through to the ordinary session-loss flow, which is honest
(the session really did just become unauthenticated) rather than
pretending otherwise.

The real fix — the server either exempting its own request's token from
the revocation, or returning a freshly issued token in the PATCH response
body the way `login.php` does — is logged as an open backend question
(`React-Password-Recovery-and-Account-Management.md` §14), not implemented
here (no backend change, per this epic's constraint).

## 12. Related Documentation

- Feeds: KAN-60 and its child epics (Core, Auth & Session, Password Recovery
  & Account Management, Live Tracking, History Dashboard, Device Pilotage,
  Client Branding & Assets).
- Depends on: KAN-116 (`/api/v1`, Done), KAN-126 (Technical Feature
  Discovery, Done — source of the CB-n/GAP-n citations above).
- [`cpal.docs/sdlc/README.md`](../sdlc/README.md) — SSoT and workflow this
  decision is recorded under.
