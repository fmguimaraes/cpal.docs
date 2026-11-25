# c-PAL brand — constraints for the app UI

Source: Confluence, [2026-11-20 Charte graphique Présentation candidature Aéroport Marseille Provence](https://c-pal.atlassian.net/wiki/spaces/COC/pages/116588545/2026-11-20+Charte+graphique+Pr+sentation+candidature+A+roport+Marseille+Provence)
(read 2026-11-24), which applies the
[charte commerciale](https://c-pal.atlassian.net/wiki/spaces/COC/pages/100958209/2026-10-15+Charte+graphique+Pr+sentation+commerciale)
(white, cyan `#08B9E8`, Montserrat). Visual reference: the Ariane deck slides in
[`directions/blue-theme/`](directions/blue-theme/). That page is the source of truth for
the brand. It's written for presentation decks, so this file only records what an app UI
takes from it, and the checks that apply to a screen. If the Confluence page changes,
update this file.

**Colors since 2026-11-25:** [`theme.md`](theme.md) ("Clair minéral" / "Sombre graphite").
This file keeps what the app takes from the charter: typography, logo, signature elements.

**Supersedes** the orange charter ([Référentiel de design — présentations c-PAL](https://c-pal.atlassian.net/wiki/spaces/COC/pages/49119233),
dark background, orange `#DD8C3E`, Arial and Georgia), which the app followed until
2026-11-24. Palette changes and their checks:
[`../audits/2026-11-24-blue-theme-palette-changes.md`](../audits/2026-11-24-blue-theme-palette-changes.md).

These are **fixed constraints**: whichever UI reference direction is chosen, its
colors are replaced by these.

## What carries over

**Character.** Since 2026-11-25 the app follows the operator's theming spec "Clair minéral"
(light) / "Sombre graphite" (dark), not the deck's white-and-cyan look: enterprise, calm,
slightly earthy, with **color for the data and status, not for the chrome**. What the app still
takes from the charter: Montserrat, the logo and the brand cyan **inside the logo only**, few
words, and illustration with real data. Cards are flat, with no shadow.

**Colors.** Every app color, its role, the values changed for WCAG AA, and how the theme reaches
the map and the charts: [`theme.md`](theme.md). The brand cyan `#08B9E8` is the token
`--brand`. It is used by the logo files, the rail mark and the favicon, and by nothing else in
the UI. The app's signal color is the muted teal `--signal` (`#2997AC` light, `#4DB6C6` dark).

The blue-charter app palette (2026-11-24: white, `#EAF2F7` cards, cyan fills) is superseded. Its
tables and checks are kept in
[`../audits/2026-11-24-blue-theme-palette-changes.md`](../audits/2026-11-24-blue-theme-palette-changes.md).

**Status colors** (D3) are not brand colors: red alert, amber warning, green OK, grey not
reporting, periwinkle info. Values in [`theme.md`](theme.md#values).

**Typography.** **Montserrat** everywhere, self-hosted (`@fontsource/montserrat`, no
CDN: CSP unchanged). Weights: 400 text, 500 labels, 600/700 headings, **800
ExtraBold for key figures** (the deck's "chiffre clé"): KPI tiles, numbered lists,
timeline dates, donut center, through `font-display font-extrabold`. Tables keep
tabular figures (`tabular-nums`). Georgia is no longer used.

**Signature components → kit items.** In the app, the deck's cyan becomes `--signal`.

| Brand element | Kit item |
|---|---|
| Title with the final keyword in cyan; category label in bold uppercase, letter-spaced | Page header eyebrow (NAV-HEADER); section labels |
| Card: rounded blue-grey rectangle, no outline, no shadow | DAT-CARD plain |
| Emphasis card: white, 1.5 pt cyan outline | DAT-CARD accent variants |
| Key figure: ExtraBold cyan number, label above, text below | DAT-STAT |
| Numbered badge: white circle, cyan outline, cyan number | DAT-LIST |
| Planning: rounded cyan bars on a month grid | DAT-TIMELINE (history views) |
| Line icon (Lucide) in a cyan-outlined circle | Icons are Lucide already; circle framing only in empty states |

**Logo.** Never redrawn or approximated. The app uses the wordmark files
`c-pal-black-transparent.png` (light backgrounds) and `c-pal-white-transparent.png`
(dark backgrounds), in `public/` of `cpaltracker-react` and the reference kit. There is
no square icon variant of the logo. Where a square mark is needed, the app uses the
**rail mark** (2026-11-25, `src/ui/patterns/rail-mark.tsx`): a cyan `#08B9E8` dot in a
soft cyan halo (the charter's map-marker motif), not a redrawn logo. It is
shown in the collapsed sidebar rail (dot `--brand`, halo 20 % `--brand`), and it is the favicon: `favicon.svg` (halo follows the browser's
`prefers-color-scheme`), with `favicon.ico` 16/32/48, `favicon-32.png` and
`apple-touch-icon.png` (white tile) as fallbacks. If a square c-PAL logo is supplied, it
replaces the rail mark in both places.

## What doesn't carry over

- Slide sizes, margins in cm, the 16:9 layout. The app's type scale and spacing come
  from the kit's tokens.
- The deck's photos with GPS markers (illustration renders): the app illustrates with
  data only.
- The slide-9 exception (pure yellow and red text).

## Contrast checks (WCAG 2.1 AA)

The brand cyan is 2.30:1 on white: it is never text, a thin icon or a UI boundary on a light
background. That is one reason it stays in the logo. The app's color pairs and their ratios
(both themes, checked by a unit test over `tokens.css`) are in
[`theme.md`](theme.md#where-the-values-differ-from-the-spec-wcag-21-aa).
