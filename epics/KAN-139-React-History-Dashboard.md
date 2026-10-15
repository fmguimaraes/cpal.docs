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
— 26 FRs, 9 NFRs, 25 ACs (corrected from the original 22-FR estimate against
the file's actual FR1–FR26 numbering, confirmed during Workflow 3). This
epic references requirements by ID only; requirement text lives solely in
the feature doc.

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
| KAN-170 | Date Range Selection & Validation | 5 | FR3, FR5-8, NFR4 | N/A |
| KAN-172 | Auth, Contract/Device Scoping & Access Security | 3 | FR1, FR2, FR25, NFR5 | N/A |
| KAN-174 | Time-Slider & Trail Replay Map | 13 | FR10-18, NFR1, NFR3 | N/A |
| KAN-175 | Charts, Device Checklist, Lot-Hooks & Selection Summary | 8 | FR4, FR9, FR19-24, NFR3 | N/A |
| KAN-176 | Design System Theming, Stack Compliance & Test Coverage | 8 | FR26, NFR2, NFR6-9 | N/A |

*Total: 37 points across 5 stories. All stories pointed and assigned
to Jira active sprint 3 (S2, 2026-10-11 → 2026-10-22).*

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |
| FR1 | KAN-172 |
| FR2 | KAN-172 |
| FR3 | KAN-170 |
| FR4 | KAN-175 |
| FR5 | KAN-170 |
| FR6 | KAN-170 |
| FR7 | KAN-170 |
| FR8 | KAN-170 |
| FR9 | KAN-175 |
| FR10 | KAN-174 |
| FR11 | KAN-174 |
| FR12 | KAN-174 |
| FR13 | KAN-174 |
| FR14 | KAN-174 |
| FR15 | KAN-174 |
| FR16 | KAN-174 |
| FR17 | KAN-174 |
| FR18 | KAN-174 |
| FR19 | KAN-175 |
| FR20 | KAN-175 |
| FR21 | KAN-175 |
| FR22 | KAN-175 |
| FR23 | KAN-175 |
| FR24 | KAN-175 |
| FR25 | KAN-172 |
| FR26 | KAN-176 |
| NFR1 | KAN-174 |
| NFR2 | KAN-176 |
| NFR3 | KAN-174, KAN-175 |
| NFR4 | KAN-170 |
| NFR5 | KAN-172 |
| NFR6 | KAN-176 |
| NFR7 | KAN-176 |
| NFR8 | KAN-176 |
| NFR9 | KAN-176 |

Coverage: all 26 FRs and 9 NFRs mapped to at least one story.
100% structural coverage.

## Related Documentation

- Feature spec: [`React-History-Dashboard.md`](../features/React-History-Dashboard.md)
- Technical-Feature source: [`History-Dashboard.md`](../features/technical/History-Dashboard.md)
- Core epic: [KAN-60](https://c-pal.atlassian.net/browse/KAN-60)
- Stack decision: [`react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md)

## Jira Link

- Epic: [KAN-139](https://c-pal.atlassian.net/browse/KAN-139)
