# c-PAL theme: "Clair minéral" (light) and "Sombre graphite" (dark)

Status: **implemented, uncommitted, awaiting operator review** (2026-11-25).
Supersedes the blue-charter palette of 2026-11-24
([`../audits/2026-11-24-blue-theme-palette-changes.md`](../audits/2026-11-24-blue-theme-palette-changes.md)).
Changes from the spec and their checks:
[`../audits/2026-11-25-mineral-graphite-palette-changes.md`](../audits/2026-11-25-mineral-graphite-palette-changes.md).

This page explains how the theme works: the token set, how the theme is chosen, the rules for
chrome, accent and components, and how the theme reaches the map and the charts. It applies to
`cpaltracker-react` and the reference kit `cpaltracker.ui-reference`.

| | |
|---|---|
| Spec | "C-PAL theming: Clair minéral + Sombre graphite", operator, 2026-11-25 |
| Values | `cpaltracker.ui-reference/src/styles/tokens.css`. This is the only file that defines a color, font, radius or shadow. It is copied byte-identical to `cpaltracker-react/src/styles/tokens.css` |
| Token → utility mapping | `src/index.css` (`@theme inline`), identical in both repos |
| Contrast guard | `cpaltracker-react/src/styles/tokens.contrast.test.ts` (vitest, 162 pairs) |
| Brand constraints | [`brand.md`](brand.md): logo, typography, the brand cyan |

## 1. Intent

The look is enterprise, calm and slightly earthy. **Color belongs to the data and the status, not
to the chrome.** The chrome is quiet: one neutral surface, separated by thin borders. A muted
teal, `--signal`, marks what is selected, active or focused. Red, amber and green mark states.
The brand cyan `#08B9E8` stays in the logo. Montserrat everywhere, Lucide icons. No card
shadows, no gradients, no italics.

## 2. Tokens

### Spec names and token names

The spec names its tokens `--bg`, `--accent` and so on. The code keeps the shadcn/Tailwind names
that every component already uses, so the spec maps onto them:

| Spec token | Token(s) in `tokens.css` | Tailwind utility |
|---|---|---|
| `--bg`, `--surface` | `--background`, `--sidebar` | `bg-background`, `bg-sidebar` |
| `--surface-subtle` | `--card`, `--muted`, `--secondary`, `--accent`*, `--sidebar-accent` | `bg-card`, `bg-muted`, … |
| `--surface-elevated` | `--popover` | `bg-popover` |
| `--shadow-elevated` | `--shadow-overlay` (`--shadow-media` for the map frame) | `shadow-overlay` |
| `--border` | `--border`, `--sidebar-border`, `--chart-grid` | `border-border` |
| — (added) | `--input`: form-control outline, see §2.3 | `border-input` |
| `--text` | `--foreground`, `--body` | `text-foreground`, `text-body` |
| `--text-secondary` | `--muted-foreground`, `--sidebar-foreground` | `text-muted-foreground` |
| `--text-muted` | `--faint`, `--chart-axis` | `text-faint` |
| `--accent` | `--signal`, `--ring`, `--chart-1` | `bg-signal`, `ring-signal`, `border-signal` |
| `--accent-text` | `--primary-text` | `text-primary-text` |
| `--accent-ring` | `--signal-ring` | `ring-signal-ring`, `shadow-[…var(--signal-ring)]` |
| `--primary-bg`, `--primary-fg` | `--primary`, `--primary-foreground` | `bg-primary`, `text-primary-foreground` |
| label on the selected marker (`#151617`) | `--on-signal` | `text-on-signal` |
| `--danger`, `--warning`, `--success` | same names | `text-danger`, … |
| `--danger-mark` | same name | `bg-danger-mark` |
| `--*-tint` | `--danger-soft`, `--warning-soft`, `--success-soft` | `bg-danger-soft`, … |
| `--neutral-pill-bg`, `--neutral-pill-fg` | `--offline-soft`, `--offline` | `bg-offline-soft`, `text-offline` |
| `--map-water`, `--map-land`, `--map-road` | same names; `--media` = map-land | `bg-map-land`, `bg-media` |
| logo cyan `#08B9E8` | `--brand` | `bg-brand` (the rail mark only) |

\* `--accent` in the code is the **shadcn hover/selected surface**, not the spec's accent. The
spec's accent is `--signal`. Read `bg-accent` as "hover surface".

### Values

| Token | Clair minéral | Sombre graphite | Role |
|---|---|---|---|
| `--background` | `#FAFBFA` | `#151617` | page and chrome: one surface |
| `--card` / `--muted` / `--accent` | `#EDF1EE` | `#1D1E20` | cards, hover, segmented track, selected nav item |
| `--popover` | `#FFFFFF` | `#25272A` | popovers, menus, map controls, selected segment |
| `--border` | `#DCE3DE` | `#303235` | separators, outlines (decorative) |
| `--input` | `#76917E` | `#6B7077` | text field, select and checkbox outlines |
| `--foreground` | `#17201C` | `#ECEDEC` | text, active icons |
| `--muted-foreground` | `#536159` | `#AAAEAB` | secondary text, default icons |
| `--faint` | `#65706A` | `#8B8E87` | separators (›), disabled, placeholders, axis labels |
| `--signal` | `#2997AC` | `#4DB6C6` | selection, active, focus, primary series, markers |
| `--primary-text` | `#1D6F80` | `#6CC6D3` | links, small accent text |
| `--signal-ring` | `#2997AC` at 20 % | `#4DB6C6` at 25 % | selected-marker halo, focus glow |
| `--on-signal` | `#151617` | `#151617` | label on a signal fill |
| `--primary` / `--primary-foreground` | `#17201C` / `#FAFBFA` | `#4DB6C6` / `#151617` | primary button |
| `--danger` | `#A33A26` | `#E9826D` | "À traiter", excess values, thresholds |
| `--danger-mark` | `#B4432F` | `#C24D38` | status dots, chart anomaly dots |
| `--warning` | `#8A5A12` | `#D9A54A` | "En cours", medium risk |
| `--success` | `#2E7854` | `#5FBF8E` | OK, low risk |
| `--danger-soft` | `#F6E5E1` | danger at 15 % | pill background |
| `--warning-soft` | `#F3EBDC` | warning at 15 % | pill background |
| `--success-soft` | `#E2EFE7` | success at 15 % | pill background |
| `--offline` / `--offline-soft` | `#536159` / `#EDF1EE` | `#AAAEAB` / `#1D1E20` | neutral pill ("Clôturée", not reporting) |
| `--info` | `#4A4FB0` | `#A9B1F2` | info status (not in the spec, kept) |
| `--brand` | `#08B9E8` | `#08B9E8` | logo, rail mark, favicon |
| `--map-water` / `--map-land` / `--map-road` | `#D5DFDB` / `#F1F4F1` / `#E1E7E2` | `#0F1011` / `#1C1D1F` / `#27292B` | basemap palette, §6 |
| `--shadow-overlay` | `0 4px 16px rgb(23 32 28 / .08)` | `0 4px 16px rgb(0 0 0 / .4)` | elevated surfaces only |

The full set (sidebar aliases, chart series, load ramp) is in `tokens.css`, with a comment on
every value that differs from the spec.

### Where the values differ from the spec (WCAG 2.1 AA)

The spec requires every text pair to pass AA. Five spec values did not, so each kept its hue and
moved in lightness by the smallest step that passes on every surface it is used on:

| Token | Theme | Spec | App | Reason |
|---|---|---|---|---|
| `--faint` (text-muted) | light | `#8A968F` | `#65706A` | 2.96:1 on the background. Now 4.96 background, 4.52 subtle, 5.15 elevated |
| `--faint` | dark | `#83877F` | `#8B8E87` | 4.09:1 on the elevated surface. Now 5.45 background, 5.02 subtle |
| `--signal` (accent) | light | `#2B9DB3` | `#2997AC` | 2.80:1 on the subtle surface (selected nav item, focus on cards). Now ≥ 3.01 on every surface |
| `--success` | light | `#2F7A55` | `#2E7854` | 4.39:1 on its own tint (the "OK" pill). Now 4.51 |
| `--input` (added) | both | — | `#76917E` / `#6B7077` | `--border` is 1.26:1 (light) and 1.41:1 (dark). Separators can be decorative, but a text field's outline is a UI boundary (WCAG 1.4.11, 3:1). Border's hue, darkened or lightened to ≥ 3:1 |

The spec uses `--text-muted` for placeholders and axis labels. These are text, so they need
4.5:1. That is why `--faint` moved, rather than keeping it for "non-essential hints" only.

### The automated check

`src/styles/tokens.contrast.test.ts` parses `tokens.css` and checks, in both themes:

- every text role (`foreground`, `body`, `muted-foreground`, `faint`, `primary-text`,
  `sidebar-foreground`) on every surface (`background`, `card`, `popover`, `sidebar`), 4.5:1;
- every `*-foreground` on its own fill, including `on-signal` on `signal`, 4.5:1;
- each status on the background, the subtle surface and its own tint, and
  `status-foreground` on a filled status, 4.5:1;
- `input`, `ring` and `signal` on every surface, `ring-media` on the map frame, `signal` on
  map-land, `danger-mark`, and `chart-1…5`, 3:1;
- axis labels, 4.5:1.

Translucent values (`rgb(… / a)`, `color-mix(… N%, transparent)`) are composited over their
surface first. A token change that breaks a pair fails the unit suite. The kit's
`/guideline/foundations` page shows the same pairs computed live from the rendered theme.

## 3. Choosing the theme

- `<html data-theme="light|dark">` is the only switch. Every token block is keyed on it, and
  Tailwind's `dark:` variant follows it (`@custom-variant dark`), never a class.
- The **default is the OS preference** (`system`, decision D7 revised 2026-11-25). The toggle
  overrides it and the choice persists in `localStorage` under `cpaltracker:theme`
  (`light | dark | system`). Storage access is wrapped in try/catch; when it is unavailable, the
  default applies.
- **Before first paint:** `public/theme-init.js`, a same-origin classic script in `<head>`
  (no inline script, since the CSP is `script-src 'self'`), resolves the stored choice or the OS
  preference and sets `data-theme`. `index.html` ships `data-theme="dark"`, which is overridden
  before the body renders. When the OS states no preference, `system` resolves to dark.
- **At runtime:** `ThemeProvider` (`src/app/theme.tsx`) holds the choice, follows
  `prefers-color-scheme` changes while the choice is `system`, and writes `data-theme`.
  `useTheme()` needs the provider. `useResolvedTheme()` works without one: it falls back to the
  `<html>` attribute, so kit components such as the maps run anywhere.
- Switching theme never reloads the page. CSS variables change, Tailwind utilities follow, the
  map tiles re-filter (§6), and the charts re-read their colors (§7).

## 4. Chrome

- Sidebar, top bar and drawers (`Sheet`) use `--background`, the same surface as the page.
  They are separated by 1 px `--border` lines only, with no other background shade and no shadow.
- Active nav item: a 3 px `--signal` bar on the left edge, `--sidebar-accent` (subtle)
  background, `--foreground` text at weight 600 (`NAV_ITEM_CLASS` in
  `src/ui/patterns/app-shell.tsx`). In the collapsed icon rail the bar is hidden and the subtle
  background alone marks the item.
- Inactive nav items: `--muted-foreground`; hover `--sidebar-accent`.
- Breadcrumb: parents `--muted-foreground`, separators `--faint`, current level `--foreground`
  at weight 600.
- Logo: the anthracite wordmark in light, the white one in dark. In the collapsed rail and as
  the favicon, the **rail mark** (a `--brand` dot in a 20 % `--brand` halo) stands in, see
  [`brand.md`](brand.md#what-carries-over) § Logo.

## 5. Accent rules and components

**`--signal` is allowed for:** the active nav bar, the selected tab underline (2 px), the focus
ring (2 px `--ring` + `--signal-ring` glow), the selected card outline (1.5 px), map marker and
cluster outlines, the selected marker fill, the primary chart series, checked checkboxes, radios
and switches, slider and progress fill, and key figures ≥ 20 px.

**`--signal` is forbidden for:** large fills (except the selected marker, and the primary button
in dark, which is `--primary`), body text, segmented-control and chip fills, and header
backgrounds. Small accent text uses `--primary-text`, never `--signal`.

| Component | Treatment | Where |
|---|---|---|
| Primary button | `--primary` / `--primary-foreground`, radius 8 px, weight 600. One per view at most | `ui/button.tsx` `default` |
| Secondary button | transparent, 1 px `--border`, `--foreground`; hover subtle | `outline` |
| Tertiary / link | text only, `--primary-text` | `link`, `ghost` |
| Icon button | 36 px (`size="icon"`); on the map, `--popover` in the control group (§6) | `ui/button.tsx` |
| Segmented control | track `--muted`; selected segment `--popover` + 1 px `--border` + `--foreground` 600; others `--muted-foreground` | `ui/tabs.tsx` (default variant), app `period-control.tsx` |
| Tabs | text tabs; selected = 2 px `--signal` underline + `--foreground` 600 | `ui/tabs.tsx` `variant="line"` (now the default) |
| Chips | page surface, 1 px `--border`, `--foreground`; pressed = subtle surface | `ui/toggle.tsx`, filter bars |
| Status pills | tint background + semantic text, 11 px, 600, radius 10 px | `ui/badge.tsx` status variants |
| Cards | light: `--card`, no border, no shadow. Dark: `--card` + 1 px `--border` ring. Accent variants: one `--signal` edge (`accent-left`, `accent-top`). Selected card (spec): page surface + 1.5 px `--signal`, applied by the feature that owns the selection | `ui/card.tsx` |
| Popovers, menus, selects, tooltips | `--popover` + `--shadow-overlay` + 1 px `--border`, radius 10 px (`rounded-xl`) | every `bg-popover` component |
| Inputs | page surface, 1 px `--input`, focus ring as above, placeholder `--faint` | `ui/input.tsx`, `ui/input-group.tsx`, `ui/textarea.tsx` |
| Tables | header `--muted-foreground` 12 px 600; rows 1 px `--border`; hover `--muted`; tabular figures | `ui/table.tsx` |

Typography: Montserrat (self-hosted, `@fontsource/montserrat` 400–800). Key figures 18–28 px
700/800 (`font-display`). `font-feature-settings: 'tnum'` is set on `body`, so every number in
tables, cards and charts has tabular figures.

## 6. The map and the theme

Both maps follow the theme: the live fleet map (`src/ui/map/FleetMap.tsx`, both repos) and the
history timeline map (`src/history/TimelineMap.tsx`, app). Three layers are themed separately.

```
┌──────────────── .leaflet-container  (bg-media = --map-land, shows while tiles load) ───┐
│  .leaflet-tile-pane   filter: var(--map-tile-filter)    ← basemap: recolored per theme │
│  .leaflet-marker-pane (no filter)                       ← markers, clusters: tokens    │
│  .leaflet-control-*   --popover + --border               ← zoom, fullscreen, layers    │
│  attribution          --card backdrop, --body text, --primary-text links              │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

### Basemap

The spec asks for CARTO Positron (light) and CARTO Dark Matter (dark), "or a custom style using
the `--map-*` tokens". **CARTO's basemap tiles now answer "API key required" without a CARTO
account** (checked 2026-11-25: the tiles render as the text "API KEY REQUIRED"). The app
therefore uses the alternative: **OpenStreetMap raster tiles, recolored to the `--map-*`
palette** by one CSS filter on the tile pane.

| | Clair minéral | Sombre graphite |
|---|---|---|
| `--map-tile-filter` | `grayscale(0.85) brightness(1.03) contrast(0.9)` | `invert(1) grayscale(1) brightness(0.9) contrast(0.85)` |
| Result | mineral grey-green, land ≈ `--map-land` `#F1F4F1` | graphite, no hue, land ≈ `#1C1D1F` |
| Frame under the tiles (`--media`) | `#F1F4F1` | `#1C1D1F` |

How it works:

- `src/ui/map/basemap.ts` holds the one tile URL and attribution (`useBasemap()`). Both maps
  pass them to their `TileLayer`, so a new basemap is a one-file change.
- `src/index.css` applies `filter: var(--map-tile-filter)` to `.leaflet-tile-pane` only. The
  filter is a token, so switching theme restyles the tiles in place, with no tile refetch and no
  reload. The marker pane is a sibling, so markers keep their exact token colors.
- The light filter keeps a trace of OSM's color (water, parks), so the map stays readable while
  matching the mineral grey. The dark filter inverts and drops all hue, so the dark map has no
  navy or green cast (spec acceptance).
- CSP: `img-src` allows `https://tile.openstreetmap.org` only, with no wildcard (KAN-222).
- Attribution is required and stays visible. It sits on an opaque `--card` backdrop with `--body`
  text and `--primary-text` links, so it passes AA whatever tile is underneath.
- The `--map-water`, `--map-land` and `--map-road` values are the target palette. They are
  exposed as utilities (`bg-map-land`, …) for map legends and for a future vector style.

**To switch to CARTO later:** get a CARTO API key, put its two tile URLs in `basemap.ts`
(pick by `useResolvedTheme()`), set `--map-tile-filter: none` in both themes, and add the CARTO
host(s) to `img-src` in `vite-plugin-security-headers.ts` and its test.

### Markers and clusters (`src/ui/map/markers.ts`)

Markers are Leaflet `divIcon`s whose HTML uses token utilities only (a unit test asserts there
is no `style=` and no hex in the HTML).

| Element | Light and dark |
|---|---|
| Pallet marker | `--background` fill, 1.5 px `--signal` outline, `--foreground` label 12 px 600, state dot |
| State dot | `--success`, `--warning`, `--danger-mark` or `--offline`, with the state glyph. The state lives in the dot, never in the outline |
| Selected marker | `--signal` fill, `--on-signal` label (`#151617` in both themes), 6 px `--signal-ring` halo, scaled 110 % |
| Cluster | `--background` fill, 2 px `--signal` outline, `--foreground` count at 700. The worst state of its members is in the `aria-label` |
| Keyboard focus | `--ring` outline. Inside the map frame `--ring` becomes `--ring-media` (`.bg-media`), checked ≥ 3:1 on map-land |
| Map controls | vertical group, `--popover` surface, 1 px `--border`, `--foreground` icons |

App only (Pulse): when a pallet has a remaining-load level, the outline takes the load ramp
`--load-1…5` (the signal hue, empty → full) instead of `--signal`, since the outline is then
data. The state stays in the dot.

Contrast on the map: the signal outline is ≥ 3:1 on map-land in both themes (tested). The
marker's label sits on its own opaque fill, so its contrast does not depend on the tiles.

## 7. Charts

`src/ui/charts/use-chart-theme.ts` reads the tokens with `getComputedStyle` and re-reads them
when `data-theme` changes. Plotly gets resolved colors and never a literal.

- Series: `--chart-1` (= `--signal`) is the primary series. `--chart-2…5` keep the approved
  per-metric hues (violet, sage, sand, slate), each ≥ 3:1 on the background and the subtle
  surface.
- Thresholds: dashed 1 px `--danger` lines with a `--danger` label (`time-series-chart.tsx`,
  app `pulse/palletFigure.ts`). Out-of-range bands: `--danger` at 14 %.
- Anomaly dots: solid while open, hollow when closed. The spec color is `--danger-mark`;
  `pulse/palletFigure.ts` still draws them in `--danger` (open item, see the audit).
- Axes: labels `--chart-axis` (= `--faint`), lines `--border`. **No gridlines**
  (`showgrid: false`), only threshold lines.
- Hover labels: `--popover` background, `--foreground` text.
- Risk zones (Tendances, `trends/TrendsChart.tsx`): danger, warning and success bands at low
  opacity.
- Paper and plot backgrounds are transparent. The card behind the chart gives the surface.

## 8. Rules for new code

- Never write a color literal (hex, `rgb()`, named color) outside `tokens.css`. Use the
  utilities above. Check: `grep -rnE "#[0-9a-fA-F]{6}\b" src --include=*.tsx --include=*.ts`
  returns nothing outside tests.
- A new token goes in `tokens.css` (both theme blocks, with a comment if it differs from the
  spec), in the `@theme inline` map of `index.css`, and in the contrast test if it is used for
  text or a UI boundary. Then copy both files to the other repo unmodified.
- The chrome never gets a new background shade. Hierarchy comes from borders, weight and the
  elevated surface.
- Before using `--signal`, check §5. If the use is not in the allowed list, use
  `--foreground`, `--primary-text` or a status token.
