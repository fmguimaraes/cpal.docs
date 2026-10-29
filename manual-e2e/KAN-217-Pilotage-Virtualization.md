# KAN-217 / KAN-227 - Pilotage with 5000 devices (manual E2E scenarios)

Spec: `cpaltracker-react/e2e/pilotage-virtualization.spec.ts` (4 tests: 2 scenarios x dark and light theme, all automated).
Mock: `/devices/positions.php` returns 5000 devices (`CHIP-0000` to `CHIP-4999`). Set `SHOTS_DIR` to write screenshots
(audit: `ui/audits/2026-10-29/shots/pilotage-virtualized-*.png`).

| ID | Tag | Scenario | automation |
|----|-----|----------|------------|
| V1 | @SI-227-6 | At 1440 px the table is virtualized: fewer than 100 rows in the DOM (19 measured), the header is sticky while scrolling, scrolling the container to the end mounts `CHIP-4999`, the selection survives scrolling away and back, sorting "Device" descending puts `CHIP-4999` first, no console error. Dark and light. | playwright |
| V2 | @SI-227-7 | At 390 px the list is the windowed kit `MobileList` (`data-windowed="true"`, KAN-217): fewer than 60 cards in the DOM at load, the count reads "5000 sur 5000", there is no "Afficher plus", selecting a card sets `aria-pressed`, scrolling the page to the end of the list brings `CHIP-4999` into the viewport while the mounted-card count stays below 60 throughout (24 measured) and `CHIP-0000` is unmounted, the last card can be selected, no horizontal scroll, no console error. Dark and light. | playwright |
