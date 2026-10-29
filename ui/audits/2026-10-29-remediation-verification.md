# UI Remediation Verification — cpaltracker-react 2026-10-29

SDLC W4 end, before W5 human acceptance. Verification only: no product code was modified.
Baseline: [`2026-10-28-audit.md`](2026-10-28-audit.md) @ `407bc61`. Feature:
[`IN-PROGRESS-UI-Remediation.md`](../../features/IN-PROGRESS-UI-Remediation.md) §13 and §15.

| | |
|---|---|
| App | `cpaltracker-react`, HEAD `c8e7bcb` **plus 89 uncommitted working-tree changes** (see §5). The verification ran on the working tree. |
| Kit reference | `cpaltracker.ui-reference/src/ui` |
| Procedure | `cto-tools/docs/visual-guideline/procedures/audit.md` (static pass, runtime pass) |
| Build under test | `VITE_API_BASE_URL=http://localhost:4199/api/v1 npx vite build --outDir dist-e2e-4199`, `vite preview --port 4199` |
| Runtime script | Throwaway Playwright script in the session scratchpad (not committed); API mocked with `page.route` as in the E2E specs |

## 1. Static metrics (§13)

Counts from `src/`, excluding `src/ui` and test files, same method as the audit.

| Metric | Audit 2026-10-28 | Now | Target | Result |
|---|---|---|---|---|
| Raw interactive elements outside `src/ui` | 48 | 0 (1 grep hit is a doc comment in `src/history/DateRangePicker.tsx:27`) | 0 | met |
| Inline `style` outside `src/ui` | 194 | 0 | 0 | met |
| Hard-coded colors in `src/` (`.ts`, `.tsx`, and `.css` outside `styles/tokens.css`) | 36 | 0 | 0 | met |
| `npm run lint` (guideline rules are errors outside the kit) | not run (no config) | exit 0, 0 problems | 0, blocking | met |
| `npm run lint:fixtures` | n/a | exit 0, all 7 rules report (FR10-raw-element, radix-import, inline-style, arbitrary-length, arbitrary-color, color-literal, jsx-a11y/alt-text) | pass | met |
| `npm run typecheck` | n/a | exit 0 | pass | met |
| Kit identical to reference (`diff -r src/ui ../cpaltracker.ui-reference/src/ui`) | no kit | identical, no output | identical | met |
| Conforming pages (structure) | 0/9 | 8/9 by structure; `/change-password` normal mode renders outside the shell (R-1) | 9/9 | **not met** |
| Major findings open | 41 | 1 confirmed regression (F-41 / FR12, R-1), 1 production defect on F-17 (R-2); the rest closed (§2) | 0 in scope | **not met** |
| Pages with a console error at runtime | not measured | 9/9 (R-2, same error) | 0 | **not met** |

## 2. Findings status

Legend. **Closed**: the finding's symptom is gone, evidence in the column. **Open**: not
closed or regressed. **Out of scope**: excluded by feature §6 or by decision.

Evidence kinds: **S** = static (lint 0, grep, diff against the reference), **R** = runtime
(screenshots in `2026-10-29/shots/`, DOM probes). Only items marked R were seen working in
a browser. Items marked S only were not individually re-observed; they rest on the lint
rules and the kit being identical to the reference (see §5).

### Kit gaps

| ID | Status | Evidence |
|---|---|---|
| K-1 | Closed | S: kit identical to the reference; reference decoupled from mock data (lint and typecheck pass in the app) |
| K-2 | Closed | S: React 19.3.1 in `package.json`; typecheck 0 |
| K-3 | **Partly closed** | S: pre-paint script exists in `index.html`. R: the production CSP blocks it (R-2), so the mechanism does not run in a production build |
| K-4 | Closed | R: skip link first in tab order (`Aller au contenu`, `#main-content`), nav items `Accueil / Suivi en direct / Historique / Pilotage`, client-logo slot present in header |
| K-5, K-6, K-7, K-8, K-9, K-10 | Closed | S only: items live in the kit that is identical to the reference. R for K-8: history map and live map render |

### Major layout

| ID | Status | Evidence |
|---|---|---|
| F-1 | Closed for 4 routes, **open for `/change-password`** | S: `App.tsx` has one `AppShellLayout` route for `/`, `/live-tracking`, `/history`, `/pilotage`. R: `change-password-*` shots have no sidebar or header (R-1) |
| F-2 | Closed | R: kit sidebar in shots |
| F-3 | Closed | R: at 390 px the sidebar trigger opens a drawer with the 4 nav links |
| F-4 | Closed | R: `PageHeader` on `/live-tracking`, `/history`, `/pilotage`, `/change-password`, 404 |
| F-5 | Closed | R: content widths in the 1440 shots follow the patterns |
| F-6 | Closed | R: live tracking is a fleet panel plus map split; stacks on 390 |
| F-7 | Closed | R: History uses stat tiles, filter bar, time picker, chronology |
| F-8 | Closed | R: device table plus transfer form cards on `/pilotage` |
| F-9 | Closed | R: `/login`, `/forgot-password`, `/reset-password` use the AuthLayout column |

### Major UI

| ID | Status | Evidence |
|---|---|---|
| F-10 | Closed | S: 0 raw elements, 0 inline style, lint blocking and passing |
| F-11, F-12, F-20 | Closed | S: `tokens.css` is the reference one; 0 hard-coded colors. R: brand orange/charcoal palette in all shots |
| F-13 | Closed | S: `--warning` defined in both themes (`tokens.css:51`, `:122`) |
| F-14, F-15 | Closed (not re-measured) | S: tokens ported from the reference. Contrast was not re-computed (§4) |
| F-16 | Closed | S: `color-scheme: dark` / `light` in `tokens.css` |
| F-17 | **Open (production)** | R: toggle works and `data-theme` follows the stored value in all 36 runs, but the pre-paint script is blocked by CSP in the production build (R-2); the theme is applied only after React mounts. Flash not measured |
| F-18 | Closed | R: CSP `img-src 'self' data: blob: https://tile.openstreetmap.org` in the built `index.html`; both maps requested `tile.openstreetmap.org` tiles with no CSP block; with tiles aborted the live map shows the notice "Les tuiles de la carte n'ont pas pu être chargées. Les marqueurs restent exacts." and the marker stays. See header note in §3 |
| F-19 | Closed (S) | S: `--map-tile-filter` in both themes, applied via `index.css:102` (live and history maps). Real tile appearance not seen (tiles mocked, §4) |
| F-21 | Closed (not re-measured) | S: charts use `useChartTheme`. Series contrast not re-computed |
| F-22, F-36 | Closed | R: status badges/dots in the fleet panel and device list (`Position connue`, `Aucune position`, `Avertissement`, `Hors ligne`) |
| F-23, F-24 | Closed | R: kit pill markers, kit zoom/fit/layers controls on live map; history map has kit timeline controls |
| F-25 | Closed | R: brand mark in shell and on auth pages in both themes |
| F-26 | Closed | R: header is a single compact row at 1440 and 390 (logo slot in the header) |
| F-27 | Closed | R: skip link first in tab order, single `#main-content` |
| F-28, F-29, F-30, F-31, F-32, F-34, F-35 | Closed | S: lint 0 (no raw controls, no local Card; `src/chrome/Card.tsx` removed). R: kit buttons, fields, banners, toasts seen on pages |
| F-33 | Closed | R: kit table on `/pilotage` (virtualization above threshold not exercised: 2 rows only) |
| F-37 | Out of scope | Feature §6: confirmation dialog needs a product decision; layout part delivered |
| F-38 | Closed (partly observed) | R: empty state (`Aucune position`), tile-error notice; loading and error states not forced on every page |
| F-39 | Closed | R: history charts use the kit chart wrappers |
| F-40 | Closed | R: stat tiles (`Mesures`, `Appareils`, `Température amb.`) on `/history` |
| F-41 | **Open** | Account page body uses `PageHeader`, `Separator`, sections, and toasts, but it is not in the shell (R-1) |

### Minor

| ID | Status | Evidence |
|---|---|---|
| F-42 | Closed | S: brand body/muted roles from tokens |
| F-43 | Closed (S) | S: reference scale via tokens |
| F-44 | Closed | R: 404 uses the Empty pattern with a Home button |
| F-45 | Closed (title part); home content out of scope | R: titles per route (`Suivi en direct · cpaltracker-react`, `Historique ...`, `Pilotage des devices ...`, `Mon compte ...`, `404 — Page not found ...`). Exception: `/` title is still `cpaltracker-react` (no route title). Home content out of scope per feature §6 |
| F-46 | Closed | R: kit link style on auth pages |
| F-47 | Closed (S) | S: `index.css` themes the leaflet layers. Tooltip and attribution look not inspected closely |
| F-48 | Closed (S) | S: `tabular-nums` in the live and history panels |
| F-49 | Out of scope (decided) | X-COPY decided 2026-10-29: not a defect. Each page keeps its language: auth login and 404 remain English, shell and data pages French. `index.html` stays `lang="en"` |

Counts: 49 findings plus 10 kit gaps. Closed 41 findings plus 9 kit gaps (of which 3
items rest on static evidence only, see Evidence kinds); open 2 (F-17 production defect,
F-41 plus F-1 for one route, counted once as R-1); partly closed 1 (K-3, same cause as
F-17); out of scope 2 (F-37 confirmation, F-49) plus F-45 home content.

## 3. Runtime matrix

9 routes x 2 viewports x 2 themes = 36 runs, all captured:
`2026-10-29/shots/<route>-<theme>-<width>.png` (routes: `home`, `live-tracking`,
`history`, `pilotage`, `login`, `forgot-password`, `reset-password`, `change-password`,
`not-found`). `/login` family were run unauthenticated (account 401); the others
authenticated as a `client` with a MAIN contract `REF-012` and two devices.

| Route | 1440 dark | 1440 light | 390 dark | 390 light | Console errors | Horizontal scroll |
|---|---|---|---|---|---|---|
| `/` | ok | ok | ok | ok | R-2 | none |
| `/live-tracking` | ok | ok | ok | ok | R-2; 404 on `messages/global.php` and `messages/contract.php` (my mock omitted them, not an app defect) | none |
| `/history` | ok | ok | ok | ok | R-2 | none |
| `/pilotage` | ok | ok | ok | ok | R-2 | none |
| `/login` | ok | ok | ok | ok | R-2; 401 on `account.php` (the mock, expected) | none |
| `/forgot-password` | ok | ok | ok | ok | R-2; 401 expected | none |
| `/reset-password?token=abc` | ok | ok | ok | ok | R-2; 401 expected | none |
| `/change-password` | R-1 | R-1 | R-1 | R-1 | R-2 | none |
| `*` (404) | ok | ok | ok | ok | R-2 | none |

"ok" = renders, correct theme applied (`data-theme` equals the requested theme in all 36),
no horizontal page scroll (`scrollWidth <= clientWidth` in all 36), no page error. Screenshots
were looked at for a sample of 12 (home, live, history, pilotage, login, account, 404, at
both widths and themes), not all 36.

Map tiles: live-tracking requested `tile.openstreetmap.org` (18 tile requests at 1440, 6 at
390), history 9, with no CSP console error for tiles. Note: the CSP is a `<meta
http-equiv>` tag in the built `index.html`, not an HTTP response header, so `curl -sI`
on the preview shows no CSP header (the app has no serving layer, as documented in
`vite-plugin-security-headers.ts`). The meta content contains `https://tile.openstreetmap.org`
in `img-src`.

### Problems found

- **R-1. `/change-password` is outside the shell in normal mode (F-41 / FR12, F-1).**
  `src/App.tsx` mounts it outside the `AppShellLayout` route, but `src/routes/Account.tsx`
  says "the shell is the route's layout" and renders only the page body. Result: the
  account page has no sidebar, no header, no navigation and no theme toggle at 1440 or
  390 (see `change-password-*` shots). First-connection mode is meant to be shell-less;
  normal mode is not.
- **R-2. The production CSP blocks the pre-paint theme script (F-17, K-3).** Every one of
  the 36 runs logs: `Refused to execute inline script because it violates ... script-src
  'self'`. The inline script in `index.html` that resolves the stored theme before first
  paint cannot run in a production build; `index.html` ships `data-theme="dark"` and the
  theme is applied after React mounts. Light-theme users probably see a dark flash; I did
  not measure it. Fix options (not done here): move the script to a file in `public/`, or
  add its sha256 to `script-src`.

## 4. Not closed, or could not be verified

- R-1 and R-2 above are not closed.
- Per-finding statuses in §2 marked S rest on lint, grep and the kit-identical check, not on
  inspection of each finding's specific symptom. In particular F-14, F-15 and F-21 (contrast
  ratios) were **not recomputed**, and no runtime computed-color or font sampling was done
  (audit procedure step 2). Keyboard focus was checked only for skip link and drawer.
- Map basemap look: tiles were mocked with a 1x1 PNG (and aborted for the failure case), so
  the real OSM raster, the dark and light tile filters, and the tooltip and attribution
  theming (F-19, F-47) were not seen. No real network tile fetch was attempted.
- The client logo shows its alt text (`Logo du client`) in the shell header in the shots.
  Cause is my mock (the logo endpoint was not mocked and returned HTML from the preview
  fallback), not verified against a real logo or the 404 placeholder path.
- Pilotage with many devices (virtualization, F-33), loading and error states on each page
  (F-38), the transfer form submission, history with 10 series (F-21), and the
  `mustChangePassword` first-connection layout were not exercised.
- Unit tests (`vitest`) and the E2E suite were not run in this verification (AC5 not
  verified here). Only lint, lint fixtures and typecheck were run.
- 89 files are modified or deleted but uncommitted in `cpaltracker-react`. This verification
  covers the working tree; the commit that is reviewed must contain the same state.
- `curl -sI` does not show CSP because it is a meta tag (see §3).
- AC1, AC3 and AC6 are not met until R-1 and R-2 are resolved. AC2 (reference showcase and
  guideline doc updates) was not checked.
