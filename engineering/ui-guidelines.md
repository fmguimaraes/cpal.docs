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

- **X-COLOR.** Color belongs to the data and the status, not to the chrome
  ([`theme.md`](../ui/reference/theme.md)). The accent `--signal` (`bg-signal`,
  `ring-signal`, `border-signal`) only marks selection, active, focus, the primary series,
  map markers and key figures, as listed in theme.md §5. It is never a large fill, body text,
  or a chip or segment fill. Small accent text is `text-primary-text`. The brand cyan
  (`--brand`) is for the logo and rail mark only. Status colors (`success`, `warning`, `danger`, `offline`, `info`)
  mean state only, never branding. Warning is **yellow**; info is **periwinkle**, never
  the brand cyan. Red is for
  alerts and destructive actions. Text uses `text-foreground` (titles), `text-body`
  (running text), `text-muted-foreground` (secondary). Never signal state with color
  alone: pair it with an icon or text (`StatusBadge` does).
- **X-TYPE.** Montserrat (`font-sans`) for everything. Montserrat ExtraBold
  (`font-display font-extrabold`) only for big numbers: KPI values (`Stat`), numbered
  lists, timeline dates and the donut center. Tables stay sans with tabular figures
  (`tabular-nums`). Headings come from `PageHeader` and `Card*`, do not restyle them.
- **X-DENSITY (D5).** Dense in data views (tables, dashboards, lists: `DataTable`
  density `compact`/`comfortable`, `WidgetCard`); the brand's generous spacing in page
  headers, empty states and login. Page padding and section gaps come from the pattern.
- **X-LINKS (D2).** Links are `text-primary-text` and underlined (the `link` button
  variant). Never `text-signal` or the brand cyan.
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
| Text | `text-foreground`, `text-body`, `text-muted-foreground`, `text-faint` | Titles, running text, secondary, placeholders / separators / disabled |
| Primary / accent | `bg-primary` + `text-primary-foreground` (primary button); `bg-signal` + `text-on-signal`, `ring-signal`, `ring-signal-ring` (selection, focus); `text-primary-text` (links, small accent text) | See theme.md §5 for where the accent is allowed |
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
| Secondary / tertiary action | `Button variant="outline"` / `"ghost"` | A second primary button |
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
| Outcome of an action (FBK-TOAST) | `toast.success/info/error()` from `sonner`; one `<Toaster position="top-center" />` in `AppShellLayout` (the account pages keep their own). A form's save result is a toast, never a flash `Alert` in its action bar | An alert at the page top; a flash `Alert` beside the submit button |
| Persistent problem (FBK-ALERT) | `Alert` variants `default/info/success/warning/danger/note` (`@/ui/alert`) | A toast |
| Nothing to show (FBK-EMPTY) | `Empty*` (`@/ui/empty`), with a next step | Blank area |
| Failure (FBK-ERROR) | `Alert variant="danger"` inline or `Empty` for a section; retry action | Raw error text |
| Progress | `Progress` (`@/ui/progress`) | |
| Card (DAT-CARD) | `Card*` (`@/ui/card`) | A bordered `div` |
| Dashboard tile with menu and period | `WidgetCard` (`@/ui/widget-card`) | |
| Key number (DAT-STAT) | `Stat` (`@/ui/stat`; `goodDirection`, `change`, `accent` for the one accent KPI) | Hand-built KPI |
| List rows | `Item*` (`@/ui/item`), `NumberedList` | |
| Tabular data (DAT-TABLE) | `DataTable` (`@/ui/data-table`; sorting, selection, row menu, density, pagination; `labels` for the page language; `rowProps` for the app's own row attributes such as `data-testid`, `aria-selected` or a selection class, applied after the defaults with `className` merged; `containerProps` on the root); `Table*` only for static tables | Raw `<table>`; testids on an inner cell element to work around the row |
| Timeline / feed (DAT-TIMELINE) | `Timeline`, `ActivityTimeline` (`@/ui/timeline`; per item `badge` after the title, e.g. a `Badge` for an ongoing entry, and `testId`; `formatTime` and `label` for the page language) | A dot-and-line list composed by hand |
| Fleet overview grid | `HealthGrid` (`@/ui/health-grid`) | |
| Chart (DAT-CHART) | `TimeSeriesChart`, `DonutChart`, `RankedBars`, `BarChart`, `Sparkline` from `@/ui/charts`; `label` is required | Plotly directly |
| Map (DAT-MAP) | `FleetMap` from `@/ui/map` | Leaflet directly |
| Navigation shell (NAV-SHELL) | `AppShell` (`@/ui/patterns/app-shell`); the app passes `sections` (paths + `testId`s), `userMenu` (account link, sign out), `logo` (client-logo slot) and optional copy and hooks (`skipToContentLabel`, `navigationLabel`, `sidebarTriggerLabel`, `themeLabels`, `sidebarTestId`, `navTestId`; `userMenu.testId`, `userMenu.signOut` = `{ label, onSelect, pending, testId }` for an async sign-out, omitted when signed out); it renders the skip-to-content link and the single `<main id="main-content">` | A per-page layout, a second `<main>`, a nav hard-coded in the kit |
| Page header (NAV-HEADER) | `PageHeader` + `Eyebrow` (`@/ui/patterns/page-header`, `eyebrow`) | Hand-built titles |
| Breadcrumb, tabs, pagination | `Breadcrumb*`, `Tabs*`, `Pagination*`; `PageHeader` `breadcrumb` prop | |
| Theme switch | `ThemeChoiceControl` (`@/ui/patterns/theme-toggle`), a segmented control (clair / sombre / système) inside the user menu, rendered by `AppShell` from `themeLabels`; the top bar carries no theme control (Pulse layout v2, 2026-11-25). The selected segment uses the selected-toggle fill (see "Selected state" below) | A theme button in the top bar; a selected segment styled so it reads as unselected |
| Device status type (K-1) | `DeviceStatus`, `StatusDevice`, `statusLabel`, `countByStatus` from `@/ui/status`: the kit owns the type; the app maps its own data to it | A status union redefined per app or per page |
| Typed date entry (K-5) | `TypedDateField` + `parseTypedDate` (`@/ui/typed-date-field`): strict `YYYY-MM-DD` text field with `min`/`max` and overridable `TypedDateMessages` (app language) | Raw `type="date"`; a second parser |
| Long table (K-6) | `DataTable` virtualizes rows above `virtualizeThreshold` (default 100): fixed row height, cells truncate, no pagination in that mode | Paginating or rendering thousands of DOM rows |
| More than 5 series (K-7) | `seriesStyle(i)` / `useChartTheme` (`@/ui/charts`): colour cycles `--chart-1..5`, each further pass changes dash and marker; `FleetMap` markers use the same index (`MapDevice.series`) | A sixth colour; colour as the only cue |
| Track playback (K-8) | `TrackPlayback` + `useTrackPlayback(points)` (`@/ui/map`): instant slider, step/play/pause, trail switch, fit-to-track, as the companion of `FleetMap` | A custom player outside the kit |
| Theme before first paint | Same-origin classic script `public/theme-init.js`, loaded from the `<head>` of `index.html` with `<script src="/theme-init.js">` (render-blocking, before the app bundle), reads `cpaltracker:theme` (`light`/`dark`/`system`, default light) and sets `html[data-theme]`; `ThemeProvider` (`@/app/theme`) keeps it in sync. The production CSP stays `script-src 'self'` | Setting the theme only after React mounts (flash of the wrong theme); an inline `<script>` in `index.html` (blocked by the CSP), or a CSP hash / `'unsafe-inline'` to let it run |
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

**PAT-DETAIL header/section rhythm (unified shell).** In `cpaltracker-react`, a sub-page
a list row opens onto its own item's information (an anomaly's own view, a device's own
page) follows one rhythm, not a bespoke header per screen:
- Header: `<header className="flex min-w-0 flex-col gap-0.5 pb-3">` — a first row
  (`flex items-center gap-1.5`) with a `Button variant="ghost" size="icon-sm"
  className="-ml-2 shrink-0"` back button (the `-ml-2` flushes it to the content edge,
  `ArrowLeft` icon, `aria-label`/`title` "Retour à la liste") then the `h2` title
  (`min-w-0 truncate text-base font-semibold`, plus `font-mono` when the title is an
  identifier like a chip id), then an optional muted subtitle row
  (`text-xs text-muted-foreground`, items separated by an `aria-hidden` `·`).
- Body: each section is a labelled block, not a second page-level card —
  `<section className="flex min-w-0 flex-col gap-2 border-t border-border pt-5 pb-4">`
  with an uppercase `h3` (`text-xs font-semibold tracking-wider uppercase`) above the
  content. The section wrapper is the frame: what goes inside it is never wrapped in a
  `Card` (no form card, no box around a box). A form inside a section keeps PAT-FORM's
  field rules (labels above, `FieldSet` + `FieldLegend variant="label"` for sub-groups)
  and ends on an action bar `border-t pt-4`, primary button right-aligned
  (`sm:justify-end`); its outcome is a toast (FBK-TOAST), not an alert in the bar.
- Full-page width (a detail that is a route, not a drawer): the body is a 12-column grid,
  `grid grid-cols-1 lg:grid-cols-12 lg:gap-x-10`. The main task (e.g. a device's
  configuration) spans `lg:col-span-7 xl:col-span-8`, a side column
  `lg:col-span-5 xl:col-span-4` holds the at-a-glance facts and short secondary tasks
  (latest readings, a transfer). Both columns open on a section divider, so their tops
  line up. Below `lg` the side column flattens into the single column with
  `max-lg:contents`, and the facts section moves first (`max-lg:order-first`) so a phone
  sees state before forms. A wide main form may go 2 columns (`md:grid-cols-2`); a
  side-column form stays 1 column.
- Facts read as tiles, the same as the anomaly view's "Relevés": a `grid grid-cols-2
  gap-2` of `h-14 rounded-md border px-2.5` tiles, icon (`TYPE_ICON`) + formatted value
  (`formatMeasure`, tabular numbers) on the first line, muted label below; position takes
  a full row (`col-span-2`, `formatCoordinates`).
- Help text says each rule once. A rule shared by every field (e.g. "Laissez un champ vide
  pour conserver sa valeur actuelle.") is a muted line at the top of the form, referenced
  by every input's `aria-describedby`. Each field's own hint keeps only what is specific
  to it (range, unit, current value: `entier de 0 à 255 · valeur actuelle : 3`).
- A measured value is never a bare text box: `InputGroup` with the unit as suffix (°C,
  % HR, kg, G, °, V, min), `inputMode="decimal"` (`text` when it can be negative, the iOS
  decimal pad has no minus); a small bounded integer gets −/+ stepper buttons; a duration
  gets a live readout ("= 6 h") and a few `ToggleGroup` presets. Validation (format, type
  range, physical bounds) lives in one module per form, runs on blur and on submit, shows
  an inline `FieldError` naming the range and unit, and submit focuses the first invalid
  field (reference: `src/pilotage/SettingsForm.tsx`, `settingsValidation.ts`).
- Reference implementation: `AnomalyDetailView.tsx`'s local `Section` helper (the
  original), reused the same way by `PilotageDeviceDetail.tsx`. Any new detail sub-page
  follows this rhythm rather than inventing its own header/section markup.
- A section's own link ("Voir la palette →") sits right-aligned on the section title row
  (`Section` `action` prop), not on a line of its own.
- A record with a status workflow (anomalies: Détectée → En cours → Clôture provisoire →
  Clôturée) shows its lifecycle as a horizontal step bar at the bottom of the header
  (`AnomalyLifecycle.tsx`): reached steps filled and joined, the current one ringed
  (`aria-current="step"`), later steps hollow; under each reached step the date it was last
  entered, its author in the tooltip. After a reopening, the later steps are pending again.
  The full history (with comments) stays in the last section, "Historique".

**Lists that open a detail.** One click on a row or card opens its detail directly. No
intermediate "Voir le détail" step or expanded preview.

### Map pages (Pulse), learned 2026-11-25

These come from the Pulse redesign in `cpaltracker-react`. Each one fixes a defect seen
in the browser.

- **Stacking.** Wrap the map in an `isolate` stacking context (`PositionsMap` root, the
  Pulse split). Leaflet's panes and controls (z 400–1000) then stay inside it, and the
  shell's portalled menus (z-50: user menu, top-bar popovers) render above the map.
  Overlays drawn inside the map (legend, hover card) use `z-1000`.
- **Hover card on a marker.** Anchor it to the marker element and write all four offsets
  (`left/top/right/bottom`) in one imperative step. Mixing corner classes with anchor styles
  stretches the card for a frame. While the marker element is missing (icon swap, off
  screen), keep the card `visibility: hidden`; never fall back to a corner, which flashes.
  Track `mouseout` with `relatedTarget`, so moving between a marker's dot and label doesn't
  unmount the card.
- **Marker labels.** On a pallet map the id label is always shown next to the dot, on one
  line (an inner `whitespace-nowrap` span, `shrink-0` dots). It is never shown only on
  hover. `not-sr-only` resets `white-space`, so it can't be the only nowrap.
- **Legends.** A legend the markers depend on (the load ramp "Poids restant transporté")
  is a card always visible over the map, bottom-left. It never sits only inside the
  layers popover, and its title never wraps (`w-max`, `whitespace-nowrap`).
- **Deselect.** One click on the map background clears the pallet or anomaly selection
  (`FleetMap` `onMapClick`). Don't use a double-click: that is Leaflet's zoom.
- **No data in the period.** Keep the map on screen and show the empty state centred over
  it. Never replace the map with an empty block.
- **Collapsed drawer.** The icon rail of a collapsed drawer shows what the drawer holds.
  With a pallet open, that is its latest readings (icon above, value, unit below:
  `PalletRail`), not the anomaly count.
- **Replay timeline.** A permanent bar under the map: play/pause, the period presets
  `1 j · 7 j · 15 j · 1 mois` (+ custom), the track, and the speed. The anomalies of the
  shown pallets in the period are ticks on the track: label in the tooltip, and a click
  jumps there and opens the anomaly. Ticks are one colour for now; colouring them by
  status is an open question.

**Selected state of toggles and segments.** A selected `Toggle`/`ToggleGroup` item uses
the primary fill (`SELECTED_TOGGLE` in `@/ui/patterns/period-control`:
`data-[state=on]:bg-primary text-primary-foreground font-semibold`). `bg-muted` alone
read as unselected in both themes and was reported as a bug twice (period presets, theme
segments).

**Document title.** `<page> · cPal` (`DocumentTitle.tsx`, `APP_NAME = 'cPal'`; `index.html`
`<title>cPal</title>`), never the package name.

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
| `cpaltracker-react` `src/anomalies/AnomalyLifecycle.tsx`, `AnomalyDetailView.tsx` (`History`) | The lifecycle step bar and the history list are dot-and-line markup built by hand. The kit has no stepper, and `ActivityTimeline` doesn't yet take a comment block per item. | A `Stepper` is added to the kit and `ActivityTimeline` takes comments (rule 5) |
| `src/showcase/lib/contrast.ts` | Computes contrast from rendered tokens and needs a canvas sentinel color. Lint must pass; use a scoped, reasoned disable if the sentinel is flagged. | Permanent |
