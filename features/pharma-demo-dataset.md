# Pharma demo dataset tooling

Feature spec (W1, [`FEATURE.md`](../sdlc/product/FEATURE.md) template). Epic:
[KAN-67 New Use Case - Pharma](https://c-pal.atlassian.net/browse/KAN-67).
Related: [KAN-28](https://c-pal.atlassian.net/browse/KAN-28) (first, manual load of
the dataset), [KAN-73](https://c-pal.atlassian.net/browse/KAN-73) (load the Pharma
data), [KAN-74](https://c-pal.atlassian.net/browse/KAN-74) (device simulator, a
separate project).

Stories:

| Story | Area | Covers |
|---|---|---|
| [KAN-103](https://c-pal.atlassian.net/browse/KAN-103) Capture the lot tracking schema | Data Model | FR3, FR4 |
| [KAN-104](https://c-pal.atlassian.net/browse/KAN-104) Import the Pharma demo-data tool (blocked by KAN-103) | Tooling | FR1, FR2, FR5–FR10, NFR1–NFR5 |

Dataset spec and simulation assumptions (imported from Confluence on
2026-10-09): [pharma-use-case-dataset.md](pharma-use-case-dataset.md),
[pharma-use-case-simulation.md](pharma-use-case-simulation.md).

## 1. Feature name

Pharma demo dataset tooling: the scripts that simulate the Pharma use case and
load it into cpaltracker, versioned in `cpaltracker.web` and runnable against the
local Docker stack.

## 2. Problem

The Pharma use-case dataset (350 instrumented pallets, 7 clients, 974 simulated
days, 1.4 M `T_DATA` rows, two injected anomalies) was generated and loaded into
cpaltracker on 2026-09-29 by a set of Python and SQL scripts. Those scripts live
in a loose local folder (`SQL & PY Scripts/`), not in git. Their spec is a
Confluence page ("2026-09-15 : Use Case Pharma - Jeu de données").

As they stand:

- **They can be lost.** Nothing is versioned or reviewed.
- **They only run against one database.** Client and user IDs are hardcoded
  (`CID_INNO=3`, `CID_C01=4`, `UID=4` = a real user, Frédéric Marteau).
- **They change the schema inside a seed.** The generated SQL runs
  `ALTER TABLE T_DDLOT` (widens `Gross_Weight`, drops two CHECK constraints).
- **The purge is invasive.** It drops and recreates trigger `trg_ddlot_bd`, and
  moves a real user between clients.
- **They can't run locally.** The local schema
  (`cpaltracker.web/docker/mysql/init/01-schema.sql`) is reconstructed from the
  PHP pages. It has none of `T_DDLOT`, `T_DDLOT_DEVICE_HIST`, `T_TRAITEMENT`,
  `T_ANOMALIE`, their triggers, or `sp_anomalies_recalc`.
- **The archive is incomplete.** `routes.py` reads one file per route
  (`itineraires/<name>.json`), but the archive holds a single combined
  `itineraires.json`.

So the only place this realistic dataset exists is the shared server, where there
is no staging.

## 3. Goal

A developer runs `make seed-demo` on a fresh local stack. A few minutes later,
cpaltracker shows the Pharma use case: lots, pallets moving on real road routes,
and the two anomalies. `make purge-demo` removes it cleanly. Running the same
inputs always produces the same dataset.

## 4. User stories

- As a developer, I want to load the Pharma dataset into my local stack, so that
  I can work on the dashboard, map timeline and anomaly logic with realistic
  volume.
- As a sales or product person, I want to rerun the threshold scenarios and get
  the Excel/dashboard report, so that I can size a fleet for a prospect.
- As an operator, I want to remove the demo data without touching real clients,
  users or devices, so that a shared database stays clean.

## 5. In scope

- Move the simulation and loader sources into `cpaltracker.web/tools/demo-data/pharma/`.
- Capture the production schema the loader depends on into the local schema.
- Replace hardcoded IDs with lookups or self-created demo entities.
- Add Make targets to generate, load and purge, against the local Docker database.
- Write documentation for the tool, and move the Confluence spec here.

## 6. Out of scope

- **Pushing data through the ingestion path** (`insert_data.php` / TTN uplink).
  That is the device simulator's job ([KAN-74](https://c-pal.atlassian.net/browse/KAN-74),
  a separate repo).
- **A dedicated front end.** cpaltracker is the viewer, and `rapport.py` already
  produces the scenario report.
- **Running against production.** Removing the dataset already loaded in
  production is a separate decision, to be taken once staging exists.
- **Anomaly detection** (`T_TRAITEMENT`, `T_ANOMALIE`, `trg_data_anomalies_ai`,
  `sp_anomalies_*`). Its source exists only in production, the rule generator
  (`charger_regles_anomalies.sql`) was not archived, and no page reads it.
  Locally, the two injected anomalies appear as values in the readings only.
- **Loading `T_REGLAGES`.** It stays empty until
  [KAN-34](https://c-pal.atlassian.net/browse/KAN-34) settles its content.
- **Splitting the scenario model** (`simulate.py`, `rapport.py`) into its own repo.
  Revisit only if it gets used independently of cpaltracker.

## 7. Flow

1. The developer runs `make up` (existing). The stack starts with the full schema.
2. The developer runs `make seed-demo`:
   1. The system generates the inputs, runs the simulation (scenario
      threshold 0, seed 2026), and writes `T_DATA_use_case_pharma.csv` and
      `charger_tables_metier.sql` to an ignored build directory.
   2. The system runs the pre-load checks and stops on the first failure.
   3. The system loads the business tables, then `T_DATA`.
   4. The system prints row counts per table.
3. The developer logs in as the admin and sees the Pharma contracts, lots,
   map and the readings that carry the two anomalies.
4. The developer runs `make purge-demo`. The system removes every demo row and
   prints zero counts.
5. Optionally, the developer runs `make demo-report` to get
   `Simu_Pharma_resultats.xlsx` and the comparative dashboard for the 3 scenarios.

## 8. Functional requirements

- **FR1.** The simulation sources (`simulate.py`, `entrees_defaut.py`,
  `run_scenarios.py`, `rapport.py`, `routes.py`), the generators
  (`generer_tdata.py`, `generer_tables.py`), the SQL scripts
  (`controle_pre_chargement.sql`, `charger_T_DATA.sql`, `purge_simulation.sql`)
  and the route data SHALL be versioned in
  `cpaltracker.web/tools/demo-data/pharma/`. Generated files (`*.csv`, generated
  `*.sql`, `*.xlsx`, dashboard outputs) SHALL be gitignored.
- **FR2.** The route data SHALL be stored in the form `routes.py` reads, or
  `routes.py` SHALL read the combined `itineraires.json`. One of the two, not both.
- **FR3.** The local schema SHALL include every table, column, generated
  column and trigger the loader and purge use: `T_DDLOT`,
  `T_DDLOT_DEVICE_HIST` (with `Active_ChipID`), `trg_ddlot_bi`, `trg_ddlot_bd`,
  `T_CONTRATS.ParentContratID`, and the client, contract, device and
  measurement columns the seed fills. These objects are rebuilt from the
  loader's pre-load checks, generator and purge script. All of this work stays
  on localhost (decision of 2026-10-09), so there is no production dump.
- **FR4.** The `T_DDLOT` weight change (`Gross_Weight` / net weight widening,
  CHECK constraints) SHALL live in the schema, not in a generated seed file.
- **FR5.** The generators SHALL NOT hardcode client or user IDs. They SHALL
  either create their own demo client(s) and demo user and use the IDs the
  database returns, or look them up by business key (client name, user email).
- **FR6.** The loader SHALL NOT modify any pre-existing user, client, contract
  or device.
- **FR7.** `make seed-demo` SHALL generate, check and load the dataset into the
  local Docker database in the order: pre-load checks → business tables →
  `T_DATA`. It SHALL stop at the first failing step.
- **FR8.** `make purge-demo` SHALL remove every row created by the seed, and
  only those rows, then print zero counts for each affected table.
- **FR9.** `make demo-report` SHALL run the 3 threshold scenarios and produce the
  Excel report and the dashboard JSON. It also renders the HTML dashboard when
  `dashboard_template.html` is present.
- **FR10.** The `T_DATA` load SHALL work inside the Docker database container:
  the CSV is copied into the container and loaded server-side, or
  `local_infile` is enabled for the local stack.

## 9. Non-functional requirements

- **NFR1.** Determinism: the same inputs and seed (2026) SHALL produce
  byte-identical output files.
- **NFR2.** A full `make seed-demo` on a developer laptop SHOULD finish in
  under 15 minutes.
- **NFR3.** Python dependencies SHALL be pinned in a `requirements.txt`
  (`openpyxl`, `python-dateutil`). The tool SHALL run either in a container or in
  a local virtualenv, as documented.
- **NFR4.** No credentials in the tool's files. Database access comes from the
  stack's `.env`.
- **NFR5.** The Make targets SHALL refuse to run unless the database host is the
  local Docker service.

## 10. Data inputs and outputs

| Input | Output |
|---|---|
| `entrees_defaut.py` parameters (10 products, 7 clients, 34 order lines, fleet of 350, horizon 2029-05-31, seed 2026) | `entrees_defaut.csv` / `.xlsx` |
| Inputs + `itineraires` (8 real road routes) | `T_DATA_use_case_pharma.csv` (~1.4 M rows, ~165 MB) |
| Simulation state | `charger_tables_metier.sql` (clients, contracts, devices, histories, lots, hooks: ~2,500 rows) |
| Inputs, 3 thresholds (0 / 1 t / 2 t) | `Simu_Pharma_resultats.xlsx`, `dashboard.json`, `flux-palettes-innopharm.html` |

Demo devices use the `cpaldev*` ChipID prefix. The purge relies on that prefix
and on the demo client IDs.

## 11. Integration points

- **cpaltracker.web local stack** (`docker-compose.yml`, `Makefile`,
  `docker/mysql/init/`). Write access to the database, local only.
- **cpaltracker.web schema.** This feature makes the local schema match
  production for lot tracking (FR3). It's a step towards versioned
  schema changes ([deployment.md](../../cpaltracker.web/docs/deployment.md)).
- **Device simulator** ([KAN-74](https://c-pal.atlassian.net/browse/KAN-74)).
  It may reuse `routes.py` and the route data later, by copying or packaging
  them. This tool doesn't depend on the simulator.

### Placement decision

| Option | Verdict |
|---|---|
| Keep it as a loose folder | Rejected: not versioned, can be lost, not reviewable. |
| `cpaltracker.web/tools/demo-data/pharma/` | **Chosen.** The loader is tied to cpaltracker's schema, triggers and procedures, so it should change in the same commit as them. |
| Its own repo, with a front end | Rejected for now. The data goes straight into the database (it never uses the ingestion path), and cpaltracker already shows it. A live device simulator is a different tool and belongs to [KAN-74](https://c-pal.atlassian.net/browse/KAN-74). |

## 11a. Software items

Not applicable (see [SDLC, applicability](../sdlc/README.md#applicability-to-c-pal-today)).

## 12. Edge cases and constraints

- **Some IDs are already taken.** The seed must not assume free IDs. FR5 removes
  the risk.
- **`trg_ddlot_bd` blocks every `DELETE` on `T_DDLOT`.** The purge has to drop it
  and recreate it identically. It must recreate it even if a later step fails
  (put the recreation in a `finally`-style step, or in a check run after purge).
- **`trg_ddlot_bi` checks who created a lot.** The demo user must belong to the
  demo client before lots are inserted. With a dedicated demo user (FR5), no real
  user ever needs to be moved.
- **`Active_ChipID` is unique, so a pallet can have only one open hook.** The
  simulation respects this. Keep the existing check that runs before generation.
- **MariaDB vs MySQL.** Production runs MariaDB 10.11; the local stack runs
  MySQL 8.0. The scripts avoid MariaDB-only syntax (`DROP CONSTRAINT IF
  EXISTS`), and no DDL runs inside the seed.

## 13. Metrics

- `make seed-demo` succeeds on a fresh `make reset` stack.
- `make seed-demo` then `make purge-demo` leaves every table's row count equal to
  what it was before the seed.

## 14. Risks and mitigations

| Risk | Mitigation |
|---|---|
| The rebuilt lot schema drifts from production | The schema follows the columns, ranges, index and triggers that the production loader relied on. The `cPAL_Lot` format is the only invented part, and it's documented. Re-check if a production dump becomes available. |
| A purge run against a shared database deletes real data | NFR5 (local host guard). Scope the purge to the demo client IDs and `cpaldev*` devices only. |
| Fixing the hardcoded IDs changes the dataset | Determinism (NFR1) covers the measurements. IDs are expected to change; ChipIDs and lot references must not. |

## 15. Acceptance criteria

- [x] Sources versioned under `tools/demo-data/pharma/`, generated files ignored (FR1, FR2)
- [x] Local schema contains the lot tracking tables and triggers (FR3, FR4)
- [x] No hardcoded client or user IDs; no pre-existing row modified (FR5, FR6)
- [x] `make seed-demo`, `make purge-demo` and `make demo-report` work on a fresh stack (FR7–FR10)
- [x] Same output for the same inputs; dependencies pinned; local-only (NFR1–NFR5)
- [x] Tool README written; the Confluence spec's content moved here; local-setup doc updated

## 15a. Delivery notes (2026-10-09)

Verified on a fresh local stack:

- **Seed.** 1,401,155 readings, 350 pallets, 2026-10-01 00:01 → 2029-05-31 23:59.
  Business tables match the generator's expected counts (2,500 rows). Takes
  about 26 s once the files are generated; generation takes about 30 s.
- **Purge.** Every table's row count returns to its pre-seed value, and
  `trg_ddlot_bd` is restored. Takes about 50 s.
- **Determinism.** Two generations give byte-identical files.
- **App.** On contract MAIN-INNOPHARM, the dashboard API returns readings and
  lot hooks, and `pilotage.php` lists the pallets.
- **Fixed along the way (KAN-103).** `T_DEVICE.MORT` was `NOT NULL DEFAULT 0`
  locally, while `pilotage.php` lists only devices `WHERE MORT IS NULL`. No
  device was ever listed locally.

Open points:

- **No row cap in the dashboard API.** `api_dashboard.php` loads every reading
  in the requested range. Over the full contract period (1.4 M rows) it hits
  PHP's 128 MB `memory_limit`. The default range (contract start → now) is fine.
- **The HTML scenario dashboard can't be rebuilt.** Its template wasn't
  archived, and the published dashboard (a claude.ai artifact) couldn't be read.

## 16. Future extensions

- Other use cases (aerospace, retail) as sibling folders under `tools/demo-data/`.
- Packaging `routes.py` and the route data for the device simulator (KAN-74).
- A staging seed, once staging exists.
