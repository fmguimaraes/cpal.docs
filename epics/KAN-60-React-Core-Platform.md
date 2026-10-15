## Jira

| Field | Value | Note |
| --- | --- | --- |
| Key | `KAN-60` | epic (authoritative — sprint mechanics) |

## Subject

React Core Platform & Hello World — cpaltracker.web UI rebuild foundation

## Business Case

KAN-60 ("UI Refactor") previously had no scope beyond "a beautiful UI for
the December salon." The Technical Feature Discovery epic (KAN-126, Done)
and the REST API Extraction epic (KAN-116, Done) now give this rebuild a
trustworthy inventory of existing behavior and a stable API to build
against. Before any of the 6 feature-area epics it seeds can start, one
epic must fix the stack, the project scaffold, API connectivity
(local/staging and prod, switchable without a code change), and a
centralized, theme-based design system — otherwise each feature epic
re-decides these independently and drifts. This epic is that foundation,
delivered as a working scaffold plus a hello-world proof, with zero change
to the existing PHP monolith.

## Description

Stand up a new React 18.3 + TypeScript 5.5 (Vite 5) application: routing,
state, an `ApiClient` against `/api/v1` with a runtime-switchable
local/staging-vs-prod target, an `AuthContext` scaffold backed by a REST/
cookie client (not Supabase — see stack decision), a centralized design-
token system driving light/dark themes with desktop as the primary target,
and a working unit (Vitest/Testing Library) + E2E (Playwright) test harness.
Deliver a hello-world screen proving live `/api/v1` connectivity end to end.
Purely additive: no backend/DB change, no modification to any
`cpaltracker.web` PHP page.

## Requirement References (SSoT = Feature doc)

Feature spec: [`React-Core-Platform.md`](../features/React-Core-Platform.md)
— 11 FRs, 5 NFRs, 8 ACs. This epic references requirements by ID only;
requirement text lives solely in the feature doc.

Stack and design-system decisions are recorded once in
[`react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md)
and referenced by this epic and all 6 sibling feature epics rather than
restated.

## KPIs

- 100% of `npm run lint`, `npm run typecheck`, `npm run test`, `npm run
  test:e2e` passing on a clean checkout.
- 0 modified files under `cpaltracker.web` as a result of this epic.
- 1 working end-to-end connectivity proof against a real `/api/v1`
  endpoint, both local/staging and prod targets demonstrated.

## Dependencies

- KAN-116 (REST API Extraction, Done) — `/api/v1` is consumed as-is; no
  endpoint changes requested or made under this epic.
- KAN-126 (Technical Feature Discovery, Done) — source of the CB-n/GAP-n
  citations the 6 sibling feature epics build against.
- No Jira parent-epic link exists in this project's Jira schema (Epic has
  no parent-Epic hierarchy here); the 6 sibling epics below reference this
  one by id in their own `Dependencies` section instead.

## Child Stories

| Jira | Story | Pts | Covered Requirements | Software Items |
| --- | --- | --- | --- | --- |
| KAN-142 | Project Scaffold & Tooling | 5 | FR1, FR11, NFR1-3 | N/A |
| KAN-143 | API Client & Dual-Target Connectivity | 5 | FR2, FR3, NFR4 | N/A |
| KAN-144 | AuthContext Scaffold | 3 | FR4 | N/A |
| KAN-147 | Design Token System & Theming | 8 | FR5-7, NFR5 | N/A |
| KAN-150 | Routing & Hello-World Screen | 5 | FR8 | N/A |
| KAN-153 | Test Harness (Unit+E2E) | 5 | FR9, FR10 | N/A |

*Total: 31 points across 6 stories. All stories pointed and assigned
to Jira active sprint 3 (S2, 2026-10-11 → 2026-10-22).*

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |
| FR1 | KAN-142 |
| FR2 | KAN-143 |
| FR3 | KAN-143 |
| FR4 | KAN-144 |
| FR5 | KAN-147 |
| FR6 | KAN-147 |
| FR7 | KAN-147 |
| FR8 | KAN-150 |
| FR9 | KAN-153 |
| FR10 | KAN-153 |
| FR11 | KAN-142 |
| NFR1 | KAN-142 |
| NFR2 | KAN-142 |
| NFR3 | KAN-142 |
| NFR4 | KAN-143 |
| NFR5 | KAN-147 |

Coverage: all 11 FRs and 5 NFRs mapped to at least one story.
100% structural coverage.

## Related Documentation

- Feature spec: [`React-Core-Platform.md`](../features/React-Core-Platform.md)
- Stack decision: [`react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md)
- Sibling epics (built on this foundation): React UI — Auth & Session,
  Password Recovery & Account Management, Live Tracking, History Dashboard,
  Device Pilotage, Client Branding & Assets.
- Task-tracking policy: [SDLC — Single source of truth](../sdlc/README.md#single-source-of-truth-cpaldocs)

## Jira Link

- Epic: [KAN-60](https://c-pal.atlassian.net/browse/KAN-60)
