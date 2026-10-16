# Epic Template

> **Single source of truth:** requirement **text** lives in exactly one place, the
> feature spec. The epic doc and the tracker description reference requirements
> **by ID only**. Do NOT copy FR/NFR/AC prose, schemas or pseudo-code into this
> document: duplicated requirement text drifts from the spec and nothing
> reconciles it. The full flow is in [`README.md`](README.md).

## Authoring Rules

Write as the Technical Leader defining an epic for the platform.

- Description is short, actionable, and objective.
- Be technical — use software-engineering knowledge and straightforward language.
- Avoid jargon and buzzwords; be assertive, not verbose.
- Expand from the initial input, but change the wording as little as possible to
  keep it authentic.
- Sections below map to the tracker's epic fields: Subject, Business Case,
  Description, Requirement References, KPIs, Dependencies.

## Tracker

| Field | Value |
| --- | --- |
| Key | `KAN-…` |

## Subject
<!-- The epic title -->

## Business Case
<!-- A brief explanation of the business need or value -->

## Description
<!-- Short, actionable, and objective description -->

## Requirement References
<!-- IDs ONLY, with a link to the feature spec. No requirement text. Example:

Feature spec: [`IN-PROGRESS-Feature-Name.md`](../features/IN-PROGRESS-Feature-Name.md)
— 17 FRs, 14 NFRs, 15 ACs. This epic references requirements by ID only;
requirement text lives solely in the feature spec.
-->

## KPIs
<!-- Copy from the feature spec's Metrics/KPIs section -->
-

## Dependencies
<!-- Other projects, teams, or third-party services -->
-

## Child Stories

<!-- One row per story. "Software Items" applies to regulated software only
     (IEC 62304): the SI UIDs the story changes; the union across stories must
     equal the feature spec's Software Items section. Delete the column otherwise. -->
| Key | Story | Pts | Covered Requirements | Software Items |
| --- | --- | --- | --- | --- |

<!-- Follow with: *Total: N points across M stories.* -->

## Requirement Traceability

<!-- MANDATORY: every FR, NFR and AC ID from the feature spec maps to at least one
     story. Fill after creating the stories and verify 100% coverage.
     IDs only — no requirement text. -->

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
- Feature spec: [`<Feature-Name>.md`](../features/<Feature-Name>.md)

## Tracker Link
<!-- Link to the Jira epic and its sprint window. Example:
- Epic: [KAN-12](https://c-pal.atlassian.net/browse/KAN-12) — sprint S3, <start> → <end>
-->
