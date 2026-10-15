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
| KAN-158 | Auth-Gated Contract Selector & Scoping | 5 | FR1, FR2, NFR1, NFR6 | N/A |
| KAN-160 | Last-Known-Position Map Rendering | 8 | FR3-7, NFR5, NFR7 | N/A |
| KAN-162 | Manual Refresh, Last-Updated & Device Filter | 5 | FR8-10, NFR10 | N/A |
| KAN-165 | Messaging, Branding & History Dashboard Navigation | 5 | FR11-14 | N/A |
| KAN-168 | Error & Empty States, Theming & Test Coverage | 8 | FR15, NFR2-4, NFR8, NFR9 | N/A |

*Total: 31 points across 5 stories. All stories pointed and assigned
to Jira active sprint 3 (S2, 2026-10-11 → 2026-10-22).*

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |
| FR1 | KAN-158 |
| FR2 | KAN-158 |
| FR3 | KAN-160 |
| FR4 | KAN-160 |
| FR5 | KAN-160 |
| FR6 | KAN-160 |
| FR7 | KAN-160 |
| FR8 | KAN-162 |
| FR9 | KAN-162 |
| FR10 | KAN-162 |
| FR11 | KAN-165 |
| FR12 | KAN-165 |
| FR13 | KAN-165 |
| FR14 | KAN-165 |
| FR15 | KAN-168 |
| NFR1 | KAN-158 |
| NFR2 | KAN-168 |
| NFR3 | KAN-168 |
| NFR4 | KAN-168 |
| NFR5 | KAN-160 |
| NFR6 | KAN-158 |
| NFR7 | KAN-160 |
| NFR8 | KAN-168 |
| NFR9 | KAN-168 |
| NFR10 | KAN-162 |

Coverage: all 15 FRs and 10 NFRs mapped to at least one story.
100% structural coverage.

## Related Documentation

- Feature spec: [`React-Live-Tracking.md`](../features/React-Live-Tracking.md)
- Technical-Feature source: [`Live-Tracking.md`](../features/technical/Live-Tracking.md)
- Core epic: [KAN-60](https://c-pal.atlassian.net/browse/KAN-60)
- Stack decision: [`react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md)

## Jira Link

- Epic: [KAN-138](https://c-pal.atlassian.net/browse/KAN-138)
