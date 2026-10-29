# UI Remediation Verification — cpaltracker-react 2026-10-29

SDLC W4 end, before W5 human acceptance. Verification only: no product code was modified.

**Update 2026-10-29 (final verification, KAN-217).** Re-run on HEAD `7624eab` (KAN-229) with a clean tree. R-1 (`/change-password` shell) and R-2 (CSP blocks the pre-paint script) are fixed and re-verified; F-1, F-17, F-41, K-3 and FR12 are closed. The first-pass text below is kept; changed cells are marked "(update)". New: contrast recomputed ([`2026-10-29-contrast.md`](2026-10-29-contrast.md): 6 findings), pilotage with 5000 devices measured (§6), vitest and E2E run (§5).
**Update 2 2026-10-29 (KAN-217 contrast fixes and windowed mobile pilotage).** The contrast fixes made in the reference kit (`cpaltracker.ui-reference` `24d578e`) are copied unmodified into the app (`src/ui`, `src/styles`, `src/index.css`; `diff -r` empty). The app's own global rules that used to sit in `src/index.css` (KAN-171 reduced motion, mobile `h1` size, solid focus outline) moved unchanged to `src/app.css`, imported after it, so `index.css` can stay identical to the reference. Contrast re-run: **0 failures apart from the 4 exempt disabled buttons**, at 1440 and 390 px ([`2026-10-29-contrast.md`](2026-10-29-contrast.md)). The pilotage list at 390 px is now the kit `MobileList`, windowed above 50 devices (§6). Cells changed by this pass are marked "(update 2)".
Baseline: [`2026-10-28-audit.md`](2026-10-28-audit.md) @ `407bc61`. Feature:
[`IN-PROGRESS-UI-Remediation.md`](../../features/IN-PROGRESS-UI-Remediation.md) §13 and §15.

| | |
|---|---|
| App | `cpaltracker-react`, HEAD `c8e7bcb` **plus 89 uncommitted working-tree changes** (first pass). **Update: HEAD `7624eab`, clean tree.** |
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
| Kit identical to reference (`diff -r src/ui ../cpaltracker.ui-reference/src/ui`) | no kit | identical, no output. **Update 2:** also `src/styles` and `src/index.css` identical to the reference after the KAN-217 sync | identical | met |
| Conforming pages (structure) | 0/9 | 8/9 by structure; `/change-password` normal mode renders outside the shell (R-1). **Update: 9/9**, `/change-password` is in the shell (header, sidebar nav, theme toggle, skip link); first-connection mode (`mustChangePassword`) stays shell-less by design | 9/9 | met (update) |
| Major findings open | 41 | first pass: F-41 / FR12 (R-1), F-17 (R-2). **Update: 0 open**; contrast findings F1 to F7 are new, in [`2026-10-29-contrast.md`](2026-10-29-contrast.md). **Update 2: contrast findings fixed** (F1 to F5, F7, T1 to T3), only the exempt disabled buttons (F6) remain below threshold | 0 in scope | met (update 2) |
| Pages with a console error at runtime | not measured | 9/9 (R-2, same error). **Update: 0/9** app errors (36 runs; only the mocked 401 on auth pages and the mocked logo 404 appear) | 0 | met (update) |
| `npx tsc -b` | n/a | not run. **Update: exit 0** | pass | met |
| `npx vitest run` | not run | **Update: 64 files, 478 tests, all passed.** Update 2: 64 files, 480 tests (2 new `MobileList` cases in `DeviceList.test.tsx`), all passed | pass | met |
| E2E (`scripts/e2e-isolated.sh 4220`), full suite | not run | **Update: 138 passed, 0 failed, 0 flaky, twice in a row (1.3 min each)**; the new `pilotage-virtualization.spec.ts` (4 tests) is on top, 4 passed. **Update 2 (`scripts/e2e-isolated.sh 4240`, commit `e8f2436`): 142 passed (138 + the 4 virtualization tests), 0 failed, twice in a row**; two flaky tests that already fail on HEAD appeared in other runs, see §5 | pass | met |

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
| K-3 | Closed (update) | R: `index.html` loads `/theme-init.js` (same-origin, 200, `text/javascript`), 0 inline scripts, CSP `script-src 'self'`. With the app bundle aborted and `cpaltracker:theme=light` stored, `data-theme` is already `light` and body background is `rgb(245, 242, 237)`: the script runs in the production build. No CSP console error in 36 runs |
| K-4 | Closed | R: skip link first in tab order (`Aller au contenu`, `#main-content`), nav items `Accueil / Suivi en direct / Historique / Pilotage`, client-logo slot present in header |
| K-5, K-6, K-7, K-8, K-9, K-10 | Closed | S only: items live in the kit that is identical to the reference. R for K-8: history map and live map render |

### Major layout

| ID | Status | Evidence |
|---|---|---|
| F-1 | Closed (update) | S: `App.tsx` has one `AppShellLayout` route for `/`, `/live-tracking`, `/history`, `/pilotage`, and `/change-password` under `AccountLayout` with the shell. R: refreshed `change-password-*` shots show sidebar, header, theme toggle; probe: header present, theme toggle present, skip link `Aller au contenu`, H1 `Mon compte` |
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
| F-14, F-15 | Closed for the palette; **new contrast findings** (update) | Recomputed: [`2026-10-29-contrast.md`](2026-10-29-contrast.md), 350 token pairs (10 below threshold, none on background or card text roles) and 397 runtime rows. Findings F1 to F7 are kit or third-party combinations, not the F-14/F-15 palette defects. **Update 2: fixed**; re-run 328 token pairs and 378 (1440) / 280 (390) runtime rows, 0 below threshold apart from the 4 exempt disabled buttons |
| F-16 | Closed | S: `color-scheme: dark` / `light` in `tokens.css` |
| F-17 | Closed (update) | R: toggle works and `data-theme` equals the requested theme in all 36 refreshed runs. Pre-paint: `theme-init.js` applies the stored theme before the bundle (probe above, bundle aborted). Flash itself not measured as a paint timeline; the mechanism is verified |
| F-18 | Closed | R: CSP `img-src 'self' data: blob: https://tile.openstreetmap.org` in the built `index.html`; both maps requested `tile.openstreetmap.org` tiles with no CSP block; with tiles aborted the live map shows the notice "Les tuiles de la carte n'ont pas pu être chargées. Les marqueurs restent exacts." and the marker stays. See header note in §3 |
| F-19 | Closed (S) | S: `--map-tile-filter` in both themes, applied via `index.css:102` (live and history maps). Real tile appearance not seen (tiles mocked, §4) |
| F-21 | Closed (update, re-measured) | R: Plotly series strokes and donut fills on card, both themes: lowest 3.00 (chart-5 dark), 3.58 (chart-1 light); chart axis, tick and legend text 4.63 to 13.66. All at or above 3:1 / 4.5:1 |
| F-22, F-36 | Closed | R: status badges/dots in the fleet panel and device list (`Position connue`, `Aucune position`, `Avertissement`, `Hors ligne`) |
| F-23, F-24 | Closed | R: kit pill markers, kit zoom/fit/layers controls on live map; history map has kit timeline controls |
| F-25 | Closed | R: brand mark in shell and on auth pages in both themes |
| F-26 | Closed | R: header is a single compact row at 1440 and 390 (logo slot in the header) |
| F-27 | Closed | R: skip link first in tab order, single `#main-content` |
| F-28, F-29, F-30, F-31, F-32, F-34, F-35 | Closed | S: lint 0 (no raw controls, no local Card; `src/chrome/Card.tsx` removed). R: kit buttons, fields, banners, toasts seen on pages |
| F-33 | Closed (update) | R: kit table on `/pilotage`; virtualization exercised with 5000 devices (§6): 19 rows mounted at 1440 px. At 390 px the `MobileList` is paginated, not virtualized (§6) |
| F-37 | Out of scope | Feature §6: confirmation dialog needs a product decision; layout part delivered |
| F-38 | Closed (partly observed) | R: empty state (`Aucune position`), tile-error notice; loading and error states not forced on every page |
| F-39 | Closed | R: history charts use the kit chart wrappers |
| F-40 | Closed | R: stat tiles (`Mesures`, `Appareils`, `Température amb.`) on `/history` |
| F-41 | Closed (update) | Account page uses `PageHeader`, `Separator`, sections, toasts, and is now inside the shell (R-1 fixed); `mustChangePassword` redirects `/` to `/change-password` with no sidebar (`nav` links: 0) |

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

Counts (update): 49 findings plus 10 kit gaps. Closed 47 findings plus 10 kit gaps (3 kit
items rest on static evidence only, see Evidence kinds); open 0; out of scope 2 (F-37
confirmation, F-49) plus F-45 home content. First pass was 2 open (F-17/R-2, F-41 plus F-1/R-1)
and 1 partly closed (K-3).

## 3. Runtime matrix

**Refreshed 2026-10-29 on HEAD `7624eab`** (36 screenshots regenerated, build and preview on port 4221). 9 routes x 2 viewports x 2 themes = 36 runs, all captured:
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

Refresh result: `data-theme` equals the requested theme in 36/36, `scrollWidth <= clientWidth` in 36/36, a sidebar is present at 1440 on the 6 shell routes (`/`, live, history, pilotage, `/change-password`, 404) and absent on the 3 auth routes, titles per route as in F-45 (`/` is still `cpaltracker-react`). Console: no CSP error and no page error; only the mocked 401 on the auth routes and the mocked 404 for the client logo. The pre-refresh console column above (R-2 in every row) no longer applies. 12 of the 36 refreshed screenshots were looked at (change-password and pilotage at 1440 in both themes among them).

### Problems found (first pass, both fixed, see update)

- **R-1 (fixed, update: closed). `/change-password` is outside the shell in normal mode (F-41 / FR12, F-1).**
  `src/App.tsx` mounts it outside the `AppShellLayout` route, but `src/routes/Account.tsx`
  says "the shell is the route's layout" and renders only the page body. Result: the
  account page has no sidebar, no header, no navigation and no theme toggle at 1440 or
  390 (see `change-password-*` shots). First-connection mode is meant to be shell-less;
  normal mode is not.
- **R-2 (fixed, update: closed; `public/theme-init.js`, no inline script). The production CSP blocks the pre-paint theme script (F-17, K-3).** Every one of
  the 36 runs logs: `Refused to execute inline script because it violates ... script-src
  'self'`. The inline script in `index.html` that resolves the stored theme before first
  paint cannot run in a production build; `index.html` ships `data-theme="dark"` and the
  theme is applied after React mounts. Light-theme users probably see a dark flash; I did
  not measure it. Fix options (not done here): move the script to a file in `public/`, or
  add its sha256 to `script-src`.

## 4. Not closed, or could not be verified (update)

Closed since the first pass: R-1, R-2, the unit tests and the E2E suite not run (now run, §1), contrast not recomputed (done), pilotage with many devices (done, §6), the 89 uncommitted files (HEAD `7624eab`, clean tree), AC1/AC3/AC6 blockers.

Still not verified:

- Map basemap look: tiles mocked with a 1x1 PNG, so the real OSM raster, the dark and light tile filters, tooltip and attribution theming (F-19, F-47) were not seen. The attribution no longer depends on them (update 2: opaque card backdrop, F2 fixed). Contrast finding F2 (attribution link) depends on it.
- Client logo: the mock returns 404 (placeholder path) or a 1x1 PNG; a real logo was not seen.
- Loading and error states on each page (F-38), history with 10 series, the transfer form submission against a real API (the E2E suite covers it with mocks).
- The `mustChangePassword` first-connection layout was probed (no sidebar, redirect to `/change-password`) but not reviewed visually.
- Keyboard focus: 12 Tab stops per route sampled for ring contrast (one failure, F7; update 2: fixed, all 26 pass); no full keyboard walk of dialogs and the drawer.
- Hover, error and toast states, tooltips (see the contrast doc, "Not covered"). Contrast at 390 px: sampled in update 2 (drawer closed), same result as 1440.
- ~~Pilotage at 390 px is paginated, not virtualized (§6).~~ Update 2: windowed (kit `MobileList`), every device reachable by scrolling, §6.
- Pre-paint theme flash was verified by mechanism (script runs before the bundle), not as a measured paint timeline.
- AC2 (reference showcase and guideline doc updates) was not checked.
- `curl -sI` does not show CSP because it is a meta tag (see §3).

## 5. Test run (update)

HEAD `7624eab`, clean tree. `npx tsc -b`: exit 0. `npx vitest run`: 64 files, 478 tests, 0 failed. `npm run lint`: exit 0, no problems. `npm run lint:fixtures`: exit 0, all 7 rules reported. `scripts/e2e-isolated.sh 4220`, twice: 138 passed, 0 failed each time. Plus `e2e/pilotage-virtualization.spec.ts`: 4 passed (not in the 138).

**Update 2 (KAN-217 sync and windowed mobile list).** Tree under test: `cpaltracker-react` commit `e8f2436` ("KAN-217 - contrast fixes and windowed mobile pilotage list") checked out alone in a separate worktree. The shared checkout had unrelated uncommitted edits from parallel work (for example `src/routes/Login.tsx`, which fails KAN-171 FR12 "soft-shadow card"), so it was not used for the final runs. `npx tsc -b`: exit 0. `npx vitest run`: 64 files, 480 tests, 0 failed. One run under heavy host load (load average 46) had `TransferForm.test.tsx` cases time out at 5 s; the re-run passed. `npm run lint`: exit 0. `npm run lint:fixtures`: exit 0, all 7 rules reported. `scripts/e2e-isolated.sh 4240`, full suite on `e8f2436`: **142 passed, 0 failed, twice in a row**. Earlier runs on the same changes had 1 failure each in one of two tests. Both tests are also flaky on HEAD `d551e4b` without these changes, measured on an untouched worktree: `live-tracking.spec.ts:104` (the tooltip closes between two assertions; 5 of 30 repeats fail on HEAD) and `KAN-169-transfer-form-history.spec.ts:137` (calendar day button detached; 7 of 80 repeats fail on HEAD). They were not fixed here because they belong to other stories' specs; flagged.

## 6. Pilotage with 5000 devices (update, F-33)

Spec: `cpaltracker-react/e2e/pilotage-virtualization.spec.ts`, scenarios in `cpal.docs/manual-e2e/KAN-217-Pilotage-Virtualization.md`. Screenshots: `2026-10-29/shots/pilotage-virtualized-*.png`.

| Check | 1440 px (dark and light) | 390 px (dark and light) |
|---|---|---|
| Rows in the DOM | **19 of 5000** (`data-virtualized="true"`), stays below 100 at the end of the list | first pass: 20 of 5000 at load, 5000 after paging to the end. **Update 2: windowed** (`data-windowed="true"`), at most **24 cards** mounted at any point of a scripted scroll to the end (asserted below 60); count "5000 sur 5000" |
| Last device reachable | by scrolling the table container to the end: `CHIP-4999` mounted and visible | first pass: only through "Afficher plus" 249 times (about 15 s). **Update 2: by scrolling the page**: `CHIP-4999` in the viewport, `CHIP-0000` unmounted, no "Afficher plus" (about 5 s for the scripted scroll, 2 frames per step) |
| Sticky header | `thead` is `position: sticky`, offset 0 to 1 px from the container top at mid-scroll | not applicable (cards) |
| Selection | works; survives scrolling away and back; default first device selected | works (`aria-pressed`); update 2: also on `CHIP-4999` after scrolling to it |
| Sorting | "Device" header: ascending then descending, `aria-sort` correct, `CHIP-4999` first when descending | no sort control on cards |
| Test ids | `pilotage-device-<id>`, `-select`, `-last-seen`, `pilotage-device-list`, `pilotage-contract-selector` unchanged | same, plus `pilotage-device-list` is the `ul` |
| Console errors | 0 | 0 |
| No horizontal scroll | n/a | 0 px overflow |
| Scroll smoothness | 121 frames over a 120-step scripted scroll of the full 5000 rows: median 16.7 ms, p95 16.7 to 33.4 ms, max 33.4 to 49.9 ms, 0 frames above 50 ms (4 runs, headless Chromium) | n/a |

Finding (first pass, closed in update 2): the 390 px list is paginated, not virtualized. With 5000 devices the first view is fine (20 cards) but reaching the end mounts 5000 cards and takes about 15 s of clicking in the test. Not a defect against the current spec (AC4 asks for count and "Afficher plus"); flagged for a decision.

**Update 2 (KAN-217):** the 390 px list is the kit `MobileList` with `windowThreshold` and `pageSize` equal to the table's threshold (50): up to 50 devices every card is shown with the count (no "Afficher plus"); above 50 the list is windowed against the page scroll. Testids (`pilotage-device-list` on the `ul`, `pilotage-device-<id>` on each `li`, `-select`, `-last-seen`), selection (`aria-pressed`, `bg-accent`) and the count text ("{n} sur {total}") are unchanged. The "Afficher plus" paging is removed because the window makes it redundant. Specs: `pilotage-virtualization.spec.ts` V2 @SI-227-7 and `KAN-227-pilotage-list-form.spec.ts` AC4 @SI-227-4 updated (scenarios in `manual-e2e/KAN-217-Pilotage-Virtualization.md` and `manual-e2e/KAN-217-Pilotage.md`, feature §12). Screenshots `2026-10-29/shots/pilotage-virtualized-*-390.png` regenerated.
