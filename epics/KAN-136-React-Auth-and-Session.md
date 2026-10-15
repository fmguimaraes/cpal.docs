## Jira

| Field | Value | Note |
| --- | --- | --- |
| Key | `KAN-136` | epic (authoritative — sprint mechanics) |

## Subject

React UI — Auth & Session

## Business Case

Login, session handling, CSRF and logout are the entry point to every other
screen in the React rebuild; nothing else can be built or tested end to end
without them. `cpaltracker.web`'s current auth/session machinery is solid
but has real holes (an IP throttle bypassable via malformed POST, no clean
5-attempt reset after lockout, one rejection branch leaking account state —
see [`Auth-and-Session.md`](../features/technical/Auth-and-Session.md)
GAP1–GAP5). This epic delivers the React equivalent, deliberately fixing
what's fixable at the UI layer and logging the rest as backend open
questions rather than papering over them.

## Description

Build the React/TS login, session-bootstrap, CSRF, lockout UX and logout
screens against `/api/v1`'s existing auth endpoints, on top of KAN-60's
scaffold (`ApiClient`, `AuthContext`, design-token theming). Replicate
security-critical UI behavior (CB1–CB36) faithfully; fix GAP4 (locked-account
message leaking account state) at the UI layer with a neutral message; make
logout an explicit authenticated POST only. No backend/DB change; no
modification to `cpaltracker.web`.

## Requirement References (SSoT = Feature doc)

Feature spec: [`React-Auth-and-Session.md`](../features/React-Auth-and-Session.md)
— 15 FRs, 9 NFRs, 24 ACs. This epic references requirements by ID only;
requirement text lives solely in the feature doc.

## KPIs

- 100% of CB1–CB36 (security-critical UI behavior) replicated or explicitly
  logged as a backend-owned open question — none silently dropped.
- GAP4 fixed: locked-account message is neutral and indistinguishable from
  other lockout causes.
- 0 modified files under `cpaltracker.web`; 0 backend/DB changes.

## Dependencies

- KAN-60 (Core Platform) — scaffold, `ApiClient`, `AuthContext`, design
  tokens, test harness.
- KAN-116 (`/api/v1`, Done) — auth endpoints consumed as-is; exact endpoint
  set to confirm early per feature spec §11.
- GAP1, GAP2, GAP3, GAP5 (server-/docs-owned) logged as open backend
  questions, not addressed here.

## Child Stories

| Jira | Story | Pts | Covered Requirements | Software Items |
| --- | --- | --- | --- | --- |
| KAN-145 | Login Screen & Credential Submission | 5 | FR1, FR4, FR5, NFR4-6 | N/A |
| KAN-149 | Session Bootstrap & Protected Routes | 8 | FR2, FR3, FR13-15, NFR7 | N/A |
| KAN-151 | CSRF Token Lifecycle & Client-Side Security Constraints | 5 | FR9, NFR3, NFR8 | N/A |
| KAN-155 | Auth Outcome Messaging & Lockout UX | 5 | FR6-8, NFR9 | N/A |
| KAN-156 | Logout & Session Loss Handling | 5 | FR10-12 | N/A |
| KAN-157 | Unit + E2E Test Coverage | 8 | NFR1, NFR2 | N/A |

*Total: 36 points across 6 stories. All stories pointed and assigned
to Jira active sprint 3 (S2, 2026-10-11 → 2026-10-22).*

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |
| FR1 | KAN-145 |
| FR2 | KAN-149 |
| FR3 | KAN-149 |
| FR4 | KAN-145 |
| FR5 | KAN-145 |
| FR6 | KAN-155 |
| FR7 | KAN-155 |
| FR8 | KAN-155 |
| FR9 | KAN-151 |
| FR10 | KAN-156 |
| FR11 | KAN-156 |
| FR12 | KAN-156 |
| FR13 | KAN-149 |
| FR14 | KAN-149 |
| FR15 | KAN-149 |
| NFR1 | KAN-157 |
| NFR2 | KAN-157 |
| NFR3 | KAN-151 |
| NFR4 | KAN-145 |
| NFR5 | KAN-145 |
| NFR6 | KAN-145 |
| NFR7 | KAN-149 |
| NFR8 | KAN-151 |
| NFR9 | KAN-155 |

Coverage: all 15 FRs and 9 NFRs mapped to at least one story.
100% structural coverage.

## Related Documentation

- Feature spec: [`React-Auth-and-Session.md`](../features/React-Auth-and-Session.md)
- Technical-Feature source: [`Auth-and-Session.md`](../features/technical/Auth-and-Session.md)
- Core epic: [KAN-60](https://c-pal.atlassian.net/browse/KAN-60)
- Stack decision: [`react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md)

## Jira Link

- Epic: [KAN-136](https://c-pal.atlassian.net/browse/KAN-136)
