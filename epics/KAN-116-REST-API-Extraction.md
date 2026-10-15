# Epic Template

## Jira

| Field | Value | Note |
| --- | --- | --- |
| Key | `KAN-116` | epic (authoritative — sprint mechanics) |

## Subject
REST API Extraction

## Business Case
`cpaltracker.web` has no general-purpose API, which blocks the planned UI
refactor (KAN-60) from ever feeding a React frontend: there is no stable,
documented JSON contract, no token auth, no CORS. Extracting the existing
page logic into a versioned `/api/v1/...` REST API — additively, with zero
change to current page behavior — unblocks KAN-60's "Prepare React
transition" story and removes the gap `cpaltracker.web/docs/architecture.md`
already names explicitly.

## Description
Extract every capability currently only reachable via server-rendered PHP
pages (auth, account, live tracking, history, device pilotage, client
assets) into a versioned, documented, fully unit-tested JSON REST API,
built by moving business logic into shared functions called by *both* the
existing page and the new endpoint — never by changing what the existing
page does.

## Requirement References (SSoT = Feature doc)
Feature spec: [`BACKLOG-REST-API-Extraction.md`](../features/BACKLOG-REST-API-Extraction.md)
— 15 FRs, 8 NFRs, 11 ACs, 9 edge cases. This epic references requirements
by ID only; requirement text lives solely in the feature doc.

## KPIs
- 100% of FR1-FR15 implemented and unit-tested (NFR2).
- 0 regressions in existing page behavior (NFR1), verified against the
  Area 9 regression checklist.
- 100% of FR1-FR15 and NFR1-NFR8 traced to at least one story (below).

## Dependencies
- Unblocks KAN-60 (UI Refactor) — "Prepare React transition" story depends
  on this epic's API contract.
- No dependency on other epics; builds on the existing `T_AUTH_TOKEN` table
  and PDO layer, no schema change.

## Child Stories

| Jira | Story | Pts | Covered Requirements |
| --- | --- | --- | --- |
| KAN-117 | API Foundation | 8 | FR13, NFR4, NFR6, NFR2 (harness), EC5, AC7 |
| KAN-118 | Auth API (token lifecycle) | 8 | FR1, FR2, NFR5, NFR3, EC1, EC6 (login/logout), EC "must-change-password", EC "IP throttle", AC1 (login/logout) |
| KAN-119 | Password Recovery API | 5 | FR3, FR4, NFR8, EC4, EC "password change revokes tokens", AC1 (forgot/reset) |
| KAN-120 | Account API | 5 | FR5, FR6, NFR8, AC2 |
| KAN-121 | Live Tracking API | 3 | FR7, AC3, EC3 |
| KAN-122 | History API | 5 | FR8, AC4, EC3, EC "date range" |
| KAN-123 | Device Pilotage API | 8 | FR9, FR10, FR11, AC5, EC2, EC3, EC6 (config), EC "inactive contract", EC "transfer target" |
| KAN-124 | Client Assets API | 2 | FR12, AC6, EC3, EC "logo info-leak" |
| KAN-125 | API Contract & Release Verification | 5 | FR14, FR15, NFR1, NFR7, AC8, AC9, AC10, EC "CORS disallowed origin" |

*Total: 49 points across 9 stories. All stories pointed and assigned to
Jira active sprint S2 (id 3, 2026-10-11 → 2026-10-22).*

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |
| FR1 | KAN-118 |
| FR2 | KAN-118 |
| FR3 | KAN-119 |
| FR4 | KAN-119 |
| FR5 | KAN-120 |
| FR6 | KAN-120 |
| FR7 | KAN-121 |
| FR8 | KAN-122 |
| FR9 | KAN-123 |
| FR10 | KAN-123 |
| FR11 | KAN-123 |
| FR12 | KAN-124 |
| FR13 | KAN-117 |
| FR14 | KAN-125 |
| FR15 | KAN-125 |
| NFR1 | KAN-125 |
| NFR2 | KAN-117 (harness), all stories (per-story tests) |
| NFR3 | KAN-118, KAN-120, KAN-121, KAN-122, KAN-123, KAN-124 |
| NFR4 | KAN-117, all stories |
| NFR5 | KAN-118 |
| NFR6 | KAN-117 |
| NFR7 | KAN-125 |
| NFR8 | KAN-119, KAN-120 |

*Coverage: all 15 FRs, 8 NFRs, 11 ACs and 9 edge cases mapped to at least
one story. 100% structural coverage.*

## Related Documentation
- Feature spec: [`BACKLOG-REST-API-Extraction.md`](../features/BACKLOG-REST-API-Extraction.md)
- Related epic: [KAN-60 — UI Refactor](https://c-pal.atlassian.net/browse/KAN-60)
- Task-tracking policy: [SDLC — Single source of truth](../sdlc/README.md#single-source-of-truth-cpaldocs)

## Jira Link
- Epic: [KAN-116](https://c-pal.atlassian.net/browse/KAN-116) — sprint S2, 2026-10-11 → 2026-10-22
