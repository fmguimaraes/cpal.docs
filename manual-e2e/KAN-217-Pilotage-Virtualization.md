# KAN-217 / KAN-227 - Pilotage with 5000 devices (manual E2E scenarios)

Spec: `cpaltracker-react/e2e/pilotage-virtualization.spec.ts` (4 tests: 2 scenarios x dark and light theme, all automated).
Mock: `/devices/positions.php` returns 5000 devices (`CHIP-0000` to `CHIP-4999`). Set `SHOTS_DIR` to write screenshots
(audit: `ui/audits/2026-10-29/shots/pilotage-virtualized-*.png`).

| ID | Tag | Scenario | automation |
|----|-----|----------|------------|
| V1 | @SI-227-6 | At 1440 px the table is virtualized: fewer than 100 rows in the DOM (19 measured), the header is sticky while scrolling, scrolling the container to the end mounts `CHIP-4999`, the selection survives scrolling away and back, sorting "Device" descending puts `CHIP-4999` first, no console error. Dark and light. | playwright |
| V2 | @SI-227-7 | At 390 px the list is the paginated `MobileList`: 20 cards and "20 sur 5000", selecting a card sets `aria-pressed`, clicking "Afficher plus" 249 times reaches `CHIP-4999` (all 5000 cards then mounted: not virtualized, about 15 s), no horizontal scroll, no console error. Dark and light. | playwright |
