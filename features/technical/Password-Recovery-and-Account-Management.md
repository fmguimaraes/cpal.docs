# Technical-Feature: Password Recovery & Account Management

> Reverse-engineered current behavior of `cpaltracker.web`'s forgot-password,
> reset-password, and self-service account pages, extracted per
> [`TECH-FEATURE.md`](../../sdlc/product/TECH-FEATURE.md). Written under
> **KAN-128** (epic **KAN-126**, "Technical Feature Discovery —
> cpaltracker.web"), feeding the planned React UI rebuild (**KAN-60**).

## 1. Feature Area

Password Recovery & Account Management — the forgot-password request flow,
the reset-password (token-redemption) flow, and the authenticated
self-service account page (password / phone / e-mail changes, including the
forced "first connection" password-change flow).

## 2. Scope (files / pages covered)

- `cpaltracker.web/forgot_password.php`
- `cpaltracker.web/reset_password.php`
- `cpaltracker.web/account.php`
- `cpaltracker.web/password_policy.php`
- `cpaltracker.web/auth.php` — not independently in scope, but its
  `cpal_token_*`, `cpal_ip_bloquee`, `cpal_current_user`, `cpal_require_login`,
  `cpal_log`, and `cpal_login_session` functions are called by every file
  above and are cited directly wherever this doc relies on their behavior.
- `cpaltracker.web/mailer.php` — cited only for what e-mails get sent
  (driver selection, template wrapper); not analyzed in depth, per the task
  brief.
- `cpaltracker.web/docs/security.md`, `cpaltracker.web/docs/design-and-standards.md`
  — cross-referenced for already-documented rules/gaps, cited rather than
  re-derived.
- `cpaltracker.web/config.php` — read only for the constants
  (`RESET_MAX_PER_HOUR`, `RESET_TOKEN_MINUTES`, `PWD_MIN_LENGTH`,
  `PWD_MAX_LENGTH`, `PWD_CHECK_HIBP`, `LOGIN_IP_MAX_FAILS`,
  `LOGIN_IP_WINDOW_MINUTES`) this area's logic depends on.
- Excluded from this area's scope, deliberately: `login.php` (initial
  login/credential check — a different feature area), and the admin-side
  script that first sets `MustChangePassword = 1` on an account (not found
  in the files read for this story — see §9).

## 3. Current Behavior (CB-n)

**Forgot-password request (`forgot_password.php`)**

- CB1. On POST, the page always renders the same "check your mailbox"
  success state (`$envoye = true`, set unconditionally before the
  account lookup, `forgot_password.php` line 45) whether the e-mail is
  unknown, inactive, rate-limited, or the send itself fails — only the
  internal `cpal_log(...)` calls differ (`forgot_password.php` lines
  44–118).
- CB2. A POST is rejected with a "session expired" error unless
  `cpal_csrf_ok($_POST['csrf_token'] ?? null)` passes (`forgot_password.php`
  line 33).
- CB3. Before any account lookup, the request is refused if
  `cpal_ip_bloquee($pdo)` returns true — i.e. if the requesting IP has
  `LOGIN_IP_MAX_FAILS` or more `login_ko`/`reset_lien_invalide`/`code_ko`
  events logged within the last `LOGIN_IP_WINDOW_MINUTES` minutes
  (`forgot_password.php` line 39; `auth.php` `cpal_ip_bloquee`, lines
  143–160; `config.php` lines 63–64).
- CB4. Per-account throttling: the account's existing `T_AUTH_TOKEN` rows
  with `TokenType = 'reset'` created in the last hour are counted, and the
  request is refused (logged `reset_demande`/`false`/`'plafond horaire'`) if
  that count is `>= RESET_MAX_PER_HOUR` (default `3`, `config.php` line 74)
  (`forgot_password.php` lines 59–68).
- CB5. Otherwise, any still-live `reset` tokens for the account are first
  revoked via `cpal_token_revoquer($pdo, $userId, 'reset')`, then exactly one
  new token is minted via `cpal_token_creer($pdo, $userId, 'reset',
  RESET_TOKEN_MINUTES)` — at most one live reset token per account at a time
  (`forgot_password.php` lines 71–72, matching its own docblock lines 14–18).
- CB6. The raw token is embedded in a link (`BASE_URL . '/reset_password.php?t=' .
  urlencode($jeton)`) and e-mailed to the account's stored address in both
  HTML and plain-text bodies via `cpal_envoyer_mail()`/`cpal_gabarit_mail()`
  (`forgot_password.php` lines 73, 78–112; `mailer.php` lines 14–52).
- CB7. `cpal_token_creer()` generates the raw token as
  `base64url(random_bytes(32))` and persists only `hash('sha256', $brut,
  true)` in `T_AUTH_TOKEN.TokenHash` — the plaintext token is never stored
  (`auth.php` lines 427–439, docblock lines 422–426).

**Reset-password redemption (`reset_password.php`)**

- CB8. The token is accepted from either the query string (`$_GET['t']`) or
  the POST body (`$_POST['t']`); it is rendered into the page only via a
  hidden form field on resubmission, never regenerated server-side
  (`reset_password.php` line 24, 172).
- CB9. `cpal_token_valide($pdo, $jeton, 'reset')` requires: a matching
  `TokenHash`, matching `TokenType`, `UsedAt IS NULL`, `InvalidatedAt IS
  NULL`, `ExpiresAt > NOW()`, and the owning account's
  `SuppressionDate IS NULL` (`auth.php` lines 453–473).
- CB10. The same `cpal_ip_bloquee($pdo)` check used by forgot-password also
  gates reset-password, evaluated before token validation
  (`reset_password.php` line 31).
- CB11. At submit time, the token is re-validated a second time
  (`cpal_token_valide` again) *inside* a DB transaction, immediately before
  the `UPDATE`, specifically so that a second browser tab holding the same
  link cannot also consume it (`reset_password.php` lines 65–71, docblock
  lines 7–10).
- CB12. The candidate password is checked by
  `cpal_valider_motdepasse($nouveau, [Mail, local-part-of-Mail, Nom, Prenom])`
  of the token's owning account (`reset_password.php` lines 54–59).
- CB13. On success, a single `UPDATE` sets `password_hash`,
  `MustChangePassword = 0`, and `MailVerifieAt = COALESCE(MailVerifieAt,
  NOW())` — redeeming a mailed reset link also marks the account's e-mail
  verified if it wasn't already (`reset_password.php` lines 73–78).
- CB14. In the same transaction, the token is marked consumed
  (`cpal_token_consommer`) and any other live `reset` tokens for the account
  are revoked (`cpal_token_revoquer`), both committed together with the
  password `UPDATE` (`reset_password.php` lines 80–83).
- CB15. Per the file's own docblock, the `UPDATE` causes a DB trigger
  (`trg_user_role_guard_bu`) to increment `SessionEpoch`; `cpal_current_user()`
  compares `T_USER.SessionEpoch` against `$_SESSION['epoch']` on every
  request and logs the session out on mismatch — so a password reset
  invalidates every other open session on the account the next time each is
  used (`reset_password.php` docblock lines 12–14; `auth.php` lines
  169–213).
- CB16. On success, a notification e-mail is sent to the account's address
  stating the password was changed, that all open sessions were closed, and
  to contact the administrator if the user didn't request it
  (`reset_password.php` lines 89–110).

**Password policy (`password_policy.php`)**

- CB17. `cpal_valider_motdepasse()` rejects, in order: length below
  `PWD_MIN_LENGTH` (default 12) or above `PWD_MAX_LENGTH` (default 128);
  all-whitespace input; a single character repeated for the whole string
  (`^(.)\1+$`); a case-insensitive substring match (length ≥ 6) against one
  of four fixed sequences (`0123456789`, `abcdefghijklmnopqrstuvwxyz`,
  `azertyuiop`, `qwertyuiop`); a case-insensitive substring match (length ≥
  4) against the caller-supplied personal context (e-mail, e-mail
  local-part, Nom, Prenom); and a case-insensitive substring match against a
  fixed word denylist (`cpal`, `tracker`, `motdepasse`, `password`,
  `azerty`, `qwerty`) (`password_policy.php` lines 70–110).
- CB18. If and only if all prior checks pass *and* `PWD_CHECK_HIBP` is true,
  `cpal_hibp_count($mdp)` is called and a result `> 0` is rejected, with the
  breach count shown to the user (`password_policy.php` lines 112–118;
  `config.php` line 71).
- CB19. `cpal_hibp_count()` sends only the password's 5-character SHA-1
  prefix to `api.pwnedpasswords.com/range/{prefix}` (k-anonymity), compares
  the returned suffixes with `hash_equals()`, and on any transport/HTTP
  failure (`curl_exec` false or HTTP code ≠ 200) logs the error and returns
  `-1`; `cpal_valider_motdepasse()` only rejects on `$n > 0`, so both `0`
  (clean) and `-1` (unreachable) pass — the check fails open
  (`password_policy.php` lines 30–61, 112–118).
- CB20. `cpal_hacher_motdepasse()` hashes with `PASSWORD_ARGON2ID` when
  `password_algos()` reports it available, otherwise `PASSWORD_BCRYPT` at
  cost `12` (`password_policy.php` lines 128–134).

**Self-service account page (`account.php`)**

- CB21. Authentication is checked via `cpal_current_user($pdo)` directly
  (not `cpal_require_login($pdo)`); on `null` it redirects to
  `login.php?suite=...&nbsp;` (`account.php` lines 28–32). This differs from
  other authenticated pages' use of `cpal_require_login()`, which itself
  redirects to `account.php?premiere=1` whenever `MustChangePassword = 1`
  (`auth.php` lines 217–231) — `account.php` cannot call that function on
  itself without looping.
- CB22. The page is in "first connection" (`$premiere`) state when either
  `?premiere` is present in the query string or `MustChangePassword` is `1`
  (`account.php` line 34). In that state, the Téléphone and Adresse
  e-mail panels, the "return to app" link, and the "log out" link are not
  rendered — only the password-change form is shown (`account.php` lines
  215–219, 223–229, 288–338).
- CB23. All three self-service actions (`motdepasse`, `telephone`, `mail`)
  require the `actuel` field to `password_verify()` against the account's
  stored `password_hash` before any action-specific branch runs
  (`account.php` lines 41, 48–56).
- CB24. A failed current-password check is logged with event name
  `login_ko` (`account.php` lines 54–55) — the identical event string
  `cpal_ip_bloquee()`'s window query counts for `login.php`/
  `reset_password.php` (`auth.php` line 151).
- CB25. The password-change action additionally rejects a new password
  identical to the current one (`account.php` lines 65–66) before validating
  it with `cpal_valider_motdepasse($nouveau, [Mail, local-part, Nom,
  Prenom])` — the same personal-data context `reset_password.php` uses
  (`account.php` lines 68–70).
- CB26. On a successful password change, the code re-reads the row's new
  `SessionEpoch`, writes it into `$_SESSION['epoch']`, and calls
  `session_regenerate_id(true)` — explicitly to keep the acting session
  alive despite the same `SessionEpoch`-increment trigger described in CB15
  (`account.php` lines 80–87, comment lines 80–83).
- CB27. On success, a "password changed" notification e-mail is sent to the
  account's address, worded for a self-service change ("depuis votre espace
  personnel") rather than a reset-by-link; its body does not state that
  other sessions were closed (`account.php` lines 89–103).
- CB28. The phone action strips whitespace/`.`/`-`/`()`, converts a French
  national-format number (`0` + 9 digits) to E.164 (`+33` + remaining 9
  digits), otherwise requires the input to already match
  `^\+[1-9][0-9]{7,14}$`, and treats an empty (post-strip) value as "clear
  the field" (`Telephone = NULL`) (`account.php` lines 117–130).
- CB29. The e-mail action lower-cases and trims the candidate address,
  rejects it via `FILTER_VALIDATE_EMAIL` if malformed, rejects it if
  case-insensitively identical to the current address, and rejects it if
  another `UserID` already holds that address (`account.php` lines 147–158).
- CB30. On a successful e-mail change, `T_USER.Mail` is updated and
  `MailVerifieAt` is reset to `NULL` (`account.php` lines 161–163).
- CB31. On a successful e-mail change, an identical notification e-mail is
  sent to *both* the old and the new address — the only `account.php` action
  that notifies an address other than the account's (new) current one
  (`account.php` lines 168–185).

## 4. Current Data Model & Contracts

- **`T_USER`** columns touched by this area: `UserID`, `Mail`, `Nom`,
  `Prenom`, `Telephone`, `Role`, `ClientID`, `password_hash`,
  `MustChangePassword`, `SessionEpoch`, `MailVerifieAt`, `SuppressionDate`,
  `LastLoginAt`, `PasswordSetAt` (`account.php` lines 48, 73–78, 134,
  161–163; `reset_password.php` lines 73–78; `auth.php` lines 192–199).
- **`T_AUTH_TOKEN`** columns: `TokenID`, `UserID`, `TokenType`, `TokenHash`,
  `ExpiresAt`, `RequestIP`, `RequestUA`, `CreatedAt`, `UsedAt`,
  `InvalidatedAt` (`auth.php` lines 427–479; `forgot_password.php` lines
  60–72).
- **`T_AUTH_LOG`** columns: `UserID`, `MailTente`, `Evenement`, `Succes`,
  `IP`, `UserAgent`, `Detail`, `EventAt` (`auth.php` `cpal_log`, lines
  111–132).
- **Request shapes (HTML form POSTs, no JSON API in this area today):**
  - `forgot_password.php` POST: `{ mail, csrf_token }`.
  - `reset_password.php` GET/POST: `{ t, csrf_token, motdepasse,
    confirmation }` (`t` may arrive via either verb, CB8).
  - `account.php` POST: `{ action: 'motdepasse'|'telephone'|'mail', actuel,
    csrf_token, + action-specific fields (motdepasse/confirmation |
    telephone | mail) }`.
- **Session state** (`$_SESSION`): `uid`, `epoch`, `login_at`, `last_seen`,
  `csrf_token` (`auth.php` `cpal_login_session` lines 260–268,
  `cpal_current_user` lines 174–213).

## 5. Current Integration Points

- **MySQL via PDO** (read/write) — `T_USER`, `T_AUTH_TOKEN`, `T_AUTH_LOG`,
  all four files in scope, through prepared statements with bound
  parameters.
- **Outbound mail** (write) — `mailer.php`'s `cpal_envoyer_mail()` /
  `cpal_gabarit_mail()`, called from `forgot_password.php`,
  `reset_password.php`, and `account.php` for: reset-link mail, post-reset
  confirmation mail, post-self-service-password-change mail, and
  old+new-address e-mail-change notifications. Driver is `log` (writes to
  PHP error log) by default, or raw SMTP to `MAIL_HOST:MAIL_PORT` if
  `MAIL_DRIVER=smtp` (`mailer.php` lines 3–10, 32–52).
- **Have I Been Pwned range API** (read) — `password_policy.php`'s
  `cpal_hibp_count()`, `GET https://api.pwnedpasswords.com/range/{5-char
  SHA-1 prefix}`, gated by `PWD_CHECK_HIBP` (CB18–CB19).
- **Font Awesome CDN stylesheet** (read) — `https://cdnjs.cloudflare.com/...`
  loaded in the `<head>` of `forgot_password.php`, `reset_password.php`, and
  `account.php`.

## 6. Edge Cases Already Handled

- Two browser tabs opening the same reset link: the token is re-checked for
  liveness a second time inside the commit transaction, so only one tab's
  submission can succeed (`reset_password.php` lines 65–71, CB11).
- Unknown or soft-deleted (`SuppressionDate IS NOT NULL`) account on a
  forgot-password request: logged internally, but the user-visible response
  is identical to a successful request (`forgot_password.php` lines 55–56,
  CB1).
- Outbound mail-send failure on a reset request: logged internally
  (`reset_mail_echec`) without changing the user-visible "check your
  mailbox" response, to avoid leaking account existence through a visible
  failure (`forgot_password.php` lines 104–112).
- HIBP API unreachable: treated as "not found in a breach," not as a
  rejection (`password_policy.php` lines 49–52, 112–118, CB19).
- Self-service password change would otherwise log the acting session out
  via the same `SessionEpoch` mechanism that invalidates *other* sessions;
  the code re-syncs `$_SESSION['epoch']` and regenerates the session ID so
  the acting user stays logged in (`account.php` lines 80–87, CB26).
- French national-format phone input (`0XXXXXXXXX`) is normalized to E.164
  rather than rejected (`account.php` lines 120–124, CB28).
- Clearing the phone number (submitting an empty value) is treated as a
  valid "remove the number" action, not a validation error (`account.php`
  lines 126–127, CB28).
- E-mail change to an address already owned by another account is rejected
  before any `UPDATE` runs (`account.php` lines 154–157, CB29).
- A successful e-mail-link password reset also opportunistically marks the
  account's e-mail as verified (`MailVerifieAt = COALESCE(...)`) since
  redeeming the mailed link is itself proof of mailbox control
  (`reset_password.php` lines 75–77, CB13).

## 7. Gaps (GAP-n)

- GAP1. `account.php`'s three self-service actions never call
  `cpal_ip_bloquee($pdo)`, unlike `login.php`-style checks used by
  `forgot_password.php` (CB3) and `reset_password.php` (CB10). A failed
  current-password attempt on `account.php` is logged as `login_ko` — the
  same event type that feeds `cpal_ip_bloquee()`'s window query (CB24,
  `auth.php` line 151) — but `account.php` itself never *checks* that
  counter before processing a submission. An authenticated session can
  therefore be used to guess the account's current password on
  `account.php` without being throttled by this app-level control, which is
  inconsistent with the IP-lockout rule the other three password-adjacent
  pages in this area apply.
- GAP2. `reset_password.php`'s post-change e-mail explicitly states "Toutes
  les sessions ouvertes ont été fermées" (CB16), reflecting the
  `SessionEpoch`-increment effect described in CB15. `account.php`'s
  post-change e-mail is sent by the same `password_hash` `UPDATE`
  mechanism and the code is itself aware the same `SessionEpoch` bump
  occurs (it re-syncs its own session to survive it, CB26), yet that e-mail
  never mentions that other open sessions were closed (CB27). A user who
  was logged in on a second device when changing their password via
  `account.php` gets no explanation for why that session was dropped, even
  though the underlying mechanism is identical to the one
  `reset_password.php` does disclose.
- GAP3. `config.php` defaults `PWD_CHECK_HIBP` to `'false'` (line 71), so on
  an unconfigured deployment `cpal_hibp_count()` is never called at all
  (CB18). `docs/security.md`'s "What's done well" section lists the HIBP
  breach check as an active mitigation ("a breach check against Have I Been
  Pwned... fails open... if HIBP is unreachable," lines 14–19) without
  noting that the check is off by default — the documented security
  posture and the default runtime behavior disagree.
- GAP4. `docs/security.md`'s "Gaps and risks" section states "No rate
  limiting or lockout was found on `login.php`, `forgot_password.php` or
  `insert_data.php` in this repo" (lines 66–70). That is inconsistent with
  the code read for this story: `forgot_password.php` does call
  `cpal_ip_bloquee($pdo)` (CB3), and `auth.php` implements the underlying
  `LOGIN_IP_MAX_FAILS`/`LOGIN_IP_WINDOW_MINUTES` mechanism it relies on
  (CB3, `config.php` lines 63–64). `forgot_password.php` is one of the two
  pages that security doc entry names specifically, so this is a stale
  documented gap rather than a confirmed current one for this file (whether
  it still holds for `login.php` and `insert_data.php` was not re-verified
  here, as both are out of this story's scope).

## 8. Suggested Improvements (IMP-n)

*(Author's own recommendations — not current behavior, not confirmed scope.)*

- IMP1. Add an explicit `cpal_ip_bloquee($pdo)` (or a dedicated per-account
  attempt counter) check to `account.php`'s current-password verification,
  mirroring `forgot_password.php`/`reset_password.php`, to close GAP1.
- IMP2. Make `account.php`'s post-password-change e-mail state explicitly
  that other open sessions were closed, matching `reset_password.php`'s
  wording, to close GAP2 and avoid confusing a user logged in elsewhere.
- IMP3. Default `PWD_CHECK_HIBP` to `true` (or require an explicit,
  documented opt-out) for new deployments, so the out-of-the-box behavior
  matches what `docs/security.md` already describes as a strength — closes
  GAP3.
- IMP4. Update `docs/security.md`'s "No rate limiting" entry to reflect the
  current `cpal_ip_bloquee()` implementation (or re-verify and narrow it to
  the pages that genuinely still lack it) — closes GAP4.
- IMP5. For the React rebuild (KAN-60) and the parallel REST extraction
  (KAN-119/120), keep `RESET_TOKEN_MINUTES` and `RESET_MAX_PER_HOUR` as the
  single source for both the new API's responses and any copy shown in the
  new UI, rather than re-hardcoding those numbers in frontend code.
- IMP6. Since the reset token is deliberately carried only in the URL query
  string (`reset_password.php`'s own docblock, CB8), consider adding a
  `Referrer-Policy: no-referrer` (or `same-origin`) response header on
  `reset_password.php`, which currently also loads a third-party Font
  Awesome stylesheet (§5) from the same page that has the live token in its
  URL — this reduces (does not eliminate) the token's exposure surface
  without changing the documented "token in the URL" design choice itself.

## 9. Open Questions

- Where/how `MustChangePassword` is first set to `1` (presumably an
  admin-only user-creation or admin-reset script) was not found in the
  files read for this story and is outside its scope — the "first
  connection" flow (CB22) is documented here only from `account.php`'s
  consuming side.
- `config.php` and `mailer.php`'s actual SMTP credentials/transport
  configuration are gitignored per `docs/security.md` (lines 57–65, "scope"
  note) and could not be inspected beyond the function signatures and
  driver-selection logic already cited (CB6, §5).
- The DB trigger `trg_user_role_guard_bu`, which `reset_password.php`'s
  docblock says increments `SessionEpoch` (CB15), is referenced only by
  name in that comment — its exact firing condition (every `T_USER`
  `UPDATE`, or specifically a `password_hash` change) lives in the DB
  schema, which is not in this repository, so it is taken on the comment's
  word rather than verified from trigger source.
- Per the task brief: a parallel in-progress story (KAN-119/KAN-120) is
  extracting this area's logic into shared functions and a new
  `/api/v1/account` + `/api/v1/auth/forgot-password`/`reset-password` REST
  path, additively, including a new rule that a password change/reset
  revokes all of the account's live API tokens. No `api_token`,
  `T_API_TOKEN`, or `/api/v1/...` references were found in
  `forgot_password.php`, `reset_password.php`, `account.php`, or
  `password_policy.php` as read for this story — that work does not appear
  to have landed in this area's files yet. Not analyzed further here per
  the brief; re-check this doc against KAN-119/120's actual diff once it
  merges, since it will add a CB this doc does not yet have.
- Whether `login.php` applies the same `cpal_valider_motdepasse()`/HIBP
  checks to an admin-created provisional password (the one a "first
  connection" user starts with) was not verified — `login.php` is out of
  this story's scope.

## 10. Coverage Checklist

- [x] Every file in Scope (§2) cited by at least one CB-n.
- [x] Every authz/validation rule found is captured (as CB-n or GAP-n).
