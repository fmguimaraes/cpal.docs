# c-PAL "Clair minéral" / "Sombre graphite": changes from the spec, and checks

Status: **implemented, uncommitted, awaiting operator review** (2026-11-25).
Supersedes the blue palette of [`2026-11-24-blue-theme-palette-changes.md`](2026-11-24-blue-theme-palette-changes.md).
How the theme works (tokens, switching, chrome, components, map, charts): [`../reference/theme.md`](../reference/theme.md).

| | |
|---|---|
| Spec | "C-PAL theming: Clair minéral (light) + Sombre graphite (dark)", operator, 2026-11-25 |
| Scope | `cpaltracker-react` and the reference kit: tokens, theme default, kit components, map markers and controls, chart theme, the app's accent spots. Layout, copy and `data-testid`s are unchanged, apart from the decisions listed below |
| Values | `cpaltracker.ui-reference/src/styles/tokens.css`, copied unmodified to the app |

## Decisions

| # | Decision | Before | Now |
|---|---|---|---|
| 1 | Palette | blue charter: white, cyan `#08B9E8` fills | mineral light, graphite dark; muted teal `--signal` |
| 2 | Default theme (D7) | light, whatever the OS | **follows the OS**; the toggle overrides and persists |
| 3 | Brand cyan | primary fill, nav marker, badges | logo, rail mark and favicon only (`--brand`) |
| 4 | Primary button | cyan + anthracite label | anthracite in light, `--signal` fill in dark |
| 5 | Chrome | sidebar `#F3F7FA` (light), stepped surfaces (dark) | one surface with the page, borders only; drawers too |
| 6 | Active nav item | cyan marker | 3 px `--signal` bar, subtle background, weight 600 |
| 7 | Tabs | segmented by default | text tabs with a 2 px `--signal` underline by default; segmented = elevated selected segment |
| 8 | Cards | flat blue-grey | subtle surface, no shadow; dark adds a 1 px border |
| 9 | Popovers, menus | radius 8 px | elevated surface + shadow + border, radius 10 px |
| 10 | Status pills | 12 px | 11 px, 600, radius 10 px, on their tint |
| 11 | Map basemap | OSM, light `saturate(0.55)`, dark inverted toward anthracite | OSM recolored to `--map-*`: light grey-mineral, dark graphite (CARTO needs a key, below) |
| 12 | Map markers | card fill, outline in the state color | page fill, 1.5 px `--signal` outline, state as a dot; selected = `--signal` fill + halo |
| 13 | Clusters | card fill, 4 px ring in the worst state | page fill, 2 px `--signal` outline, count at 700 |
| 14 | Map controls | separate secondary buttons | one vertical group, elevated surface, 1 px border |
| 15 | Charts | gridlines, threshold lines dotted in warning | no gridlines; thresholds dashed in `--danger` |
| 16 | Numbers | `tabular-nums` on tables | `tnum` on `body`: every number |

## Values changed from the spec (WCAG 2.1 AA)

Method as before: keep the hue, move the lightness by the smallest step that passes on every
surface the token is used on.

| Token | Theme | Spec | App | Failing pair → result |
|---|---|---|---|---|
| `--faint` | light | `#8A968F` | `#65706A` | 2.96 on background → 4.96 / 4.52 subtle / 5.15 elevated |
| `--faint` | dark | `#83877F` | `#8B8E87` | 4.09 on elevated → 5.45 / 5.02 subtle |
| `--signal` | light | `#2B9DB3` | `#2997AC` | 2.80 on subtle → ≥ 3.01 on every surface |
| `--success` | light | `#2F7A55` | `#2E7854` | 4.39 on its tint → 4.51 |
| `--input` | both | — | `#76917E` / `#6B7077` | `--border` is 1.26 / 1.41: too faint for a form-control outline (1.4.11) → ≥ 3.00 |

Not in the spec, kept: `--info` (periwinkle, 6.07 on subtle), the chart series 2–5 (approved
hues, each ≥ 3:1), the Pulse remaining-load ramp `--load-1…5` (moved to the signal hue).

## Basemap: why not CARTO

The spec's first choice is CARTO Positron / Dark Matter. On 2026-11-25, CARTO's tile servers
returned tiles reading "API KEY REQUIRED" without an account. Using CARTO also needed four new
hosts in the CSP. The app uses the spec's alternative instead: a style matched to the `--map-*`
tokens, made by recoloring the OSM tiles with `--map-tile-filter`. No new host, no key, and
the CSP is unchanged. The way back to CARTO is in [`theme.md` §6](../reference/theme.md#6-the-map-and-the-theme).

## Checks

- **Token contrast, automated:** `cpaltracker-react/src/styles/tokens.contrast.test.ts`,
  162 pairs (81 per theme), 0 below threshold.
- **No color literal outside `tokens.css`:** a grep for hex values in `src/**/*.{ts,tsx,css}`
  returns only comments and the kit's contrast helper (`showcase/lib/contrast.ts`, a canvas
  sentinel), in both repos.
- **Map, visual:** both themes checked in the running app (live tracking and history). Tiles
  are recolored, markers are unfiltered, and the attribution is readable. No CSP violation.
- **Kit snapshots:** the 68 `e2e/showcase.spec.ts` baselines regenerated (both themes, desktop
  and mobile), then re-run: 68 / 68 pass.
- **App unit tests (targeted: ui, map, charts, theme, history, live tracking):** 390 tests. The
  only failures are the 7 that already failed before this change (TimelineMap ×2,
  HistoryReadyPanel ×4, FleetPanel ×1).
- **E2E updated for the new decisions (not yet re-run):** KAN-176 (card colors
  `rgb(237, 241, 238)` / `rgb(29, 30, 32)`, light tile filter), KAN-223 AC2 and KAN-224 AC4
  (default follows the OS), TimelineMap F-22 (state in the dot, outline `ring-signal`).

## Open items

- `cpaltracker-react/src/pulse/palletFigure.ts` (uncommitted Pulse work by another
  contributor): anomaly dots use `--danger`; the spec says `--danger-mark`. It is a one-token
  change, left to the owner of that file.
- The shell's theme toggle is being reworked in parallel (`ThemeChoiceControl`).
  `AppShellLayout.test.tsx` ("flips the theme… no system button") will follow that rework.
- Full runtime contrast crawl of the app routes (as in [`2026-10-29-contrast.md`](2026-10-29-contrast.md)).
- Operator approval of the adapted values above.
