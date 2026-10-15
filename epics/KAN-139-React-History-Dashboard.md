## Jira

| Field | Value | Note |
| --- | --- | --- |
| Key | `KAN-139` | epic (authoritative — sprint mechanics) |

## Subject

React UI — History Dashboard

## Business Case

The history dashboard's time-slider/trail-replay/tooltip/auto-fit feature
set (`map_timeline.js` wired into `dashboard.php`) is the one area where the
current UI's richest functionality is real and actually used — it is the
authoritative behavior to replicate, not `Live-Tracking.md`'s read of the
same file. Separately, the current date-handling is more permissive than
KAN-116's REST endpoint is specified to be (reversed ranges silently
swapped, malformed dates silently dropped, no max span — see
[`History-Dashboard.md`](../features/technical/History-Dashboard.md) and
the Readiness note in `CONSOLIDATED-Coverage-and-Gaps.md` §5). This epic
decides, explicitly, to surface the REST layer's stricter rules instead.

## Description

Build the React/TS history dashboard (time-slider, trail replay, tooltip,
auto-fit, charts) against `/api/v1`, on top of KAN-60's scaffold, bundling
Leaflet + Plotly instead of CDN. Reject reversed date ranges and malformed
dates with clear errors, and enforce a sane max span — surfacing the REST
layer's stricter validation deliberately, not the legacy page's
permissiveness. No backend/DB/REST-contract change; no modification to
`cpaltracker.web`.

## Requirement References (SSoT = Feature doc)

Feature spec: [`React-History-Dashboard.md`](../features/React-History-Dashboard.md)
— 22 FRs, 9 NFRs, 25 ACs. This epic references requirements by ID only;
requirement text lives solely in the feature doc.

## KPIs

- 100% of map_timeline.js's time-slider/trail/replay/tooltip/auto-fit
  behavior (CB1–CB28) replicated.
- 0 silently-swapped or silently-dropped dates accepted by the new UI
  (GAP2/GAP3/GAP4 fixed at the UI layer, consuming the REST layer as-is).
- 0 modified files under `cpaltracker.web`; 0 backend/DB/REST-contract
  changes.

## Dependencies

- KAN-60 (Core Platform) — scaffold, `ApiClient`, design tokens, test
  harness.
- KAN-116 (`/api/v1`, Done) — date-validation rules consumed as the
  deliberately-chosen stricter contract; no endpoint changes requested.
- GAP5 (nonexistent vs. not-yours indistinguishable) carried forward as a
  deliberate anti-enumeration behavior, not a defect to fix.
- GAP1 (map_timeline.js's own live/historical framing) not resolved here;
  the scrubbable map is built as a self-contained component with no
  assumption it's shared with Live Tracking (KAN-138).

## Child Stories

| Jira | Story | Pts | Covered Requirements | Software Items |
| --- | --- | --- | --- | --- |

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |

## Related Documentation

- Feature spec: [`React-History-Dashboard.md`](../features/React-History-Dashboard.md)
- Technical-Feature source: [`History-Dashboard.md`](../features/technical/History-Dashboard.md)
- Core epic: [KAN-60](https://c-pal.atlassian.net/browse/KAN-60)
- Stack decision: [`react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md)

## Jira Link

- Epic: [KAN-139](https://c-pal.atlassian.net/browse/KAN-139)
