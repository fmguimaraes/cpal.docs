## Jira

| Field | Value | Note |
| --- | --- | --- |
| Key | `KAN-140` | epic (authoritative — sprint mechanics) |

## Subject

React UI — Device Pilotage

## Business Case

The "MAIN contract only" and "contract is active today" rules gating device
settings exist only inside `pilotage.php` itself, with no shared, reusable
definition anywhere else in the codebase
([`Device-Pilotage.md`](../features/technical/Device-Pilotage.md)
GAP1–GAP5) — this epic must consume whatever `/api/v1` already enforces
server-side rather than reimplementing that business logic in React.
Separately, the current settings form presents 10 fields as if equivalent
when only 5 actually reach the device (GAP6); this epic resolves that
ambiguity in the new UI instead of carrying it forward.

## Description

Build the React/TS device list and settings/pilotage screens against
`/api/v1`, on top of KAN-60's scaffold. Consume server-side
eligibility/active/validity flags as-is — no client-side reimplementation of
MAIN-contract or active-contract gating. Split the settings form into two
visibly labeled groups: the 5 fields actually transmitted to the device, and
the 5 stored-only fields (kept and labeled, not omitted, since their future
use is an open backend question). Surface last-calibration read-only where
`/api/v1` exposes it. No backend/DB change; no modification to
`cpaltracker.web`.

## Requirement References (SSoT = Feature doc)

Feature spec: [`React-Device-Pilotage.md`](../features/React-Device-Pilotage.md)
— 20 FRs, 10 NFRs, 22 ACs. This epic references requirements by ID only;
requirement text lives solely in the feature doc.

## KPIs

- GAP6 fixed: settings form visibly distinguishes the 5 transmitted fields
  from the 5 stored-only fields — no field presented ambiguously.
- 0 client-side reimplementation of MAIN-contract/active-contract gating —
  100% consumed from `/api/v1` responses.
- 0 modified files under `cpaltracker.web`; 0 backend/DB changes.

## Dependencies

- KAN-60 (Core Platform) — scaffold, `ApiClient`, design tokens, test
  harness.
- KAN-116 (`/api/v1`, Done) — eligibility/active/validity flags and field
  values consumed as-is.
- GAP1–GAP5 (gating logic centralization) remain server-owned; this epic
  does not fix them, only refuses to duplicate them client-side.

## Child Stories

| Jira | Story | Pts | Covered Requirements | Software Items |
| --- | --- | --- | --- | --- |

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |

## Related Documentation

- Feature spec: [`React-Device-Pilotage.md`](../features/React-Device-Pilotage.md)
- Technical-Feature source: [`Device-Pilotage.md`](../features/technical/Device-Pilotage.md)
- Core epic: [KAN-60](https://c-pal.atlassian.net/browse/KAN-60)
- Stack decision: [`react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md)

## Jira Link

- Epic: [KAN-140](https://c-pal.atlassian.net/browse/KAN-140)
