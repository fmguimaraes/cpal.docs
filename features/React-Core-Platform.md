1. Feature Name

React Core Platform & Hello World — cpaltracker.web UI rebuild foundation.

2. Problem / Pain Point

`cpaltracker.web`'s UI is server-rendered PHP + vanilla JS, with styling and
behavior duplicated per page (e.g. two different mobile breakpoints in the
shared CSS, no single design-token source — see
[`Client-Branding-and-Assets.md`](technical/Client-Branding-and-Assets.md)
CB22). It is not practical to start any feature-area React rebuild (Auth &
Session, Password Recovery, Live Tracking, History Dashboard, Device
Pilotage, Client Branding) without first deciding, once, the stack, the
project scaffold, how it talks to `/api/v1`, and how theming/design tokens
work — otherwise each feature epic reinvents these decisions independently
and drifts. This feature is that one-time foundation.

3. Goal / Outcome (Success Definition)

A new, empty-but-working React/TypeScript application exists: it builds,
lints, type-checks, runs a unit test and an E2E test in CI-equivalent
fashion, renders a single "hello world" screen that proves it can
authenticate-free fetch from `/api/v1` pointed at either a local/staging
backend or production (switchable without a code change), and renders that
screen correctly in both a light and a dark theme using a centralized token
system. Every later feature epic builds directly on this scaffold instead of
each re-deciding tooling.

4. User Stories (Primary Scenarios)

- As a frontend developer, I want a Vite+React+TS project scaffold with
  linting, type-checking and test runners already wired, so that I can start
  a feature epic without first solving tooling.
- As a frontend developer, I want a single `ApiClient` with a runtime-
  configurable base URL, so that the same build can point at a local/staging
  `/api/v1` or prod without a code change or rebuild.
- As a QA engineer, I want one unit test and one E2E test passing in this
  scaffold, so that I know the test harness itself works before any feature
  code is written on top of it.
- As a designer/developer, I want one centralized set of theme tokens
  driving both a light and a dark theme, so that changing a color/spacing
  value once updates every screen, instead of hunting per-component styles.
- As a product stakeholder, I want this new app to run side by side with the
  existing PHP site with zero change to it, so that the current monolith
  keeps serving real traffic throughout the rebuild.

5. In Scope (What WILL be delivered)

- Vite 5 + React 18.3 + TypeScript 5.5 project scaffold (new package/repo;
  binding TBD — see [`sdlc/README.md`](../sdlc/README.md#project-bindings-c-pal)
  `<frontend-repo>` placeholder for the React app).
- React Router v6 wired with at least two routes (hello-world screen + a
  not-found screen) to prove routing works end-to-end.
- Zustand store wired for at least one piece of global state (active theme)
  to prove the state layer works end-to-end.
- `ApiClient` module: wraps `fetch` against `/api/v1`, base URL resolved from
  a build-time/runtime env var (e.g. `VITE_API_BASE_URL`), with no
  hardcoded host. Includes a typed client for at least one real `/api/v1`
  read-only endpoint (e.g. a health/status or client-info endpoint) used by
  the hello-world screen.
- `AuthContext`: a React context providing auth state (authenticated user or
  null) and actions (login/logout placeholders), backed by a thin REST
  client against `/api/v1`'s auth endpoints over cookies — not Supabase (see
  [stack decision](../architecture/react-frontend-stack-decision.md) §3).
  This feature only scaffolds the context/interface; full login UI is Auth &
  Session epic's job.
- Design system foundation: a single token source (CSS variables / Tailwind
  4 theme config) for color, spacing, typography, radii, shadows; a light
  and a dark theme both defined from these tokens; a theme switch (stored in
  the Zustand store, persisted) that changes every themed surface via one
  state change — no component-level conditional styling.
- Tailwind CSS 4 (`@tailwindcss/vite`) wired against the token set; Lucide
  React available for icons.
- Responsive layout primitives exist and are exercised by the hello-world
  screen's layout (breakpoints defined in the token set), with desktop as
  the primary, first-class target (per product decision 2026-10-15); mobile
  rendering is best-effort only, not separately acceptance-tested here.
- Hello-world screen: renders a welcome message, the current theme, and data
  fetched live from `/api/v1` (proving connectivity), with a visible toggle
  to flip between local/staging and prod `/api/v1` targets in non-prod
  builds (e.g. via env-driven config, not user-facing in a prod build).
- Test harness: Vitest 4 + Testing Library (jsdom) configured and running
  one real unit test (hello-world component renders, theme toggle works);
  Playwright configured and running one real E2E test (loads the app,
  asserts hello-world content, toggles theme, asserts the visual/DOM change)
  against a running local build.
- ESLint 9 + typescript-eslint configured; `npm run typecheck` running
  `tsc --noEmit` against the app's own tsconfig (root tsconfig intentionally
  checks nothing, per the two-tsconfig project convention).
- `npm run lint`, `npm run typecheck`, `npm run test` (unit), `npm run
  test:e2e` all pass from a clean checkout with documented setup steps.

6. Out of Scope (What WILL NOT be delivered)

- Any backend, API, or database change of any kind. `/api/v1` (KAN-116) is
  consumed exactly as it exists today.
- Any modification to, or disabling of, any `cpaltracker.web` PHP page. The
  existing monolith is untouched and keeps serving traffic throughout and
  after this epic.
- Real login/session UI, password recovery, live tracking, history
  dashboard, device pilotage, or client branding screens — each is its own
  child epic built on top of this scaffold.
- A real Supabase backend or package — explicitly dropped per the stack
  deviation (see [stack decision](../architecture/react-frontend-stack-decision.md) §3).
- Realtime/WebSocket infrastructure (`socket.io-client`) — held until a
  feature epic needs it and the backend has a realtime endpoint to call.
- Mobile-first design, or mobile-specific acceptance testing.
- Production deployment/hosting setup (CI deploy pipeline, domain, TLS) —
  this feature proves the app runs locally and against prod `/api/v1`
  data, not that it is deployed to a public prod URL.

7. User Flow / UX Breakdown

- Developer runs `npm install && npm run dev` → Vite dev server starts →
  browser loads the hello-world screen at `/`.
- Screen mounts → `ApiClient` fires a request to the configured `/api/v1`
  base URL → response renders inline (e.g. "Connected to `/api/v1` at
  `<base-url>` — status: ok").
- User clicks the theme toggle → Zustand store updates `theme` →
  `data-theme` attribute (or equivalent) changes on the root element → every
  token-driven surface re-renders in the new theme, no page reload.
- User navigates to an undefined route → React Router renders the
  not-found screen, still themed correctly.
- CI (or local equivalent) runs lint → typecheck → unit tests → E2E tests,
  in that order; any failure blocks the feature as Done.

8. Functional Requirements (FRs)

FR1. The app SHALL build with Vite 5 targeting React 18.3 and TypeScript
5.5, with zero TypeScript errors from `npm run typecheck` on a clean
checkout.

FR2. The app SHALL resolve its `/api/v1` base URL from an environment
variable at build/runtime, with no backend hostname hardcoded in source, so
the identical build artifact can target local/staging or prod by
configuration alone.

FR3. The app SHALL expose an `ApiClient` module used by at least one screen
to fetch live data from a real `/api/v1` endpoint and render the result.

FR4. The app SHALL provide an `AuthContext` exposing authenticated-user
state and login/logout actions, implemented against `/api/v1`'s auth
endpoints over cookies, with no Supabase dependency.

FR5. The app SHALL define a single token source (design tokens: color,
spacing, typography, radii, shadows) consumed by every themed component —
no component SHALL hardcode a raw value the token set already defines.

FR6. The app SHALL ship both a light and a dark theme derived from the
token set in FR5, switchable via one state change (e.g. a Zustand-held theme
value), with the choice persisted across reloads.

FR7. The app SHALL define responsive breakpoints in the token set, with
desktop as the primary rendering target; mobile layout SHALL NOT crash or
visibly break but is not held to the same acceptance bar.

FR8. The app SHALL include at least two routes via React Router v6 (the
hello-world screen and a not-found screen), both themed per FR5/FR6.

FR9. The app SHALL include at least one Vitest + Testing Library unit test
that passes against the hello-world screen (rendering and theme toggle).

FR10. The app SHALL include at least one Playwright E2E test that passes
against a locally running build (loads the app, asserts content, toggles
theme).

FR11. The app SHALL run ESLint 9 + typescript-eslint with zero errors on a
clean checkout.

9. Non-Functional Requirements (NFRs)

NFR1. `npm run lint`, `npm run typecheck`, `npm run test`, and `npm run
test:e2e` SHALL each exit 0 on a clean checkout with documented setup steps.

NFR2. No change under this feature SHALL modify, disable, or alter the
behavior of any file in `cpaltracker.web` (the PHP monolith).

NFR3. No change under this feature SHALL add, modify, or remove any
backend endpoint, server logic, or database schema/data.

NFR4. The hello-world screen SHALL render the connectivity result (success
or error) within 3 seconds on a local `/api/v1` target under normal
conditions, with a visible error state (not a blank screen) if the fetch
fails.

NFR5. Theme switching SHALL apply to the whole screen in a single
synchronous UI update — no visible flash of unthemed content and no
partial/inconsistent theme state across components.

10. Data Inputs & Outputs

- Input: `VITE_API_BASE_URL` (or equivalent) env var, read at build/runtime.
- Input: response from one real `/api/v1` read-only endpoint (exact shape is
  whatever that endpoint already returns — this feature does not define a
  new contract, it consumes an existing one).
- Output: rendered hello-world screen, reflecting the fetched data, current
  theme, and current route.
- Output: persisted theme preference (e.g. `localStorage`), read back on
  next load.

11. Integration Points

- `/api/v1` (KAN-116, read direction) — one existing read-only endpoint,
  consumed as-is.
- `/api/v1` auth endpoints (read/write direction, cookie-based) — wired by
  `AuthContext` but not exercised by any UI flow in this feature (that's
  Auth & Session epic's job); this feature only proves the context compiles
  and the client can call the endpoints.
- No other external system.

11a. Software Items (IEC 62304 §5.3 — MANDATORY)

N/A — IEC 62304 Software Items are not applicable to this project today, per
[`sdlc/README.md`](../sdlc/README.md#applicability-to-c-pal-today).

12. Edge Cases & Constraints

- `/api/v1` unreachable or returns an error: hello-world screen shows a
  visible error state, not a blank page or unhandled exception.
- Env var for the API base URL missing/malformed: app fails fast with a
  clear build/startup error, not a silent fallback to a wrong host.
- Theme preference missing/corrupted in storage: app falls back to a
  defined default theme (e.g. light) without crashing.
- Browser without `prefers-color-scheme` support: app still defaults
  correctly via the persisted/explicit theme state, not relying solely on
  the media query.

13. Metrics / KPIs

- 100% of `npm run lint`, `npm run typecheck`, `npm run test`, `npm run
  test:e2e` passing on a clean checkout.
- 0 modified files under `cpaltracker.web` as a result of this feature.
- 1 working end-to-end connectivity proof against a real `/api/v1`
  endpoint, local and prod target both demonstrated.

14. Risks & Mitigations

- Risk: scaffold decisions made here lock in a bad pattern that every later
  feature epic then repeats. Mitigation: this feature's acceptance includes
  a short usage note (README) in the app explaining the `ApiClient`,
  `AuthContext`, and token/theme patterns so later epics reuse rather than
  reinvent them.
- Risk: no realtime/polling pattern exists yet and a later epic (e.g. Live
  Tracking) might silently add `socket.io-client` without a backend
  endpoint to call. Mitigation: explicitly called out as held/out-of-scope
  in the stack decision doc; a later epic needing it must first confirm a
  backend endpoint exists.
- Risk: theme/token system designed too narrowly, forcing per-component
  overrides later. Mitigation: token set reviewed against at least the
  known two legacy breakpoints (768px/480px, CB22 in
  [`Client-Branding-and-Assets.md`](technical/Client-Branding-and-Assets.md))
  before being treated as final.

15. Acceptance Criteria (Definition of Done)

- [ ] `npm run dev` serves the hello-world screen locally.
- [ ] Hello-world screen fetches and displays live data from a real
      `/api/v1` endpoint.
- [ ] Switching the `/api/v1` target (local/staging vs. prod) requires only
      a config/env change, no code change.
- [ ] Light and dark themes both render correctly from the same token set;
      switching is instant and total (no unthemed flash, no partial state).
- [ ] Desktop layout renders correctly; mobile does not crash.
- [ ] `npm run lint`, `npm run typecheck`, `npm run test`, `npm run
      test:e2e` all exit 0.
- [ ] Zero files changed under `cpaltracker.web`.
- [ ] All FR1–FR11 and NFR1–NFR5 validated.

16. Future Extensions (Optional, Not Required Now)

- Realtime data (WebSocket/polling) once a backend endpoint exists.
- Production deploy pipeline (hosting, domain, TLS, CI deploy stage).
- Mobile-first acceptance testing, if product later re-prioritizes mobile.
- i18n/localization (not mentioned in current scope or legacy site).
