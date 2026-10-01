# Epic Template

> **SSoT rule (authoritative — see [SDLC.md Workflow 2](../../../cto-tools/SDLC.md#workflow-2--epic-creation) steps 2 and 3):**
> Requirement **text** lives in exactly one place: the Feature specification.
> The epic doc and the Jira description reference requirements **by ID only**.
> Do NOT copy FR/NFR/AC prose, schemas, or pseudo-code into this document —
> duplicated requirement text drifts from the feature spec and there is no
> mechanism to reconcile it.

## Authoring Rules

Write as the Technical Leader defining an epic for the platform.

- Description is short, actionable, and objective.
- Be technical — use software-engineering knowledge and straightforward language.
- Avoid jargon and buzzwords; be assertive, not verbose.
- Expand from the initial input, but change the wording as little as possible to
  keep it authentic.
- Sections below map to the required Jira fields: Subject, Business Case,
  Description, Requirement References, KPIs, Dependencies. Jira tooling and the
  assignment policy are in [`../JIRA-REFERENCE.md`](../../../cto-tools/JIRA-REFERENCE.md).

## Jira

<!-- Jira is the sole tracker. Qualify keys as `KAN-…`. -->

| Field | Value | Note |
| --- | --- | --- |
| Key | `KAN-…` | epic (authoritative — sprint mechanics) |

## Subject
<!-- The epic title -->

## Business Case
<!-- A brief explanation of the business need or value -->

## Description
<!-- Short, actionable, and objective description -->

## Requirement References (SSoT = Feature doc)
<!-- IDs ONLY, with a link to the feature spec. No requirement text. Example:

Feature spec: [`IN-PROGRESS-Feature-Name.md`](features/IN-PROGRESS-Feature-Name.md)
— 17 FRs, 14 NFRs, 15 ACs. This epic references requirements by ID only;
requirement text lives solely in the feature doc.
-->

## KPIs
<!-- Copy from feature doc Metrics/KPIs section -->
-

## Dependencies
<!-- Other projects, teams, or third-party services -->
-

## Child Stories

<!-- Table tracking all user stories created for this epic. "Software Items" = the SI UIDs
     (04 - architecture/software-items.md) the story changes — IDs only; the union across
     stories must equal the feature doc's "Software Items" section. -->
| Jira | Story | Pts | Covered Requirements | Software Items |
| --- | --- | --- | --- | --- |

<!-- Follow with: *Total: N points across M stories. All stories pointed and assigned
     to Jira active sprint <id>.* -->

## Requirement Traceability

<!-- MANDATORY: Every FR, NFR, and AC ID from the feature doc MUST map to at least one
     User Story. After creating all stories, fill this table and verify 100% coverage.
     IDs only — no requirement text (that would violate the SSoT rule). -->

| Requirement | Story key(s) |
| --- | --- |
<!-- Example:
| FR1 | KAN-338 |
| FR2 | KAN-333, KAN-335 |
| NFR1 | KAN-333 |
-->

<!-- Close with: *Coverage: all N FRs, M NFRs, and K ACs mapped to at least one story.
     100% structural coverage.* -->

## Related Documentation
- Feature spec: [`<Feature-Name>.md`](features/<Feature-Name>.md)
- Task-tracking policy: [SDLC — Single source of truth](../README.md#single-source-of-truth-cpaldocs)

## Jira Link
<!-- Required: link to the Jira epic + its sprint window. Example:
- Epic: [KAN-12]($JIRA_URL/browse/KAN-12) — sprint S3, <start> → <end>
-->
