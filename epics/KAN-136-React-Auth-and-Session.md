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

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |

## Related Documentation

- Feature spec: [`React-Auth-and-Session.md`](../features/React-Auth-and-Session.md)
- Technical-Feature source: [`Auth-and-Session.md`](../features/technical/Auth-and-Session.md)
- Core epic: [KAN-60](https://c-pal.atlassian.net/browse/KAN-60)
- Stack decision: [`react-frontend-stack-decision.md`](../architecture/react-frontend-stack-decision.md)

## Jira Link

- Epic: [KAN-136](https://c-pal.atlassian.net/browse/KAN-136)
