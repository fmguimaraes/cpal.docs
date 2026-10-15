# Client Branding & Assets — Technical Feature Doc

> Reverse-engineered from the current codebase for KAN-132 (epic KAN-126,
> "Technical Feature Discovery — cpaltracker.web"), feeding the planned React
> UI rebuild (KAN-60). Read-only analysis; no application code was modified.

## 1. Feature Area

Client Branding & Assets — how the application resolves, authorizes, serves
and renders a client's own logo, and the shared CSS conventions (`auth.css`,
`styles.css`) that define the app's visual branding system.

## 2. Scope (files / pages covered)

- `cpaltracker.web/client_logo.php` — full file.
- `cpaltracker.web/auth.css` — full file.
- `cpaltracker.web/styles.css` — full file.
- `cpaltracker.web/auth.php` — only the `cpal_a_un_logo()` function and its
  call site; the rest of `auth.php` (session/login/CSRF internals) is out of
  scope for this doc and is covered elsewhere.
- `cpaltracker.web/index.php` — cited only for the single call site of
  `client_logo.php`/`cpal_a_un_logo()` (the `<img>` tag and the `$logoClient`
  gate); the rest of `index.php` is out of scope.
- `cpaltracker.web/docs/security.md` and
  `cpaltracker.web/docs/design-and-standards.md` — cross-referenced for
  already-documented rules/gaps, not re-derived here.

## 3. Current Behavior (CB-n)

- **CB1.** `client_logo.php` requires an authenticated session for every
  request — `$user = cpal_require_login($pdo);` — before any logo logic runs
  (`client_logo.php:23`, using `auth.php`'s `cpal_require_login()`).
- **CB2.** The target client is resolved from `?client_id=` if present,
  otherwise from the logged-in user's own `ClientID` in session
  (`client_logo.php:25`).
- **CB3.** A non-admin caller requesting a `client_id` other than their own
  gets `403` and the script exits immediately, with no DB query executed
  (`client_logo.php:27-30`).
- **CB4.** A resolved `clientId <= 0` (missing/non-numeric `client_id` for a
  user with no `ClientID`, or an explicit `?client_id=0` /negative value)
  returns `404` immediately, also with no DB query executed
  (`client_logo.php:31-34`).
- **CB5.** An admin (`cpal_is_admin($user)` true) bypasses the ownership
  check entirely and may request any client's logo via `?client_id=`; this
  is explicit, stated intent in the file's own docblock, which also states
  that `index.php` never renders a logo for an admin but that direct URL
  access to `client_logo.php` is protected the same way as any other
  endpoint regardless of what the UI shows (`client_logo.php:7-13`, check at
  `client_logo.php:27`).
- **CB6.** The logo and its update timestamp are fetched with a parameterized
  prepared statement, `SELECT Logo, LogoUpdatedAt FROM T_CLIENTS WHERE
  ClientID = ?` (`client_logo.php:37-39`) — consistent with the PDO
  prepared-statement convention documented for the whole codebase
  (`docs/security.md` "SQL" bullet under "What's done well").
- **CB7.** A `PDOException` from that query is logged via `error_log()` and
  answered with `500`, not a stack trace or DB error text
  (`client_logo.php:40-44`).
- **CB8.** If no row is found for the resolved `clientId`, or the row's
  `Logo` column is `NULL`, the script returns `404` (`client_logo.php:46-49`).
  This single branch already covers both "client does not exist" and
  "client exists but has no logo" uniformly within this one query path (see
  CB-25 below) — the only place in the file where the two cases get
  different status codes is the earlier admin-independent ownership check
  (see GAP1).
- **CB9.** On success, `Content-Type: image/png` is sent unconditionally
  (`client_logo.php:51`) — the header is hardcoded, not derived from the
  stored bytes.
- **CB10.** `Content-Length` is set to `strlen($row['Logo'])`
  (`client_logo.php:52`).
- **CB11.** `Cache-Control: private, max-age=300` is sent, with an inline
  comment stating the rationale: a logo changes rarely, but a URL-versioning
  cache-busting scheme isn't worth it for such a minor gain
  (`client_logo.php:53-55`).
- **CB12.** If `LogoUpdatedAt` is non-empty and parses via `strtotime()`, a
  `Last-Modified` header is emitted from it; if `strtotime()` returns `false`
  the header is simply skipped rather than emitting a malformed date
  (`client_logo.php:56-61`). The script never reads
  `$_SERVER['HTTP_IF_MODIFIED_SINCE']` anywhere in the file, so no request in
  this file ever receives a conditional `304` — every successful request
  re-runs the query and re-sends the full binary body.
- **CB13.** The logo bytes are served directly from the `T_CLIENTS.Logo`
  database column (`echo $row['Logo'];`, `client_logo.php:63`) — there is no
  filesystem-stored logo path in this file; the binary lives in the DB as a
  BLOB-like column, not on disk.
- **CB14.** `index.php` only renders the `<img src="client_logo.php">` tag
  when `$logoClient` is true, where `$logoClient = cpal_a_un_logo($pdo,
  $user);` (`index.php:61`, tag at `index.php:294-298`).
  `cpal_a_un_logo(PDO $pdo, array $u): bool` in `auth.php` returns `false`
  outright for an admin or for a user with an empty `ClientID`, and
  otherwise runs its own prepared-statement check, `SELECT 1 FROM
  T_CLIENTS WHERE ClientID = ? AND Logo IS NOT NULL`, catching
  `PDOException` and returning `false` (not throwing) on a DB error
  (`auth.php:345-358`).
- **CB15.** The `<img>` tag in `index.php` has an `onerror` handler that
  hides its containing `.logo-client` element client-side if the image fails
  to load (`index.php:296-298`) — a cosmetic fallback only; it has no bearing
  on `client_logo.php`'s server-side authorization, which the file's own
  docblock states is enforced independently of what the UI chooses to show
  (`client_logo.php:9-13`).

### Shared CSS / design-system conventions (`styles.css`, `auth.css`)

- **CB16.** `styles.css` defines the app's color tokens once as CSS custom
  properties on `:root` — `--primary: #1A237E`, `--primary-hover: #303F9F`,
  `--bg: #f5f5f5`, `--white: #ffffff`, `--border: #ddd`, `--text: #333`,
  `--error: #c62828`, `--success: #2e7d32` (`styles.css:5-14`) — reused by
  name throughout both `styles.css` and `auth.css` rather than hardcoded
  per rule.
- **CB17.** The c-PAL brand logo (sidebar) is styled by `.cpal-logo-sidebar`
  (centered flex row with a bottom border separator) and `.cpal-logo-icone`
  (fixed `height: 48px`, `width: auto`) (`styles.css:530-541`); at
  `max-width: 768px` the icon shrinks to `height: 40px`
  (`styles.css:585-586`).
- **CB18.** The page title is styled by `.cpal-titre` (`color:
  var(--primary)`, `font-size: 1.7rem`) (`styles.css:544-548`), shrinking to
  `1.3rem` at `max-width: 768px` (`styles.css:587`).
- **CB19.** The logged-in client's own logo (the one `client_logo.php`
  serves) is laid out by `.logo-client` / `.logo-client img`: a
  flex container with a bottom border separator, and the `<img>` itself
  capped at `max-width: 220px; max-height: 80px` with `object-fit: contain`
  so logos of different aspect ratios don't distort (`styles.css:550-562`).
  Both `.cpal-logo-sidebar`/`.cpal-logo-icone` and `.logo-client`/`.logo-client
  img` plus the adjacent `.msg-annonce` announcement-banner classes were
  added in the same dated block, `"Ajouté le 2026-08-28"`
  (`styles.css:524-528, 564-583`), the same date as `client_logo.php`'s own
  `VERSION 1.0 — 2026-08-28` header (`client_logo.php:5`).
- **CB20.** `auth.css`'s own header comment states it is loaded **after**
  `styles.css`, extends the same variables, and reuses the same
  "soft-shadow card" idiom for auth pages (`.page-auth`, `.carte-auth`,
  `.carte-auth-tete`, `.carte-auth-corps`) and the "my account" page
  (`.page-compte`, `.compte-conteneur`, `.fiche`, `.fiche-ligne`)
  (`auth.css:1-7, 19-92`) — none of these classes style the client logo
  directly, but they draw on the same `--primary`/`--white` tokens and
  card/shadow pattern as the branding components in `styles.css`.
- **CB21.** `auth.css` defines one global accessibility rule —
  `:focus-visible` outline on links/buttons/inputs/selects
  (`auth.css:161-167`) — and one global `prefers-reduced-motion` override
  disabling all transitions/animations (`auth.css:169-171`); both apply
  site-wide, not just to auth pages, despite living in `auth.css`.
- **CB22.** Two different responsive breakpoints are used across the two
  stylesheets for what is otherwise one shared visual system:
  `styles.css` breaks at `max-width: 768px` (sidebar/logo/title sizing,
  `styles.css:266, 519, 585`), while `auth.css` breaks at `max-width: 480px`
  (auth-card padding, `auth.css:173-177`).
- **CB23.** CSS class names follow the same French-leaning naming convention
  documented for PHP helper functions elsewhere in this codebase (`cpal_` +
  French words) — e.g. `logo-client`, `carte-auth`, `compte-conteneur`,
  `fiche-ligne`, `msg-annonce`, `bandeau`/`barre-user` — consistent with the
  bilingual-naming pattern already noted in
  `docs/design-and-standards.md:48-53` ("helper functions and all comments
  are French"), extended here to the CSS layer.
- **CB24.** Neither `styles.css` nor `auth.css` declares a `font-family`
  beyond the single `body { font-family: 'Segoe UI', Arial, sans-serif; }`
  rule in `styles.css` (`styles.css:21`) and a monospace override for a few
  numeric/code-like displays (`'Consolas', 'Menlo', monospace`,
  `styles.css:353, 465`); there is no separate typography scale beyond the
  ad hoc `font-size` values set per class.
- **CB25.** Within the authorized-and-queried branch (i.e. once a request
  passes CB3/CB4), `client_logo.php` already collapses "no such client" and
  "client exists but has no logo" into the same `404` response
  (`client_logo.php:46-49`) — this part of the current page already matches
  the uniform-404 intent documented for the in-progress `GET
  /api/v1/clients/{clientId}/logo` REST endpoint (KAN-116). The distinction
  that endpoint is designed to eliminate exists only in the earlier
  ownership-check branch — see GAP1.

## 4. Current Data Model & Contracts

- `T_CLIENTS.Logo` — binary column holding the PNG bytes, read via
  `SELECT Logo, LogoUpdatedAt FROM T_CLIENTS WHERE ClientID = ?`
  (`client_logo.php:37`); `NULL` means "no logo for this client" (CB8).
  The column's exact SQL type/size limit is not verifiable from this repo
  (schema is not in version control — see Open Questions).
- `T_CLIENTS.LogoUpdatedAt` — a datetime-like column, parsed with
  `strtotime()` and only used to emit `Last-Modified`
  (`client_logo.php:56-61`); not used for any cache-invalidation logic
  beyond that header.
- Session contract consumed: `$user['ClientID']` and `$user['Role']` (the
  latter via `cpal_is_admin($user)`), both populated by
  `cpal_require_login($pdo)` (`client_logo.php:23, 25, 27`).
- Request contract: optional query parameter `client_id` (int-cast,
  `client_logo.php:25`); no request body; `GET` only (no method check is
  present in the file itself, but the page is only ever linked as an `<img
  src>`, i.e. invoked with `GET`).
- Response contract: binary body = raw PNG bytes; headers
  `Content-Type: image/png`, `Content-Length`, `Cache-Control: private,
  max-age=300`, optional `Last-Modified`; no `ETag`, no `X-Content-Type-Options`
  (`client_logo.php:51-61`).

## 5. Current Integration Points

- **Database (read)**: `T_CLIENTS` table, two independent queries across the
  feature area — one in `client_logo.php` (CB6) and a separate existence
  check in `auth.php`'s `cpal_a_un_logo()` (CB14) — both via PDO prepared
  statements. No filesystem, mailer, HIBP, TTN, or CDN integration appears
  anywhere in this feature area's scoped files.

## 6. Edge Cases Already Handled

- Missing/omitted `client_id` for a client-role user falls back to their own
  session `ClientID` rather than erroring (`client_logo.php:25`).
- `client_id` resolving to `0` or negative is rejected with `404` before any
  DB access (`client_logo.php:31-34`).
- A DB error during the logo lookup is caught and logged, returning `500`
  rather than leaking a DB exception message (`client_logo.php:40-44`).
- `LogoUpdatedAt` values that fail to parse via `strtotime()` are silently
  skipped rather than emitting a malformed `Last-Modified` header
  (`client_logo.php:58-59`).
- `cpal_a_un_logo()` treats a `PDOException` as "no logo" (`return false`)
  rather than propagating/throwing, so a transient DB error on the
  `index.php` dashboard render degrades to "don't show a logo" instead of a
  fatal error (`auth.php:350-357`).
- An admin's missing/empty `ClientID` is treated the same as "admin" for the
  purposes of `cpal_a_un_logo()` (both short-circuit to `false`)
  (`auth.php:347-349`).

## 7. Gaps (GAP-n)

- **GAP1.** `client_logo.php` returns two different HTTP status codes for
  two different failure reasons that the in-progress REST replacement
  (`GET /api/v1/clients/{clientId}/logo`, KAN-116) is explicitly documented
  to unify: a non-admin requesting a `client_id` other than their own gets
  `403` (`client_logo.php:27-30`), while a resolved, in-scope `client_id`
  that has no logo (or doesn't exist, for an admin) gets `404`
  (`client_logo.php:31-34, 46-49`). This means today's page already lets an
  authenticated caller distinguish "this `client_id` isn't mine" (403) from
  "this `client_id` is mine/accessible but has no logo" (404) by status code
  alone — exactly the two cases KAN-116's new endpoint is designed to
  collapse into one uniform `404` so it never confirms whether an
  out-of-scope client exists. Note: the `403` branch here is a pure
  in-memory comparison against the caller's own `ClientID` (`client_logo.php:27`)
  and never queries the DB, so it does not by itself confirm the *existence*
  of the requested client — but it does unambiguously tell a non-admin
  caller which of the two reasons caused the failure, which the new
  endpoint's rule exists specifically to prevent.

## 8. Suggested Improvements (IMP-n)

- **IMP1** *(author's recommendation, addresses GAP1)*. When building the
  new `/api/v1/clients/{clientId}/logo` endpoint (KAN-116), verify not only
  that both failure cases return the same status code, but that they also
  take a comparable code path/response time. As written today, the `403`
  branch in `client_logo.php` returns before any DB access
  (`client_logo.php:27-30`) while the `404`-for-no-logo branch performs a DB
  round trip first (`client_logo.php:37-49`); if the new endpoint keeps that
  same asymmetry it would re-introduce a timing side-channel even after the
  status codes are unified.
- **IMP2** *(author's recommendation, standalone)*. Add conditional-request
  support (`If-Modified-Since`/`ETag` handling, returning `304` with no
  body) to complement the `Last-Modified` header that is already set but
  currently unused for anything but informational purposes
  (`client_logo.php:56-61`, CB12). This would reduce bandwidth for the
  common case of a client revisiting the dashboard after the 300s
  `max-age` window has lapsed, without needing a URL-versioning scheme.
- **IMP3** *(author's recommendation, standalone)*. Consolidate the
  duplicated "does this client have a logo" query logic — currently
  implemented twice, once in `auth.php`'s `cpal_a_un_logo()` (`auth.php:345-358`)
  and once inline inside `client_logo.php` (`client_logo.php:36-49`) — into
  one shared function that both call, consistent with
  `docs/design-and-standards.md`'s standing recommendation to extract
  query/business logic out of page files into shared, reusable functions
  (`docs/design-and-standards.md:70-78`). This is directly relevant to the
  React rebuild (KAN-60), which will need exactly one such function behind
  whatever REST endpoint replaces both call sites.
- **IMP4** *(author's recommendation, standalone)*. Before the React rebuild
  defines its own design tokens, reconcile the two different responsive
  breakpoints currently used for what is otherwise one shared visual system
  — `768px` in `styles.css` vs. `480px` in `auth.css` (CB22) — into a single
  set of breakpoint tokens, so the rebuilt component library doesn't inherit
  two inconsistent cutoffs for logically equivalent "mobile" layouts.
- **IMP5** *(author's recommendation, standalone)*. Record (or validate) an
  actual MIME type/format for a stored client logo rather than hardcoding
  `Content-Type: image/png` on every response (`client_logo.php:51`). This
  file's docblock and behavior both assume PNG (CB9), but nothing in this
  file's scope validates that assumption on the write/upload side; if the
  React rebuild's upload path ever accepts other formats, a hardcoded
  `image/png` header would mislabel the response.

## 9. Open Questions

- The exact SQL type/size of `T_CLIENTS.Logo` and `T_CLIENTS.LogoUpdatedAt`
  cannot be confirmed — the DB schema is not committed to this repo (per
  `docs/security.md`'s stated scope limitation covering `config.php`/schema).
- Whether a logo-upload path (not in this story's scope) validates that
  uploaded bytes are actually PNG before storing them in
  `T_CLIENTS.Logo` could not be verified — no upload-handling file was in
  scope for this doc.
- Whether any security response headers (CSP, `X-Content-Type-Options`,
  etc.) are added at the Traefik/proxy layer around this binary response is
  not verifiable from this repo, consistent with the same limitation already
  noted for the rest of the app in `docs/security.md`'s "Gaps and risks"
  section.
- The in-progress KAN-116 REST extraction's exact implementation (status
  code uniformity, and whether it also addresses the timing asymmetry noted
  in IMP1) is being built in a parallel epic and was not inspected here;
  this doc only establishes what `client_logo.php` itself does today.

## 10. Coverage Checklist

- [x] Every file in Scope (§2) cited by at least one CB-n.
- [x] Every authz/validation rule found is captured (as CB-n or GAP-n).
