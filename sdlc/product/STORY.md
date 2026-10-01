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
  FR/NFR/AC IDs it covers — text stays in the feature doc (SSoT).
- Acceptance Criteria use Given/When/Then and must validate every covered
  requirement. Jira tooling and the assignment policy are in
  [`../JIRA-REFERENCE.md`](../../../cto-tools/JIRA-REFERENCE.md).

## Subject
<!-- "As a [persona], I [want to], [so that]." -->

## Feature Area
<!-- Which feature area this story belongs to (e.g., Backend API, Frontend UI, Data Model, Event Feed, Governance).
     Stories MUST be split by feature area — one area per story. -->

## Covered Requirements
<!-- MANDATORY: List ALL Feature Requirements this story addresses.
     Every story must cover at least one FR or NFR from the parent feature doc.
     Use the original requirement IDs. -->
- <!-- e.g., FR1, FR3, NFR2 -->

## Software Items
<!-- MANDATORY: the SI UIDs (04 - architecture/software-items.md) this story changes —
     a subset of the parent feature doc's "Software Items" section. IDs only.
     These are what the story's unit-test IDs (UT-<tag>-<seq>) and E2E tags (@SI-nnn)
     carry, and what the review gate checks the diff against (declared ⊇ touched). -->
- <!-- e.g., SI-015, SI-034 -->

## Description
<!-- Short, actionable, and objective description -->

## Functional Requirements
<!-- Functional requirements for THIS story, derived from the feature doc FRs listed above -->
-

## Technical Requirements
<!-- Technical requirements if any -->
-

## Acceptance Criteria
<!-- Format: Given, When, Then. Must validate ALL Covered Requirements above. -->
- **Given**:
  **When**:
  **Then**:
