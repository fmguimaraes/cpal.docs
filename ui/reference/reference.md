# UI references — cpaltracker-react

The brief the visual guideline is built from. It says which reference wins for
what, and the constraints every screen keeps. Evidence and the detailed reasoning:
[`../analysis/2026-10-28-direction-newrelic.md`](../analysis/2026-10-28-direction-newrelic.md).

## Sources

| Folder | Source | Used for |
|---|---|---|
| [`brand.md`](brand.md) | c-PAL design reference (Confluence) | **All colors, type and signature elements.** Fixed: overrides every other source |
| `directions/newrelic/` | New Relic (one Northflank shot, `18-31-54`) | App structure: shell, page header, time picker, filter bar, dashboards, KPI tiles, tables, detail pages, forms, menus, empty states |
| `directions/reui-map/` | ReUI "Dispatch" fleet map block | Live tracking layout: full map, floating fleet panel, needs-attention list, device markers, map controls. Layout only; no ReUI code |
| `directions/login/` | Datadog login | Login, forgot password, reset password layout |

## Take from each

- **Shell (New Relic):** one sidebar, collapsible to an icon rail with an accent bar
  on the active item; no second-level sidebar. Four sections today.
- **Page header (New Relic):** small breadcrumb, bold title, status chips beside it,
  **time picker top right** with previous/next arrows, filter bar with removable
  `attribute = value` chips underneath.
- **Dashboards and history (New Relic):** widget cards (title, period subtitle, ⋯
  menu, empty state inside); KPI tiles with trend arrow and %; **threshold bands**
  on charts for excursions; ranked "top N" bar lists.
- **Fleet overview (New Relic home):** a health grid, one glyph per device, colored by
  state, with "not reporting" as its own state.
- **Detail page (New Relic):** charts and tables in the main area, a fixed right
  column for activity and context.
- **Tables (New Relic):** status dot first, monospace right-aligned numbers and
  dates, row ⋯ menu, link in the name column.
- **Forms:** dialog forms in two columns (New Relic `18-31-33`); page forms in one
  column at about 800 px with bordered inputs (Northflank `18-31-54`). Date and time
  as adjacent fields.
- **Live tracking (ReUI):** map filling the content area; a floating, collapsible
  fleet panel on the left with a state donut, two key numbers, a **needs
  attention** list (temperature excursion, shock, low battery, not reporting,
  geofence…) whose rows filter the map, and footer actions; device markers as pills
  (icon + device ID) with a state-colored ring; clusters for devices at one site;
  zoom/locate/layers controls top right; **Map/List** toggle in the header.
- **Login (Datadog):** large logo left, narrow form column right, upper third of the
  page; title, fields, full-width primary button, "forgot password" link; small
  footer.

## Avoid

- New Relic's filled borderless inputs (fail 3:1 boundary contrast), its blue links
  everywhere, its saturated rainbow chart palette, its two-level navigation.
- Datadog's patterned background, Google/SSO buttons, region selector.
- ReUI's white primary button, pure-grey neutrals, purple state color.
- Decorative imagery of any kind (brand rule): illustrate with data and the product.

## Decisions

| # | Decision |
|---|---|
| D1 | One sidebar, collapsible to an icon rail |
| D2 | Links text-colored and underlined; orange only for the primary action, the active nav item and key numbers |
| D3 | Status colors: red alert, **yellow** warning (never the brand orange), green OK, grey not reporting |
| D4 | Chart palette: orange first, then muted cool hues, each ≥ 3:1 on the background in both themes |
| D5 | Dense data views (tables, dashboards); the brand's spacing in headers, empty states and login |
| D6 | Georgia for big numbers in KPI tiles and the donut center; tables stay sans with tabular figures |
| D7 | Dark theme by default; light and system available |
| D8 | Maps: **Leaflet 1.9.4 + OpenStreetMap raster tiles**, as the monolith; marker clustering as the monolith (`leaflet.markercluster`). Markers are HTML `divIcon`s styled with the tokens; dark basemap through the CSS tile filter, tuned toward the brand charcoal |
| D9 | Tile source: `tile.openstreetmap.org`, as the monolith. The production CSP must allow it (`img-src`) |
| D10 | ReUI not used: layout reference only |
| D11 | Charts: **Plotly** kept. Colors read from the tokens at runtime; threshold bands as Plotly shapes; donut as a Plotly pie with a 0.62 hole |

## Constraints

- **Brand:** every color from [`brand.md`](brand.md), using its *app values*
  (contrast-corrected). Primary button: orange fill, charcoal text, both themes.
- **Logo:** the canonical files only, never redrawn. Off-white and white variants
  are still to be supplied.
- **Language (decided 2026-10-29):** no page is translated; each page keeps the
  language it has in the monolith (French navigation: "Accueil", "Suivi en direct";
  English login title "Sign in"). The kit carries no default copy. See X-COPY.
- **Desktop first** (stack decision §4); phone width best-effort.
- **No behavior change** while restyling: the existing E2E tests and their
  `data-testid`s keep passing.
- **Accessibility:** WCAG 2.1 AA in both themes (the project's NFRs).

## Still missing

- Light-theme references (any of the sources above, in light mode). Until then the
  light theme is derived from `brand.md`'s light palette.
- Off-white and white logo files.
