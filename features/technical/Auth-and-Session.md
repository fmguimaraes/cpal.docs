# Technical-Feature: Auth & Session

> Reverse-engineered current behavior (pre-/concurrent-with KAN-118). See
> `TECH-FEATURE.md` for the authoring rules this document follows: CB-n is
> descriptive fact with a code citation, GAP-n is an inconsistency relative
> to the code's own stated rules, IMP-n is the author's own recommendation
> — never blended with CB-n.

## 1. Feature Area

Auth & Session — login form handling, session lifecycle, logout, CSRF
issuance/verification, and the per-account/per-IP brute-force counters in
`cpaltracker.web`.

## 2. Scope (files / pages covered)

- `cpaltracker.web/login.php`
- `cpaltracker.web/logout.php`
- `cpaltracker.web/auth.php`

Cross-referenced (not re-derived at length, cited where relevant):
- `cpaltracker.web/docs/security.md`
- `cpaltracker.web/docs/design-and-standards.md`
- `cpaltracker.web/config.php` — only the `LOGIN_*`/`SESSION_*` constant
  definitions (lines 61–67), consulted to resolve the default values the
  three in-scope files reference by name.

Deliberately excluded from this doc's analysis (other feature areas or out
of KAN-127's scope): `account.php` (password-change UI, beyond the
`MustChangePassword` gate itself), `forgot_password.php`/`reset_password.php`
(password-reset flow), `password_policy.php` (HIBP/Argon2id rules), and the
in-progress KAN-118 `/api/v1/auth/*` REST extraction.

## 3. Current Behavior (CB-n)

**Login form — `login.php`**

- CB1. If a valid session already exists, `login.php` skips the form
  entirely and redirects to `index.php` (`cpal_current_user($pdo) !== null`
  check, `login.php` lines 27–30).
- CB2. The `suite` return-target (from `$_GET`/`$_POST`) is reset to empty
  string unless it starts with `/` and contains no `://`, preventing an
  open redirect to a third-party host (`login.php` lines 36–40).
- CB3. On POST, a failed `cpal_csrf_ok($_POST['csrf_token'] ?? null)` check
  short-circuits to the generic message "Votre session a expiré..."
  without touching the database (`login.php` lines 45–46).
- CB4. An empty `mail` or `motdepasse` field short-circuits to "Renseignez
  votre adresse e-mail et votre mot de passe." before any DB query
  (`login.php` lines 48–49).
- CB5. `login.php` rejects a login attempt when `cpal_ip_bloquee($pdo)`
  returns true (`auth.php`), independent of which account is targeted; it
  logs event `login_debit_ip` and returns a message naming
  `LOGIN_IP_WINDOW_MINUTES` (`login.php` lines 51–54).
- CB6. `cpal_ip_bloquee()` counts `T_AUTH_LOG` rows for the caller's IP
  where `Succes = 0` and `Evenement IN ('login_ko','reset_lien_invalide','code_ko')`
  within the last `LOGIN_IP_WINDOW_MINUTES` minutes, and returns true once
  that count reaches `LOGIN_IP_MAX_FAILS` (`auth.php` lines 143–160).
  Defaults: `LOGIN_IP_MAX_FAILS=10`, `LOGIN_IP_WINDOW_MINUTES=15`
  (`config.php` lines 63–64).
- CB7. On a `PDOException` inside `cpal_ip_bloquee()`, the function logs
  the error and returns `false` — it fails open rather than blocking a
  legitimate user (`auth.php` lines 156–159).
- CB8. For an unknown `Mail` (no `T_USER` row), `login.php` still runs
  `password_verify()` against a fixed dummy bcrypt hash before responding,
  so an unknown-account response takes comparable time to a known-account
  one (`login.php` lines 68–72).
- CB9. Unknown account, soft-deleted account, locked-but-not-yet-expired...
  (see CB10–CB12) and wrong password all return the identical string
  "Adresse ou mot de passe incorrect." (`login.php` lines 74, 78, 89, 107),
  per the file's own stated anti-enumeration rationale (lines 14–18).
- CB10. A soft-deleted account (`SuppressionDate !== null`) is rejected
  with the generic message and logged as `login_compte_inactif`
  (`login.php` lines 76–78).
- CB11. A still-locked account (`LockedUntil` in the future) is rejected
  with a *distinct* message stating the remaining minutes, logged as
  `login_verrouille` (`login.php` lines 80–84).
- CB12. An account with `password_hash IS NULL` (created by an admin but
  never initialized) is rejected with the generic message, logged as
  `login_ko` with detail `sans mot de passe` (`login.php` lines 86–89).
- CB13. A wrong password increments `FailedLoginCount`; when the
  incremented value reaches `LOGIN_MAX_FAILS` (default 5), `T_USER` is
  updated with that count and `LockedUntil = NOW() + INTERVAL
  LOGIN_LOCK_MINUTES MINUTE` (default 15), event `lockout` is logged, and a
  distinct lockout message is returned (`login.php` lines 91–102;
  `config.php` lines 61–62).
- CB14. A wrong password that does not reach `LOGIN_MAX_FAILS` only
  updates `FailedLoginCount`, logs `login_ko`, and returns the generic
  message (`login.php` lines 103–108).
- CB15. A successful login resets `FailedLoginCount = 0`,
  `LockedUntil = NULL`, updates `LastLoginAt`/`LastLoginIP`, calls
  `cpal_login_session($u)`, and logs `login_ok` (`login.php` lines
  110–120).
- CB16. After a successful login, if `MustChangePassword == 1` the
  response redirects to `account.php?premiere=1` regardless of `suite`;
  otherwise it redirects to `suite` if non-empty, else `index.php`
  (`login.php` lines 122–127).
- CB17. Any `PDOException` raised during the login flow is caught, written
  to `error_log`, and surfaced to the user only as "Le service est
  momentanément indisponible..." — the DB error itself is never shown
  (`login.php` lines 129–132).
- CB18. The rendered form always carries a hidden `csrf_token` (from
  `cpal_csrf()`) and a hidden, already-sanitized `suite` field
  (`login.php` lines 161–162).

**Logout — `logout.php`**

- CB19. `logout.php` accepts the CSRF token from either `$_GET` or
  `$_POST` (`logout.php` line 17), so it can be reached via GET.
- CB20. If a user is currently logged in (`cpal_current_user($pdo) !==
  null`) and the supplied token fails `cpal_csrf_ok()`, the script does
  **not** log out; it instead renders an HTML confirmation page (with a
  fresh CSRF-protected form) and exits (`logout.php` lines 19–51), per the
  file's own stated rationale (lines 7–9: block an unsolicited
  `<img src="logout.php">` from a third-party page).
- CB21. In every other case (no session, or a valid CSRF token),
  `cpal_logout($pdo)` is called unconditionally and the response redirects
  to `login.php?fin=1` (`logout.php` lines 53–55).

**Session/CSRF/role core — `auth.php`**

- CB22. `cpal_is_https()` trusts `$_SERVER['HTTPS']` first, falling back to
  `X-Forwarded-Proto` (lowercased) only when that header says `https`
  (`auth.php` lines 36–43); the function's own comment states this is
  only safe because Traefik is the sole trusted reverse proxy.
- CB23. `cpal_session_start()` forces `session.use_strict_mode` and
  `session.use_only_cookies`, sets the cookie `httponly`, `SameSite=Lax`,
  `secure` tied to `cpal_is_https()`, `lifetime=0` (session cookie), names
  the session `CPALSESSID`, starts it, and seeds `$_SESSION['csrf_token']`
  if absent (`auth.php` lines 46–66); it runs automatically as a top-level
  side effect the moment `auth.php` is required (line 68).
- CB24. `cpal_csrf()` returns `$_SESSION['csrf_token'] ?? ''`;
  `cpal_csrf_ok($recu)` requires a non-empty string equal to the session
  token via `hash_equals()` (`auth.php` lines 74–83).
- CB25. `cpal_log()` writes one row to `T_AUTH_LOG`
  (`UserID, MailTente, Evenement, Succes, IP, UserAgent, Detail`) and
  swallows any `PDOException` (logs to `error_log` only), per its own
  docblock: a logging outage must never block a legitimate login
  (`auth.php` lines 106–132).
- CB26. `cpal_current_user($pdo)` returns `null` immediately if
  `$_SESSION['uid']` is empty (`auth.php` lines 174–178).
- CB27. It enforces an idle timeout (`SESSION_IDLE_MAX`, default 1800 s)
  since `$_SESSION['last_seen']` and an absolute timeout
  (`SESSION_ABSOLUTE_MAX`, default 28800 s) since `$_SESSION['login_at']`;
  exceeding either calls `cpal_logout($pdo, 'session_expiree')` and
  returns `null` (`auth.php` lines 180–188; `config.php` lines 66–67).
- CB28. When neither timeout is exceeded, `$_SESSION['last_seen']` is
  refreshed to the current time on that same call — i.e. the idle timer
  resets on every authenticated request (`auth.php` line 189).
- CB29. On every call, `Role`, `ClientID`, `SessionEpoch`,
  `MustChangePassword` and `SuppressionDate` are re-read from `T_USER`
  rather than trusted from the session, specifically so that disabling an
  account or changing its role/password takes effect on already-open
  sessions without waiting for them to expire (`auth.php` docblock lines
  14–20, query at lines 191–203).
- CB30. If the user row no longer exists or `SuppressionDate` is non-null,
  `cpal_current_user()` logs out (`login_compte_inactif`) and returns
  `null` (`auth.php` lines 205–208).
- CB31. If `T_USER.SessionEpoch` no longer matches `$_SESSION['epoch']`,
  `cpal_current_user()` logs out (`session_epoch_perime`) and returns
  `null` — this is the mechanism that invalidates all other open sessions
  after a password change (`auth.php` lines 209–212).
- CB32. `cpal_require_login($pdo)` redirects to
  `login.php?suite=<urlencoded current REQUEST_URI>` and exits when
  `cpal_current_user()` returns `null` (`auth.php` lines 217–224).
- CB33. `cpal_require_login()` also redirects to
  `account.php?premiere=1` and exits whenever `MustChangePassword == 1`
  and the current script's basename is not `account.php` — i.e. every
  page except `account.php` is blocked until the password is changed
  (`auth.php` lines 226–230).
- CB34. `cpal_is_admin($u)` is `true` only when `$u` is non-null and
  `$u['Role'] === 'admin'` (`auth.php` lines 234–237).
- CB35. `cpal_logout($pdo, $evenement='logout')` logs the event (only if a
  `uid` was present) with `Succes=true`, empties `$_SESSION`, expires the
  session cookie via `setcookie(..., time()-42000, ...)`, and calls
  `session_destroy()` (`auth.php` lines 240–253).
- CB36. `cpal_login_session($u)` calls `session_regenerate_id(true)`
  *before* writing `uid`/`epoch`/`login_at`/`last_seen` and issuing a new
  `csrf_token`, per its own docblock rationale: a session id obtained
  before authentication must grant nothing after it (fixation defense)
  (`auth.php` lines 255–268).

## 4. Current Data Model & Contracts

- `T_USER` columns read/written by this area: `UserID`, `ClientID`,
  `Mail`, `Nom`, `Prenom`, `Role`, `password_hash`, `SessionEpoch`,
  `MustChangePassword`, `SuppressionDate`, `FailedLoginCount`,
  `LockedUntil`, `LastLoginAt`, `LastLoginIP`, `Telephone`,
  `PasswordSetAt` (`login.php` lines 59–63, 94–117; `auth.php` lines
  192–196).
- `T_AUTH_LOG` columns written by `cpal_log()`: `UserID`, `MailTente`,
  `Evenement`, `Succes`, `IP`, `UserAgent`, `Detail`, plus an implicit
  `EventAt` used by `cpal_ip_bloquee()`'s window filter (`auth.php` lines
  111–132, 148–155).
- Session keys: `uid`, `epoch`, `login_at`, `last_seen`, `csrf_token`
  (`auth.php` lines 63–65, 263–267). Cookie name `CPALSESSID`,
  `httponly`, `SameSite=Lax`, `secure` driven by `cpal_is_https()`
  (`auth.php` lines 53–60).
- Request contracts: `login.php` POST fields `mail`, `motdepasse`,
  `csrf_token`, `suite`; GET/POST `suite` as return target. `logout.php`
  GET/POST field `csrf_token`.
- Named constants consumed (defined in `config.php` lines 61–67, not
  re-derived here): `LOGIN_MAX_FAILS=5`, `LOGIN_LOCK_MINUTES=15`,
  `LOGIN_IP_MAX_FAILS=10`, `LOGIN_IP_WINDOW_MINUTES=15`,
  `SESSION_IDLE_MAX=1800`, `SESSION_ABSOLUTE_MAX=28800`.

## 5. Current Integration Points

- MySQL/MariaDB via PDO — read/write `T_USER`, write `T_AUTH_LOG`, read
  `T_AUTH_LOG` for the IP-throttle count (`login.php`, `auth.php`,
  throughout).
- Reverse proxy (Traefik, per `auth.php`'s own comment) — read of the
  `X-Forwarded-Proto` request header to determine the `secure` cookie
  flag (`auth.php` lines 30–43). The proxy's trusted-proxy configuration
  itself is outside this repo (see §9 and `docs/security.md`).
- No calls to HIBP, mailer, or TTN from these three files; those belong to
  other feature areas (`password_policy.php`, `forgot_password.php`).

## 6. Edge Cases Already Handled

- Already-authenticated visit to `login.php` is redirected away rather
  than re-rendering the form (`login.php` lines 27–30).
- Open-redirect attempt via `suite` is neutralized (`login.php` lines
  36–40).
- Unknown-account timing is equalized with a dummy `password_verify()`
  call (`login.php` lines 68–72).
- A DB outage during login degrades to a generic "service unavailable"
  message instead of leaking a stack trace/SQL error (`login.php` lines
  129–132).
- A DB outage inside `cpal_log()` or `cpal_ip_bloquee()` never blocks a
  legitimate login attempt (`auth.php` lines 129–131, 156–159).
- `logout.php` reached via an unsolicited cross-site GET (no valid CSRF
  token) does not destroy the session; it requires an explicit confirm
  click (`logout.php` lines 19–51).
- Session fixation: the session id is regenerated at the moment of
  authentication, before any session data is trusted (`auth.php` line
  262).
- A role/status change or password change on an account takes effect on
  that account's already-open sessions without waiting for cookie
  expiry, via the `SessionEpoch` check and the per-request `T_USER`
  re-read (`auth.php` lines 191–212).
- Both an idle timeout and an independent absolute session lifetime are
  enforced (`auth.php` lines 180–188).

## 7. Gaps (GAP-n)

- GAP1. **Rate-limiting/logging gap bypasses the per-IP throttle.** A
  failed CSRF check (`login.php` lines 45–46) and an empty-field
  submission (`login.php` lines 48–49) both return before any call to
  `cpal_log()`, and `cpal_ip_bloquee()`'s query only counts
  `Evenement IN ('login_ko','reset_lien_invalide','code_ko')`
  (`auth.php` lines 149–151). A scripted client can therefore submit an
  unbounded number of malformed/CSRF-less POSTs to `login.php` without
  ever being logged or counted toward the per-IP lockout that is supposed
  to catch "someone hammering this endpoint" (`auth.php` lines 138–142
  state that purpose explicitly).
- GAP2. **`FailedLoginCount` is never reset when a lockout expires.**
  `login.php`'s own header comment states the per-account rule as
  "verrouille après 5 essais, 15 minutes" (lines 7–9), implying a fresh
  budget of attempts after the lock lifts. In the code, `FailedLoginCount`
  is reset to 0 only on a *successful* login (`login.php` line 114);
  nothing resets it when `LockedUntil` passes. Consequently, once the
  lock window elapses, a single further wrong password immediately
  re-locks the account for another `LOGIN_LOCK_MINUTES` (the `$echecs =
  FailedLoginCount + 1 >= LOGIN_MAX_FAILS` check at `login.php` lines
  92–93 is already satisfied from the prior cycle's count), not after 5
  fresh attempts as the stated rule implies.
- GAP3. **Timing side channel across rejection branches.** Only the
  "unknown account" branch performs a `password_verify()` against a dummy
  hash to flatten response time (`login.php` lines 68–72). The
  soft-deleted (lines 76–78), still-locked (lines 80–84), and
  uninitialized-password (lines 86–89) branches all return without any
  `password_verify()` call, so they complete measurably faster than a
  branch that does hash a password — a timing signal that can distinguish
  these account states from a genuine wrong-password attempt, despite the
  file's own stated goal of not revealing which addresses have accounts
  (lines 14–18).
- GAP4. **Locked-account response discloses account existence and
  state.** The anti-enumeration design is stated as "message unique...
  dans tous les cas" (`login.php` lines 14–18), but the still-locked
  branch returns a distinct message disclosing the remaining lockout
  minutes (`login.php` lines 80–84) rather than the generic message used
  by every other rejection path — this is an exception to the file's own
  stated single-message rule.
- GAP5. **`docs/security.md` is stale relative to the current code.** It
  states "No rate limiting or lockout was found on `login.php`..." and
  "repeated failed logins... aren't visibly throttled here"
  (`docs/security.md` lines 66–69), but `login.php`/`auth.php` as read in
  this review implement both a per-account lockout
  (`FailedLoginCount`/`LockedUntil`, `login.php` lines 91–102) and a
  per-IP throttle (`cpal_ip_bloquee()`, `auth.php` lines 143–160). The
  documented security posture and the code's actual behavior disagree.

## 8. Suggested Improvements (IMP-n)

*(Author's own recommendations — not current behavior, not confirmed
scope.)*

- IMP1. Reset `FailedLoginCount` to 0 when a login attempt is evaluated
  after `LockedUntil` has passed (not only on success), so the "5
  attempts, then a 15-minute lock" cycle restarts cleanly instead of
  re-arming on a single attempt. Addresses GAP2.
- IMP2. Apply the same dummy-hash (or otherwise constant-cost) step
  uniformly across the soft-deleted, locked, and uninitialized-password
  branches, not only the unknown-account branch, so every rejection path
  before password comparison costs the same wall-clock time. Addresses
  GAP3.
- IMP3. Record a lightweight event (e.g. `login_malformed`) for
  CSRF-failed and empty-field submissions and include it in
  `cpal_ip_bloquee()`'s counted event set, so that path can no longer be
  used to bypass the per-IP throttle. Addresses GAP1.
- IMP4. For the locked-account branch, consider replying with the same
  generic message/timing as every other rejection and communicating the
  remaining lockout time out of band (e.g. account-recovery email) rather
  than in the synchronous HTTP response, to close the disclosure in GAP4
  without removing the user-facing lockout information entirely.
- IMP5. Update `docs/security.md`'s "Gaps and risks" section to reflect
  the lockout/throttle mechanisms that already exist in code (addresses
  GAP5), and add a note on their specific gaps (GAP1–GAP4) so the
  documented security posture and the code stay in sync going forward.
- IMP6. For the planned React UI rebuild (KAN-60) and the in-flight
  KAN-118 extraction, keep the lockout/CSRF/epoch decision logic in the
  single shared `auth.php`-style layer rather than duplicating any of it
  in the new REST handlers, and add unit tests around the counter/timeout
  edge cases identified above (GAP1–GAP2) — `docs/testing-and-cicd.md` is
  cited elsewhere as noting no tests currently exist for this logic.

## 9. Open Questions

- Whether `X-Forwarded-Proto` is actually only accepted from a trusted
  Traefik hop in the deployed proxy configuration (`RemoteIPTrustedProxy`)
  cannot be verified from this repo — `auth.php`'s own comment names this
  as the precondition for trusting the header (`auth.php` lines 30–34),
  but the proxy config itself is not checked in (also flagged in
  `docs/security.md` lines 105–109).
- Whether any additional rate limiting exists at the reverse-proxy/WAF
  layer outside this repo, which would change the real-world severity of
  GAP1, is unknown from the code alone (`docs/security.md` lines 66–69).
- Whether the lockout-reset behavior in GAP2 (no reset of
  `FailedLoginCount` after `LockedUntil` expires) is an intentional
  progressive-lockout design or an oversight could not be determined from
  the code or docs alone; this needs confirmation from whoever owns the
  account-lockout business rule.
- KAN-118 is concurrently extracting this logic into shared functions and
  a new `/api/v1/auth/login`/`/api/v1/auth/logout` path. This document
  describes the page-level behavior of `login.php`/`logout.php`/
  `auth.php` as read during this review; if KAN-118 has already
  refactored internals by the time this doc is consumed, the externally
  observable behavior described here (CB1–CB36) should still hold, but
  the exact line citations may shift and should be re-checked before
  being relied on for the React rebuild.

## 10. Coverage Checklist

- [x] Every file in Scope (§2) cited by at least one CB-n.
  (`login.php`: CB1–CB18; `logout.php`: CB19–CB21; `auth.php`:
  CB22–CB36.)
- [x] Every authz/validation rule found is captured (as CB-n or GAP-n).
