# c-PAL blue theme: palette changes from the charter, and contrast audit

Status: **implemented, uncommitted, awaiting operator review** (2026-11-24). Typography
(Montserrat, Georgia dropped) and the light default (D7) were decided by the operator on
2026-11-24; the color values below are proposed and verified, not yet approved.
Supersedes the orange palette of [`2026-10-29-palette-changes.md`](2026-10-29-palette-changes.md).

| | |
|---|---|
| Charter | [Confluence COC 116588545](https://c-pal.atlassian.net/wiki/spaces/COC/pages/116588545/2026-11-20+Charte+graphique+Pr+sentation+candidature+A+roport+Marseille+Provence), slides in [`../reference/directions/blue-theme/`](../reference/directions/blue-theme/) |
| Brand constraints | [`../reference/brand.md`](../reference/brand.md) (rewritten for the blue charter) |
| Source of truth for values | `cpaltracker.ui-reference/src/styles/tokens.css`, copied unmodified to `cpaltracker-react/src/styles/tokens.css` |
| Scope | Theming only: tokens, font, default theme, logo files in the kit, comments and kit copy that named orange or Georgia. No layout, behavior or `data-testid` change |

Rule behind every change: WCAG 2.1 AA, 4.5:1 for normal text, 3:1 for large text, form
control boundaries, focus indicators and graphical objects. Method: a brand color that
fails keeps its hue and saturation; only its lightness moves, by the smallest step that
passes on every surface it is used on.

## Summary

| # | Change | Theme | Brand | App | Kind |
|---|---|---|---|---|---|
| 1 | Accent | Both | orange `#DD8C3E` | cyan `#08B9E8` (fills) | charter change |
| 2 | Text on primary | Both | white | anthracite `#111820` | value changed (white is 2.30:1) |
| 3 | Cyan as text | Light | `#08B9E8` / `#0A8FB5` | `#086F8C` | value changed |
| 4 | Focus ring, chart-1 | Light | `#0A8FB5` Cyan foncé | `#0A8FB5` | brand value, new role |
| 5 | Muted text | Light / Dark | `#668AA5` Bleu acier | `#4D6B82` / `#809EB4` | value changed |
| 6 | Input border | Both | `#668AA5` Bleu acier | `#668AA5` | brand value, new role |
| 7 | Body text | Light | `#111820` | `#30435A` | value changed (second text level) |
| 8 | Separator | Light | `#AABAC6` | `#D5E0E8` | value changed (decorative) |
| 9 | Light surfaces | Light | `#FFFFFF`, `#EAF2F7` | + `#F2F6F9` inset, `#E3ECF2` hover, `#F3F7FA` sidebar | new roles |
| 10 | Dark theme | Dark | none (charter is white only) | anthracite-based, see `brand.md` | derived |
| 11 | Info status | Both | none | periwinkle `#4A4FB0` / `#A9B1F2` (was blue) | changed: never the accent |
| 12 | Chart palette | Both | cyan bars | cyan, violet, sage, sand, slate | changed: blue `chart-2` removed |
| 13 | Typography | Both | Montserrat | Montserrat 400–800 self-hosted; ExtraBold for key figures | charter change (Arial, Georgia dropped) |
| 14 | Default theme (D7) | — | white charter | light by default | operator decision |
| 15 | Light map tiles | Light | — | `saturate(0.55)` (sepia removed) | cool theme, no cream cast |

## Rationale per change

1. **Accent.** The charter has one accent, cyan `#08B9E8`. It replaces orange for fills
   (primary button, badges, active nav marker), in both themes.
2. **Text on primary.** White on cyan is 2.30:1. Anthracite on cyan is 7.76:1: the
   primary button stays "bright fill, dark label", as with orange.
3. **Cyan as text, light.** Cyan is 2.30:1 on white, Cyan foncé `#0A8FB5` 3.74:1 (large
   text only). Cyan foncé darkened to `#086F8C`: 5.73 background, 5.06 card, 4.70 hover,
   ≥ 4.60 over the 12 % cyan soft fill (selected rows). Dark keeps `#08B9E8` (≥ 5.67).
4. **Focus ring and chart-1, light.** Cyan is 2.03:1 on the card. The charter's own
   Cyan foncé passes 3:1 everywhere (3.74 background, 3.31 card, 3.13 hover) and is used
   as it is. Inside the dark map frame (`--ring-media`) it is 4.77 on the frame and 3.26
   on light tiles.
5. **Muted text.** Bleu acier is the charter's caption color but is 3.66 / 3.23:1 on
   white / card. Light `#4D6B82` (≥ 4.60 on every light surface), dark `#809EB4` (≥ 4.65
   on every dark surface and over the soft fill). Inside `bg-primary-soft` muted text
   still reads as body (kept rule, no margin there: 4.51 light).
6. **Input border.** Bleu acier as it is passes 3:1 on every surface of both themes
   (lowest: 3.05 on the light hover surface).
7. **Body text, light.** The charter sets running text in anthracite. Dense tables need a
   quieter second level: anthracite's hue lightened to `#30435A` (10.1 / 8.9:1).
8. **Separator, light.** Gris industriel draws the rules of one comparison table in the
   deck; the app draws separators everywhere, so a lighter step of the same hue.
9. **Light surfaces.** The hover surface was first `#E0EAF1`; the input border fell to
   2.996:1 on it, so it was lightened to `#E3ECF2` (3.05).
10. **Dark theme.** Kept (D7 keeps dark available). Surfaces step up from the anthracite
    `#111820`; body is Gris industriel `#AABAC6` as it is.
11. **Info status.** The former info blue (`#2B5C8A` / `#86AFDA`) sat next to the accent.
    Moved to periwinkle, mirroring D3's "warning is never the brand orange".
12. **Charts.** Former `chart-2` was blue and read as the accent. New order: cyan, violet,
    sage, sand, slate. Every series ≥ 3:1 on background and card in both themes (lowest:
    light chart-1 on card 3.31).
13. **Typography.** `@fontsource/montserrat` 5.3.0, weights 400/500/600/700/800, bundled
    same-origin (CSP `default-src 'self'` unchanged). The `font-serif` role is renamed
    `font-display`; big numbers use `font-display font-extrabold`. The Plotly donut
    center uses `weight: 800`, and its number/label are now anchored bottom/top around
    the hole's center (the taller Montserrat glyphs overlapped the label in the small
    fleet-panel donut).
14. **Default theme.** `theme-init.js` and `ThemeProvider` default to `light`.

## Contrast results

**Token matrix** (every hex in `tokens.css`, per theme): text roles (foreground, body,
muted, primary-text, sidebar-foreground) on all seven surfaces and over the opaque
primary soft fill; every `*-foreground` on its surface; status-foreground on each filled
status; each status on background, card, secondary and sidebar and on its own 12 % soft
fill over them; input border and ring on all surfaces; ring-media on the media frame;
chart-1…5 on background and card; axis labels on card.

| | Light | Dark |
|---|---|---|
| Pairs | 117 | 117 |
| Below threshold | **0** | **0** |
| Lowest margins | input on hover 3.05, ring on hover 3.13, chart-1 on card 3.31 | chart-5 on card 3.63, muted on secondary 4.97 |

**Runtime** (reference kit production build, Chromium, 1440 px): the Foundations page,
which computes each token pair live from the rendered theme, shows **96 / 96
"Conforme", 0 "Non conforme"** in both themes. Montserrat 400–800 loaded on every page.
Pages reviewed visually in both themes: login, overview, live tracking (map), history,
devices, foundations, charts, data & navigation.

The full route crawl of [`2026-10-29-contrast.md`](2026-10-29-contrast.md) (text nodes,
controls, focus stops on the 9 app routes) is **not re-run yet**: to do once the
History unit/E2E failures below are fixed, so every route renders.

## Tests

- `cpaltracker-react` unit (vitest): 567 passed, 8 failed. The 8 (HistoryReadyPanel,
  TimelineMap, HistoryDashboard, FleetPanel collapse) **fail identically on a clean
  checkout of `71d1195` without these changes**: pre-existing.
- E2E updated for the new decisions: KAN-167 (primary is the same in both themes now;
  compares `--card`), KAN-176 (new card colors, light tile filter), KAN-223 and KAN-224
  (light default, OS set to dark to prove the default is not the system choice).
  Theme E2E (5 specs, 21 tests): 18 passed; 3 failures not caused by this change: KAN-176 ×2 wait for a
  History chart that does not render (the History breakage above), KAN-171 FR12
  expects the login card shadow that the uncommitted split-screen login rework removed.
- Reference kit: the 68 `e2e/showcase.spec.ts` baselines regenerated with the blue
  theme (every route, both themes, desktop and mobile; Playwright 1.64.1, Chromium
  1248), then re-run: 68 / 68 pass. `app-device-detail` (`/app/devices/CP-0412`)
  shows "Appareil introuvable" because no mock device has that ID. The old orange
  baseline showed the same page, so it is not a theme regression.

## Open items

- Square logo mark: not supplied. Since 2026-11-25 the collapsed rail and the favicon use
  the rail mark (cyan dot in a halo, theme-aware), see `brand.md` § Logo.
- Operator approval of the color values above.
- Full runtime contrast crawl.
- The public site `c-PAL.web` and the monolith `cpaltracker.web` still use orange: out
  of scope (platform = `cpaltracker-react` + reference kit).
