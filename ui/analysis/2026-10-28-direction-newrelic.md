# Direction analysis — New Relic (2026-10-28)

Evidence for choosing the cpaltracker-react UI direction. Source:
`ui/reference/directions/newrelic/` (16 screenshots, no notes). Colors in this
direction are replaced by [`../reference/brand.md`](../reference/brand.md); this
analysis is about structure, density and components.

Screenshots are cited by their time stamp (`18-28-34` = `Screenshot from 2026-10-28 18-28-34.png`).
`18-31-54` is **Northflank**, not New Relic; it's kept as a form reference.

## 1. Character

A dark, data-dense operations console. Near-black neutral surfaces, one blue accent
for actions, links and the time picker, and a saturated multi-color palette
reserved for charts and status. Cards are separated by surface shade rather than
borders. Every page answers the same three questions in the same places: *what am
I looking at* (breadcrumb and title, top left), *over what period* (time picker,
top right), *filtered how* (filter bar under the header).

That matches cpaltracker's job: monitoring a fleet of devices over time.

## 2. Measured surfaces

| Surface | New Relic (sampled) | c-PAL replacement | Note |
|---|---|---|---|
| Page background | `#10181C` (teal-black) | `#1A1D22` | Close luminance: a direct swap |
| Card / panel | `#182024` | `#22252E` | Same step between page and card |
| Sidebar | about `#0C1010` | `#20232B` | NR's sidebar is darker than the page; the brand's is between page and card |
| Filled input | `#384448`, no border | input border `#686E7F` | NR's filled inputs are 1.8:1 against the page, below the 3:1 a form control boundary needs. Keep bordered inputs |
| Accent | blue | orange `#DD8C3E` | See decision D2 |

## 3. Per screenshot

| Shot | Shows | Maps to | Takeaways |
|---|---|---|---|
| 18-28-34 | Logs: global sidebar + section sidebar, search bar with query button, histogram, log table | NAV-SHELL, NAV-SIDEBAR, PAT-LIST | Two-level navigation; the full-width search bar is the page's main control; the table uses monospace rows with a red status dot |
| 18-28-55 | APM errors: three columns (global nav, entity nav with group labels, content); chip filters; grouped error rows with sparklines | NAV-SIDEBAR, PAT-LIST, DAT-TABLE | Filters as removable `attribute = value` chips; sparkline per row; status selector in the row |
| 18-29-02 | APM databases: ranked list with bars + stacked area charts | DAT-LIST (ranked), DAT-CHART | "Top N" bar list: label, value, thin bar. Simple, readable |
| 18-29-10 | APM summary: chart grid + right column (activity stream, related entities) | PAT-DETAIL, PAT-DASH | Detail page = charts in the main area, context in a fixed right column. Healthy/critical banner in the side column |
| 18-29-16 | Dashboard: widget grid; big numbers; "No value" / "No chart data" states | PAT-DASH, DAT-STAT, FBK-EMPTY | Every widget: title, period subtitle, ⋯ menu. Empty state inside the widget, not a blank box |
| 18-29-24 / 18-29-31 | Install wizard: full-screen overlay, steps on the left, status table with badges; choice cards in a 2-column grid | PAT-WIZARD, OVL-SHEET (full), CMP-BADGE | Step list + content; status badges (green "Installed", grey "Detected") |
| 18-29-53 | Kubernetes dashboard: tab row, variable + filter bar, big numbers, ranked bars, stacked chart | NAV-TABS, PAT-DASH | Underline tabs for dashboard pages; big numbers in a light-weight font |
| 18-30-06 | Widget ⋯ menu (collapsed icon rail visible) | OVL-MENU, NAV-SIDEBAR (collapsed) | Plain text menu; the collapsed sidebar is an icon rail with an accent bar on the active item |
| 18-30-31 | Home: hexagon health grid (critical / warning / operational / no alerts / not reporting), "what's changed" timelines per category | DAT-STAT, DAT-TIMELINE, PAT-DASH | **The strongest fit for c-PAL**: one glyph per device, colored by state, with "not reporting" as a state of its own; event bars per category over a week |
| 18-30-52 | Dashboards list: thumbnail carousel, filter bar, table with star, link names, right-aligned dates, row ⋯ | PAT-LIST, DAT-TABLE | Clean list pattern; monospace, right-aligned dates |
| 18-31-33 | Modal form: two columns, required asterisks, date + time pairs, tabs, selection table with checkboxes, Cancel / primary bottom right | OVL-DIALOG, FRM-LAYOUT, FRM-DATE | The reference for a dialog form; date and time as two adjacent fields |
| 18-31-54 | Northflank settings form: single column (~800 px), bordered inputs, secret toggle, collapsible advanced options, primary button bottom left | PAT-SETTINGS, PAT-FORM, FRM-INPUT | The reference for a page form; bordered inputs pass contrast where NR's filled ones don't |
| 18-27-09 | Network flows: sankey, KPI tiles with trend arrow, info panel on the right | DAT-STAT, FBK-ALERT | KPI: label, big number, trend with arrow and % |
| 18-27-46 | Hosts: view switcher, suggested filter chips, KPI tiles with an action, chart small multiples with **red violation bands**, summary table | PAT-DASH, DAT-CHART, DAT-TABLE | Threshold breaches shown as red bands behind the series: directly useful for sensor history (excursions) |
| 18-28-00 | Browser app: status chips in the header ("50% warning alerts"), metric gauges, side column | NAV-HEADER, CMP-BADGE | Health summary as chips next to the title |

## 4. Patterns to adopt

1. **Header**: small breadcrumb link above a bold title; status chips and secondary
   actions next to the title; **time picker top right** with previous/next arrows
   ("Since 3 days ago"). History needs exactly this.
2. **Filter bar** under the header: removable `attribute = value` chips plus "add
   filter". Replaces the device checklist filters.
3. **Widget card**: title, period subtitle, ⋯ menu (expand, export), empty state
   inside. Every history chart and stat uses it.
4. **KPI tile**: label, big number, trend (arrow + %), optional action.
5. **Health grid**: one glyph per device, colored by state (alert, warning, OK,
   not reporting). A natural home page for the fleet.
6. **Chart threshold bands**: shaded regions where a value broke a limit.
7. **Detail page**: main area of charts and tables plus a fixed right column for
   activity and context (for a device: last positions, messages, settings shortcuts).
8. **Tables**: status dot first, monospace right-aligned numbers and dates, row ⋯
   menu, links in the name column.
9. **Collapsed sidebar** as an icon rail with an accent bar on the active item.
10. **Forms**: dialog forms two columns, page forms one column at about 800 px,
    required asterisks, date and time as adjacent fields, primary action stated by
    the page type (dialog: bottom right; page: bottom left).

## 5. Not to adopt

- **Filled, borderless inputs**: they fail the 3:1 boundary contrast. Use bordered
  inputs (`brand.md`).
- **Blue links everywhere**: too much accent once the accent is orange (D2).
- **The saturated rainbow chart palette**: it competes with the orange (D4).
- **Two-level navigation** as the default: cpaltracker has four sections (D1).
- **Sankey, AI prompt bar, account switcher**: no use in cpaltracker today.

## 6. Coverage gaps

| Missing | Why it matters | Suggestion |
|---|---|---|
| ~~A map page~~ | Covered by the ReUI fleet map (§8) | — |
| ~~Login~~ | Covered by the Datadog login (§9) | — |
| Light theme | The light theme needs its own reference | New Relic has one: same pages in light |
| Phone width | X-RESP | One page at about 390 px, if mobile matters |

## 7. Decisions for the operator

| # | Decision | Options | Recommendation |
|---|---|---|---|
| D1 | Navigation | (a) one sidebar, collapsible to an icon rail; (b) New Relic's global + section sidebars | **(a)**: four top-level sections don't need two levels. A device detail page can carry tabs instead |
| D2 | Link color | (a) orange links; (b) text-colored links, underlined, orange reserved for primary actions, active nav and key numbers | **(b)**: the brand says orange signals, it doesn't decorate; with orange links, every table would be orange |
| D3 | Warning color | Warning must not be mistaken for the brand orange | A distinct yellow for warning, red for alert, green for OK, grey for "not reporting" |
| D4 | Chart palette | (a) New Relic's saturated palette; (b) orange first, then muted cool hues, each ≥ 3:1 on the background | **(b)**: the orange stays the "main series" as in the brand |
| D5 | Density | (a) New Relic density everywhere; (b) dense in data views (tables, dashboards), the brand's spacing in headers, empty states and login | **(b)** |
| D6 | Big numbers | (a) New Relic's light sans; (b) the brand's Georgia serif for KPI tiles | **(b)**: the brand's signature; tables keep the sans |
| D7 | Default theme | Dark (both the brand and New Relic) or light | **Dark** by default, light available, system option |
| D8 | Map library | (a) keep Leaflet with raster tiles and the CSS invert filter; (b) MapLibre GL with vector tiles, the basemap styled from the tokens | **(b)**, see §8 |
| D9 | Map tile source (with D8 b) | (a) OpenFreeMap, a third-party host, CSP opened for it; (b) self-hosted PMTiles (Protomaps) served from the app's own origin | **(b)**: keeps the CSP at `'self'`, in line with the History feature's "no third-party CDN" rule (NFR1); (a) is faster to start |
| D10 | ReUI Pro license | (a) buy it and start from the block's code; (b) build from the screenshot with the free kit | Depends on budget: the block is the same stack (shadcn, MapLibre, Recharts, TanStack Table), so (a) saves the most work on the hardest page. Check the license terms for client projects first |

### Operator's answers (2026-10-28)

- **D1–D7:** recommendations accepted as written.
- **D8, D9 — map:** keep the technology the monolith (`cpaltracker.web`) uses, to
  avoid disruption: **Leaflet 1.9.4 with OpenStreetMap raster tiles**
  (`tile.openstreetmap.org`), plus **marker clustering** as the monolith does with
  `leaflet.markercluster`. No MapLibre, no vector tiles. The ReUI map layout (§8)
  is built on Leaflet: markers as HTML (`divIcon`) styled with the tokens,
  floating panel and controls as regular React components over the map, dark
  basemap through the existing CSS filter, tuned toward the brand charcoal.
- **D10 — ReUI:** not used, neither the code nor a license. The screenshot is a
  layout reference only.
- **D11 — charts:** keep **Plotly**. Chart colors are read from the tokens at
  runtime rather than copied into `chartColors.ts`; threshold bands are Plotly
  shapes; the brand donut is a Plotly pie with a 0.62 hole.

## 8. Map reference — ReUI fleet map

Source: `ui/reference/directions/reui-map/PAT-SPLIT__dispatch-map-reui.png`,
<https://reui.io/blocks/solutions/map>. A shadcn-based block; same stack as the
target kit.

| Element | What it does | cpaltracker use |
|---|---|---|
| Full-bleed dark vector map | Basemap in the theme's neutrals; roads light, land dark, labels muted | Live tracking and history maps. The basemap colors come from the tokens, in both themes, instead of inverting light raster tiles |
| Floating left panel over the map, collapsible from an edge handle | Fleet summary: title, subtitle ("31 stops still to deliver"), ⋮ menu | Fleet panel: devices reporting, alerts, last update |
| Donut + legend with counts | Day progress by state (delivered, failed, at risk, to go) | Devices by state (OK, alert, warning, not reporting) |
| Two-cell stat row | "On time 94%" / "Done by 4:57 PM" | Two key numbers, for example "Reporting 94%" and "Last position 2 min ago" |
| **Needs attention** list: icon, label, count, chevron | Breakdown, **cargo temp**, low range, late stops, failed attempt | Directly c-PAL: temperature excursion, shock, low battery, not reporting, geofence exit. Each row filters the map |
| Dot-progress table | Deliveries per hour | Optional: messages received per hour |
| Panel footer: secondary + primary action | "Van list" / "New delivery" | "Device list" / main action |
| Map markers: pill with icon + ID, status ring | Ring color = state (red, yellow, neutral); a green check for done | Device markers: pill with the device ID, ring color = state |
| Cluster marker | "16 parked" at the depot | Devices grouped at one site |
| Map controls top right | Zoom, locate, layers | Same |
| Header right | Map/List segmented toggle, density/speed select, search, alerts bell with count, fullscreen, primary action | Map/List toggle switches live tracking between map and device table |
| Scale bar, attribution bottom right | | Attribution must stay visible (OpenStreetMap / tile source) |

**Against the brand:**

- The primary button is a white fill; becomes orange fill with charcoal text.
- The neutrals are pure grey; become the brand charcoal.
- Status colors (green, red, yellow, purple "to go") follow D3; "to go" / neutral
  states use the chart secondary `#676E80`, not purple.
- The donut is the brand's donut (orange main segment, about 62 % hole, big serif
  number in the center) extended with the status colors.

**Technical consequence (D8, D9).** cpaltracker-react uses react-leaflet with
OpenStreetMap raster tiles, darkened in dark mode by a CSS invert filter. A
basemap in the brand's colors needs vector tiles and MapLibre GL. That's a stack
change, to be recorded in the stack decision. Separately, the production build's
CSP (`vite-plugin-security-headers.ts`) has `img-src 'self' data: blob:`, which
would block the `tile.openstreetmap.org` images the current maps load. **Likely
production defect, not yet verified on a built app.** Self-hosted PMTiles (D9 b)
removes the third-party tile host altogether.

## 9. Login reference — Datadog

Source: `ui/reference/directions/login/PAT-AUTH__login-datadog.png`. Operator's
instruction: keep the layout, **no patterned background**.

| Element | Datadog | c-PAL |
|---|---|---|
| Background | Polygon pattern, purple gradient | Flat `#1A1D22` (light theme: `#F5F2ED`) |
| Left column | Large logo + wordmark, vertically aligned with the form's top | The orange logo variant (`cPAL_logo_orange.png`) on dark, the dark variant on light; never redrawn |
| Right column | "Log in to Datadog", region selector, Google button, SSO link, separator, two fields, full-width "Log in", "Forgot password?" | Title, the two fields (labelled; the current page's labels, `aria` wiring and lockout notices are kept), full-width primary button, "forgot password" link. No SSO, no region |
| Primary button | Blue fill, white text | Orange fill, charcoal text (`brand.md`) |
| Inputs | Dark fill, thin grey border | Bordered inputs, border `#686E7F` |
| Footer | Small legal links | Small muted line: `www.c-pal.eu`, legal links if any |
| Narrow screens | — | The logo stacks above the form, smaller |

The same layout serves forgot-password and reset-password (one title and one
field set each). The brand's eyebrow ("CONNEXION" in orange capitals with the short
orange rule under it) above the title is optional; it's the brand's signature
element and fits here.

Today's `Login.tsx` puts the form in a centered `Card` under a page title. The new
layout drops the card: the form sits directly on the page background, next to the
logo.
