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
| Unit testing | Vitest 4 + Testing Library (jsdom) |
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
  every single `/api/v1` request (`client.ts`'s `request()` — not
  overridable per-call) and is locked down by a Vitest regression test
  (`src/api/client.test.ts`) asserting it on every call, so nothing can
  silently start sending cookies or inventing a CSRF header/field the backend
  never reads.
- **In-memory-only token (NFR3).** The bearer token lives in `client.ts`'s
  module-scoped `authToken` variable only — never written to
  `localStorage`/`sessionStorage`/a cookie, and cleared via `setAuthToken(null)`
  on logout/failed bootstrap. Regression-tested in `client.test.ts` and
  `AuthContext.test.tsx` (spies on `Storage.prototype.setItem` across a full
  login→logout cycle, and on `console.error`, to ensure the token is never
  persisted or logged).
- **Server cookie flags.** NFR3's "SHALL not weaken any server cookie flag
  (CB23)" is vacuously satisfied: this client never sets, reads, or
  participates in any cookie at all, so there is no flag it could weaken.
- **NFR8** (no client-side rate limiting; no copy built around
  `docs/security.md`'s stale "no rate limiting" claim) is unaffected by this
  reconciliation and remains satisfied as-is — no rate-limiting code exists
  anywhere in this app (lockout/throttle copy is FR7/FR8's scope, KAN-155).

This reconciliation was flagged as a judgment call for review: if a future
story finds a literal backend CSRF mechanism was added to `/api/v1` (it is
not present as of KAN-151 per `openapi.yaml`), this section must be revisited
before any FR9-shaped work is marked done against it.

## 7. Related Documentation

- Feeds: KAN-60 and its child epics (Core, Auth & Session, Password Recovery
  & Account Management, Live Tracking, History Dashboard, Device Pilotage,
  Client Branding & Assets).
- Depends on: KAN-116 (`/api/v1`, Done), KAN-126 (Technical Feature
  Discovery, Done — source of the CB-n/GAP-n citations above).
- [`cpal.docs/sdlc/README.md`](../sdlc/README.md) — SSoT and workflow this
  decision is recorded under.
