# React Client Branding & Assets

> Forward-looking product spec for the KAN-60 React UI rebuild of
> `cpaltracker.web`. One of 6 feature-area epics + 1 Core epic under KAN-60.
> This spec describes the NEW React 18.3 + TypeScript 5.5 / Vite standalone
> frontend only; it consumes the existing `/api/v1` REST layer (KAN-116, Done)
> as-is and makes NO backend, DB, or PHP-monolith change.
>
> Single Source of Truth for current behavior and gaps:
> [`features/technical/Client-Branding-and-Assets.md`](technical/Client-Branding-and-Assets.md)
> (CB-n / GAP-n items). Stack decisions:
> [`architecture/react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md).
> Cross-epic readiness note:
> [`features/technical/CONSOLIDATED-Coverage-and-Gaps.md`](technical/CONSOLIDATED-Coverage-and-Gaps.md) §5.
> Requirement *text* lives only in those docs and this spec; downstream epics and
> stories reference IDs, they do not copy requirement text.

## 1. Feature Name

React Client Branding & Assets — the React UI's rendering of a client's own logo
and the centralized, theme-token design system (colors, typography, spacing,
breakpoints, accessibility rules) that defines the rebuilt app's visual brand.

## 2. Problem / Pain Point

Today's branding lives in the PHP monolith: `client_logo.php` serves the logo and
two hand-maintained stylesheets (`styles.css`, `auth.css`) carry the brand tokens.
This has three concrete pains the React rebuild must resolve:

- The current logo endpoint leaks, by HTTP status code and by response timing,
  whether a `client_id` is "mine but logo-less" (`404`) versus "not mine" (`403`)
  — an authenticated caller can distinguish the two (GAP1). The React UI must not
  carry that distinction forward into user-visible behavior.
- The shared CSS carries two different "mobile" breakpoints for one visual system
  — `768px` in `styles.css` and `480px` in `auth.css` (CB22) — so logically
  equivalent layouts break at different widths.
- Brand tokens, typography, and accessibility rules are scattered across two
  stylesheets with ad-hoc per-class `font-size` values and no single token source
  (CB16, CB24), and there is no dark theme at all.

Without this feature, the React UI would either re-leak the client-existence
side-channel, inherit two inconsistent breakpoints, or hardcode brand values the
rebuild's design system is supposed to centralize.

## 3. Goal / Outcome (Success Definition)

A logged-in client sees their own logo rendered cleanly in the React app exactly
where the legacy dashboard showed it, with correct sizing and a graceful,
non-revealing fallback when no logo is available — and they can never tell, from
the UI, whether a logo is merely absent or forbidden. The whole app draws its
colors, typography, spacing, radii, shadows, breakpoints, and accessibility
behavior from one centralized theme-token design system that ships both a light
and a dark theme from day one, with a single reconciled responsive breakpoint
convention. No PHP page, API contract, or database row changes in the process.

## 4. User Stories (Primary Scenarios)

- **US1 — Client sees their brand.** As a client-role user, I want my company's
  logo rendered in the app chrome when I'm signed in, so that the tool feels like
  our own branded environment. *(FR1, FR4, FR8, NFR5, NFR6)*
- **US2 — Graceful, non-revealing fallback.** As a client-role user whose client
  has no logo (or whose logo request is refused), I want the UI to simply show a
  neutral placeholder with no error or distinguishing message, so that I'm never
  shown a broken image and the app never reveals whether a logo exists or is
  forbidden. *(FR2, FR3, NFR7)*
- **US3 — Admin / no-client user.** As an admin (or a user with no `ClientID`), I
  want the app to not attempt to render a client logo at all, matching the legacy
  behavior where the dashboard never shows a logo for me. *(FR2, FR3)*
- **US4 — Consistent, theme-aware branding.** As any user, I want every screen to
  use one consistent brand palette, typography scale, spacing, and a single
  responsive breakpoint, and to switch between light and dark themes, so that the
  app looks coherent in either theme and at any supported width. *(FR5, FR6, FR7,
  FR9, FR10, FR13, NFR9)*
- **US5 — Accessible branding.** As a keyboard or reduced-motion user, I want a
  visible focus ring on interactive elements and animations disabled when I ask my
  OS to reduce motion, applied app-wide, so that the branded UI stays usable.
  *(FR11, NFR8)*
- **US6 — Verified rebuild.** As the delivery team, I want the logo-rendering
  logic and the design-token system covered by unit and end-to-end tests, so that
  branding behavior (including the unified-fallback rule) is proven, not assumed.
  *(NFR1, NFR2)*

## 5. In Scope (What WILL be delivered)

- A React logo component that fetches the signed-in client's logo from
  `GET /api/v1/clients/{clientId}/logo` and renders it in the app chrome
  (replicates CB2/CB14 placement and the CB19 layout).
- Client-side gating so the logo is not requested/rendered for admins or users
  with no `ClientID` (replicates CB5/CB14).
- A single unified fallback path: any non-success response (unified `404`,
  forbidden, network/other error) renders one neutral placeholder with no
  distinguishing text (adopts GAP1's unified-404 model; supersedes the legacy
  `onerror`-hide cosmetic in CB15 with a deliberate placeholder).
- Logo render constraints — `max-width: 220px`, `max-height: 80px`,
  `object-fit: contain` — expressed as design-token-driven styles (CB19).
- A centralized theme-token design system: color tokens (CB16), typography scale
  (CB24), spacing/radii/shadows, and a single reconciled breakpoint token set
  (reconciles CB22's 768px/480px).
- Light and dark themes, switchable via a single state change (stack decision §4).
- The c-PAL brand logo in the app sidebar/chrome with responsive sizing via tokens
  (CB17) and token-driven page-title styling (CB18).
- Shared card/shadow visual idioms for auth and account screens rebuilt on the
  same tokens (CB20).
- App-wide accessibility rules: `:focus-visible` focus ring and a
  `prefers-reduced-motion` override disabling transitions/animations (CB21).
- Unit tests (Vitest + Testing Library) and E2E tests (Playwright) for the above.

## 6. Out of Scope (What WILL NOT be delivered)

- Any backend or database change. `GET /api/v1/clients/{clientId}/logo` and its
  status-code/caching/timing behavior are consumed exactly as KAN-116 ships them;
  the unified-404 behavior is assumed to be provided by the endpoint, not
  implemented in the frontend (stack decision §5). GAP1's server-side timing
  side-channel (IMP1), conditional-request/`ETag` support (IMP2), shared
  server-side query consolidation (IMP3), and logo-format validation (IMP5) are
  backend concerns owned by separate stories, not this feature.
- Any change to, or disruption of, the `cpaltracker.web` PHP monolith, including
  `client_logo.php`, `styles.css`, and `auth.css`, which keep serving the legacy
  app unmodified in parallel.
- A logo upload / management UI (no upload path is in scope for this area; Open
  Questions in the technical doc confirm no upload handler was analyzed).
- Mobile-first design. Desktop is the primary, first-class, acceptance-tested
  target; mobile is best-effort (stack decision §4). The CB22 breakpoint
  *reconciliation* still applies to the design tokens themselves.
- Carrying forward the legacy French-leaning CSS class naming (CB23) as a
  requirement; the new design system defines its own component API.

## 7. User Flow / UX Breakdown

1. A client-role user signs in; the app chrome mounts.
2. The app reads auth/session state (via the Core epic's `AuthContext`); if the
   user is an admin or has no `ClientID`, the logo component renders nothing and
   makes no request (CB5/CB14).
3. Otherwise the logo component requests `GET /api/v1/clients/{clientId}/logo`.
4. On a successful image response, the system renders the logo in the branding
   slot, capped at 220×80 with `object-fit: contain` so aspect ratios never
   distort (CB19).
5. On ANY non-success (unified `404`, forbidden, or error), the system renders a
   single neutral placeholder — identical UI in every failure case, with no
   message that could reveal whether the logo is absent or forbidden (GAP1).
6. Throughout, the c-PAL brand logo, page title, colors, typography, spacing and
   focus/motion behavior come from the centralized theme tokens; toggling the
   theme control flips every screen between light and dark via one state change.

## 8. Functional Requirements (FRs)

- **FR1.** The app SHALL render the signed-in client's logo by consuming
  `GET /api/v1/clients/{clientId}/logo` and placing it in the app chrome's
  branding slot (replicates CB2/CB14 placement). *(US1)*
- **FR2.** The app SHALL NOT request or render a client logo when the current
  user is an admin or has no `ClientID`; the branding slot renders no logo in
  those cases (replicates CB5/CB14). *(US2, US3)*
- **FR3.** The app SHALL treat every non-success logo outcome — unified `404`,
  a forbidden response, and any network/other error — identically, rendering one
  generic placeholder with NO text or behavior that distinguishes "no logo" from
  "forbidden" (adopts GAP1's unified-404 model; supersedes CB15's cosmetic
  `onerror` hide with a deliberate placeholder). *(US2, US3)*
- **FR4.** The app SHALL render the client logo constrained to `max-width: 220px`,
  `max-height: 80px`, `object-fit: contain`, driven by design tokens, so logos of
  differing aspect ratios do not distort (replicates CB19). *(US1)*
- **FR5.** The app SHALL source all brand colors from centralized theme tokens
  (at minimum the CB16 palette: primary, primary-hover, bg, white, border, text,
  error, success); no component SHALL hardcode a raw value the token set covers.
  *(US4)*
- **FR6.** The app SHALL ship both a light and a dark theme, selectable at runtime
  via a single state change (not per-component conditionals). *(US4)*
- **FR7.** The design system SHALL define ONE reconciled set of responsive
  breakpoint tokens, replacing the legacy two-convention split (768px in
  `styles.css` vs 480px in `auth.css`, CB22); every responsive style SHALL consume
  these tokens rather than literal widths. *(US4)*
- **FR8.** The app SHALL render the c-PAL brand logo in the sidebar/chrome with
  token-driven, responsive sizing (replicates CB17's base and reduced sizes via
  the FR7 breakpoint tokens). *(US1, US4)*
- **FR9.** The app SHALL style the page title from theme tokens (color = primary
  token, token-driven responsive font size), replicating CB18 without per-class
  literals. *(US4)*
- **FR10.** The design system SHALL define a typography scale as tokens (font
  family and a defined size scale) consumed by components, replacing the ad-hoc
  per-class `font-size` values of CB24. *(US4)*
- **FR11.** The app SHALL apply, app-wide, a `:focus-visible` focus indicator on
  interactive elements and a `prefers-reduced-motion` override that disables
  transitions/animations (replicates CB21). *(US5)*
- **FR12.** The app SHALL present the shared auth-page and account-page card /
  soft-shadow visual idiom (CB20) built from the same theme tokens as the branding
  components. *(US4)*

## 9. Non-Functional Requirements (NFRs)

- **NFR1.** Logo-component logic — user-type gating (FR2), the unified fallback
  rule (FR3), and render constraints (FR4) — SHALL be covered by Vitest + Testing
  Library unit tests. *(US6)*
- **NFR2.** End-to-end Playwright scenarios SHALL cover: logo renders for a client
  with a logo; a neutral placeholder renders (with no distinguishing message) for
  the no-logo/forbidden cases; and no logo is rendered for an admin/no-`ClientID`
  user. *(US6)*
- **NFR3.** The feature SHALL make NO backend, API-contract, or database change;
  it consumes `GET /api/v1/clients/{clientId}/logo` exactly as KAN-116 ships it,
  including its status codes and caching headers (CB11). *(US1, US2)*
- **NFR4.** The feature SHALL NOT modify or disrupt any `cpaltracker.web` PHP page
  or stylesheet; the legacy app keeps running unmodified in parallel. *(US1)*
- **NFR5.** Desktop SHALL be the primary, acceptance-tested target; mobile layout
  is best-effort and not separately acceptance-tested (stack decision §4). The
  FR7 breakpoint reconciliation still applies to the tokens. *(US1)*
- **NFR6.** The logo SHALL rely on the browser's handling of the endpoint's
  `Cache-Control: private, max-age=300` response (CB11) and SHALL NOT issue extra
  or duplicate logo requests per render. *(US1)*
- **NFR7.** The UI SHALL never expose, by message, markup difference, or
  observable timing of its own logic, whether a client exists / has a logo / is
  forbidden — the fallback is indistinguishable across all failure causes (GAP1).
  *(US2)*
- **NFR8.** Branding and interactive components SHALL honor keyboard focus
  visibility and reduced-motion preferences app-wide (FR11). *(US5)*
- **NFR9.** Theme switching SHALL be a single state change (e.g. a `data-theme`
  attribute + CSS/Tailwind theme variables), not a per-component conditional.
  *(US4)*

## 10. Data Inputs & Outputs

- **Inputs:**
  - Auth/session state from the Core epic's `AuthContext`: the caller's
    `ClientID` and whether they are an admin (consumed, not re-derived).
  - `GET /api/v1/clients/{clientId}/logo` — binary image response on success;
    a unified `404` (and/or other non-2xx) on any failure case.
  - OS/browser signals: `prefers-reduced-motion`, keyboard focus, theme choice.
- **Outputs:**
  - Rendered logo image (≤ 220×80, `object-fit: contain`) or a neutral
    placeholder, in the branding slot.
  - A themed UI (light/dark) whose every brand value derives from the token set.
- **Transformations:** none to the image bytes; the UI only places/sizes the
  returned image or substitutes the placeholder.

## 11. Integration Points

- **`/api/v1` (KAN-116, read-only):** `GET /api/v1/clients/{clientId}/logo`,
  consumed via the Core epic's REST client (fetch + session cookie). Direction:
  frontend → API, read-only. No other service, filesystem, CDN, or mailer is
  touched by this area.
- **Core epic (same KAN-60 rebuild):** `AuthContext` for user type/`ClientID`,
  the shared REST client, the theme-token design system, and the app-chrome
  layout slot the logo renders into.

## 11a. Software Items (IEC 62304 §5.3)

N/A. IEC 62304 software-item tracking is not yet applicable to c-PAL per the
"Applicability to c-PAL today" table in
[`sdlc/README.md`](../sdlc/README.md); no Software Item Registry entries are
assigned to this feature.

## 12. Edge Cases & Constraints

- Client has no logo → unified `404` → neutral placeholder (FR3); must be
  indistinguishable from the forbidden case (NFR7).
- Forbidden/out-of-scope `client_id` → treated identically to "no logo"; the UI
  shows the same placeholder and never surfaces a `403`-style distinction (GAP1).
- Admin or empty-`ClientID` user → no request, no logo (FR2) — mirrors CB5/CB14.
- Network error / API down → same neutral placeholder, no error text (FR3, NFR7);
  the app chrome still renders.
- Transient failure then recovery → the component may retry on remount but SHALL
  NOT reveal failure cause and SHALL NOT duplicate requests within a render
  (NFR6).
- Extreme logo aspect ratios → contained by the 220×80 / `object-fit: contain`
  cap (FR4), never distorted or overflowing the slot.
- Reduced-motion / keyboard-only users → focus ring + motion override apply
  app-wide (FR11).
- Constraint: the endpoint's exact `T_CLIENTS.Logo` MIME type is unverified (tech
  doc Open Questions / IMP5); the UI renders whatever image bytes the endpoint
  returns and does not assume or enforce a format itself.

## 13. Metrics / KPIs

- Logo renders successfully for 100% of client-role users whose client has a logo
  (measured in E2E).
- 0 UI code paths distinguish forbidden vs. missing logo (verified by NFR1/NFR2
  tests and code review) — the GAP1 side-channel is closed UI-side.
- 1 breakpoint-token convention in the shipped design system (0 literal-width
  responsive rules; verified by review/lint).
- Both themes (light, dark) pass visual/E2E smoke with a single-state toggle.
- Unit + E2E coverage present for all FRs touching logo logic and theming.

## 14. Risks & Mitigations

- **Risk:** KAN-116's endpoint does not actually unify `403`/`404` or still leaks
  timing (IMP1). **Mitigation:** the UI treats all non-success identically
  regardless (FR3/NFR7), so a residual *server* leak is logged as a backend open
  question (stack decision §5), never worked around in the frontend.
- **Risk:** developers reintroduce hardcoded colors/breakpoints, eroding the token
  system. **Mitigation:** tokens-only rule (FR5/FR7) enforced via review and lint.
- **Risk:** dark theme regressions in the branding slot. **Mitigation:** both
  themes covered by E2E smoke (NFR2); single-state switch (NFR9) keeps logic
  centralized.
- **Risk:** accidental coupling to or disruption of the PHP monolith.
  **Mitigation:** additive standalone frontend; NFR4 is an acceptance condition.
- **Risk:** unknown logo MIME type (IMP5/Open Questions) causes a broken render.
  **Mitigation:** any decode/load failure falls into the FR3 placeholder path.

## 15. Acceptance Criteria (Definition of Done)

- [ ] A client-role user with a logo sees it rendered in the branding slot,
      sized ≤ 220×80 with `object-fit: contain` (FR1, FR4).
- [ ] No logo is requested or rendered for an admin or a user with no `ClientID`
      (FR2).
- [ ] Every non-success logo outcome (unified `404`, forbidden, error) renders one
      identical neutral placeholder with no distinguishing message or behavior
      (FR3, NFR7).
- [ ] All brand colors come from centralized theme tokens; no hardcoded raw values
      the token set covers (FR5).
- [ ] Light and dark themes both ship and switch via a single state change (FR6,
      NFR9).
- [ ] The design system defines ONE reconciled breakpoint token set; no literal
      768px/480px split remains (FR7).
- [ ] c-PAL sidebar logo and page title render from tokens with responsive sizing
      via the reconciled breakpoints (FR8, FR9).
- [ ] A token-based typography scale is defined and consumed (FR10).
- [ ] `:focus-visible` focus ring and `prefers-reduced-motion` override apply
      app-wide (FR11, NFR8).
- [ ] Auth/account card + soft-shadow idiom is rebuilt on the shared tokens (FR12).
- [ ] Vitest + Testing Library unit tests cover gating, fallback, and render
      constraints (NFR1).
- [ ] Playwright E2E covers logo-present, placeholder (no-logo/forbidden), and
      admin/no-client cases (NFR2).
- [ ] No backend/API/DB change and no modification/disruption of the PHP monolith
      (NFR3, NFR4, NFR5, NFR6).
- [ ] All FRs and NFRs validated.

## 16. Future Extensions (Optional, Not Required Now)

- A client-facing logo upload/management UI (would depend on a new/validated
  `/api/v1` write endpoint and the IMP5 format-validation work).
- Consuming conditional-request / `ETag` `304` support (IMP2) once the backend
  adds it, to further reduce logo bandwidth.
- Per-client theming beyond the single logo (brand color overrides) if a future
  epic requires it.
- Separately acceptance-testing the mobile layout if a later epic reprioritizes
  mobile from best-effort to first-class.
