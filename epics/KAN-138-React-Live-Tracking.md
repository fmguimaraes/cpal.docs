## Jira

| Field | Value | Note |
| --- | --- | --- |
| Key | `KAN-138` | epic (authoritative — sprint mechanics) |

## Subject

React UI — Live Tracking

## Business Case

`index.php`'s "live" view does not actually render a working map today,
despite `map_timeline.js`'s own header comment framing itself as that page's
live view — there is no polling anywhere in the current code (see
[`Live-Tracking.md`](../features/technical/Live-Tracking.md) GAP1–GAP3). The
React rebuild must not assume a live feature exists to replicate; this epic
builds a genuinely new, minimal last-known-position view instead of
pretending to port something that was never shipped.

## Description

Build a React/TS last-known-position map from `/api/v1`, on top of KAN-60's
scaffold, with a user-initiated manual refresh and a visible "last updated"
timestamp — explicitly no WebSocket/server-push/background polling (no
backend realtime endpoint exists). Framed throughout as new functionality,
not a replication of current behavior. No backend/DB change; no
modification to `cpaltracker.web`.

## Requirement References (SSoT = Feature doc)

Feature spec: [`React-Live-Tracking.md`](../features/React-Live-Tracking.md)
— 15 FRs, 10 NFRs, 24 ACs. This epic references requirements by ID only;
requirement text lives solely in the feature doc.

## KPIs

- 1 working last-known-position view backed by a real `/api/v1` read.
- 0 realtime/polling infrastructure added (explicitly out of scope).
- 0 modified files under `cpaltracker.web`; 0 backend/DB changes.

## Dependencies

- KAN-60 (Core Platform) — scaffold, `ApiClient`, design tokens, test
  harness.
- KAN-116 (`/api/v1`, Done) — R-1 (open risk): `/api/v1` may not yet expose
  a last-known-positions endpoint; this must be confirmed as a separate
  backend open question/story before/during implementation, never worked
  around in the frontend.
- No dependency on History Dashboard (KAN-139)'s scrubbable map component;
  this epic's component is self-contained.

## Child Stories

| Jira | Story | Pts | Covered Requirements | Software Items |
| --- | --- | --- | --- | --- |

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |

## Related Documentation

- Feature spec: [`React-Live-Tracking.md`](../features/React-Live-Tracking.md)
- Technical-Feature source: [`Live-Tracking.md`](../features/technical/Live-Tracking.md)
- Core epic: [KAN-60](https://c-pal.atlassian.net/browse/KAN-60)
- Stack decision: [`react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md)

## Jira Link

- Epic: [KAN-138](https://c-pal.atlassian.net/browse/KAN-138)
