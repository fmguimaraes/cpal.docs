# Story Template

## Authoring Rules

Write as the Product Manager (user stories) or Technical Leader (technical
stories) for the platform.

- Description is short, actionable, and objective; be technical and
  straightforward.
- Avoid jargon and buzzwords; be assertive, not verbose.
- Keep the title simple and short — a few words.
- Change the input wording as little as possible to keep it authentic; expand
  only where needed.
- A story MUST be split by feature area (one logical area per story) and list the
  FR/NFR/AC IDs it covers — text stays in the feature spec (single source of
  truth, see [`README.md`](README.md)).
- Acceptance Criteria use Given/When/Then and must validate every covered
  requirement.

## Subject
<!-- "As a [persona], I [want to], [so that]." -->

## Feature Area
<!-- Which feature area this story belongs to (e.g., Backend API, Frontend UI, Data Model, Navigation, Permissions).
     Stories MUST be split by feature area — one area per story. -->

## Covered Requirements
<!-- MANDATORY: list ALL feature-spec requirements this story addresses.
     Every story covers at least one FR or NFR from the parent feature spec.
     Use the original requirement IDs. -->
- <!-- e.g., FR1, FR3, NFR2 -->

## Software Items
<!-- Regulated software only (IEC 62304); delete this section otherwise.
     The SI UIDs this story changes — a subset of the feature spec's Software Items
     section. IDs only. The review checks the diff touched only declared items. -->
- <!-- e.g., SI-015, SI-034 -->

## Description
<!-- Short, actionable, and objective description -->

## Functional Requirements
<!-- Functional requirements for THIS story, derived from the feature-spec FRs listed above -->
-

## Technical Requirements
<!-- Technical requirements if any -->
-

## Acceptance Criteria
<!-- Format: Given, When, Then. Must validate ALL Covered Requirements above. -->
- **Given**:
  **When**:
  **Then**:
