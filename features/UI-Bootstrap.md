# Feature: UI Bootstrap — cpaltracker

## 1. Feature Name

UI Bootstrap — cpaltracker: tokens, kit, page patterns, showcase and visual guideline,
delivered as a standalone **reference UI** (`cpal.global/cpaltracker.ui-reference`).

## 2. Problem / Pain Point

cpaltracker-react has no shared visual system. Each screen is styled on its own:
colors, spacing and components drift, dark mode breaks piece by piece, and reviewers
have nothing to check a screen against. The brand (Confluence design reference) is
written for slide decks and several of its colors fail WCAG AA in an app.

## 3. Goal / Outcome (Success Definition)

A runnable reference UI — the "bought template" — in which every token, kit component
and page pattern of cpaltracker is visible, themed in light and dark, built on the
same stack as cpaltracker-react, and documented in one guideline. It is the yardstick
the audit of cpaltracker-react (and every later frontend change) is measured against.

## 4. User Stories (Primary Scenarios)

- As a developer, I want a kit component and a page pattern for each common need,
  so that a new page looks right without design decisions of my own.
- As the product owner, I want a showcase of every component and page pattern, so
  that I can approve the look before it spreads through the app.
- As a user, I want light and dark themes that both work everywhere, so that I can
  use the app comfortably in any lighting.
- As a reviewer, I want written rules and lint checks, so that a deviation is caught
  before merge.

## 5. In Scope (What WILL be delivered)

- Tokens for light and dark (§10) in one file, mapped into Tailwind, with a
  light/dark/system theme switch.
- The kit items listed in FR3–FR6, generated from shadcn/ui (Radix) and themed.
- The page patterns listed in FR7, each reproducing its reference (§7), with mock data.
- A showcase (the reference app itself) covering foundations, components and patterns.
- The guideline doc at `cpal.docs/engineering/ui-guidelines.md`.
- Lint rules, the Review Gate item and Playwright showcase snapshots.

## 6. Out of Scope (What WILL NOT be delivered)

- Product behavior, API calls, authentication: patterns use mock data
  (`src/mocks/fleet.ts`) only.
- Changing cpaltracker-react: that is the remediation feature produced by the audit.
- ReUI code (D10), MapLibre, Recharts: maps are Leaflet (D8), charts Plotly (D11).
- Native mobile theming; a design-token JSON source.
- Opt items not selected: NAV-STEPPER, DAT-KANBAN, DAT-TREE, FRM-RICHTEXT, FRM-UPLOAD
  (use FRM-INPUT / DAT-LIST instead).

## 7. User Flow / UX Breakdown

The reference app has one shell with two sidebar sections: **Application** (the
product patterns, French nav labels and `data-testid`s as cpaltracker-react) and
**Guideline** (foundations, components, charts, map, states). `/login` is outside the
shell. A toolbar in the shell switches theme (light / dark / system) and viewport
(desktop / 390 px mobile frame).

Pattern-to-reference mapping (folder: `cpal.docs/ui/reference/`):

| Pattern | Route | Reference(s) | Intended differences |
|---|---|---|---|
| NAV-SHELL | all | `directions/newrelic/` (sidebar, page header) | one sidebar only, collapsible to an icon rail (D1); brand colors |
| NAV-HEADER | all | `directions/newrelic/` | breadcrumb, title, status chips, time picker top right, filter chips; brand eyebrow |
| PAT-AUTH | `/login`, `/forgot-password` | `directions/login/PAT-AUTH__login-datadog.png` | flat background, no SSO / region selector, c-PAL logo per EC6 (orange on dark, dark on light) |
| PAT-HOME (fleet overview) | `/app/overview` | `directions/newrelic/` (home, health grid) | glyph grid by device state, "what's changed" feed |
| PAT-SPLIT (live tracking) | `/app/live` | `directions/reui-map/PAT-SPLIT__dispatch-map-reui.png` | Leaflet + OSM tiles; token-styled pill markers; Plotly donut; Map/List toggle |
| PAT-DASH (history) | `/app/history` | `directions/newrelic/` (dashboards) | Georgia KPI numbers, threshold bands 2–8 °C, brand chart palette |
| PAT-LIST | `/app/devices` | `directions/newrelic/` (entity tables) | status dot first, mono right-aligned numbers, row ⋯ menu |
| PAT-DETAIL | `/app/devices/:id` | `directions/newrelic/` (entity detail) | charts main area, fixed right column (activity, context) |
| PAT-FORM | `/app/settings` + transfer dialog | `directions/newrelic/` `18-31-33` (dialog), `18-31-54` (page form) | bordered inputs (3:1), page form ≤ 800 px, dialog in 2 columns |
| PAT-STATES | `/app/states`, `*` | `directions/newrelic/` (empty states) | brand tone, no illustrations |

Evidence and conflict choices: `cpal.docs/ui/analysis/2026-10-28-direction-newrelic.md`
and `cpal.docs/ui/reference/reference.md` (decisions D1–D11).

## 8. Functional Requirements (FRs)

- FR1. Tokens SHALL be defined once, in `src/styles/tokens.css`, for every role in
  §10, with light and dark values, and exposed as Tailwind utilities via
  `@theme inline` in `src/index.css`. No color value is defined anywhere else.
- FR2. The theme SHALL switch with one attribute on `<html>` (`data-theme`), between
  light, dark and system (following `prefers-color-scheme`), default dark (D7), and
  persist the choice under the key `cpaltracker:theme`.
- FR3. The kit SHALL provide the primitives: CMP-BUTTON (primary, secondary, outline,
  ghost, destructive, link; sm/md/lg/icon; loading, disabled), CMP-ICONBUTTON,
  CMP-BADGE (neutral, status ×4, brand pill), CMP-AVATAR, CMP-KBD, CMP-TOGGLE,
  CMP-BUTTONGROUP, CMP-TOOLTIP, CMP-SEPARATOR.
- FR4. The kit SHALL provide the form items: FRM-FIELD (label, help, error, ARIA
  wiring, react-hook-form + zod), FRM-INPUT (with prefix/suffix group), FRM-TEXTAREA,
  FRM-SELECT, FRM-COMBOBOX, FRM-CHECKBOX, FRM-RADIO, FRM-SWITCH, FRM-SLIDER,
  FRM-DATE (date + time as adjacent fields), FRM-DATERANGE, FRM-OTP, FRM-SEARCH.
- FR5. The kit SHALL provide the overlays and feedback items: OVL-DIALOG,
  OVL-ALERTDIALOG (confirm destructive), OVL-SHEET, OVL-POPOVER, OVL-MENU (dropdown,
  row ⋯ menu), OVL-HOVERCARD, OVL-COMMAND (⌘K palette), FBK-TOAST, FBK-ALERT (info,
  success, warning, danger, brand "note" quote-box), FBK-PROGRESS, FBK-SPINNER,
  FBK-SKELETON, FBK-EMPTY.
- FR6. The kit SHALL provide the data-display and navigation items: DAT-CARD
  (plain, orange left border, orange top border), DAT-WIDGET (title, period subtitle,
  ⋯ menu, empty state inside), DAT-STAT (Georgia number, label, trend arrow + %),
  DAT-STATUS (dot + label, the four states of D3), DAT-TABLE (TanStack: sort, select,
  status dot first, mono right-aligned numbers, row menu, pagination), DAT-LIST
  (incl. the brand numbered list), DAT-TIMELINE, DAT-HEALTHGRID, DAT-CHART (FR12),
  DAT-MAP (FR13), NAV-SIDEBAR (collapsible to icon rail, active accent bar),
  NAV-HEADER (eyebrow, breadcrumb, title, chips, actions), NAV-TABS,
  NAV-BREADCRUMB, NAV-PAGINATION, NAV-TIMEPICKER (presets + prev/next arrows),
  NAV-FILTERBAR (removable `attribute = value` chips), NAV-ACCORDION.
- FR7. The patterns SHALL be provided as pages with mock data, each defining its
  mobile behavior: NAV-SHELL, PAT-AUTH, PAT-HOME, PAT-SPLIT, PAT-DASH, PAT-LIST,
  PAT-DETAIL, PAT-FORM, PAT-STATES (§7 routes).
- FR8. The showcase SHALL render every token (swatch, value, and the computed
  contrast against the surfaces it sits on), every kit item in every variant and
  state, and every pattern, with global theme and viewport toggles, and an index page
  linking every entry with its inventory ID.
- FR9. The guideline doc SHALL state, for each common need, which kit item or
  pattern to use, with do/don't rules, and SHALL cover X-COLOR (orange signals, never
  decorates; status colors only for state), X-TYPE (Georgia only for big numbers),
  X-DENSITY (D5), X-LINKS (D2), X-COPY (language, open), X-FORMAT (dates, numbers,
  °C), X-A11Y, X-MOTION, X-ICONS (lucide only), X-LOGO.
- FR10. Lint rules SHALL fail the lint script for raw interactive elements, direct
  `radix-ui`/`@radix-ui/*` imports, inline `style` and arbitrary length Tailwind
  values outside `src/ui` and `src/styles`, and for hard-coded color literals and
  arbitrary color Tailwind values anywhere outside `src/styles/tokens.css` (FR1);
  the Plotly and Leaflet token bridges (FR12, FR13) read colors from the CSS
  variables and hold no color literal.
- FR11. The Review Gate (`cpal.docs/engineering/code-standards.md`) SHALL include the
  UI item, linking the guideline.
- FR12. Plotly chart colors (series, axis, grid, font, threshold bands) SHALL be read
  from the tokens at runtime and update on theme change; threshold bands as Plotly
  shapes; donut as a pie with a 0.62 hole and a Georgia center number.
- FR13. Maps SHALL use Leaflet 1.9.4 + react-leaflet 5 with
  `tile.openstreetmap.org` tiles, marker clustering (react-leaflet-cluster), token-styled
  `divIcon` pill markers (icon + device ID, state-colored ring), zoom/locate/layers
  controls top right, and the dark basemap through the `--map-tile-filter` token.

## 9. Non-Functional Requirements (NFRs)

- NFR1. WCAG 2.1 AA in both themes: text contrast ≥ 4.5:1 (≥ 3:1 for large text, UI
  boundaries and chart marks), visible keyboard focus, every kit item keyboard-operable.
- NFR2. Desktop is the primary target at ≥ 1280 px; every pattern stays usable at
  390 px.
- NFR3. Stack as cpaltracker-react: React 19, TypeScript, Vite 7, Tailwind 4,
  shadcn/ui on Radix, lucide-react, react-router 7; exact versions pinned in
  `package.json`.
- NFR4. A Playwright snapshot of every showcase route, both themes, desktop and
  mobile, runs with `npm run test:e2e`.
- NFR5. `npm run typecheck`, `npm run lint` and `npm run build` pass with zero errors.

## 10. Data Inputs & Outputs

Token values — hex, kept identical to `cpal.docs/ui/reference/brand.md` so each value
traces to the brand; "why changed" for every non-brand value is a comment in
`tokens.css`.

| Role | Dark (default) | Light | Notes |
|---|---|---|---|
| background | #1A1D22 | #F5F2ED | brand |
| foreground (titles) | #F3F2EE | #1E2128 | brand |
| body | #AAACB2 | #55585F | brand |
| card | #22252E | #EEE6D4 | light: public site cream card |
| popover | #22252E | #FFFDF8 | |
| secondary / accent (hover) | #2C2F38 | #ECE7DD | light: #E6DFD1 left muted text at 4.33:1 |
| muted (inset) | #24262C | #ECE6DA | |
| muted-foreground | #96989D | #64666B | brand #83858B fails 4.5:1 on cards |
| media frame | #0E0F12 | #1A1D22 | map frame, screenshots |
| border (separator) | #353841 | #D9D3C9 | decorative |
| input (control boundary) | #686E7F | #938267 | 3:1 |
| ring (focus) | #DD8C3E | #85511F | reuses primary-text |
| primary (fills) | #DD8C3E | #C57A30 | brand |
| primary-foreground | #1A1D22 | #1A1D22 | white fails on orange |
| primary-text (orange as text) | #DD8C3E | #85511F | light brand orange 3.03:1; #915923 failed over the soft fill (4.14:1) |
| danger / warning / success / offline / info | #F39083 / #E3C34F / #5DBE8A / #A8ABB0 / #86AFDA | #A23126 / #705800 / #276742 / #585A60 / #2B5C8A | D3; ≥ 4.6:1 as text on background, card, secondary and on their own 12 % soft fill (chips) |
| chart-1…5 | #DD8C3E #6FA8C7 #93B79C #B4A2D6 #676E80 | #A8692A #3A6E8A #4A7456 #6D5C99 #5F656F | D4; ≥ 3:1 on background and card |
| chart-axis / chart-grid | #96989D / #2C2F38 | #64666B / #DDD5C6 | |
| sidebar | #20232B | #EFEAE1 | |

| Scale | Values |
|---|---|
| Font family | Arial (sans, everything); Georgia (big numbers only, D6); system mono (IDs) |
| Type scale | Tailwind steps: 12 / 14 (body, dense D5) / 16 / 18 / 20 / 24 / 30 / 48 (Georgia stat) px — no arbitrary sizes (FR10) |
| Spacing | Tailwind 4 px scale; dense data views (D5), brand spacing in headers, empty states, login |
| Radius base | 8 px (`--radius: 0.5rem`) |
| Breakpoints | Tailwind defaults; desktop ≥ 1280, mobile 390 |

## 11. Integration Points

- `cpal.global/cpaltracker.ui-reference` (new, local repo): `src/styles/tokens.css`,
  `src/index.css`, `src/ui/`, `src/ui/patterns/`, `src/showcase/`,
  `eslint.config.js`, `e2e/`.
- `cpal.docs`: `engineering/ui-guidelines.md`, `engineering/code-standards.md`.
- Later: the audit of `cpaltracker-react` uses this app as its yardstick.

## 11a. Software Items

Not applicable: cpal has no software-items registry bound.

## 12. Edge Cases & Constraints

- EC1. A token change moves showcase snapshots; their baselines are updated in the same change.
- EC2. Plotly (SVG with inline colors) and Leaflet tiles can't read CSS variables:
  bridged at runtime (FR12) and by the tile filter token (FR13).
- EC3. OSM tiles unreachable (offline, CSP): the map frame shows the media background
  and markers still render; an inline notice says tiles are unavailable.
- EC4. A device with no reading (offline) shows "—" and the grey state, never 0 °C.
- EC5. Long device names and site names truncate with a tooltip; IDs never truncate.
- EC6. Logo: canonical files only, never redrawn; orange on dark, dark on light.
- EC7. Orange is never used for a status (warning is yellow, D3).

## 13. Metrics / KPIs

- 100% of the inventory items in FR3–FR7 present in the showcase.
- 0 lint violations.
- Every token pair shown on Foundations passes its contrast threshold in both themes.

## 14. Risks & Mitigations

- Sampled reference colors fail contrast → replaced by brand app values (§10), each change commented.
- Parallel stories editing shared files → the lead owns `App.tsx`, `nav.ts`,
  `tokens.css`, `index.css`, `mocks/`, `package.json`; each story owns its listed files.

## 15. Acceptance Criteria (Definition of Done)

- [ ] AC1. Every role in §10 is defined once in `tokens.css` and used through Tailwind utilities.
- [ ] AC2. The theme switch works in light, dark and system modes, persists, and dark is the default.
- [ ] AC3. Every kit item in FR3–FR6 is in the showcase in every variant and state.
- [ ] AC4. Every pattern in FR7 is in the showcase at its route; the product owner has
      compared it with its reference (§7) in both themes, desktop and mobile, and approved it.
- [ ] AC5. The guideline doc exists and covers every kit item, pattern and X rule.
- [ ] AC6. Linting a fixture file with a deliberate violation of each FR10 rule reports
      every rule (the fixture is excluded from the default `npm run lint` scope), and
      `npm run lint` passes on `src/` (NFR5).
- [ ] AC7. The Review Gate item is in `code-standards.md`.
- [ ] AC8. Showcase snapshot tests exist and pass.
- [ ] AC9. Charts and map follow the theme on toggle without reload.
- [ ] AC10. All FRs and NFRs validated; typecheck, lint and build pass.

## 16. Future Extensions (Optional)

- Deferred Opt items (§6).
- Design-token JSON source for a native app.
- Publishing the kit as a package shared by cpaltracker-react.
