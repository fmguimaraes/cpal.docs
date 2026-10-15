# Technical-Feature: Device Pilotage

> Reverse-engineered current behavior, per
> [`sdlc/product/TECH-FEATURE.md`](../../sdlc/product/TECH-FEATURE.md). Written
> for KAN-131 (epic KAN-126, Technical Feature Discovery — cpaltracker.web),
> feeding the planned React UI rebuild (KAN-60).

## 1. Feature Area

Device Pilotage — the MAIN-contract-only page for reading/changing a device's
uplink configuration and for temporarily transferring a device's data
visibility to a STD sub-client contract.

## 2. Scope (files / pages covered)

- `cpaltracker.web/pilotage.php` — the whole feature (context resolution,
  both POST actions, and the rendered page/JS).
- `cpaltracker.web/auth.php` — cited only for the two calls `pilotage.php`
  makes into it (`cpal_require_login`, `cpal_contrat_autorise`) and the
  `cpal_csrf_ok`/`cpal_log`/`h()` helpers it uses; auth.php's own internals
  are not re-documented here beyond what pilotage.php relies on.
- `cpaltracker.web/docs/security.md`, `cpaltracker.web/docs/design-and-standards.md`
  — cross-referenced, not re-derived.
- Deliberately excluded: `ttn_downlink_reglages.php` (consumes
  `V_DEVICE_REGLAGES`/`T_REGLAGES` on the device's next uplink) and
  `ttn_webhook.php`. Neither is in this repo's tracked scope the way
  `docs/security.md` already notes for the TTN bridge scripts, so this doc
  cites their *documented effect* (via pilotage.php's own comments) without
  analyzing their code.

## 3. Current Behavior (CB-n)

**Access gate**

- **CB1.** Every request first calls `cpal_require_login($pdo)` (`auth.php`),
  redirecting to `login.php` if there is no session (`pilotage.php` line 49).
- **CB2.** `contrat_id` is read from POST or GET and resolved through
  `cpal_contrat_autorise($pdo, $user, (int) $contratId)` (`auth.php` lines
  309-334), which returns the contract row only if the caller is an admin or
  the contract's `ClientID` matches the user's own `ClientID` — otherwise it
  logs `acces_refuse` and returns `null` (`pilotage.php` lines 153-170).
- **CB3.** `pilotage.php` additionally requires
  `$contrat['ContratType'] === 'MAIN'`; if the authorized contract is a STD
  contract, the page sets `$contrat = null`, shows "Le pilotage des devices
  est réservé aux contrats MAIN.", and separately logs `acces_refuse` with
  detail `'pilotage sur contrat STD ' . ContratID` (`pilotage.php` lines
  167-178). `cpal_contrat_autorise()` itself has no notion of `ContratType`
  and would return a STD contract to its rightful owner — the MAIN-only rule
  is enforced only here, after the fact (also see GAP1).
- **CB4.** "Active" is computed locally, not read from a stored flag:
  `$contratActif = (DebutContrat === null || now >= DebutContrat) &&
  (FinContrat === null || now <= FinContrat)` — a null start means "already
  started", a null end means "open-ended" (`pilotage.php` lines 183-192).
- **CB5.** Reading context — the device list, each device's last known
  settings, the MAIN's STD sub-contracts, and the device transfer history —
  runs unconditionally whenever `$contrat` is resolved, regardless of
  `$contratActif` (`pilotage.php` lines 200-267). An inactive MAIN contract
  only triggers an informational banner (lines 277-283, "Les formulaires sont
  consultables mais désactivés"); it never blocks the reads themselves.
- **CB6.** The device list is scoped to
  `T_CONTRATS_DEVICES cd JOIN T_DEVICE d ON d.ChipID = cd.ChipID WHERE
  cd.ContratID = ? AND d.MORT IS NULL`, i.e. only devices currently assigned
  to this contract and not marked dead (`pilotage.php` lines 203-211).
- **CB7.** `d.LASTCALIBRATION` is selected in that same device query
  (`pilotage.php` line 204) but is never read or rendered anywhere else in
  the file (see GAP5).

**Reading current configuration**

- **CB8.** Each device's current settings are read from `V_DEVICE_REGLAGES`
  (one row per `ChipID`, the file's own header comment states this view
  keeps only the highest `REGLAGE_ID`), and reduced into a `$filigrane`
  array that is the only settings data sent to the browser, as JSON, for use
  as placeholder ("filigrane") text (`pilotage.php` lines 217-235, 756-176
  in the embedded script around line 766).
- **CB9.** `$CHAMPS` defines 10 configurable fields. Per the file's own
  header comment, only the first 5
  (`FREQ_MESURE_CL`, `WAKEUP_ESP_CL`, `Seuil_TEMPAMB_CL`, `Seuil_HRAMB_CL`,
  `Seuil_MASSE_CL`) are actually encoded into the device downlink by
  `ttn_downlink_reglages.php`; the other 5
  (`SeuilG_CL`, `Seuil_AccelG_CL`, `Seuil_LONG_CL`, `Seuil_LAT_CL`,
  `Seuil_TEMPMPU_CL`) are persisted in `T_REGLAGES` but never transmitted to
  the device (`pilotage.php` lines 66-90). The rendered form (lines 615-625)
  presents all 10 identically — see GAP6.

**Writing a configuration change (Action 1)**

- **CB10.** Triggered by `isset($_POST['envoyer_reglages'])` with
  `$contrat !== null` and a valid `cpal_csrf_ok($_POST['csrf_token'])`
  (`pilotage.php` lines 273-277).
- **CB11.** Refused with `InvalidArgumentException` if `!$contratActif`
  ("Ce contrat n'est pas actif aujourd'hui : aucun réglage ne peut être
  envoyé.") (`pilotage.php` lines 279-282).
- **CB12.** The target device must be one of `$devices` (i.e. belong to this
  contract), else rejected with "Device inconnu ou hors de ce contrat."
  (`pilotage.php` lines 284-287).
- **CB13.** Each submitted field is validated by `valider_champ()` against
  its `$CHAMPS` definition: `uint8` ≤ 255, `uint16` ≤ 65535, `float` accepts
  a comma as decimal separator (normalized to a dot before storage) and is
  capped at 100 characters (`pilotage.php` lines 102-137, 293-300).
- **CB14.** A field left blank is **not** cleared: it carries forward the
  device's last known value from `$reglages[$chip]` (`pilotage.php` lines
  289-300). The file's own header comment explains why this matters:
  `encodeReglagesBinary()` in `ttn_downlink_reglages.php` replaces any `NULL`
  with a hardware default (`FREQ=1, WAKEUP=5, seuils=0`), so an un-reported
  blank field would silently reset the device on that field (`pilotage.php`
  lines 21-28).
- **CB15.** If no field actually changed vs. the previous row
  (`$modifies` empty), the submission is rejected with "Aucune valeur
  modifiée : la ligne n'a pas été créée." and no row is inserted
  (`pilotage.php` lines 305-308).
- **CB16.** The two binary WiFi fields (`WiFi_SSID_CL`, `WiFi_PASS_CL`) are
  never exposed in the form; they are re-read from the latest existing
  `T_REGLAGES` row for that `ChipID` and carried forward unchanged, bound
  with `PDO::PARAM_LOB` specifically to avoid character-set conversion on
  these `varbinary` columns (`pilotage.php` lines 310-342).
- **CB17.** The accepted change is applied as a plain, unconditional
  `INSERT INTO T_REGLAGES (...)` (`pilotage.php` lines 321-343). It is not a
  direct device write and not a separate command-queue table: the file's own
  header states the newly inserted row becomes "the" setting for that device
  only because `V_DEVICE_REGLAGES` returns the highest `REGLAGE_ID` per
  `ChipID`, and it only reaches the device when
  `ttn_downlink_reglages.php` reads that view on the device's next uplink on
  port 3 (`pilotage.php` lines 11-14). The success flash message states this
  explicitly: "Ils seront transmis au device lors de sa prochaine remontée
  sur le port 3." (`pilotage.php` lines 348-353).
- **CB18.** On success, `cpal_log($pdo, 'reglage_ajout', true, ...)` is
  written with the changed field labels as detail (`pilotage.php` lines
  345-346). On failure (validation or `PDOException`), a `ko` flash message
  is stored instead, and DB errors are additionally written to
  `error_log()` (`pilotage.php` lines 355-364).
- **CB19.** Either way, the handler ends with
  `header('Location: pilotage.php?contrat_id=...#reglages'); exit;`
  (`pilotage.php` lines 365-367) — the result is read back from
  `$_SESSION['pil_flash']` on the next page load (lines 158-165), which the
  file's own comment says is specifically to stop a page refresh from
  replaying the insert (`pilotage.php` lines 158-159).

**Transferring a device to a STD contract (Action 2)**

- **CB20.** Triggered by `isset($_POST['envoyer_transfert'])` with
  `$contrat !== null` and a valid CSRF token (`pilotage.php` lines 373-377),
  refused the same way if `!$contratActif` (`pilotage.php` lines 379-382).
- **CB21.** The target device must again be one of `$devices`
  (`pilotage.php` lines 384-387).
- **CB22.** The transfer target is resolved only against `$contratsStd`, the
  set of contracts where `ParentContratID = $contrat['ContratID']` and
  `ContratType = 'STD'` (query at `pilotage.php` lines 238-247); the
  submitted `contrat_std` id is matched against that list, not looked up
  independently (`pilotage.php` lines 389-400). If no match is found:
  "Contrat destinataire inconnu ou non rattaché à ce contrat MAIN." The
  `ClientID` used for the transfer is read from the matched `$std` row, not
  from client input — the file's own comment states this makes it "jamais
  falsifiable depuis le navigateur" (`pilotage.php` lines 389-391, 447).
- **CB23.** `date_init` is mandatory; `date_end` is optional. When omitted,
  `EventDateEnd` is set to the sentinel constant
  `HIST_SENTINELLE = '9999-12-31 00:00:00'`, representing an open-ended
  transfer (`pilotage.php` lines 52, 402-407).
- **CB24.** If an end date is supplied, it must be strictly after the start
  date, else rejected: "La fin du transfert doit suivre son début."
  (`pilotage.php` lines 409-411).
- **CB25.** The transfer's start — and its end, if supplied — must fall
  within the target STD contract's own `DebutContrat`/`FinContrat` window,
  else rejected with that window named in the error message
  (`pilotage.php` lines 413-426). The file's own comment gives the reason:
  outside that window, `V_DATA_WITH_CLIENT`'s second branch returns nothing,
  so the STD client would see no measurements (`pilotage.php` lines 413-415).
- **CB26.** A new transfer is rejected if it overlaps an existing
  `'Transféré'` row for the same `ChipID`
  (`EventDateInit < new_end AND new_start < EventDateEnd`)
  (`pilotage.php` lines 428-442). Comment: `V_DATA_WITH_CLIENT` only honors
  the most recent `EventDateInit`, so overlapping periods would produce
  unpredictable read rights (`pilotage.php` lines 428-431).
- **CB27.** The accepted transfer is a plain, unconditional
  `INSERT INTO T_DEVICE_HIST (ChipID, EventType, EventDateInit, EventDateEnd,
  ClientID) VALUES (?, 'Transféré', ?, ?, ?)` (`pilotage.php` lines 444-448)
  — again an immediate DB write, not a pending/queued state.
- **CB28.** On success, `cpal_log($pdo, 'device_transfert', true, ...)` is
  written (`pilotage.php` lines 450-451); failures follow the same
  flash-message + `error_log()` pattern as Action 1 (`pilotage.php` lines
  463-472), and the handler redirects to
  `pilotage.php?contrat_id=...#transfert` either way (`pilotage.php` lines
  473-475).
- **CB29.** Rendered history (`T_DEVICE_HIST` rows for this contract's
  devices, left-joined to `T_CLIENTS`, read at lines 249-262) marks a row
  "en cours" when `EventDateEnd === HIST_SENTINELLE`, otherwise shows the
  stored end date (`pilotage.php` lines 713-739, sentinel check at the
  relative line matching `$ouvert = (string) $ev['EventDateEnd'] ===
  HIST_SENTINELLE`).

## 4. Current Data Model & Contracts

- **`T_CONTRATS`**: `ContratID, ContratRef, ContratType, ClientID,
  DebutContrat, FinContrat, ParentContratID` — read via
  `cpal_contrat_autorise()` (`auth.php` lines 309-334) and directly for STD
  children (`pilotage.php` lines 238-247).
- **`T_CONTRATS_DEVICES`**: `ChipID ↔ ContratID` join table, read-only here
  (`pilotage.php` lines 203-211).
- **`T_DEVICE`**: `ChipID, STATUT, LASTCALIBRATION, MORT` — read-only
  (`pilotage.php` line 204).
- **`V_DEVICE_REGLAGES`**: read-only view, one row per `ChipID` = highest
  `REGLAGE_ID` (`pilotage.php` lines 217-235, per header comment lines
  11-14).
- **`T_REGLAGES`**: written (INSERT only, never UPDATE/DELETE from this
  file) — columns `ChipID, FREQ_MESURE_CL, WAKEUP_ESP_CL, SeuilG_CL,
  Seuil_LONG_CL, Seuil_LAT_CL, Seuil_AccelG_CL, Seuil_TEMPMPU_CL,
  Seuil_TEMPAMB_CL, Seuil_HRAMB_CL, Seuil_MASSE_CL, WiFi_SSID_CL,
  WiFi_PASS_CL` (`pilotage.php` lines 321-343). Required SQL grant per the
  file's own header: `GRANT INSERT ON cpaltracker.T_REGLAGES TO
  'cpal_web'@'localhost'` (`pilotage.php` lines 38-41).
- **`T_DEVICE_HIST`**: written (INSERT only) — columns `DEVICE_HIST_ID,
  ChipID, EventType, EventDateInit, EventDateEnd, ClientID`
  (`pilotage.php` lines 444-448); required grant per the same header comment
  (`GRANT INSERT ON cpaltracker.T_DEVICE_HIST ...`).
- **`T_CLIENTS`**: read-only, `ClientID, Nom`, left-joined for display
  (`pilotage.php` lines 238-262).
- **`T_AUTH_LOG`**: written indirectly via `cpal_log()` (`auth.php` lines
  111-132), events `acces_refuse`, `reglage_ajout`, `device_transfert`
  observed from this file.
- **Session state**: `$_SESSION['csrf_token']` (set by `auth.php`, checked
  via `cpal_csrf_ok()`); `$_SESSION['pil_flash']` — `{'type': 'ok'|'ko',
  'msg': string}`, set by both action handlers and consumed once on the next
  load (`pilotage.php` lines 158-165, 348-353, 453-461).
- **Request shapes** (POST, both form-encoded):
  - Action 1: `contrat_id, csrf_token, chip_reglages, envoyer_reglages`, plus
    one key per `$CHAMPS` entry (`FREQ_MESURE_CL`, `WAKEUP_ESP_CL`,
    `Seuil_TEMPAMB_CL`, `Seuil_HRAMB_CL`, `Seuil_MASSE_CL`, `SeuilG_CL`,
    `Seuil_AccelG_CL`, `Seuil_LONG_CL`, `Seuil_LAT_CL`, `Seuil_TEMPMPU_CL`).
  - Action 2: `contrat_id, csrf_token, chip_transfert, contrat_std,
    date_init, date_end, envoyer_transfert`.
  - There is no JSON API response shape — both actions respond with an
    HTTP redirect (`header('Location: ...')`), not a body.

## 5. Current Integration Points

- **MySQL via PDO** (`$pdo` from `config.php`, not in repo) — read/write, as
  detailed in §4.
- **`ttn_downlink_reglages.php`** (not in scope, not in this review) —
  downstream consumer, read-only from this feature's perspective: it reads
  `V_DEVICE_REGLAGES`/`T_REGLAGES` on the device's next uplink and encodes
  the first 5 of the 10 `$CHAMPS` fields into the port-3 downlink, per
  `pilotage.php`'s own header comment (lines 11-14, 61-68). This file does
  not call it directly; the dependency is documentary only.
- **`T_AUTH_LOG`** via `cpal_log()` (`auth.php`) — write, for access-denial
  and successful-action auditing (CB3, CB18, CB28).
- **`font-awesome` CDN stylesheet** (`cdnjs.cloudflare.com`) and local
  `styles.css`/`auth.css` — static asset, read-only, no SRI attribute
  present on the `<link>` tag (`pilotage.php` line 184), consistent with the
  gap already logged in `docs/security.md`'s "Gaps and risks" section.

## 6. Edge Cases Already Handled

- A `null` `DebutContrat`/`FinContrat` is treated as "already started" /
  "open-ended" respectively when computing `$contratActif`
  (`pilotage.php` lines 188-191).
- A blank settings field preserves the device's last known value instead of
  wiping it (CB14).
- A settings submission with zero actual changes is rejected rather than
  inserting a no-op row (CB15).
- A decimal comma in a float field is normalized to a dot before validation
  and storage (`pilotage.php` lines 124-126).
- The binary WiFi columns are carried forward via `PDO::PARAM_LOB` so they
  are not mangled by a character-set conversion on an unrelated update
  (`pilotage.php` lines 338-342).
- An open-ended transfer is represented with a sentinel date rather than a
  `NULL` (`pilotage.php` line 52, consistent with "DEFAULT de la colonne"
  per the header comment line 51).
- Two transfers with overlapping periods on the same device are explicitly
  rejected rather than silently creating ambiguous read rights (CB26).
- A dead device (`T_DEVICE.MORT IS NULL` filter) is excluded from the
  pilotable device list entirely (CB6).
- A double form submission via browser refresh is prevented by routing the
  result through a one-shot session flash message plus a redirect rather
  than rendering the result directly from the POST handler (CB19, CB28).

## 7. Gaps (GAP-n)

- **GAP1.** The "pilotage is MAIN-only" rule is enforced solely inside
  `pilotage.php` (CB3), as a check bolted on *after*
  `cpal_contrat_autorise()` returns — that shared helper (`auth.php` lines
  309-334) has no `ContratType` awareness at all and will return a STD
  contract to its rightful owner just as readily as a MAIN one. The
  invariant therefore lives in one page's own code, not in a shared,
  reusable place. Any other consumer of `cpal_contrat_autorise()` —
  including the KAN-116 REST endpoints under development — must
  independently reimplement this same check (and its `acces_refuse` logging)
  rather than inherit it, with no shared source of truth to keep the two
  implementations aligned.
- **GAP2.** There is no canonical, shared "is this contract active today"
  helper. `pilotage.php` computes `$contratActif` locally from
  `DebutContrat`/`FinContrat` vs. `time()` (CB4); this is not exposed from
  `auth.php` and is not reused verbatim elsewhere — `dashboard.php` performs
  a related but distinct date computation (clamping a display window to
  `min(FinContrat, now)` rather than gating a write). The KAN-116 story's
  planned "write refused 409 if the owning contract is inactive" rule has no
  single existing definition of "inactive" to anchor to; today "inactive"
  only means "outside pilotage.php's own date comparison."
- **GAP3.** `pilotage.php` never responds with an HTTP 409 (or any
  non-redirect status) for an inactive-contract write attempt: both Action 1
  and Action 2 refuse via an `InvalidArgumentException` that becomes a
  generic flash message, then a 302 redirect (CB11, CB20). The KAN-116
  REST story's 409-on-inactive-write behavior therefore has no equivalent to
  inherit from this page beyond the underlying `$contratActif` boolean — the
  status-code semantics are net new, not an extraction of existing behavior.
- **GAP4.** Read access (viewing devices, current settings, STD contracts,
  and transfer history) is granted regardless of `$contratActif` (CB5) —
  only writes are gated. The KAN-116 story states both read and write
  require `ContratType === 'MAIN'`, which matches current behavior (CB3),
  but does not mention an inactive-contract restriction on read; this page's
  current behavior (read always allowed, even on an inactive MAIN contract)
  is consistent with that silence, and is recorded here so it isn't lost
  when the REST read endpoint is specified.
- **GAP5.** The STD contract chosen as a transfer target is validated only
  for parentage (`ParentContratID` + `ContratType = 'STD'`, CB22) and for
  the submitted transfer dates falling inside its own
  `DebutContrat`/`FinContrat` window (CB25). There is no check that the STD
  contract is itself "active today" the way the MAIN contract is
  (`$contratActif`); an expired STD contract is not excluded from the
  `contrat_std` dropdown (the `$contratsStd` query at `pilotage.php` lines
  238-247 has no date filter) and would only be rejected indirectly, if the
  submitted transfer dates also fail to fall inside its (past) window.
- **GAP6.** The rendered settings form presents all 10 `$CHAMPS` fields with
  identical styling, hints, and layout (`pilotage.php` lines 615-625), with
  no indication that 5 of them (`SeuilG_CL`, `Seuil_AccelG_CL`,
  `Seuil_LONG_CL`, `Seuil_LAT_CL`, `Seuil_TEMPMPU_CL`) are stored but never
  transmitted to the device (CB9). This contradicts the file's own stated
  intent that the hints are deliberately minimal because "le détail
  d'encodage... reste dans ttn_downlink_reglages.php, où il a un sens : à
  l'écran il n'aidait personne" (`pilotage.php` lines 61-64) — that rationale
  covers *encoding* detail, not the more basic fact of whether a field
  reaches the device at all, which the current UI does not surface.
- **GAP7.** `d.LASTCALIBRATION` is fetched in the device-list query
  (`pilotage.php` line 204) but is never read or displayed anywhere else in
  the file (CB7) — a fetched column with no current consumer.

## 8. Suggested Improvements (IMP-n)

*(Author's own recommendations — not current behavior.)*

- **IMP1** (addresses GAP1, GAP2). Extract the "MAIN contract, and active"
  check into a single shared helper in `auth.php` (e.g.
  `cpal_contrat_main_actif($pdo, $u, $contratId)`) returning the contract row
  or a typed refusal reason, so `pilotage.php` and the KAN-116 REST
  endpoints consume one definition of both "MAIN-only" and "active" instead
  of two independently maintained implementations.
- **IMP2** (addresses GAP3). When specifying the KAN-116 REST endpoints,
  state explicitly that the 409-on-inactive-write behavior is new API
  surface, not an extraction of an existing status-code convention from
  `pilotage.php` — the existing page has no status-code precedent to carry
  over, only the underlying boolean condition.
- **IMP3** (addresses GAP5). Before accepting a transfer, add an explicit
  "is the target STD contract active today" check (mirroring
  `$contratActif`) distinct from the date-window-containment check already
  in place, and exclude expired STD contracts from the `contrat_std`
  dropdown rather than relying on the date-window check to reject them
  indirectly.
- **IMP4** (addresses GAP6). In the planned React rebuild, visually
  distinguish the 5 fields that reach the device from the 5 that are
  DB-only (e.g. a "stored only, not sent to device" note), rather than
  carrying forward the current undifferentiated field list.
- **IMP5** (addresses GAP7). Either render `LASTCALIBRATION` in the device
  list (a plausible operational signal for choosing what to pilot) or stop
  selecting it.
- **IMP6** (standalone, for the rebuild). Per
  `docs/design-and-standards.md`'s own recommendation, extract this page's
  inline query/business logic (contract resolution, device/config/history
  loading, both action handlers) into plain functions independent of the
  HTML rendering, before or alongside building the REST endpoints — the
  same refactor both docs already call out as the single highest-leverage
  change for this codebase.

## 9. Open Questions

- Whether "inactive" for the KAN-116 API's 409 rule is meant to be exactly
  `pilotage.php`'s date-window `$contratActif` computation (CB4), or some
  other stored/derived status — this repo has no stored "active" flag on
  `T_CONTRATS` to confirm against, and the API story's own documentation was
  not read as part of this task (out of scope per the task brief; noted here
  per the task's instruction to flag it rather than assume).
- Whether an expired STD contract should be rejected as a transfer target
  outright (GAP5/IMP3) is a product call the KAN-116 story may already be
  making — this doc only records that `pilotage.php` does not do so today.
- The actual encoding/transport behavior of `ttn_downlink_reglages.php` and
  `ttn_webhook.php` could not be verified from this repo (not tracked here,
  consistent with `docs/security.md`'s own scope note); all claims about
  their behavior in this doc (CB9, CB17, §5) are taken from `pilotage.php`'s
  own header comments, not from reading that code directly.
- Whether the 5 DB-only `$CHAMPS` fields (GAP6) are legacy/dead columns or
  deliberately reserved for a future downlink version is unresolved from the
  code alone.

## 10. Coverage Checklist

- [x] Every file in Scope (§2) cited by at least one CB-n.
  (`pilotage.php`: CB1–CB29; `auth.php`: CB1, CB2, CB3 via
  `cpal_contrat_autorise`/`cpal_require_login`/`cpal_log`/`cpal_csrf_ok`.)
- [x] Every authz/validation rule found is captured (as CB-n or GAP-n).
  (Login gate: CB1. Contract ownership: CB2. MAIN-only: CB3/GAP1.
  Active-contract gating for write: CB4, CB11, CB20/GAP2/GAP3. Read not
  gated by active status: CB5/GAP4. Device-in-contract checks: CB6, CB12,
  CB21. Field validation: CB13. No-op rejection: CB15. STD-attached-to-MAIN
  validation: CB22/GAP5. Date-order and date-window validation: CB24, CB25.
  Overlap validation: CB26. CSRF: CB10, CB20.)
