# KAN-339 - Page Load Performance (manual E2E scenarios)

Requirement IDs only (text lives in `features/Page-Load-Performance.md`).
Specs: `cpaltracker-react/e2e/KAN-344-non-blocking-loading.spec.ts`, `cpaltracker-react/e2e/KAN-345-client-cache.spec.ts`, `cpaltracker-react/e2e/KAN-350-pulse-future-reports.spec.ts` and `cpaltracker-react/e2e/KAN-353-switch-keeps-no-stale-data.spec.ts`.
Tags: `e2e` (Playwright, automated), `manual`, `unit-only` (covered by the named test, no browser scenario).

## KAN-344 - Non-blocking loading UI

| ID | Scenario (Given / When / Then) | Covers | automation |
|----|-------------------------------|--------|------------|
| 344-1 | Given every data API response is delayed, When `/pulse` opens, Then shell, map with tiles and the "Chargement des positions" loader show within the NFR4 budget; no skeleton replaces the map; markers follow when data arrives. (spec test: "AC12 / AC19: /pulse shows shell, map ...") | AC12, AC19 | e2e |
| 344-2 | Same as 344-1 on `/live-tracking`. (spec test: "AC12 / AC19: /live-tracking ...") | AC12, AC19 | e2e |
| 344-3 | Given delayed data, When Pulse opens, Then the side-panel anomalies cell shows row skeletons (status role) that contrast with their surface. | AC13, AC19 | e2e |
| 344-4 | Given contracts answered and anomalies delayed, When the Pulse drawer opens, Then its skeletons stand out from the card surface. | AC13 | e2e |
| 344-5 | Given delayed data, When Alerts opens, Then breadcrumb, status filter with count placeholders and list skeleton show at once. | AC15, AC19 | e2e |
| 344-6 | Given delayed data, When Pilotage opens, Then breadcrumb and a list skeleton show at once. | AC14, AC19 | e2e |
| 344-7 | Given a loading Pilotage workspace, Then the skeleton is table-shaped at md+ and card-shaped below md. | AC14 | e2e |
| 344-8 | Given the positions request fails, When Pulse/Suivi en direct shows the error, Then the map stays and "Reessayer" reloads it. | AC17, EC15 | e2e |
| 344-9 | Given the contract list is still loading, Then the map and loader show and no positions request is sent. Test: `ContractPositions.test.tsx` "shows the map and the loader without fetching while the contract is unknown (EC17)". | EC17 | unit-only |
| 344-10 | Loaders and skeletons are announced and removed when content shows. Tests: `skeleton-rows.test.tsx` "announces itself and draws the requested number of rows (FR23)", `DeviceListSkeleton.test.tsx` "is one announced status region ...". | AC18 | unit-only |
| 344-11 | Error, empty, zero-contract and 404 states keep texts and test IDs. Tests: the existing `ContractPositions.test.tsx`, `Alerts.test.tsx`, `DevicePilotage.test.tsx` suites. | AC17, EC18 | unit-only |
| 344-12 | Given contracts are loading on History, Then content-shaped skeletons replace the old loading text. (spec: "344-12 / AC16: History shows its content-shaped skeleton, not text, while the contracts are delayed", added in KAN-349). The audit of every data route is recorded in `cpaltracker.web/tools/perf/baselines/REPORT-KAN-339.md` (one-time review). | AC16 | e2e |
| 344-12b | Alerts shows the status filter with count placeholders before any answer. Test: `Alerts.test.tsx` "shows the filter with count placeholders at once, before any answer (AC15, AC18)". | AC15, AC18 | unit-only |
| 344-13 | Tile failure and very slow responses (EC16, EC21) and the ineligible/empty Pilotage states (EC22) keep the page usable: no browser scenario; unchanged paths covered by `ContractPositions.test.tsx` and `DevicePilotage.test.tsx`. | EC16, EC21, EC22 | unit-only |

## KAN-345 - Client cache with background revalidation

| ID | Scenario (Given / When / Then) | Covers | automation |
|----|-------------------------------|--------|------------|
| 345-1 | Given Alerts was visited, When the user leaves and returns with the revalidation held, Then the last list shows at once with no skeleton, and updates in place (1 to 2 alertes) when the answer differs. (spec: "AC20: Alerts revisit shows the cached list at once ...") | AC20, EC13 | e2e |
| 345-2 | Given a cached Alerts list, When the background revalidation fails (503), Then the list stays and no error state shows. (spec: "AC20: a failed background revalidation keeps ...") | AC20 | e2e |
| 345-3 | Given a cached Alerts list older than 5 minutes (fake clock), When the user returns, Then the old list is not shown and the skeleton shows until data arrives. (spec: "AC20: an entry older than the maximum age ...") | AC20, EC12 | e2e |
| 345-4 | Given Pulse was visited, When the user returns with the revalidation held, Then the markers stay and the positions loader never shows. | AC20 | e2e |
| 345-5 | Given Pilotage was visited, When the user returns with the revalidation held, Then the device list stays and no skeleton shows. | AC20 | e2e |
| 345-6 | Given user A viewed Alerts, When A logs out and another user logs in with the response held, Then the skeleton shows, none of A's data appears, and nothing cached is in `localStorage`/`sessionStorage`. (spec: "AC21: logout clears the cache ...") | AC21, EC14 | e2e |
| 345-7 | Hit, expiry, per-contract and per-parameter keys, no browser storage. Tests in `src/api/responseCache.test.ts`: "hit: ...", "expiry: ...", "keys differ per contract and parameters", "writes nothing to browser storage (FR26)". | FR24, FR26, EC12 | unit-only |
| 345-8 | Contract switch in flight / logout in flight: the response is never stored. Test: "a response that started before a clear is not stored (EC11, EC14)". Also "a slower older response never overwrites a newer-started one for the same key". | EC11, EC14 | unit-only |
| 345-9 | Errors (4xx, 5xx, network) are never cached; a failed refetch leaves the cached value. Tests: "errors are never cached (EC20)", "a failed fetch leaves the cached value in place and caches nothing new (EC20, FR27)". | EC20, FR27 | unit-only |
| 345-10 | A transition or comment invalidates only that contract's anomalies; a failed mutation invalidates nothing. Tests: "transition and comment invalidate the contract anomalies only (FR25)", "a failed mutation invalidates nothing". | AC21 | unit-only |
| 345-11 | A transfer or config save invalidates positions (not anomalies). Test: "transfer and config invalidate positions, not anomalies (FR25)". | AC21 | unit-only |
| 345-12 | Refresh bypasses the cache and updates it. Test: "fetchers always hit the network and refresh the cached value (refresh bypass)". | AC21 | unit-only |
| 345-13 | Session end and identity change clear everything. Test: "setAuthToken clears everything on logout and on another identity (FR25, EC14)". | AC21, EC14 | unit-only |
| 345-14 | A cached snapshot is shown at once in `ContractPositions`, updates in place, and a failed revalidation keeps it and reports `failed`. Tests: `ContractPositions.test.tsx` "KAN-345" describe; `useLoad.test.tsx`. | AC20, FR27 | unit-only |

| 345-15 | Reopening the Pulse pallet panel shows its cached alerts at once and revalidates (KAN-349). Test: `PalletPanel.test.tsx` cache-peek test (pill on the first render, fetch still called). The panel's history is outside FR24's cached set, so its chart skeleton on reopen is expected. | FR24, AC20 | unit-only |
| 345-16 | A new data key with nothing cached (another contract, period, pallet or anomaly) shows the loading state, never the previous key's data; a new load or `reload()` of the same key keeps it (KAN-353). Tests: `useLoad.test.tsx` "useLoad on a data key change (KAN-353)", `useAnomalyDetail.test.tsx`, `PalletPanel.test.tsx`. | EC11, FR27 | unit-only |

## KAN-353 - No previous key's data after a switch (follow-up bug)

Spec: `cpaltracker-react/e2e/KAN-353-switch-keeps-no-stale-data.spec.ts` (tag `@SI-353`).

| ID | Scenario (Given / When / Then) | Covers | automation |
|----|-------------------------------|--------|------------|
| 353-1 | Given the Pulse anomalies drawer shows contract A's alerts, When the user switches to contract B with B's answer held, Then the drawer shows its skeleton and none of A's alerts, then B's alerts when the answer arrives. | AC21, EC11 | e2e |
| 353-2 | Given the Pulse pallet panel is open with its charts, When the user presses refresh, Then the charts stay (no skeleton) while the positions revalidate; the positions loader never shows. | AC20, EC13, FR27 | e2e |

## KAN-350 - Pulse map keeps reports dated after now (follow-up bug)

Pulse FR41 (amended 2026-11-26) and EC11 in `features/IN-PROGRESS-Pulse.md`. Positions and pulse responses are route-stubbed.

| ID | Scenario (Given / When / Then) | Covers | automation |
|----|-------------------------------|--------|------------|
| 350-1 | Given a pallet whose last report is in 2029, When Pulse opens on the "1 j" or "1 mois" preset, Then it is drawn ("Palettes 1/1", one marker) and `positions-period-empty` is absent. | Pulse FR41, EC11 | e2e |
| 350-2 | Given "1 mois", When a pallet's last report is 45 days old and another's is in 2029, Then the old one is not on the map (Localiser disabled), the 2029 one is, and no notice shows. | Pulse FR41 | e2e |
| 350-3 | Given typed custom days ending 10 days ago, When the last reports are in 2029 and 1 hour ago, Then both are hidden, the notice "Aucun appareil n'a émis de position sur cette période." shows and the map stays mounted. | Pulse FR41, Page-Load FR22 | e2e |
| 350-4 | Given the default period and the 2029 pallet unticked in the pallet list (hidden), When the user clicks its Localiser, Then it is selected (`palette=` in the URL), its pallet panel opens and no notice shows. | Pulse FR41, EC11 | e2e |
| 350-5 | Pharma demo check: Given contract MAIN-INNOPHARM, When Pulse opens on each preset, Then the map draws the pallets the list shows and no "no position" notice appears. | Pulse FR41 | manual |

## Backend stories

KAN-340, KAN-341, KAN-342, KAN-343 and KAN-346 have no observable UI change. They are verified by PHPUnit and by `make perf-compare` (before/after timing and JSON diff). KAN-347 (verification and tooling: the cold-run script, the baselines, the MariaDB 10.11 stand-in and the report) has no UI either; it is covered by `tests/Perf/ColdScriptTest` and the `make perf*` targets.

| ID | Scenario | automation |
|----|----------|------------|
| B-1 | Backend performance and compression stories: same JSON, faster responses. Verified by PHPUnit and `make perf-compare`. | unit-only |
