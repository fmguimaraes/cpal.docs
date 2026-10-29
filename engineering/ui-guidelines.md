# UI guidelines

The rules every screen of the c-PAL tracker UI follows. The kit is in
`src/ui`, the page patterns in `src/ui/patterns`, the tokens in
`src/styles/tokens.css`. The reference app is `cpaltracker.ui-reference` (`npm run dev`):
its showcase routes (`/guideline/*`) show every item, its pattern routes (`/app/*`,
`/login`) are the yardstick. If your screen looks or behaves differently from the
reference pattern it is based on, the screen is wrong, or the reference needs a
change first (rule 5). This doc decides; the showcase shows.

Paths below are relative to `cpaltracker.ui-reference/`. The kit is imported as
`@/ui/<file>`, for example `import { Button } from '@/ui/button'`. There is no barrel.

## Rules that always apply

1. Use kit components and page patterns. Never a raw `<button>`, `<input>`,
   `<select>`, `<textarea>`, `<table>` or `<dialog>` outside `src/ui`.
2. Colors, radius, shadows and type come from tokens, through Tailwind utilities
   (`bg-card`, `text-muted-foreground`, `p-4`). No hex/rgb/hsl/oklch literal, no inline
   `style`, no arbitrary values (`p-[13px]`, `bg-[#fff]`, `text-[var(--x)]`).
3. Never import `radix-ui` or `@radix-ui/*` outside `src/ui`. Import the kit file.
4. Every screen works in dark (default) and light, at 1440 px and at 390 px. Check
   both before asking for review.
5. A missing component is added to the kit and the showcase first (own change), then used.
6. Every data view has loading, empty and error states (see Page patterns, `/app/states`).
7. Lint enforces rules 1-3 (`npm run lint`); it is a blocking gate, see Enforcement.

## Cross-cutting rules (X-*)

- **X-COLOR.** Orange (`primary`, `text-primary-text`, `bg-primary-soft`) signals and
  never decorates: the primary action (one per page), the active nav item, key numbers
  and the eyebrow. Status colors (`success`, `warning`, `danger`, `offline`, `info`)
  mean state only, never branding. Warning is **yellow**, never orange. Red is for
  alerts and destructive actions. Text uses `text-foreground` (titles), `text-body`
  (running text), `text-muted-foreground` (secondary). Never signal state with color
  alone: pair it with an icon or text (`StatusBadge` does).
- **X-TYPE.** Sans for everything. Georgia (`font-serif`) only for big numbers: KPI
  values (`Stat`) and the donut center. Tables stay sans with tabular figures
  (`tabular-nums`). Headings come from `PageHeader` and `Card*`, do not restyle them.
- **X-DENSITY (D5).** Dense in data views (tables, dashboards, lists: `DataTable`
  density `compact`/`comfortable`, `WidgetCard`); the brand's generous spacing in page
  headers, empty states and login. Page padding and section gaps come from the pattern.
- **X-LINKS (D2).** Links are text-colored and underlined. Orange links are forbidden;
  orange is for the primary action, active nav and key numbers.
- **X-COPY (decided 2026-10-29).** No page is translated: each page keeps the language
  it has in the monolith (`cpaltracker.web`). A restyle moves components, never words.
  The kit ships no default copy: every label, placeholder and message is passed in by
  the page, so a French page stays French and an English page stays English. The
  reference app's own copy (nav in French, login in English) mirrors the monolith and is
  illustrative only. For *new* copy, follow the language of the screen it lands on; one
  language per screen, sentence case, verb-first button labels, errors say what happened
  and what to do.
- **X-FORMAT.** Dates and times through `date-fns`, never hand-built strings; 24 h
  clock; temperatures in °C with one decimal and a space-free unit (`21.4°C`); numbers
  with thousands separators; device ids as stored (`CP-0412`). Relative time ("5 min
  ago") always has the absolute time in a tooltip or `title`.
- **X-A11Y.** WCAG 2.1 AA in both themes (token pairs are checked in `/guideline/foundations`).
  Visible focus (do not remove `focus-visible` rings). Every control has a label;
  icon-only buttons have `aria-label`. Everything is keyboard operable. Charts and the
  map carry a required `label` / accessible name. State is never color-only.
  `data-testid` on nav items is kept from the monolith (`nav-home`, ...); target controls
  in E2E by role and accessible name first.
- **X-MOTION.** Short, functional transitions only (hover, open/close). No decorative
  animation. Honor `prefers-reduced-motion`; charts animate nothing.
- **X-ICONS.** `lucide-react` only, sized by the component (`size-4` in controls). No
  emoji, no second icon set, no decorative imagery.
- **X-LOGO.** The logo appears in the sidebar header and the auth pages, from
  `public/`, unmodified, on a surface that meets the brand contrast. Never recolored
  with a token or placed on a busy background.

## Tokens (`src/styles/tokens.css`, the only place a value is defined)

Theme is `html[data-theme="dark" | "light"]` (`useTheme()` from `@/app/theme`; dark by
default, light and system available). Utilities are mapped in `src/index.css`.

| Role | Utility | Use |
|---|---|---|
| Page / card / popover | `bg-background`, `bg-card`, `bg-popover` | Surfaces, from back to front |
| Inset / hover surface | `bg-muted`, `bg-secondary`, `bg-accent` | Insets, side panels, hovered or selected rows |
| Media | `bg-media` | Map frame, screenshots |
| Text | `text-foreground`, `text-body`, `text-muted-foreground` | Titles, running text, secondary |
| Brand | `bg-primary` + `text-primary-foreground`, `text-primary-text`, `bg-primary-soft` | Fill, orange text, soft highlight |
| State | `text-success`, `text-warning`, `text-danger`, `text-offline`, `text-info` and `bg-*` / `bg-*-soft` | Status only |
| Lines | `border-border` (decorative), `border-input` (control boundary, 3:1) | |
| Focus | `ring-ring` | Focus ring |
| Sidebar | `bg-sidebar`, `text-sidebar-foreground`, `sidebar-accent`, ... | Shell only |
| Charts | `useChartTheme()` (from `--chart-*`) | Never read colors elsewhere |
| Elevation | `shadow-overlay`, `shadow-media` | Overlays, media |
| Map | `--map-tile-filter` | Dark basemap (used by `FleetMap` only) |
| Type | `font-sans`, `font-serif` (big numbers only) | |

Spacing uses the Tailwind scale; radius comes from the one base (`rounded-md`, `rounded-lg`).
To add or change a value, edit `tokens.css` (comment why, with the contrast ratio) in a
dedicated change, then show it in Foundations.

## Choosing a component

Inventory IDs are explained in the Inventory section.

| Need | Use (import) | Not |
|---|---|---|
| Main action on a page (CMP-BUTTON) | `Button` (`@/ui/button`), in `PageHeader` `actions`, one per page | Several primary buttons; raw `<button>` |
| Secondary / tertiary action | `Button variant="outline"` / `"ghost"` | A second orange button |
| Icon-only action | `Button size="icon"` with `aria-label` (+ `Tooltip`) | An unlabeled icon |
| Destructive action (OVL-CONFIRM) | `Button variant="destructive"` + `AlertDialog` (`@/ui/alert-dialog`) | `window.confirm()` |
| Toggle a state / filter set | `Toggle`, `ToggleGroup` (`@/ui/toggle`, `toggle-group`) | Buttons with manual pressed styling |
| Related buttons | `ButtonGroup` (`@/ui/button-group`) | Hand-joined borders |
| Device state (CMP-BADGE) | `StatusBadge` / `StatusDot` (`@/ui/status`) | Hand-colored `Badge` |
| Tag / count | `Badge` (`@/ui/badge`) | A styled `<span>` |
| Avatar, separator, keyboard hint | `Avatar`, `Separator`, `Kbd` | |
| Tooltip | `Tooltip` (`@/ui/tooltip`, provider already in `App`) | `title` for important info |
| Wait: content / action (FBK-LOADING) | `Skeleton` for content, `Spinner` for actions | Spinner for whole pages |
| Form field (FRM-FIELD) | `Form*` (`@/ui/form`, react-hook-form + zod) or `Field*` (`@/ui/field`); `Label` | Placeholder as label |
| Text input (FRM-INPUT) | `Input`, `Textarea`, `InputGroup`, `SearchInput`, `InputOTP` | Raw `<input>` |
| Pick one, <= 7 options (FRM-SELECT) | `Select` (`@/ui/select`); `NativeSelect` for mobile-first/simple | |
| Pick one, many or searchable (FRM-COMBOBOX) | `Combobox` (`@/ui/combobox`) | `Select` with a long list |
| Boolean, one of few, on/off (FRM-CHECKBOX/RADIO/SWITCH) | `Checkbox`, `RadioGroup`, `Switch` | Checkbox for instant settings (use `Switch`) |
| Number range (FRM-SLIDER) | `Slider` (`@/ui/slider`) with label and value | |
| Date / date-time / range (FRM-DATE) | `DateTimeField`, `DateRangePicker`, `Calendar` | Raw `type="date"` |
| Time window for data | `TimePicker` (`@/ui/patterns/time-picker`; `TimeRangeValue`, `resolveRange`) | Ad hoc preset buttons |
| Filter chips on a list | `FilterBar` (`@/ui/patterns/filter-bar`) | Per-page chip markup |
| Short task in context (OVL-DIALOG) | `Dialog` (`@/ui/dialog`) | A new page |
| Side task, list still visible (OVL-SHEET) | `Sheet` (`@/ui/sheet`) | A dialog over the list |
| Contextual content / menu (OVL-POPOVER, OVL-MENU) | `Popover`, `DropdownMenu`, `HoverCard` | |
| Command palette (OVL-COMMAND) | `Command`, `CommandDialog` (`@/ui/command`) | |
| Outcome of an action (FBK-TOAST) | `toast()` from `sonner` (`Toaster` is in `App`) | An alert at the page top |
| Persistent problem (FBK-ALERT) | `Alert` variants `default/info/success/warning/danger/note` (`@/ui/alert`) | A toast |
| Nothing to show (FBK-EMPTY) | `Empty*` (`@/ui/empty`), with a next step | Blank area |
| Failure (FBK-ERROR) | `Alert variant="danger"` inline or `Empty` for a section; retry action | Raw error text |
| Progress | `Progress` (`@/ui/progress`) | |
| Card (DAT-CARD) | `Card*` (`@/ui/card`) | A bordered `div` |
| Dashboard tile with menu and period | `WidgetCard` (`@/ui/widget-card`) | |
| Key number (DAT-STAT) | `Stat` (`@/ui/stat`; `goodDirection`, `change`, `accent` for the one orange KPI) | Hand-built KPI |
| List rows | `Item*` (`@/ui/item`), `NumberedList` | |
| Tabular data (DAT-TABLE) | `DataTable` (`@/ui/data-table`; sorting, selection, row menu, density, pagination); `Table*` only for static tables | Raw `<table>` |
| Timeline / feed (DAT-TIMELINE) | `Timeline`, `ActivityTimeline` (`@/ui/timeline`) | |
| Fleet overview grid | `HealthGrid` (`@/ui/health-grid`) | |
| Chart (DAT-CHART) | `TimeSeriesChart`, `DonutChart`, `RankedBars`, `BarChart`, `Sparkline` from `@/ui/charts`; `label` is required | Plotly directly |
| Map (DAT-MAP) | `FleetMap` from `@/ui/map` | Leaflet directly |
| Navigation shell (NAV-SHELL) | `AppShell` (`@/ui/patterns/app-shell`); the app passes `sections` (paths + `testId`s), `userMenu` (account link, sign out), `logo` (client-logo slot) and optional copy and hooks (`skipToContentLabel`, `navigationLabel`, `sidebarTriggerLabel`, `themeLabels`, `sidebarTestId`, `navTestId`; `userMenu.testId`, `userMenu.signOut` = `{ label, onSelect, pending, testId }` for an async sign-out, omitted when signed out); it renders the skip-to-content link and the single `<main id="main-content">` | A per-page layout, a second `<main>`, a nav hard-coded in the kit |
| Page header (NAV-HEADER) | `PageHeader` + `Eyebrow` (`@/ui/patterns/page-header`, `eyebrow`) | Hand-built titles |
| Breadcrumb, tabs, pagination | `Breadcrumb*`, `Tabs*`, `Pagination*`; `PageHeader` `breadcrumb` prop | |
| Theme / viewport switch | `ThemeToggle` (`data-testid="theme-toggle"`: one click flips the resolved theme light/dark; `data-testid="theme-system"` next to it returns to system), `ViewportToggle` (`useViewport()`) | A group that needs two clicks to flip |
| Device status type (K-1) | `DeviceStatus`, `StatusDevice`, `statusLabel`, `countByStatus` from `@/ui/status`: the kit owns the type; the app maps its own data to it | A status union redefined per app or per page |
| Typed date entry (K-5) | `TypedDateField` + `parseTypedDate` (`@/ui/typed-date-field`): strict `YYYY-MM-DD` text field with `min`/`max` and overridable `TypedDateMessages` (app language) | Raw `type="date"`; a second parser |
| Long table (K-6) | `DataTable` virtualizes rows above `virtualizeThreshold` (default 100): fixed row height, cells truncate, no pagination in that mode | Paginating or rendering thousands of DOM rows |
| More than 5 series (K-7) | `seriesStyle(i)` / `useChartTheme` (`@/ui/charts`): colour cycles `--chart-1..5`, each further pass changes dash and marker; `FleetMap` markers use the same index (`MapDevice.series`) | A sixth colour; colour as the only cue |
| Track playback (K-8) | `TrackPlayback` + `useTrackPlayback(points)` (`@/ui/map`): instant slider, step/play/pause, trail switch, fit-to-track, as the companion of `FleetMap` | A custom player outside the kit |
| Theme before first paint | Inline script in `index.html` reads `cpaltracker:theme` (`light`/`dark`/`system`, default dark) and sets `html[data-theme]`; `ThemeProvider` (`@/app/theme`) keeps it in sync | Setting the theme only after React mounts (flash of the wrong theme) |
| Disclosure | `Accordion`, `Collapsible` | |
| Scroll region | `ScrollArea` | |

## Page patterns

All are working pages with mock data (`src/mocks/fleet.ts`) in `src/showcase/patterns`.
Every page sits in `AppShell` (sidebar, collapsible to an icon rail, drawer on mobile)
and starts with `PageHeader`.

| ID | Route | Use for | Mobile (390 px) |
|---|---|---|---|
| PAT-AUTH | `/login`, `/forgot-password` | Auth pages, no shell | Single column |
| PAT-HOME | `/` | Reference index | Cards stack |
| PAT-DASH | `/app/overview`, `/app/history` | KPI row (`Stat`), charts and lists in `WidgetCard`, `TimePicker` in the header; `/app/history` adds a `FilterBar` and the typed date entry | Tiles stack one per row |
| PAT-SPLIT | `/app/live` | Map + panel (`FleetMap`, selection synced with the list) | Panel below the map |
| PAT-LIST | `/app/devices` | `PageHeader`, `FilterBar`/search, `DataTable`, pagination | Table scrolls horizontally; filters wrap |
| PAT-DETAIL | `/app/devices/:id` | Header with breadcrumb and actions, sections, charts | Sections stack |
| PAT-FORM / PAT-SETTINGS | `/app/settings` | Form sections, one action bar | Single column |
| PAT-STATES / PAT-ERROR | `/app/states`, `*` | Loading, empty, error, 404 | |

Every data view: loading = `Skeleton` shaped like the content; empty = `Empty` with the
reason and next step; error = inline `Alert variant="danger"` with a retry.

## Forms

Use `Form*` (react-hook-form + zod through the kit's `zodResolver`) or `Field*`. One
column by default, labels above fields, required fields marked in the label. Errors
show on blur and on submit, under the field (`FormMessage`/`FieldError`), and the
control gets `aria-invalid`. The action bar sits at the bottom: primary right, cancel
left of it. Destructive actions are separate and confirmed by `AlertDialog`. Warn
before leaving a dirty form.

## Accessibility

See X-A11Y. Keyboard paths: tab order follows visual order; `Dialog`/`Sheet` trap and
restore focus; menus and the command palette (Ctrl/Cmd+K) work with arrow keys and
Enter; map markers are focusable and select with Enter/Space; charts expose a
text name through `label`.

## Enforcement

- `npm run lint` (`eslint.config.js`) fails on: raw interactive elements, `radix-ui` /
  `@radix-ui/*` imports, inline `style` and arbitrary Tailwind lengths outside
  `src/ui` and `src/styles`; hex/rgb/hsl/oklch literals and arbitrary Tailwind colors
  anywhere in `src/`. Plus `jsx-a11y` recommended and `react-hooks`.
- `npm run lint:fixtures` lints `lint-fixtures/violations.tsx` (excluded from `lint`)
  and fails unless every rule above reports.
- `npm run test:e2e` runs the showcase snapshots: every route, dark and light, desktop
  1440 and mobile 390 (`e2e/showcase.spec.ts`; map tiles masked). A kit or token change
  that moves a snapshot updates the baseline in the same change (`npx playwright test
  --update-snapshots`).
- The Review Gate in [code-standards.md](code-standards.md) has the UI item.
- Chart and map colors come from the tokens at runtime (`useChartTheme()`, CSS
  variables); the bridges in `src/ui/charts` and `src/ui/map` hold no color literal.

## Inventory

| Prefix | Meaning | Where |
|---|---|---|
| TOK-* | Tokens: color, chart, type, space, radius, elevation, theme | `src/styles/tokens.css`, `/guideline/foundations` |
| CMP-* | Primitives: button, badge, avatar, tooltip, separator, spinner, skeleton, icon, kbd | `/guideline/components/actions-forms` |
| FRM-* | Forms: field, input, textarea, select, combobox, checkbox, radio, switch, slider, date | `/guideline/components/actions-forms` |
| OVL-* | Overlays: dialog, confirm, sheet, popover, menu, command | `/guideline/components/overlays-feedback` |
| FBK-* | Feedback: toast, alert, empty, error, loading, progress | `/guideline/components/overlays-feedback` |
| DAT-* | Data display: table, card, list, stat, timeline, chart, map | `/guideline/components/data-navigation`, `/guideline/charts`, `/guideline/map` |
| NAV-* | Shell, sidebar, top bar, page header, breadcrumb, tabs, pagination | `src/ui/patterns` |
| PAT-* | Page patterns (table above) | `/app/*`, `/login` |
| X-* | Cross-cutting rules (above) | this doc |
| SHW-* | Showcase sections and snapshots | `src/showcase`, `e2e/` |

## Exceptions

| Where | Why | Until |
|---|---|---|
| `src/ui/charts`, `src/ui/map` | Runtime bridges may read CSS variables and set inline styles (Plotly/Leaflet need computed values). They hold no color literal. | Permanent |
| `src/ui/**` | Layout lengths such as `w-[390px]` are tolerated inside the kit; the arbitrary-length rule is not applied there. | Permanent |
| `src/showcase/lib/contrast.ts` | Computes contrast from rendered tokens and needs a canvas sentinel color. Lint must pass; use a scoped, reasoned disable if the sentinel is flagged. | Permanent |
