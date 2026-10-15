## Jira

| Field | Value | Note |
| --- | --- | --- |
| Key | `KAN-137` | epic (authoritative — sprint mechanics) |

## Subject

React UI — Password Recovery & Account Management

## Business Case

Users locked out of their account, or needing to self-service a password
change, are the most support-sensitive flows in the product. The current
`account.php` flow has no IP throttle on its own password check and is
silently missing the "your other sessions were closed" confirmation that
`reset_password.php` shows (see
[`Password-Recovery-and-Account-Management.md`](../features/technical/Password-Recovery-and-Account-Management.md)
GAP1–GAP4). This epic ports both flows to React, closing the messaging gap
at the UI layer while logging the throttle and HIBP-default gaps as backend
questions rather than working around them client-side.

## Description

Build the React/TS forgot-password, reset-password, and account
self-service screens against `/api/v1`, on top of KAN-60's scaffold. Fix
GAP2 (missing "other sessions closed" message on account password change) as
a presentational frontend change. Surface whatever HIBP/throttle responses
`/api/v1` returns gracefully; log GAP1 (no IP throttle on account password
check) and GAP3 (HIBP defaults off) as backend-owned open questions. No
backend/DB change; no modification to `cpaltracker.web`.

## Requirement References (SSoT = Feature doc)

Feature spec: [`React-Password-Recovery-and-Account-Management.md`](../features/React-Password-Recovery-and-Account-Management.md)
— 15 FRs, 11 NFRs, 26 ACs. This epic references requirements by ID only;
requirement text lives solely in the feature doc.

## KPIs

- GAP2 fixed: "other sessions closed" message present on account password
  change, matching the reset flow.
- 100% of 429/throttle responses from `/api/v1` handled gracefully (no raw
  error surfaced to the user).
- 0 modified files under `cpaltracker.web`; 0 backend/DB changes.

## Dependencies

- KAN-60 (Core Platform) — scaffold, `ApiClient`, `AuthContext`, design
  tokens, test harness.
- KAN-116 (`/api/v1`, Done) — password/account endpoints consumed as-is.
- GAP1 (no server-side IP throttle on account password check) and GAP3
  (HIBP defaults off) logged as open backend/deployment questions, not
  addressed here.

## Child Stories

| Jira | Story | Pts | Covered Requirements | Software Items |
| --- | --- | --- | --- | --- |
| KAN-146 | Forgot Password Flow | 3 | FR1-3, NFR3 | N/A |
| KAN-148 | Reset Password Flow | 5 | FR4-8, NFR4, NFR7 | N/A |
| KAN-152 | Account Self-Service (Gate, First-Connection, Password/Phone/E-mail Actions) | 8 | FR9-15 | N/A |
| KAN-154 | Cross-Cutting: Stack, Design System, Accessibility, Testing & Rollout Parity | 8 | NFR1, NFR2, NFR5, NFR6, NFR8-11 | N/A |

*Total: 24 points across 4 stories. All stories pointed and assigned
to Jira active sprint 3 (S2, 2026-10-11 → 2026-10-22).*

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |
| FR1 | KAN-146 |
| FR2 | KAN-146 |
| FR3 | KAN-146 |
| FR4 | KAN-148 |
| FR5 | KAN-148 |
| FR6 | KAN-148 |
| FR7 | KAN-148 |
| FR8 | KAN-148 |
| FR9 | KAN-152 |
| FR10 | KAN-152 |
| FR11 | KAN-152 |
| FR12 | KAN-152 |
| FR13 | KAN-152 |
| FR14 | KAN-152 |
| FR15 | KAN-152 |
| NFR1 | KAN-154 |
| NFR2 | KAN-154 |
| NFR3 | KAN-146 |
| NFR4 | KAN-148 |
| NFR5 | KAN-154 |
| NFR6 | KAN-154 |
| NFR7 | KAN-148 |
| NFR8 | KAN-154 |
| NFR9 | KAN-154 |
| NFR10 | KAN-154 |
| NFR11 | KAN-154 |

Coverage: all 15 FRs and 11 NFRs mapped to at least one story.
100% structural coverage.

## Related Documentation

- Feature spec: [`React-Password-Recovery-and-Account-Management.md`](../features/React-Password-Recovery-and-Account-Management.md)
- Technical-Feature source: [`Password-Recovery-and-Account-Management.md`](../features/technical/Password-Recovery-and-Account-Management.md)
- Core epic: [KAN-60](https://c-pal.atlassian.net/browse/KAN-60)
- Stack decision: [`react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md)

## Jira Link

- Epic: [KAN-137](https://c-pal.atlassian.net/browse/KAN-137)
