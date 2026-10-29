# c-PAL palette changes: what changed from the brand deck and why

Status: **all approved by the operator on 2026-10-29** (palette approval page, "I approve all").
Source of truth for values: `cpaltracker.ui-reference/src/styles/tokens.css`. Brand source: [`../reference/brand.md`](../reference/brand.md).
Rule behind every change: WCAG 2.1 AA, 4.5:1 for normal text, 3:1 for large text, form-control boundaries and graphical objects.
Ratios below are recomputed with the WCAG relative-luminance formula; "12% fill" is the color at 12% over the card.

## Summary

| # | Change | Theme | Brand | App | Kind |
|---|---|---|---|---|---|
| 1 | Orange text on light | Light | `#C57A30` | `#85511F` | value changed |
| 2 | Secondary panel | Light | `#E6DFD1` | `#ECE7DD` | value changed |
| 3 | Muted text | Dark | `#83858B` | `#96989D` | value changed |
| 4 | Muted text | Light | none | `#64666B` | new role |
| 5 | Form control border | Dark / Light | none | `#686E7F` / `#938267` | new role |
| 6 | Text on orange buttons | Both | white | `#1A1D22` | value changed |
| 7 | Focus ring | Light | none | `#85511F` | new role |
| 8 | Status colors (danger, warning, success, offline, info) | Both | none | see section 8 | new roles |
| 9 | Soft fills | Both | none | 12% of the color | new rule |
| 10 | Chart palette | Both | `#3C404A` secondary | see section 10 | changed and new |

## 1. Orange text on light: `#C57A30` to `#85511F`

- **Why:** the brand orange is 3.03:1 on the light background, which only passes for large text. Links, labels and inline numbers in an app are small. Same hue, darkened.
- **History:** the first value `#915923` passed on plain surfaces but fell to 4.14:1 over the selected-row soft fill on cards. `#85511F` fixed that. Approved earlier on KAN-201.
- **Result:** 5.89 on background, 5.29 on card, 4.74 on the 12% fill over card.
- `#C57A30` stays for fills (buttons, badges, active nav marker). `#DD8C3E` is never used as text on light (2.38:1).

## 2. Light secondary panel: `#E6DFD1` to `#ECE7DD`

- **Why:** introduced when the text orange was `#915923`, which measured 4.33:1 on `#E6DFD1`. Lightened so the orange passed (4.66:1).
- **Note:** with the final `#85511F`, the old secondary would also pass (4.96:1 vs 5.34:1 on the new one). The change is no longer needed for contrast. It is kept because it was approved, it is consistent across snapshots, and it only lightens hovered rows and side panels slightly. Revert to `#E6DFD1` if you prefer the deck value.

## 3. Dark muted text: `#83858B` to `#96989D`

- **Why:** the deck puts captions on the plain background (4.58:1, barely passes). The app puts help text, secondary table columns and timestamps inside cards, where it is 4.15:1 and fails.
- **Result:** same grey, lightened. 4.63 to 5.85:1 on every dark surface.

## 4. Light muted text (new role): `#64666B`

- **Why:** the deck has no light caption color.
- **Result:** the dark-theme grey, darkened. 5.15 on background, 4.63 on card.

## 5. Form control borders (new role): dark `#686E7F`, light `#938267`

- **Why:** the brand separators are decorative (1.44:1 dark, 1.33:1 light). AA requires 3:1 for the boundary of a form control, so an input drawn with them would be invisible.
- **Result:** the separator hue shifted until it passes. Dark 3.32 on background and 3.01 on card. Light 3.34 and 3.00.

## 6. Text on orange buttons: white to charcoal `#1A1D22`

- **Why:** white on the orange is 2.66:1 (dark) and 3.39:1 (light), so button labels fail.
- **Result:** charcoal is 6.35:1 on `#DD8C3E` and 4.99:1 on `#C57A30`.

## 7. Focus ring on light (new role): `#85511F`

- **Why:** `#C57A30` is 2.73:1 on the cream card, under the 3:1 a focus indicator needs. Reuses the text orange instead of adding a token.
- **Result:** 5.89 on background, 5.29 on card. Dark keeps `#DD8C3E` (5 to 6:1).

## 8. Status colors (new roles, decision D3)

The brand has none. Red alert, **yellow** warning (never the brand orange, so a warning cannot be confused with a primary action), green OK, grey for not reporting, blue info. Each passes as text on background, card, secondary and on its own 12% fill.

| Role | Dark | Light | Min ratio dark | Min ratio light |
|---|---|---|---|---|
| danger | `#F39083` | `#A23126` | 4.69 | 4.70 |
| warning | `#E3C34F` | `#705800` | 5.94 | 4.66 |
| success | `#5DBE8A` | `#276742` | 4.72 | 4.62 |
| offline | `#A8ABB0` | `#585A60` | 4.64 | 4.72 |
| info | `#86AFDA` | `#2B5C8A` | 4.69 | 4.76 |

- Light variants are darkened versions of the dark set.
- Label on a filled status chip: charcoal on dark (7.34 to 9.80:1), white on light (6.76 to 7.00:1).

## 9. Soft fills at 12%

- **Why:** chips, selected rows and markers use a tinted background. Contrast is computed over the real blended color, not the page color, which is stricter than computing on the page. This is what exposed the `#915923` failure in section 1.
- **Result:** orange text on its fill is 4.78 (dark) and 4.74 (light).

## 10. Chart palette (decision D4)

- **Why:** orange first, then muted cool hues so the brand orange does not drift into status meaning. Marks need 3:1 against the surface. The deck's secondary segment `#3C404A` is 1.63:1, so series 5 is lightened.
- **Dark:** `#DD8C3E` 5.75, `#6FA8C7` 5.90, `#93B79C` 6.92, `#B4A2D6` 6.62, `#676E80` 3.00 (on card).
- **Light:** `#A8692A` 3.58, `#3A6E8A` 4.48, `#4A7456` 4.31, `#6D5C99` 4.64, `#5F656F` 4.73 (on card). Series 1 darkens from `#C57A30` (2.73:1 on card).
- Series beyond five cycle the palette with a dash or marker variant, so color is never the only cue.

## Unchanged from the brand deck

Dark: background `#1A1D22`, card `#22252E`, sidebar `#20232B`, foreground `#F3F2EE`, body `#AAACB2`, orange `#DD8C3E`. Light: background `#F5F2ED`, card `#EEE6D4`, foreground `#1E2128`, body `#55585F`. Separators are decorative and unchanged.

## Verification

- Token pairs above are recomputed from the tokens.
- A runtime sampling of the rendered app (computed colors per route and theme) is in [`2026-10-29-contrast.md`](2026-10-29-contrast.md) once the fix pass finishes.
