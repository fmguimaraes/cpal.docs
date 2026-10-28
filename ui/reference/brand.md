# c-PAL brand — constraints for the app UI

Source: Confluence, [Référentiel de design — présentations c-PAL](https://c-pal.atlassian.net/wiki/spaces/COC/pages/49119233)
(read 2026-10-28). That page is the source of truth for the brand. It's written
for presentation decks, so this file only records what an app UI takes from it, and
the checks that apply to a screen. If the Confluence page changes, update this file.

These are **fixed constraints**: whichever UI reference direction is chosen (for
example `datadog-inspired/`), its colors are replaced by these.

## What carries over

**Character.** A dark background dominates; there's one accent, orange, and
everything else is neutral. Orange *signals*, it doesn't decorate. Spare, few words,
lots of empty space. Illustrate with real data and the real product, never generic
imagery.

**Dark theme (primary).** *App value* is what the tokens use. Where it differs from
the brand value, the comment says why; the checks are in
[Contrast checks](#contrast-checks-wcag-21-aa).

| App role | Brand role | Brand value | App value | Why changed |
|---|---|---|---|---|
| background | Fond principal | `#1A1D22` | `#1A1D22` | |
| card / panel | Carte / panneau | `#22252E` | `#22252E` | |
| sidebar | Bandeau latéral | `#20232B` | `#20232B` | |
| secondary panel, avatar background | Panneau latéral clair-sombre | `#2C2F38` | `#2C2F38` | |
| inset / quote background | Encadré citation | `#24262C` | `#24262C` | |
| media frame (screenshots, map frame) | Fond d'image | `#0E0F12` | `#0E0F12` | |
| foreground (titles) | Titres / blanc cassé chaud | `#F3F2EE` | `#F3F2EE` | |
| body text | Corps de texte | `#AAACB2` | `#AAACB2` | |
| muted text | Légendes / atténué | `#83858B` | **`#96989D`** | The brand value is 4.15:1 on cards, below AA's 4.5:1 for normal text. A deck puts captions on the plain background; an app puts muted text (help text, table secondary columns, timestamps) inside cards and panels everywhere. Same grey, lightened: ≥ 4.6:1 on every dark surface |
| separator | Filets de séparation | `#353841` | `#353841` | Decorative separators have no contrast requirement |
| input border | — | — | **`#686E7F`** | New role. A deck has no inputs. AA requires 3:1 for the boundary of a form control; the separator is 1.44:1, so an input drawn with it would be invisible. The separator's hue, lightened to 3:1 on background and card |
| primary / accent | Orange (principal) | `#DD8C3E` | `#DD8C3E` | |
| text on primary | — | white in decks | **`#1A1D22`** | White on the orange is 2.66:1 and fails for button labels. Charcoal on the orange is 6.35:1 |
| focus ring | — | — | `#DD8C3E` | New role; the orange is 5–6:1 on every dark surface, above the 3:1 needed |

**Light theme.** Derived from the deck's light half-slide ("demi-slide clair").

| App role | Brand value | App value | Why changed |
|---|---|---|---|
| background | `#F5F2ED` | `#F5F2ED` | |
| card | — | `#EEE6D4` | The deck has no light card; this is the public site's cream card |
| foreground (titles) | `#1E2128` | `#1E2128` | |
| body text | `#55585F` | `#55585F` | |
| muted text | — | **`#64666B`** | New role; the deck has no light caption color. The dark-theme grey, darkened to ≥ 4.6:1 on background and card |
| separator | `#D9D3C9` | `#D9D3C9` | Decorative only |
| input border | — | **`#938267`** | New role. The separator is 1.33:1, below the 3:1 a form control boundary needs. The separator's warm hue, darkened to 3:1 on background and card |
| primary / accent (fills: buttons, badges, active nav marker) | `#C57A30` | `#C57A30` | Kept for fills, with charcoal text on top |
| text on primary | — | **`#1A1D22`** | White on `#C57A30` is 3.39:1 and fails for button labels; charcoal is 4.99:1 |
| primary text (links, eyebrows below 18.7 px bold, orange numbers in text) | `#C57A30` | **`#85511F`** | The brand's dark orange is 3.03:1 on the light background, which passes only for large text. Small orange text in an app (links, labels, inline numbers) needs 4.5:1. Same hue, darkened: 5.1:1 on background, 4.6:1 on card |
| focus ring | — | **`#85511F`** | `#C57A30` is 2.73:1 on the cream card, below 3:1. Reuses the primary-text orange rather than adding a token |

`#DD8C3E` is never used for text or icons on a light background (2.38:1).

**Charts.**

| Element | Brand value | App value | Why changed |
|---|---|---|---|
| main series (`chart-1`) | `#DD8C3E` | `#DD8C3E` | |
| secondary segment | `#3C404A` | **`#676E80`** | The brand value is 1.63:1 against the background. In a deck the donut is decorative, but in the app a chart segment carries data, and AA requires 3:1 for graphical objects needed to understand the content. Same hue, lightened to 3:1 on background and card |
| axis labels | `#83858B` | **`#96989D`** | Same reason as muted text: axis labels sit on cards |
| grid | `#2C2F38` | `#2C2F38` | Decorative gridlines have no contrast requirement |

The brand defines one series only. The app needs a categorical palette
(`chart-1…5`) for multi-device history charts. It must start with the orange, keep
every series at ≥ 3:1 on the background in both themes, and is to be derived from
the chosen direction.

**Typography.** A sans-serif everywhere (the brand uses Arial, bold for titles); a
serif (Georgia) **only** for big numbers and symbols ("80 %", "01", roadmap
dates). In the app: Georgia for KPI/stat callouts only; tables and data use the sans
with tabular figures (Georgia's figures aren't tabular, so columns wouldn't align).

**Signature components → kit items.**

| Brand element | Kit item |
|---|---|
| Eyebrow: short orange uppercase text, letter-spaced, with a short orange rule below | Page header eyebrow (NAV-HEADER); section labels |
| Dark card, optional orange left border (lists) or top border (steps) | DAT-CARD variants |
| Numbered list: big orange serif number + bold title + description, separators between rows | DAT-LIST variant |
| Stat callout: big serif number + small label | DAT-STAT |
| Quote box: inset background + orange left border | FBK-ALERT "note" variant |
| Timeline: grey line, orange dots, serif date above, label below | DAT-TIMELINE (history views) |
| Donut: orange + `#3C404A`, about 62 % hole, big serif % in the centre | DAT-CHART |
| "c-PAL" pill: orange outline, no fill, orange text | CMP-BADGE "brand" variant |
| Product screenshot on `#0E0F12` with a soft shadow | Media/map frame (DAT-MAP) |

**Logo.** Never redrawn or approximated. Color variants are the canonical file
recolored. Orange `#DD8C3E` is the default; grey `#9498A0` is the discreet mark;
off-white `#D7D9DE` and white are for dark backgrounds; dark `#1A1D22` is for light
backgrounds. In this repo set: `c-PAL.web` has orange, grey and dark;
`cpaltracker-react/public/logo-cpal.png` is one version. **Off-white and white are
missing**; get them from the brand's `logos/` folder.

## What doesn't carry over

- Slide sizes in points and inches, the 16:9 layout, logo coordinates. The app's type
  scale and spacing come from the kit's tokens.
- The dark/light "sandwich" rhythm between slides. The app has a theme switch
  instead.
- pptxgenjs.

## Contrast checks (WCAG 2.1 AA)

AA requires 4.5:1 for normal text, and 3:1 for large text (≥ 24 px, or ≥ 18.7 px
bold) and for UI boundaries.

| Pair | Ratio | Verdict |
|---|---|---|
| `#F3F2EE` titles on `#1A1D22` | 15.1 | pass |
| `#AAACB2` body on `#1A1D22` / on card `#22252E` | 7.5 / 6.7 | pass |
| `#83858B` muted on `#1A1D22` | 4.58 | pass, barely |
| `#83858B` muted on card `#22252E` | 4.15 | **fail** for normal text: muted text inside cards needs a lighter value |
| `#DD8C3E` orange text on `#1A1D22` / on card | 6.4 / 5.8 | pass |
| White on `#DD8C3E` (orange button) | 2.66 | **fail**: orange buttons take dark text |
| `#1A1D22` on `#DD8C3E` | 6.4 | pass: primary button = orange fill + charcoal text |
| `#1E2128` titles / `#55585F` body on `#F5F2ED` | 14.4 / 6.4 | pass |
| `#C57A30` dark-orange text on `#F5F2ED` | 3.03 | **large text only**: fine for an eyebrow at 18.7 px+ bold, not for body text or small links |
| White on `#C57A30` | 3.39 | **fail** for normal text |
| `#1A1D22` on `#C57A30` (light-theme primary button) | 4.99 | pass |
| `#DD8C3E` on `#F5F2ED` | 2.38 | **fail**: on light backgrounds use `#C57A30`, never `#DD8C3E` for text |
| Borders `#353841` on `#1A1D22` / `#D9D3C9` on `#F5F2ED` | 1.44 / 1.33 | decorative separators only; input borders and focus rings need ≥ 3:1 |

The failures are resolved by the app values in the theme tables above. With those
values:

| Pair | Ratio | Verdict |
|---|---|---|
| Muted `#96989D` on every dark surface (`#1A1D22` … `#2C2F38`) | 4.63–5.85 | pass |
| Muted `#64666B` on `#F5F2ED` / `#EEE6D4` | 5.15 / 4.63 | pass |
| Primary text `#85511F` on `#F5F2ED` / `#EEE6D4` / over the 12 % orange soft fill on card | 5.89 / 5.29 / 4.74 | pass (`#915923`, the first value, failed over the soft fill: 4.14) |
| Charcoal `#1A1D22` on `#DD8C3E` / on `#C57A30` | 6.35 / 4.99 | pass |
| Input border `#686E7F` on `#1A1D22` / `#22252E` | 3.32 / 3.01 | pass (3:1) |
| Input border `#938267` on `#F5F2ED` / `#EEE6D4` | 3.34 / 3.00 | pass (3:1) |
| Focus ring `#85511F` on `#F5F2ED` / `#EEE6D4` | 5.89 / 5.29 | pass (3:1) |
| Chart secondary `#676E80` on `#1A1D22` / `#22252E` | 3.31 / 3.00 | pass (3:1) |

Method: each failing brand color keeps its hue and saturation; only its lightness
moves, by the smallest step that passes on every surface it's used on (with a small
margin for text). Ratios are WCAG 2.1 relative-luminance contrast.
