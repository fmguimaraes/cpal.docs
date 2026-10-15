# Technical-Feature: Device Ingestion

> Reverse-engineered current behavior of `insert_data.php`. Produced under
> KAN-133 (epic KAN-126, "Technical Feature Discovery — cpaltracker.web"),
> following `cpal.docs/sdlc/product/TECH-FEATURE.md`. Read-only analysis —
> no application code was modified to produce this doc.

## 1. Feature Area

Device Ingestion — the single inbound webhook endpoint
(`insert_data.php`) that accepts device telemetry and writes it to the
database.

**Non-UI-facing, included for completeness only.** This area is
machine-facing: it is the HTTP target a TTN (The Things Network)
LoRaWAN bridge calls into, not a page or JSON endpoint a browser or the
planned React UI talks to. Unlike the other six Technical-Feature areas
in this epic, this doc's content is **not expected to directly inform
the KAN-60 React UI rebuild** — there is no corresponding UI surface to
rebuild. It is included solely so the epic's file-by-file coverage of
`cpaltracker.web`'s web root is complete (per `IN-PROGRESS-Technical-
Feature-Discovery.md` FR7). It is also explicitly excluded from the
KAN-116 REST API extraction epic, which already treats it as "functionally
already a machine API with its own auth model" and out of scope for
extraction (`IN-PROGRESS-REST-API-Extraction.md`, "Out of Scope," item on
`insert_data.php`).

## 2. Scope (files / pages covered)

- `cpaltracker.web/insert_data.php` — the full file (auth call, JSON
  parsing, validation, DB write, response shapes).
- `cpaltracker.web/config.php` — only the `requireDeviceAuth()` function
  (lines 81–92) and the `DEVICE_SHARED_KEY` environment variable it reads
  via `cpal_env()`. The rest of `config.php` (DB bootstrap, other helpers)
  is out of scope for this doc; it is covered incidentally only where
  `insert_data.php` uses the `$pdo` connection it sets up.
- Cited, not re-derived: `cpaltracker.web/docs/security.md` (shared-secret
  auth model and its known weaknesses), `cpaltracker.web/docs/
  deployment.md` (TTN bridge / webhook delivery context),
  `cpaltracker.web/docs/design-and-standards.md` (validation-style
  conventions).
- Explicitly excluded: the TTN bridge scripts themselves
  (`ttn_webhook.php`, `ttn_downlink_reglages.php`, `ttn_recovery.php`,
  `retrieve_data.php`) and the production deployment runbook
  (`DEPLOIEMENT.md`) — all four are gitignored and not present in this
  checkout (`cpaltracker.web/docs/deployment.md`, "What is not in this
  repository"), so their behavior cannot be reverse-engineered from code
  and is only referenced as an Open Question (§9) below.

## 3. Current Behavior (CB-n)

- **CB1.** `insert_data.php` sets `CPAL_DB_PROFILE` to `'ingest'` before
  requiring `config.php`, selecting a distinct DB-connection profile
  constant from the one used by user-facing pages
  (`insert_data.php`, lines 11–12).
- **CB2.** Before any other processing, `insert_data.php` calls
  `requireDeviceAuth()` (`insert_data.php`, line 15), which is defined in
  `config.php` (lines 81–92): it reads the expected secret from the
  `DEVICE_SHARED_KEY` environment variable via `cpal_env('DEVICE_SHARED_KEY')`,
  reads the provided value from the `X-Device-Key` request header
  (`$_SERVER['HTTP_X_DEVICE_KEY']`, defaulting to `''` if absent), and
  compares them with `hash_equals()` — a timing-safe comparison
  (`config.php`, line 86).
- **CB3.** `requireDeviceAuth()` rejects the request with HTTP 401 and a
  JSON body `{"error": "Unauthorized"}` if either the expected secret is
  an empty string or the comparison fails (`config.php`, lines 86–91).
  An empty configured secret therefore never authenticates any request,
  including one with an empty/missing header.
- **CB4.** `insert_data.php` sets the response `Content-Type` to
  `application/json` only after the auth check (line 17) — the 401
  response from `requireDeviceAuth()` sets its own `Content-Type` header
  independently (`config.php`, line 88), so every response path from this
  endpoint is JSON regardless of outcome.
- **CB5.** The endpoint reads the raw POST body via
  `file_get_contents('php://input')` and returns HTTP 400 with
  `{"error": "Aucun JSON reçu"}` if the body is empty/falsy
  (`insert_data.php`, lines 20–25).
- **CB6.** The endpoint `json_decode`s the body as an associative array
  and returns HTTP 400 with `{"error": "JSON invalide: <msg>"}` (message
  from `json_last_error_msg()`) if decoding fails (`insert_data.php`,
  lines 27–32).
- **CB7.** The endpoint requires exactly these 14 keys to be present in
  the decoded payload: `ChipID`, `YEAR`, `MONTH`, `DAY`, `HOUR`, `MIN`,
  `SEC`, `LONGITUDE`, `LAT`, `ACCEL_G`, `TEMP_MPU`, `TEMP_AMB`, `HR_AMB`,
  `MASSE`. Any missing key returns HTTP 400 with
  `{"error": "Clés manquantes: <comma-separated list>"}`
  (`insert_data.php`, lines 35–45).
- **CB8.** Of those 14 keys, the 13 keys other than `ChipID` must each
  pass PHP's `is_numeric()` check; the first failing field returns HTTP
  400 with `{"error": "Le champ '<field>' doit être numérique, reçu:
  <value>"}` and stops validation at that field (`insert_data.php`, lines
  48–59). `ChipID` itself is not type-checked here.
- **CB9.** The endpoint enforces range checks, each independently
  short-circuiting with HTTP 400 on failure: `LAT` in `[-90, 90]`,
  `LONGITUDE` in `[-180, 180]`, `MONTH` in `[1, 12]`, `DAY` in `[1, 31]`,
  `HOUR` in `[0, 23]` (`insert_data.php`, lines 61–86). `MIN` and `SEC`
  have no corresponding range check in this file.
- **CB10.** `DAY` is checked against a fixed upper bound of 31 regardless
  of `MONTH`/`YEAR` — the code does not validate day-of-month against the
  specific month or leap-year rules (`insert_data.php`, lines 77–81).
- **CB11.** After validation, the endpoint checks `ChipID` exists in
  `T_DEVICE` (`SELECT 1 FROM T_DEVICE WHERE ChipID = :ChipID LIMIT 1` via
  a prepared statement) and returns HTTP 404 with
  `{"error": "ChipID inconnu"}` if no row is found
  (`insert_data.php`, lines 89–96).
- **CB12.** On a known `ChipID`, the endpoint inserts one row into
  `T_DATA` with columns `timestamp_utc` (server-set via SQL `NOW()`),
  `ChipID`, `YEAR`, `MONTH`, `DAY`, `HOUR`, `` `MIN` ``, `SEC`,
  `LONGITUDE`, `LAT`, `ACCEL_G`, `TEMP_MPU`, `TEMP_AMB`, `HR_AMB`,
  `MASSE`, using a PDO prepared statement with bound, named parameters
  (`insert_data.php`, lines 98–122). `YEAR`/`MONTH`/`DAY`/`HOUR`/`MIN`/
  `SEC` are cast to `(int)` and `LONGITUDE`/`LAT`/`ACCEL_G`/`TEMP_MPU`/
  `TEMP_AMB`/`HR_AMB`/`MASSE` to `(float)` before binding (lines 109–121).
- **CB13.** On successful insert, the endpoint responds with HTTP 200
  (implicit default) and `{"status": "ok", "message": "Donnée insérée",
  "id": <lastInsertId>}` (`insert_data.php`, lines 124–128).
- **CB14.** If the `T_DEVICE` lookup or the `T_DATA` insert throws a
  `PDOException`, the endpoint logs the exception message via
  `error_log()`, returns HTTP 500, and responds with a generic
  `{"error": "Erreur interne du serveur"}` — the underlying DB error
  message is never sent to the caller (`insert_data.php`, lines 130–134).
- **CB15.** `insert_data.php`'s own header comment documents the expected
  request shape as `POST /insert_data.php` with header
  `X-Device-Key: <SHARED_SECRET>` and a JSON body carrying the keys in
  `$requiredKeys` (`insert_data.php`, lines 1–9) — this is the file's own
  stated contract, consistent with the behavior in CB2 and CB7.

## 4. Current Data Model & Contracts

- **Request**: `POST /insert_data.php`, header `X-Device-Key: <secret>`
  (CB2), JSON body with the 14 required keys listed in CB7
  (`insert_data.php`, lines 35–38).
- **Response shapes** (all `application/json`, per CB4):
  - 200: `{"status": "ok", "message": "Donnée insérée", "id": <int>}`
    (CB13).
  - 400: `{"error": "<reason>"}` for empty body, invalid JSON, missing
    keys, non-numeric field, or out-of-range field (CB5, CB6, CB7, CB8,
    CB9).
  - 401: `{"error": "Unauthorized"}` for a missing/mismatched
    `X-Device-Key` or unset `DEVICE_SHARED_KEY` (CB3).
  - 404: `{"error": "ChipID inconnu"}` for an unknown `ChipID` (CB11).
  - 500: `{"error": "Erreur interne du serveur"}` on a `PDOException`
    (CB14).
- **Tables touched**:
  - `T_DEVICE` — read-only, `SELECT 1 ... WHERE ChipID = :ChipID` (CB11).
  - `T_DATA` — write-only, one `INSERT` per request with the 14 columns
    listed in CB12, plus server-set `timestamp_utc` (CB12). The schema
    documented elsewhere in this repo (`cpal.docs/features/pharma-use-
    case-dataset.md`, line 262) lists further `T_DATA` columns
    (`received_at_ttn`, `DIFF_UTC_TTN_SEC`, `DIFF_TTN_RECORD_SEC`) that
    this file does not write — consistent with those being generated/
    derived columns the ingestion endpoint is not responsible for.
- **Environment contract**: `DEVICE_SHARED_KEY` (read via `cpal_env()`,
  `config.php` line 83) — a single shared secret, not a per-request or
  per-device credential (see also GAP1).

## 5. Current Integration Points

- **MySQL** (read `T_DEVICE`, write `T_DATA`) via the shared `$pdo`
  connection `config.php` establishes, using the `'ingest'` DB profile
  selected by `CPAL_DB_PROFILE` (CB1, CB11, CB12).
- **TTN (The Things Network) / LoRaWAN**, inbound direction: this
  endpoint is the HTTP target a TTN webhook bridge calls into with
  device telemetry (`insert_data.php`'s own docstring, CB15; corroborated
  by `cpaltracker.web/docs/security.md`, "Device ingestion
  (`insert_data.php`) is authenticated separately from user sessions...").
  The bridge code itself (`ttn_webhook.php` and related scripts) is not
  in this repo (see §9, Open Questions).
- **PHP error log** (`error_log()`), write-only, for `PDOException`
  messages (CB14).

## 6. Edge Cases Already Handled

- Empty POST body is rejected distinctly from invalid JSON (CB5 vs CB6).
- A payload missing several required keys is reported with all missing
  key names in one response rather than failing on the first
  (`array_diff`, CB7).
- A numeric field holding a non-numeric value is caught before any range
  check or DB write is attempted (CB8 runs before CB9/CB11/CB12).
- An unset `DEVICE_SHARED_KEY` environment variable is treated as an
  always-reject condition rather than disabling the auth check (CB3 —
  `$expected === ''` is checked explicitly).
- An unknown `ChipID` is checked and rejected (404) before the `INSERT`
  is attempted, so no row referencing a non-existent device is written
  and the `T_DATA_ibfk_1` foreign-key behavior described elsewhere in
  this repo (`cpal.docs/features/pharma-use-case-dataset.md`, line 264)
  is never relied upon here (CB11).
- A DB error on either the lookup or the insert returns a generic 500
  without leaking the driver's error message to the caller (CB14).

## 7. Gaps (GAP-n)

- **GAP1.** The device authentication model uses one shared secret
  (`DEVICE_SHARED_KEY`) for every device rather than a per-device
  credential. This is already documented as a known risk in
  `cpaltracker.web/docs/security.md` ("Gaps and risks": "`X-Device-Key`
  is a single shared secret ... not a per-device credential — a leak of
  one device's key ... lets an attacker inject data for *any* device. No
  indication of per-device keys or key rotation."). Confirmed directly in
  code here: `requireDeviceAuth()` (`config.php`, lines 81–92) takes no
  device identifier as input and compares against exactly one configured
  value, so the gap applies to every call site, including
  `insert_data.php`.
- **GAP2.** No rate limiting or lockout is present on `insert_data.php`
  in this repo — repeated failed `X-Device-Key` attempts are not
  throttled anywhere in the file (`insert_data.php`, full file reviewed;
  `requireDeviceAuth()`, `config.php` lines 81–92, performs only a single
  stateless comparison per request). Already named as a cross-cutting gap
  in `cpaltracker.web/docs/security.md` ("No rate limiting or lockout was
  found on `login.php`, `forgot_password.php` or `insert_data.php`").
- **GAP3.** `MIN` and `SEC` are declared as required and numeric-checked
  (CB7, CB8) but have no range validation, unlike every other temporal
  field (`YEAR` has none either, but `MONTH`/`DAY`/`HOUR` do — CB9). A
  payload with, e.g., `MIN: 999` or `SEC: -5` passes validation and is
  cast with `(int)` and inserted into `T_DATA` as-is (`insert_data.php`,
  lines 61–86 for the existing range checks; lines 109–114 for the
  unguarded cast/insert of `MIN`/`SEC`). This is inconsistent with the
  file's own evident intent to range-check every temporal component
  (MONTH/DAY/HOUR all have explicit bounds) and is a gap relative to that
  stated pattern, not a wishlist item.
- **GAP4.** `DAY`'s range check is a fixed `[1, 31]` bound independent of
  `MONTH`/`YEAR` (CB10), so a payload like `{"MONTH": 2, "DAY": 31, ...}`
  or `{"MONTH": 4, "DAY": 31, ...}` passes validation and is written to
  `T_DATA`, producing a calendar-invalid date in the stored row. This is
  a gap relative to the file's own validation intent (reject
  out-of-range date components, CB9) since the check as written cannot
  catch a day invalid for its specific month.
- **GAP5.** The TTN bridge code that calls this endpoint
  (`ttn_webhook.php`) is not present in this repository
  (`cpaltracker.web/docs/deployment.md`, "What is not in this
  repository"), so there is no way to verify from this repo alone that
  the bridge sends exactly the shape `insert_data.php` validates (e.g.
  whether it ever sends `MIN`/`SEC` or `DAY` values that would trigger
  GAP3/GAP4, or whether retried/duplicate deliveries on webhook timeout
  are deduplicated anywhere). This is flagged as a gap in reviewability,
  not a confirmed behavioral defect in the bridge; see also Open
  Questions (§9).

## 8. Suggested Improvements (IMP-n)

- **IMP1.** (Addresses GAP1) Move from a single shared `DEVICE_SHARED_KEY`
  to a per-device credential (e.g. a key or HMAC secret stored per row in
  `T_DEVICE` and looked up by the `ChipID` already present in the
  payload), so a single device's compromised key cannot be used to inject
  data for other devices, and a compromised key can be rotated/revoked
  per device without resetting access for the whole fleet. Author's own
  recommendation, not current behavior.
- **IMP2.** (Addresses GAP2) Add request throttling to `insert_data.php`
  (e.g. a per-source-IP or per-`ChipID` rate limit, consistent with the
  equivalent recommendation already on record for `login.php`/
  `forgot_password.php` in `security.md`). Author's own recommendation.
- **IMP3.** (Addresses GAP3) Add explicit range checks for `MIN`
  (`[0, 59]`) and `SEC` (`[0, 59]`), matching the pattern already used for
  `MONTH`/`DAY`/`HOUR`. Author's own recommendation.
- **IMP4.** (Addresses GAP4) Replace the fixed `DAY` bound with a
  calendar-aware check (e.g. PHP's `checkdate($MONTH, $DAY, $YEAR)`)
  before accepting the payload, so an impossible date cannot be written
  to `T_DATA`. Author's own recommendation.
- **IMP5.** Since this area is explicitly out of scope for both the
  KAN-116 REST API extraction and the KAN-60 React UI rebuild (per
  `IN-PROGRESS-REST-API-Extraction.md`'s Out-of-Scope section and this
  epic's own framing in §1 above), no UI-facing improvement is proposed
  here. Any future change to this endpoint's auth model (IMP1) or
  validation (IMP3/IMP4) should be scoped as its own story outside both
  of those epics, since neither currently claims this file. Author's own
  recommendation, included to prevent this area from falling into a gap
  between the two adjacent epics.

## 9. Open Questions

- The TTN bridge scripts that call `insert_data.php`
  (`ttn_webhook.php`, `ttn_downlink_reglages.php`, `ttn_recovery.php`,
  `retrieve_data.php`) are gitignored and not present in this checkout
  (`cpaltracker.web/docs/deployment.md`, "What is not in this
  repository"), so the exact uplink payload TTN sends, any
  retry/deduplication behavior on webhook failure, and how
  `DEVICE_SHARED_KEY` is provisioned/rotated on the TTN side cannot be
  determined from this repo. `deployment.md` and `security.md` both flag
  this as an existing, unresolved documentation gap rather than something
  this doc can resolve by reading code.
- The production network path/reachability for the TTN webhook ingress
  (which host/port it is delivered to, whether it goes through the same
  Traefik/Apache stack documented for the rest of the app in
  `deployment.md`) is not stated anywhere in this repo
  (`cpaltracker.web/docs/deployment.md`, "Compatibility with future
  containerized deployment": "The TTN bridge scripts' network
  reachability ... needs to be re-verified once containerized — nothing
  here describes their current network path").
- The full `T_DEVICE`/`T_DATA` schema and grants (beyond the columns this
  file reads/writes) are not committed in this repo in authoritative form
  — `deployment.md` notes the local-dev schema is "reconstructed,
  approximate." This doc only describes the columns `insert_data.php`
  itself touches (§4), not the full table definitions.

## 10. Coverage Checklist

- [x] Every file in Scope (§2) cited by at least one CB-n.
  - `insert_data.php`: CB1–CB15.
  - `config.php` (`requireDeviceAuth()` / `DEVICE_SHARED_KEY`): CB2, CB3.
- [x] Every authz/validation rule found is captured (as CB-n or GAP-n).
  - Auth: CB2, CB3, CB4 (current behavior); GAP1, GAP2 (gaps relative to
    documented/stated intent).
  - Payload validation: CB5–CB10 (current behavior); GAP3, GAP4 (gaps in
    the validation's own internal consistency).
  - DB-write / error handling: CB11–CB14 (current behavior).
