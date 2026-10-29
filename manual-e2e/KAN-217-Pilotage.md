# KAN-217 / KAN-227 - Pilotage list and transfer form (manual E2E scenarios)

Spec: `cpaltracker-react/e2e/KAN-227-pilotage-list-form.spec.ts` (5 scenarios, all automated).

| ID | Tag | Scenario | automation |
|----|-----|----------|------------|
| AC1 | @SI-227-1 | The contract choice is a kit `Select` in the page header. | playwright |
| AC2 | @SI-227-2 | Devices are kit `DataTable` rows: testid on the row, selection accent, absolute UTC time in the title. | playwright |
| AC3 | @SI-227-3 | Above 50 devices the table is virtualized (`data-virtualized="true"`, fewer rows mounted than devices) and scrolling its container still reaches the last row. | playwright |
| AC4 | @SI-227-4 | At 390 px the devices are `MobileList` cards with a count and "Afficher plus". | playwright |
| AC5 | @SI-227-5 | A transfer is entered with `Select` + `DateTimeField` and posted in the datetime shape AAAA-MM-JJTHH:MM. | playwright |
