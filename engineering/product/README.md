# Product: from feature to stories

How a product need becomes specified, planned and traceable work. The templates in
this folder are the shapes each step produces.

```
Feature spec  →  Epic  →  Stories  →  code
(requirement     (IDs     (one per     (tests carry
 text, IDs)       only)    feature      the AC IDs)
                           area)
```

| Template | Holds |
|---|---|
| [`FEATURE.md`](FEATURE.md) | The feature spec: the **only** place requirement text lives (FR/NFR/AC IDs, edge cases, KPIs) |
| [`TECH-FEATURE.md`](TECH-FEATURE.md) | A reverse-engineered spec of code that already exists: current behavior (CB-n), gaps (GAP-n), suggested improvements (IMP-n). Used to inventory a system, for example before a rebuild |
| [`EPIC.md`](EPIC.md) | The epic: requirement references by ID, child stories, requirement traceability table |
| [`STORY.md`](STORY.md) | A story: subject, feature area, covered requirement IDs, Given/When/Then acceptance criteria |

## 1. Write the feature

Copy [`FEATURE.md`](FEATURE.md) into the features folder. Every requirement gets a
unique sequential ID: `FR1…`, `NFR1…`, and the acceptance criteria are a numbered
checklist (`AC1…`).

The feature's status is its **filename prefix**. There is no status field inside
the file.

| Filename | Status |
|---|---|
| `BACKLOG-<Feature-Name>.md` | written, not started |
| `IN-PROGRESS-<Feature-Name>.md` | being built |
| `<Feature-Name>.md` | delivered |

Rename the file to change status, and fix any links to it.

## 2. Build the requirement inventory

Read the whole spec and list every numbered item: all FRs, NFRs, ACs and edge
cases. Each one must end up covered by at least one story. Then group the
requirements into **feature areas** (for example: Backend API, Page Layout, Data
Model, Navigation, Permissions). A feature area is a logical boundary, not a repo
boundary; each becomes one or more stories.

## 3. Create the epic

Create the epic in the issue tracker and write its doc from [`EPIC.md`](EPIC.md),
named `<EPIC-KEY>-<Epic-Name>.md`. It lists the requirement IDs it delivers, with a
link to the feature spec. **IDs only, never requirement text.** The tracker's
description is a one- or two-sentence summary plus links to the epic and feature docs.

## 4. Create the stories

One story per feature area, written from [`STORY.md`](STORY.md). Each story lists
the requirement IDs it covers and has Given/When/Then acceptance criteria that
validate every one of them.

- Don't copy pseudo-code, schemas, formulas, edge cases or business rules into a
  story. Reference them by ID and link to the spec.
- If a story needs a detail the spec doesn't have, add it to the spec first, with a
  new ID, then reference it.

## 5. Check coverage

Fill the epic's **Requirement Traceability** table: every FR, NFR and AC ID maps to
at least one story key. An unmapped ID means a story is missing or incomplete; fix
it before work starts. This is a table completeness check, not a text comparison,
since no text is duplicated.

## Rules that hold throughout

- Requirement text is written once, in the feature spec. Everything else
  references it by ID ([single source of truth](../documentation.md#single-source-of-truth)).
- When implementation shows the spec is wrong, the spec is amended first
  ([docs change with the code](../documentation.md#docs-change-with-the-code)).
- An epic is accepted by a person, after the stories are done and the integrated
  feature is verified against its acceptance criteria.
