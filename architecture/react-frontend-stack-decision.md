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
AuthContext)". This system has no Supabase project anywhere —
`cpaltracker.web` auth is PHP session/cookie-based, re-checked against the
DB on every request ([`architecture.md`](../../cpaltracker.web/docs/architecture.md),
[`Auth-and-Session.md`](../features/technical/Auth-and-Session.md)), fronted
by KAN-116's `/api/v1`.

**Decision:** keep the `AuthContext` pattern (a single React context wrapping
auth state and actions), but back it with a thin REST client calling
`/api/v1`'s auth endpoints over `fetch`/cookies — not an actual Supabase
backend. No Supabase package is added to the stack.

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

## 6. Related Documentation

- Feeds: KAN-60 and its child epics (Core, Auth & Session, Password Recovery
  & Account Management, Live Tracking, History Dashboard, Device Pilotage,
  Client Branding & Assets).
- Depends on: KAN-116 (`/api/v1`, Done), KAN-126 (Technical Feature
  Discovery, Done — source of the CB-n/GAP-n citations above).
- [`cpal.docs/sdlc/README.md`](../sdlc/README.md) — SSoT and workflow this
  decision is recorded under.
