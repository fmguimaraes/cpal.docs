# UI references — cpaltracker-react

The brief the visual guideline is built from. It says which reference wins for
what, and the constraints every screen keeps. Evidence and the detailed reasoning:
[`../analysis/2026-10-28-direction-newrelic.md`](../analysis/2026-10-28-direction-newrelic.md).

## Sources

| Folder | Source | Used for |
|---|---|---|
| [`theme.md`](theme.md) | Theming spec "Clair minéral" / "Sombre graphite" (operator, 2026-11-25) | **All colors**: tokens, theme switching, chrome, accent rules, components, map and charts. Fixed: overrides every other source |
| [`brand.md`](brand.md) | c-PAL charter (Confluence, 2026-11-20) | Typography, logo, signature elements. Fixed |
| `directions/blue-theme/` | Ariane deck (Aéroport Marseille Provence, Oct. 2026) | Visual reference for the blue charter: cards, key figures, cyan accent. Theming only |
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

- New Relic's filled borderless inputs (fail 3:1 boundary contrast), its colored links
  everywhere, its saturated rainbow chart palette, its two-level navigation.
- Datadog's patterned background, Google/SSO buttons, region selector.
- ReUI's white primary button, pure-grey neutrals, purple state color.
- Decorative imagery of any kind (brand rule): illustrate with data and the product.

## Decisions

| # | Decision |
|---|---|
| D1 | One sidebar, collapsible to an icon rail |
| D2 | Links in `--primary-text`, underlined. The `--signal` accent only where [`theme.md` §5](theme.md#5-accent-rules-and-components) allows it (revised 2026-11-25) |
| D3 | Status colors: red alert, **yellow** warning, green OK, grey not reporting, **periwinkle** info (never the brand cyan) |
| D4 | Chart palette: `--signal` first, then muted hues away from blue (violet, sage, sand, slate), each ≥ 3:1 on background and card in both themes |
| D5 | Dense data views (tables, dashboards); the brand's spacing in headers, empty states and login |
| D6 | Montserrat everywhere; Montserrat ExtraBold (`font-display`) for big numbers in KPI tiles and the donut center; tables keep tabular figures (revised 2026-11-24, was Georgia) |
| D7 | The theme **follows the OS by default**; the toggle overrides it and persists (revised 2026-11-25 with the Clair minéral / Sombre graphite spec; was light by default) |
| D8 | Maps: **Leaflet 1.9.4 + OpenStreetMap raster tiles**, as the monolith; marker clustering as the monolith (`leaflet.markercluster`). Markers are HTML `divIcon`s styled with the tokens; the basemap follows the theme through `--map-tile-filter` (mineral grey in light, graphite in dark). CARTO, the spec's first choice, needs an API key: see [`theme.md` §6](theme.md#6-the-map-and-the-theme) |
| D9 | Tile source: `tile.openstreetmap.org`, as the monolith. The production CSP must allow it (`img-src`) |
| D10 | ReUI not used: layout reference only |
| D11 | Charts: **Plotly** kept. Colors read from the tokens at runtime; threshold bands as Plotly shapes; donut as a Plotly pie with a 0.62 hole |

## Constraints

- **Colors:** every color from [`theme.md`](theme.md) ("Clair minéral" / "Sombre graphite",
  2026-11-25). Primary button: anthracite in light, `--signal` in dark. The brand cyan is
  the logo's only.
- **Logo:** the canonical files only, never redrawn: black and white wordmarks
  (`c-pal-black-transparent.png`, `c-pal-white-transparent.png`).
- **Language (decided 2026-10-29):** no page is translated; each page keeps the
  language it has in the monolith (French navigation: "Accueil", "Suivi en direct";
  English login title "Sign in"). The kit carries no default copy. See X-COPY.
- **Desktop first** (stack decision §4); phone width best-effort.
- **No behavior change** while restyling: the existing E2E tests and their
  `data-testid`s keep passing.
- **Accessibility:** WCAG 2.1 AA in both themes (the project's NFRs).

## Still missing

- A square logo mark for the favicon and the collapsed sidebar rail (both use the rail
  mark, a cyan dot in a halo, since 2026-11-25).
