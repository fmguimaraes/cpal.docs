# KAN-217 / KAN-226 - History on the reference History page (manual E2E scenarios)

Spec: `cpaltracker-react/e2e/KAN-226-history-reference-page.spec.ts` (12 tests, 10 scenarios, all automated; contract and remembered-contract scenarios also in `history-dashboard.spec.ts`). Mocked `/api/v1`, contract REF-012 with two devices (CHIP-A has no mass reading).

| ID | Tag | Scenario | automation |
|----|-----|----------|------------|
| AC1 | @SI-226-1 | The header holds the `TimePicker`, the only period control (operator decision 2026-10-29: no typed Du/Au entry, no "période du … au …" sentence); picking `7 j` re-fetches with a day-granular `start`/`end`. | playwright |
| AC2 | @SI-226-2 | One header row (operator decision 2026-10-29): sr-only h1 `Historique`, the contract `Select` (`history-contract-selector`, "Choisir un contrat"), the device `FilterBar` immediately to its right, the `TimePicker` on the far right; at 390 px it wraps with no horizontal overflow. Unchecking a device adds the header chip `Appareils = 1 sur 2`; removing it re-checks every device. | playwright |
| AC2b | — | The first contract loads by default; the last chosen contract is remembered for the session (`sessionStorage`) and restored on reload; a `?contrat_id=` deep link wins (`history-dashboard.spec.ts`). | playwright |
| AC3 | @SI-226-3 | The Stat row (measurements, devices, min/max temperature) is shown and the text summary `N appareil(s), M mesure(s)` is gone. | playwright |
| AC4 | @SI-226-4 | The map is a `WidgetCard` titled `Chronologie`, keeps `.timeline-map`; the kit slider moves the instant by keyboard, the trail switch and play button work. | playwright |
| AC5 | @SI-226-5 | Map markers carry kit state classes (`ring-success`, `ring-offline`), no inline colour. | playwright |
| AC6 | @SI-226-6 | With the tile host blocked the markers stay and `timeline-tile-notice` (`role=status`) appears. | playwright |
| AC7 | @SI-226-7 | The second device keeps its series slot on all three charts although the first has no mass reading, and its marker swatch is `--chart-2`. | playwright |
| AC8 | @SI-226-8 | A sampled response shows `history-sampling-banner` as a warning `Alert` with `role=status`. | playwright |
| AC9 | @SI-226-9 | At 390 px the lot-hook table scrolls inside its container (`overflow-x: auto`), the page does not widen, numeric columns are right-aligned. | playwright |
| AC10 | @SI-226-10 | In a non-UTC browser zone the chart x values are the UTC instants, with no suffix. | playwright |
