## Jira

| Field | Value | Note |
| --- | --- | --- |
| Key | `KAN-141` | epic (authoritative — sprint mechanics) |

## Subject

React UI — Client Branding & Assets

## Business Case

`client_logo.php` currently tells a non-admin caller, by status code and
timing, whether a `client_id` is valid-but-logo-less versus not theirs
(GAP1, [`Client-Branding-and-Assets.md`](../features/technical/Client-Branding-and-Assets.md))
— exactly the distinction the planned `/api/v1/clients/{clientId}/logo`
endpoint is designed to erase. Separately, the shared CSS carries two
different mobile breakpoints (768px vs. 480px, CB22) that should be
reconciled into one token set rather than carried forward as two
conventions.

## Description

Build the React/TS client logo/branding rendering against `/api/v1`, on top
of KAN-60's scaffold and design-token system. Treat every non-success logo
outcome (unified 404, forbidden, network error) identically — one neutral
placeholder, no distinguishing message or timing — assuming the new
unified-404 behavior, not the legacy two-code one. Reconcile the two legacy
breakpoints into one set of responsive tokens in the centralized theme
system. No backend/DB change; no modification to `cpaltracker.web`.

## Requirement References (SSoT = Feature doc)

Feature spec: [`React-Client-Branding-and-Assets.md`](../features/React-Client-Branding-and-Assets.md)
— 12 FRs, 9 NFRs, 14 ACs. This epic references requirements by ID only;
requirement text lives solely in the feature doc.

## KPIs

- GAP1 fixed: 0 UI-observable distinction between "no logo" and "forbidden"
  outcomes.
- 1 reconciled breakpoint token set replacing the legacy 768px/480px split.
- 0 modified files under `cpaltracker.web`; 0 backend/DB changes.

## Dependencies

- KAN-60 (Core Platform) — scaffold, design-token system (color, typography,
  breakpoints), test harness.
- KAN-116 (`/api/v1`, Done) — `/api/v1/clients/{clientId}/logo` unified-404
  behavior consumed as-is.
- IMP1 (residual server-side timing leak) logged as a backend concern, not
  addressed here.

## Child Stories

| Jira | Story | Pts | Covered Requirements | Software Items |
| --- | --- | --- | --- | --- |

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |

## Related Documentation

- Feature spec: [`React-Client-Branding-and-Assets.md`](../features/React-Client-Branding-and-Assets.md)
- Technical-Feature source: [`Client-Branding-and-Assets.md`](../features/technical/Client-Branding-and-Assets.md)
- Core epic: [KAN-60](https://c-pal.atlassian.net/browse/KAN-60)
- Stack decision: [`react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md)

## Jira Link

- Epic: [KAN-141](https://c-pal.atlassian.net/browse/KAN-141)
