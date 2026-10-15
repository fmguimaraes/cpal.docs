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

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |

## Related Documentation

- Feature spec: [`React-Password-Recovery-and-Account-Management.md`](../features/React-Password-Recovery-and-Account-Management.md)
- Technical-Feature source: [`Password-Recovery-and-Account-Management.md`](../features/technical/Password-Recovery-and-Account-Management.md)
- Core epic: [KAN-60](https://c-pal.atlassian.net/browse/KAN-60)
- Stack decision: [`react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md)

## Jira Link

- Epic: [KAN-137](https://c-pal.atlassian.net/browse/KAN-137)
