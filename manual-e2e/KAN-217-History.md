# KAN-217 / KAN-226 - History on the reference History page (manual E2E scenarios)

Spec: `cpaltracker-react/e2e/KAN-226-history-reference-page.spec.ts` (10 tests, 9 scenarios, all automated). Mocked `/api/v1`, contract REF-012 with two devices (CHIP-A has no mass reading).

| ID | Tag | Scenario | automation |
|----|-----|----------|------------|
| AC1 | @SI-226-1 | The header holds the `TimePicker`; picking `7 j` re-fetches with a day-granular `start`/`end`. The typed entry (`date-range-start`, placeholder `AAAA-MM-JJ`) sits next to it and a malformed date shows the French error with no request. | playwright |
| AC2 | @SI-226-2 | `FilterBar` shows `Contrat = REF-012`; unchecking a device adds `Appareils = 1 sur 2`; removing each chip resets the selection (contract chip clears the contract). | playwright |
| AC3 | @SI-226-3 | The Stat row (measurements, devices, min/max temperature) is shown and the text summary `N appareil(s), M mesure(s)` is gone. | playwright |
| AC4 | @SI-226-4 | The map is a `WidgetCard` titled `Chronologie`, keeps `.timeline-map`; the kit slider moves the instant by keyboard, the trail switch and play button work. | playwright |
| AC5 | @SI-226-5 | Map markers carry kit state classes (`ring-success`, `ring-offline`), no inline colour. | playwright |
| AC6 | @SI-226-6 | With the tile host blocked the markers stay and `timeline-tile-notice` (`role=status`) appears. | playwright |
| AC7 | @SI-226-7 | The second device keeps its series slot on all three charts although the first has no mass reading, and its marker swatch is `--chart-2`. | playwright |
| AC8 | @SI-226-8 | A sampled response shows `history-sampling-banner` as a warning `Alert` with `role=status`. | playwright |
| AC9 | @SI-226-9 | At 390 px the lot-hook table scrolls inside its container (`overflow-x: auto`), the page does not widen, numeric columns are right-aligned. | playwright |
| AC10 | @SI-226-10 | In a non-UTC browser zone the chart x values are the UTC instants, with no suffix. | playwright |
