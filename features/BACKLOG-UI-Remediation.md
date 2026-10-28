# Feature: UI Remediation — cpaltracker-react 2026-10-28

## 1. Feature Name

UI Remediation — cpaltracker-react 2026-10-28: bring cpaltracker-react onto the c-PAL
visual guideline by porting the reference tokens, kit and page patterns, then moving
every page onto them, and close the audit's in-scope findings.

## 2. Problem / Pain Point

The 2026-10-28 audit found **9 Major-layout, 32 Major-UI and 8 Minor** findings, and
**0/9 pages (0%)** conform to the guideline. This is the first audit, so there is no
trend yet. The app has no kit, no guideline lint, an off-brand blue palette, about 190
inline styles, no navigation on a phone, and a production CSP that blocks every map
tile. Report: [`ui/audits/2026-10-28-audit.md`](../ui/audits/2026-10-28-audit.md).

The guideline is [`engineering/ui-guidelines.md`](../engineering/ui-guidelines.md); the
yardstick is `cpaltracker-ui-reference` (kit `src/ui`, patterns `src/ui/patterns`,
pages `src/showcase/patterns`, tokens `src/styles/tokens.css`). This feature follows
[`UI-Bootstrap.md`](UI-Bootstrap.md), which built that reference.

## 3. Goal / Outcome (Success Definition)

Every in-scope finding is closed and the next audit confirms it: cpaltracker-react
uses the reference tokens and kit, every shell route sits in `AppShell` on its
pattern, the guideline lint is a blocking gate, and the existing unit and E2E suites
still pass (with the explicitly listed spec adjustments in §12).

## 4. User Stories (Primary Scenarios)

- As a fleet operator, I want the live map, history and pilotage pages to use the full
  screen and the same header, controls and states, so I can read the fleet quickly.
- As a mobile user at 390 px, I want a navigation drawer, so I can reach every page.
- As a user who works in dark mode (the default), I want every surface, map and chart
  to be dark and readable, and my light or system choice remembered.
- As a user in production, I want the map basemap to load.
- As a developer, I want the kit, tokens and lint in the app, so new screens cannot
  drift and need no one-offs.

## 5. In Scope (What WILL be delivered)

- **Kit gaps** (fixed in `cpaltracker-ui-reference` first, rule 5): K-1, K-2, K-3, K-4,
  K-5, K-6, K-7, K-8, K-9, K-10.
- **Major layout:** F-1, F-2, F-3, F-4, F-5, F-6, F-7, F-8, F-9.
- **Major UI:** F-10 to F-36, F-38 to F-41. F-37 partly (transfer layout only, see §6).
- **Minor:** F-42, F-43, F-44, F-46, F-47, F-48, and the document-title part of F-45,
  batched in one area (FR16).

## 6. Out of Scope (What WILL NOT be delivered)

- **F-37 confirmation step** (AlertDialog before transfer and before the settings
  push; transfer moved into a Dialog): it changes the user flow and the KAN-166 /
  KAN-169 E2E paths. Needs a product decision. Until then transfer stays an inline
  form, restyled on PAT-FORM.
- **F-45 home content** (replacing the HelloWorld connectivity page with a PAT-DASH
  overview, `ref:src/showcase/patterns/Overview.tsx`): needs a product decision and
  data the API does not expose yet. The page is restyled (shell, header, kit) only.
- **F-49 language mix**: depends on the open X-COPY decision. Until decided, no copy is
  translated; each screen keeps its current language.
- **Route renaming** to the reference's `/app/*` paths: paths stay as in
  `src/auth/paths.ts` (`/live-tracking`, `/history`, `/pilotage`, ...), and nav testids
  stay `nav-home`, `nav-live-tracking`, `nav-history`, `nav-pilotage` (the reference's
  `nav-live` is not adopted).
- New features shown in the reference but not in the app (Map/List toggle data,
  needs-attention list beyond existing data, row menus with new actions, CSV export):
  the pattern structure is ported, empty slots are omitted.

## 7. User Flow / UX Breakdown

Delivery order; each line is a feature area for W1. Areas 1-3 are the foundation and
must land first; areas 4-8 can run in parallel once area 3 is merged; area 9 closes.
FR5 (CSP) has no dependency and can ship immediately.

| # | Area | FRs | Findings closed | Pattern / kit items (paths in `cpaltracker-ui-reference/`) |
|---|---|---|---|---|
| 0 | OSM tiles in production (hotfix, independent) | FR5 | F-18 | `vite-plugin-security-headers.ts`; tile notice from `src/ui/map/FleetMap.tsx` lands with area 5 |
| 1 | Kit gaps in the reference + guideline doc | FR1, FR2 | K-1, K-3..K-10 | `src/ui/status.tsx`, `src/ui/map/*`, `src/ui/timeline.tsx`, `src/ui/health-grid.tsx`, `src/ui/patterns/app-shell.tsx`, `src/ui/patterns/theme-toggle.tsx`, `src/ui/data-table.tsx`, `src/ui/charts/use-chart-theme.ts`, `index.html` |
| 2 | Stack upgrade in the app | FR3 | K-2 | React 19, react-leaflet 5, react-leaflet-cluster, kit dependencies |
| 3 | Tokens, theme, kit and lint port | FR4 | F-10, F-11, F-12, F-13 (token), F-14 (token), F-15, F-16, F-17 (mechanism), F-20, F-42 | `src/styles/tokens.css`, `src/index.css`, `src/app/theme.tsx`, `src/ui/**`, `src/lib/utils.ts`, `src/hooks/use-mobile.ts`, `eslint.config.js`, `lint-fixtures/` |
| 4 | Shell, header, content width | FR6, FR7 | F-1, F-2, F-3, F-4, F-5, F-17 (toggle), F-25, F-26, F-27 | NAV-SHELL `src/ui/patterns/app-shell.tsx`, `src/ui/sidebar.tsx`, NAV-HEADER `src/ui/patterns/page-header.tsx`, `eyebrow.tsx`, `theme-toggle.tsx`, `public/logo-cpal-*.png` |
| 5 | Live tracking onto PAT-SPLIT | FR8 | F-6, F-19, F-23, F-24 (live), F-47 | `src/showcase/patterns/LiveTracking.tsx`, `live/*`, `src/ui/map/*`, `src/ui/charts/donut-chart.tsx`, `src/ui/stat.tsx` |
| 6 | History onto the reference History page | FR9 | F-7, F-13, F-21, F-24 (timeline), F-30 (history), F-39, F-40 | `src/showcase/patterns/History.tsx`, `history/data.ts`, `src/ui/patterns/{time-picker,filter-bar}.tsx`, `src/ui/{stat,widget-card,table,alert}.tsx`, `src/ui/charts/time-series-chart.tsx` |
| 7 | Pilotage onto PAT-LIST + PAT-FORM | FR10 | F-8, F-22, F-30 (transfer), F-33, F-36, F-37 (layout only) | `src/showcase/patterns/Devices.tsx`, `devices/*`, `Settings.tsx`, `forms/*`, `src/ui/{data-table,status,timeline,select,date-time-field}.tsx` |
| 8 | Auth and account onto PAT-AUTH / PAT-SETTINGS | FR11, FR12 | F-9, F-25 (auth), F-41, F-46 | `src/showcase/patterns/{Login,ForgotPassword,Settings}.tsx`, `auth/*`, `src/ui/{form,field,input,input-group}.tsx` |
| 9 | Cross-page sweep, lint blocking, minor batch | FR13, FR14, FR15, FR16 | F-14, F-15, F-16, F-28, F-29, F-31, F-32, F-34, F-35, F-38 (whatever remains after 4-8), F-43, F-44, F-45 (title), F-48 | `src/ui/{button,form,field,input,checkbox,radio-group,select,toggle-group,alert,empty,skeleton,spinner,sonner,card}.tsx`, `src/showcase/patterns/{States,NotFound}.tsx`, `states/*` |

## 8. Functional Requirements (FRs)

Kit gaps first (rule 5: the reference and its showcase change first, in their own
change), then layout, then UI, then minor. `ref:` = `cpaltracker-ui-reference/`.

### Kit gaps

- **FR1. Kit decoupled from mock data and completed for the app.** In
  `cpaltracker-ui-reference`, the kit SHALL (each change shown in the showcase,
  snapshots updated in the same change):
  - own its device-status type (the four D3 states `ok | warning | alert | offline`
    and their labels) in `src/ui/status.tsx`, and `src/ui/health-grid.tsx`,
    `src/ui/timeline.tsx`, `src/ui/map/{DeviceMarker,FleetMap,markers}` SHALL take a
    minimal kit-owned device shape instead of importing `@/mocks/fleet` (K-1);
  - resolve the stored theme (`cpaltracker:theme`: `light | dark | system`) before
    first paint with an inline script in `index.html` (K-3);
  - let `AppShell` receive its nav sections (paths and `testId`s), user-menu content
    (account link, sign out) and a client-logo slot from the app, and render a
    skip-to-content link targeting its single `<main>` (K-4);
  - provide a typed date entry (`AAAA-MM-JJ` text with parse and validation, in a
    `Field`) next to `DateRangePicker`, or a typed-input mode on it (K-5);
  - support row virtualization in `DataTable` above a threshold (K-6);
  - document and implement the rule for more than five series: `useChartTheme()`
    cycles `--chart-1..5` and adds a dash / marker-symbol variant per cycle, and
    `FleetMap` markers use the same index (K-7);
  - add timeline playback to the map kit (instant slider, play/pause, trail switch,
    fit-to-track) as a `FleetMap` companion built from `Slider`, `Button`, `Switch`,
    `ButtonGroup` (K-8);
  - keep `data-testid="theme-toggle"` on a control whose single click flips the
    resolved theme light ↔ dark, with `system` offered next to it (or in the user
    menu) under `theme-system` (K-10).
- **FR2. Guideline doc aligned.** `engineering/ui-guidelines.md` SHALL map
  `/app/history` to the pattern the reference History page actually implements
  (dashboard-shaped: `Stat`, `WidgetCard`, `TimePicker`, `FilterBar`) and list the
  FR1 additions in "Choosing a component" (K-9).

### Foundation in the app

- **FR3. Stack upgrade.** cpaltracker-react SHALL move to the reference's runtime
  versions (React 19.x, react-leaflet 5.0.0, `react-leaflet-cluster` 4.1.3) and add
  the kit's dependencies (`radix-ui`, `class-variance-authority`, `tailwind-merge`,
  `clsx`, `lucide-react`, `sonner`, `date-fns`, `@tanstack/react-table`,
  `react-hook-form` + `zod` resolver as the reference pins them), with no visual or
  behavior change in this step (K-2).
- **FR4. Tokens, theme, kit and lint ported.**
  - `src/styles/tokens.css` SHALL be `ref:src/styles/tokens.css` verbatim (brand
    palette; every role: background, card, popover, muted, media, input, ring,
    primary / primary-foreground / primary-text / primary-soft, body,
    muted-foreground, success / warning / danger / offline / info and soft variants,
    status-foreground, chart-1..5, chart-axis, chart-grid, sidebar-*,
    map-tile-filter, shadow-overlay, shadow-media; `color-scheme` per theme; Arial /
    Georgia / mono stacks) and `src/index.css` SHALL carry the reference `@theme inline`
    bridge and Leaflet rules. `src/theme/tokens.css`, `src/theme/chartColors.ts` and,
    once unused, `src/theme/breakpoints.ts` SHALL be removed (closes F-11, F-12, F-13
    token part, F-14 token part, F-15, F-16, F-20, F-42, token part of F-43).
  - Until every consumer is migrated, the legacy names read by tests (`--color-bg`,
    `--color-primary`, `--color-primary-hover`, ...) SHALL remain as aliases of the new
    roles in one marked block, removed in FR16.
  - Theme SHALL come from `ref:src/app/theme.tsx` (`light | dark | system`, dark
    default, same storage key `cpaltracker:theme`), replacing
    `src/store/themeStore.ts` (closes F-17 mechanism part).
  - `src/ui/**`, `src/lib/utils.ts`, `src/hooks/use-mobile.ts` SHALL be copied from
    the reference after FR1, unmodified (local changes go upstream first).
  - `eslint.config.js` and `lint-fixtures/` SHALL be ported; the structure rules run
    in report mode until FR13, color rules blocking from the start for `src/ui` (closes
    F-10 at FR13).
- **FR5. OSM tiles allowed in production.** The production CSP in
  `vite-plugin-security-headers.ts` SHALL be `img-src 'self' data: blob:
  https://tile.openstreetmap.org`, with a case in
  `vite-plugin-security-headers.test.ts` asserting it; a tile load failure SHALL show
  the `FleetMap` inline notice over `bg-media` while markers still render (EC3)
  (closes F-18; notice delivered with FR8/FR9).

### Major layout

- **FR6. One shell.** Every authenticated route SHALL render inside one layout route
  `<RequireAuth><AppShell/></RequireAuth>` with `<Outlet/>` (`ref:src/ui/patterns/
  app-shell.tsx`, `src/ui/sidebar.tsx`): lucide icons, collapsible icon rail, active
  accent bar, mobile drawer below 768 px, theme toggle and user menu (account, sign
  out) in the shell, client logo in the FR1 slot (compact, `Skeleton` while loading,
  hidden when empty), c-PAL logo variant chosen by resolved theme
  (`ref:public/logo-cpal-{orange,dark}.png`), skip link and a single `<main>`.
  Per-route `<main>` wrappers and `src/chrome/AppChrome.tsx` are removed (closes F-1,
  F-2, F-3, F-17 toggle part, F-25 shell part, F-26, F-27).
- **FR7. Page header and pattern widths.** Every shell route SHALL start with
  `PageHeader` (+ `Eyebrow`, breadcrumb where nested, actions slot) in place of
  `src/chrome/PageTitle.tsx`, titles in `text-foreground`; content width and gaps
  SHALL come from the route's pattern (full-bleed PAT-SPLIT, fluid dashboard and
  PAT-LIST, ≤ 800 px PAT-FORM) instead of `max-w-2xl p-8` (closes F-4, F-5).
- **FR8. Live tracking on PAT-SPLIT.** `/live-tracking` SHALL follow
  `ref:src/showcase/patterns/LiveTracking.tsx`: `FleetMap` filling the content area
  (pill markers with state ring, clusters, zoom/fit/layers top right, dark tile filter
  and `bg-media` from tokens, themed tooltip), fleet panel from `live/FleetPanel.tsx`
  with the device list filtering the map (`Checkbox` rows), contract choice as
  `Select` or `ToggleGroup`, last-updated and refresh in the header actions, panel
  below the map at 390 px. `PositionsMap.tsx`, `markerStyle.ts` palettes and
  `DeviceFilter.tsx` markup are replaced; mass-threshold logic is kept and mapped to
  the four states (closes F-6, F-19, F-23, F-24 live part, F-47; F-22 live part).
- **FR9. History on the reference History page.** `/history` SHALL follow
  `ref:src/showcase/patterns/History.tsx`: `TimePicker` top right with the typed
  date entry from FR1 (FR7 of React-History-Dashboard kept), `FilterBar` for contract
  and devices, `Stat` row (measurements, devices, min/max °C) replacing the text
  summary, map and charts in `WidgetCard`, temperature in `TimeSeriesChart` with the
  2-8 °C `threshold` band and a date-typed, `date-fns`-formatted x-axis, device series
  from `useChartTheme()` with the FR1 overflow rule, timeline playback from FR1,
  sampling notice as `Alert variant="warning"`, lot-hook history as a static
  `Table` with right-aligned `tabular-nums` and horizontal scroll at 390 px (closes F-7,
  F-13, F-21, F-24 timeline part, F-30 history part, F-39, F-40; F-22 history part,
  F-33 lot part).
- **FR10. Pilotage on PAT-LIST and PAT-FORM.** `/pilotage` SHALL show devices as a
  `DataTable` per `ref:src/showcase/patterns/Devices.tsx` (status dot first, mono ID,
  formatted last-seen with absolute time in `title`, selection with accent, virtualized
  per FR1, `devices/MobileList.tsx` at 390 px), contract choice in the header as
  `Select`, settings as a PAT-FORM section per `Settings.tsx` (one column, ≤ 800 px,
  action bar), transfer as an inline PAT-FORM card in two columns using
  `Select` and `DateTimeField`, transfer history as `Timeline` with a `Badge` for
  ongoing (closes F-8, F-22, F-30 transfer part, F-33 device and transfer parts, F-36,
  F-37 layout part).
- **FR11. Auth on PAT-AUTH.** `/login`, `/forgot-password`, `/reset-password` SHALL use
  `ref:src/showcase/patterns/auth/AuthLayout.tsx` (logo, form column, footer, stacked
  at 390 px), `Form*` + `Input` / `InputGroup` (password show/hide), full-width primary
  `Button` with loading, `Alert variant="danger"` for submit errors, links from
  `auth/AuthLink.tsx`. Copy is not translated (closes F-9, F-25 auth part, F-46).

### Major UI

- **FR12. Account on PAT-SETTINGS.** `/change-password` SHALL sit in the shell
  (first-connection mode stays shell-less) and follow
  `ref:src/showcase/patterns/Settings.tsx`: sections separated by `Separator`,
  `Form*` fields, success through `toast()` (closes F-41).
- **FR13. Controls and forms everywhere.** No raw `<button>`, `<input>`, `<select>`,
  `<table>`, `<textarea>` and no inline `style` SHALL remain outside `src/ui`: buttons
  via `Button` (one primary per page, outline / ghost / link, `Spinner` loading), fields
  via `Form*`/`Field*` (label above, required marker, `FieldError` under the field,
  `aria-invalid`, one announcement), `border-input` boundaries, `ring-ring` focus,
  selectors via `RadioGroup` / `Select` / `ToggleGroup` with arrow-key navigation,
  checkboxes via `Checkbox`. The FR4 lint structure rules SHALL then be switched to
  blocking (closes F-10, F-14, F-15, F-16, F-28, F-29, F-34).
- **FR14. Feedback and states everywhere.** Every data view SHALL have loading =
  `Skeleton` shaped like the content, empty = one `Empty` with reason and next step,
  error = `Alert variant="danger"` with a retry, per
  `ref:src/showcase/patterns/states/*`; mutation outcomes via `toast()` with the
  existing flash testids kept on a rendered element; persistent messages via `Alert`
  (`warning`, `info`, `note`) (closes F-31, F-32, F-38).
- **FR15. Cards and status.** `src/chrome/Card.tsx` SHALL be replaced by `Card*` /
  `WidgetCard`, every bordered box SHALL be a `Card`, and device state SHALL be shown
  with `StatusBadge` / `StatusDot` (no reading = offline grey with `—`) (closes F-35,
  F-36).

### Minor

- **FR16. Minor batch and cleanup.** Radius, shadow, type and breakpoints SHALL be the
  reference's (Tailwind default breakpoints, no `shadow-md` on plain cards, labels from
  `FieldLabel`); headings in data views from `CardTitle`/`WidgetCard` and numbers in
  `tabular-nums`; 404 from `ref:src/showcase/patterns/NotFound.tsx` inside the shell
  when authenticated, with a way home; `auth-loading` as `Spinner`; `document.title`
  set per route; the legacy `--color-*` aliases from FR4 removed once no spec reads
  them (closes F-43, F-44, F-45 title part, F-48).

## 9. Non-Functional Requirements (NFRs)

- NFR1. No behavior change: routes, API calls, validation rules, permissions and data
  shown stay the same. Existing unit and E2E suites pass; the only allowed spec edits
  are the ones listed in §12, each made in the change that requires it with the reason
  in the commit.
- NFR2. WCAG 2.1 AA in both themes on every touched page; control boundaries and focus
  ring ≥ 3:1; chart series ≥ 3:1 on their surface.
- NFR3. Every touched route works at 1440 px and 390 px, dark and light, with no
  horizontal page scroll (tables scroll inside their container).
- NFR4. No color literal, inline style or arbitrary value outside `src/ui` and
  `src/styles`; `npm run lint` and `npm run lint:fixtures` pass (blocking from FR13).
- NFR5. Kit files in the app are identical to the reference; any difference goes
  upstream first.

## 10. Data Inputs & Outputs

No API change. Token changes come from `ref:src/styles/tokens.css` as a whole (the old
blue palette is replaced, not extended):

| Role | Dark | Light | Reason |
|---|---|---|---|
| background / card | `#1a1d22` / `#22252e` | `#f5f2ed` / `#eee6d4` | Brand (F-11) |
| primary / primary-text | `#dd8c3e` / orange | `#c57a30` / `#915923` | Orange text on light needs 4.5:1 (F-15) |
| ring | `#dd8c3e` | `#915923` | 3:1 on every surface (F-15) |
| input | `#686e7f` | `#938267` | 3:1 control boundary (F-14) |
| warning | `#e3c34f` | `#755c00` | Was undefined; yellow, not orange (F-13, F-22) |
| media | `#0e0f12` | `#1a1d22` | Map frame (F-19) |
| chart-1..5 | `#dd8c3e #6fa8c7 #93b79c #b4a2d6 #676e80` | `#a8692a #3a6e8a #4a7456 #6d5c99 #5f656f` | ≥ 3:1 series (F-21) |
| map-tile-filter | invert / hue-rotate / dimmed | `saturate(0.55) sepia(0.12)` | Both maps (F-19) |

The theme preference stays in `localStorage` under `cpaltracker:theme`; the value
`system` is added and the default becomes `dark` (D7).

## 11. Integration Points

- `cpaltracker-ui-reference`: FR1 (kit gaps, showcase, snapshots) before any port.
- `cpaltracker-react`: `src/App.tsx`, `src/chrome/*` (removed), `src/theme/*`
  (removed), `src/store/themeStore.ts` (replaced), `src/index.css`, `index.html`,
  `vite-plugin-security-headers.ts` (+ test), `eslint.config.js`, `package.json`,
  `src/routes/*`, `src/live-tracking/*`, `src/history/*`, `src/pilotage/*`,
  `src/routes/account/*`, new `src/ui/**`, `src/styles/tokens.css`, `public/logo-cpal-*.png`.
- `cpal.docs`: `engineering/ui-guidelines.md` (FR2), this feature, the audit report.

## 12. Edge Cases & Constraints

- **data-testids are kept on the same element role.** Shell: `app-sidebar`, `app-nav`,
  `nav-home`, `nav-live-tracking`, `nav-history`, `nav-pilotage`, `cpal-brand-mark`,
  `client-logo`, `client-logo-placeholder`, `theme-toggle`, `logout-button`,
  `auth-loading`, `page-content` (unit test). Auth: `login-form`, `login-email`,
  `login-password`, `login-submit`, `login-error`, `login-notice`,
  `login-email-error`, `login-password-error`, `login-forgot-password-link`,
  `forgot-password-*`, `reset-password-*`. Account: `account-*` (incl.
  `account-first-connection-notice`, `account-back-to-app`, `account-logout`,
  `account-password-*`, `account-email-address`, `account-phone-number`). Live:
  `contract-selector`, `contract-option-*`, `contract-access-error`, `positions-*`,
  `live-tracking-*`, `device-filter`, `device-filter-show-all`, `device-filter-*`,
  `marker-tooltip-*`, `global-message`. History: `history-*` (dashboard, contract
  selector and options, sampling banner, access, fetch error and dismiss, charts,
  chart-ta/hr/m, no-measurements), `date-range-*`, `device-checklist-*`,
  `selection-summary*`, `lot-hook-*`, `timeline-*`. Pilotage: `pilotage-*`,
  `settings-*` (incl. `settings-form-flash` with `data-flash-kind`), `transfer-*`
  (incl. `transfer-flash-ok/ko`, `transfer-target-option-{id}`,
  `transfer-history-row-*`). Hello: `connectivity-status`, `connected`,
  `connection-error`.
- **Selectors and roles tests depend on:** the history map container keeps the class
  `timeline-map` (KAN-176 reads `.timeline-map .leaflet-tile-pane`); messages unit
  tests query by `role=alert`/`role=status` keep that role on the rendered element.
- **Spec edits allowed (NFR1):** `e2e/KAN-176-design-system-theming-stack-compliance.spec.ts:231,248`
  assert the old Plotly paper colors `rgb(29, 33, 38)` / `rgb(244, 245, 247)` and are
  updated to the new `--card` values; `e2e/KAN-167-design-tokens-theme-breakpoints.spec.ts`
  reads `--color-bg/primary/primary-hover` and is moved to the new role names when the
  FR4 aliases are removed (FR16). The breakpoint probe in KAN-167 passes with Tailwind
  defaults (450 / 700 / 900 / 1024 px). Theme-toggle specs (`hello-world.spec.ts`,
  KAN-167, KAN-173) pass unchanged thanks to K-10.
- Mapped device colors beyond five devices cycle with a dash/symbol variant (K-7);
  markers and chart traces use the same index.
- Tiles unreachable: `bg-media` frame, markers still drawn, inline notice (EC3).
- Showcase snapshots that move because of an FR1 kit change are updated in the same
  story.

## 13. Metrics / KPIs

| Metric | Audit 2026-10-28 | Target |
|---|---|---|
| Conforming pages | 0/9 (0%) | 9/9 (100%) for structure; F-45 home content and F-49 copy excepted |
| Major findings open | 41 | 0 in scope (F-37 confirm part out) |
| Raw interactive elements outside `src/ui` | 48 | 0 |
| Inline `style` outside `src/ui` | 194 | 0 |
| Hard-coded colors in `src/` | 36 | 0 |
| Guideline lint violations | not run (no config) | 0, blocking |

## 14. Risks & Mitigations

- **React 19 / react-leaflet 5 upgrade (FR3) breaks existing code or tests.** It is
  its own area with no visual change, merged before any kit port; the full suite runs
  on it alone.
- **Kit inputs without `forwardRef` under React 18.** Mitigated by FR3 landing before
  FR4; no form is migrated before React 19.
- **Wide blast radius of the token swap (every page changes color at once).** The
  `--color-*` aliases keep unmigrated components readable; the swap is one change,
  reviewed with before/after screenshots of all 9 routes in both themes.
- **E2E coupled to implementation details** (exact colors, token names, map class,
  single-click theme toggle). Listed in §12; each adjustment is named and limited.
- **Reference and app drift during the port.** NFR5: kit copied unmodified; fixes go
  upstream (rule 5).
- **CSP fix depends on the deployment's header source.** FR5 changes the plugin and
  its test; the deployed response headers are checked once after release.

## 15. Acceptance Criteria (Definition of Done)

- [ ] AC1. Every finding in §5 is closed, with its ID cited in the closing story.
- [ ] AC2. FR1 kit items are in the reference showcase and FR2 updates the guideline doc.
- [ ] AC3. The metrics in §13 reach target on a re-run of the audit's static pass and a
      runtime pass (1440 / 390, dark / light) on all 9 routes.
- [ ] AC4. In a production build, both maps load OpenStreetMap tiles, and blocking
      tiles shows the inline notice with markers still drawn.
- [ ] AC5. All existing unit and E2E tests pass, with only the §12 spec edits.
- [ ] AC6. All FRs and NFRs validated.

## 16. Future Extensions (Optional)

- F-37 confirmation dialogs and transfer-as-dialog, after the product decision.
- F-45 home page as PAT-DASH overview (`ref:src/showcase/patterns/Overview.tsx`).
- F-49 copy in one language once X-COPY is decided.
- Route move to `/app/*` if decided.
- Next audit with a runtime crawl and screenshots in `ui/audits/<date>/shots/`.
