# SDLC — c-PAL

How work is organised, tracked, built and reviewed across the c-PAL repositories.
This document is the reference; other repos point here rather than restating it.

The rules every change follows — code standards and the Review Gate, contribution,
testing, documentation — are in [`../engineering/`](../engineering/README.md).
Product templates and the feature → epic → story flow are in
[`../engineering/product/`](../engineering/product/README.md).

Relative links below assume the repos are checked out together inside `cpal.global`
(see [Repository topology](#repository-topology)).

---

## Repository topology

All projects are git submodules of one umbrella repository, `cpal.global`. Work is
done from a `cpal.global` checkout so every repo and the docs are side by side.

```
cpal.global/                 umbrella repo — CLAUDE.md, .gitmodules, shared .env (gitignored)
├── cpal.docs/               documentation — single source of truth (this repo)
├── c-PAL.web/               public website (static HTML/CSS/JS)
├── cpaltracker.web/         c-PAL tracker web app (PHP)
├── cpaltracker-react/       React/TS rebuild of cpaltracker.web's UI (KAN-60)
└── cpaltracker.ui-reference/ Reference UI: tokens, kit, page patterns (KAN-200)
```

| Repo | Remote | Role |
|---|---|---|
| `cpal.global` | `git@github.com:fmguimaraes/cpal.global.git` | Umbrella; pins a commit of every submodule |
| `cpal.docs` | `git@github.com:fmguimaraes/cpal.docs.git` | Specs, requirements, process docs (SSOT) |
| `c-PAL.web` | `git@github.com:GCO147/c-PAL.web.git` | Product / marketing website |
| `cpaltracker.web` | `git@github.com:GCO147/cpaltracker.web.git` | Tracker application |
| `cpaltracker-react` | `git@github.com:fmguimaraes/cpaltracker-react.git` | React/TS rebuild of `cpaltracker.web`'s UI (KAN-60), consuming its `/api/v1` (KAN-116). **Temporarily hosted under `fmguimaraes`**, not `GCO147` like the other product repos — the Jira account used here has no repo-create permission on `GCO147`; transfer with `gh repo transfer` once granted. |
| `cpaltracker.ui-reference` | `git@github.com:fmguimaraes/cpaltracker.ui-reference.git` | Reference UI and showcase for the visual guideline (KAN-200): the yardstick for every frontend change |

### Working with the umbrella

```bash
# Fresh clone, all submodules included
git clone --recurse-submodules git@github.com:fmguimaraes/cpal.global.git

# Existing clone: fetch/align submodules to the pinned commits
git submodule update --init --recursive

# Add a new project
git submodule add <git-url> <dir> && git commit -m "Add <dir> as submodule"
```

- A submodule is its own repo: commit and push **inside** it first, then commit the
  updated pointer in `cpal.global` (`git add <submodule> && git commit`).
- `cpal.global` only holds cross-cutting files (`CLAUDE.md`, `.gitmodules`,
  `.gitignore`, the shared `.env`). Project code never lives at its root.

---

## Single source of truth: `cpal.docs`

- Requirements, specs, acceptance criteria, edge cases, architecture and process
  docs live in `cpal.docs` — once, under a stable id.
- Tickets (Jira) and PRs **reference by id and link**; they never copy requirement
  text. A requirement changes in its source doc only.
- When a doc and a ticket/code comment disagree, the doc wins; fix the copy or the
  code, then the doc if it was the doc that was wrong.
- Layout: one folder per topic (`engineering/`, `sdlc/`, `architecture/`,
  `features/`, `epics/`), each with a `README.md` as its entry point.

Full rules: [`../engineering/documentation.md`](../engineering/documentation.md).

---

## Jira

Work is tracked in Jira project **KAN** at <https://c-pal.atlassian.net> (project
"c-PAL", board 1, team-managed). Issue keys look like `KAN-123`.

### Board statuses

The KAN board uses French status names. Process docs use the English names:

| Lifecycle (English) | KAN board | Category |
|---|---|---|
| `Backlog` | Idée | new — idea, not yet refined |
| `To Do` | À faire | to do |
| `In Progress` | En cours | in progress |
| `Testing` | En cours | in progress — the board has no testing column; testing is recorded by comment/label while the issue stays *En cours* |
| `In Review` | En revue | in progress |
| `Done` | Terminé | done |

The same statuses apply to every issue type (Epic, Feature, Story, Task, Subtask,
Bug). An epic is moved to *Terminé* by a person, after the integrated feature is
verified against its acceptance criteria — never automatically.

### Sprints

Weekly sprints named `S<n>`. Every story being worked on belongs to the current
sprint (the one that is active and whose end date hasn't passed); start one if none
is active.

### API access

Scripts and tools that call the Jira API read credentials from the gitignored
`.env` at the `cpal.global` root:

| Key | Notes |
|---|---|
| `JIRA_URL` | `https://c-pal.atlassian.net` |
| `JIRA_EMAIL`, `JIRA_TOKEN` | Account email and Atlassian API token — never committed, never passed as a literal shell arg |
| `PROJECT_KEY` | `KAN` |
| `JIRA_BOARD_ID` | `1` |
| `STORY_POINTS_FIELD`, `SPRINT_FIELD`, `START_DATE_FIELD` | `customfield_10016`, `customfield_10020`, `customfield_10015` |
| `JIRA_STATUS_MAP` | `Backlog=Idée;To Do=À faire;In Progress=En cours;Testing=En cours;In Review=En revue;Done=Terminé` |

---

## Product templates

Requirement docs are written from these templates. How they fit together
(inventory, epic, stories, traceability check) is in
[`../engineering/product/README.md`](../engineering/product/README.md).

| Template | Holds |
|---|---|
| [`engineering/product/FEATURE.md`](../engineering/product/FEATURE.md) | The feature spec — the only place requirement text lives (FR/NFR IDs, ACs, edge cases, KPIs) |
| [`engineering/product/TECH-FEATURE.md`](../engineering/product/TECH-FEATURE.md) | **Technical-Feature**: reverse-engineered current behavior (CB-n), gaps (GAP-n) and suggested improvements (IMP-n) extracted *from* existing code — used when an epic's job is to inventory an existing system (e.g. [KAN-126](https://c-pal.atlassian.net/browse/KAN-126), seeding the KAN-60 React rebuild) rather than to implement something new |
| [`engineering/product/EPIC.md`](../engineering/product/EPIC.md) | Epic doc: requirement references by ID, Child Stories table, Requirement Traceability table |
| [`engineering/product/STORY.md`](../engineering/product/STORY.md) | Story description: subject, feature area, covered requirement IDs, Given/When/Then ACs |

The `Software Items` sections (FEATURE §11a, EPIC/STORY columns) only apply to
regulated software (IEC 62304); c-PAL leaves them empty.

---

## Delivery workflow

### 1. Specify
Write or update the feature spec in `cpal.docs/features/` from
[`engineering/product/FEATURE.md`](../engineering/product/FEATURE.md); every requirement gets a stable id
(`FR<n>`, `NFR<n>`, `AC<n>`).

### 2. Plan
Create the Epic and its User Stories in Jira and their docs in `cpal.docs/epics/`
([flow](../engineering/product/README.md)). Tickets link to the `cpal.docs` ids; they do not
restate requirements. Label by area (`front`, `back`, `infra`).

### 3. Build
Work in the relevant submodule, on a short-lived branch
([contributing](../engineering/contributing.md)), with tests
([testing](../engineering/testing.md)). Move the ticket through the board statuses.

### 4. Review & merge
c-PAL uses the **reviewed direct merge**: review the local diff against the
[Review Gate](../engineering/code-standards.md#review-gate-blocking), run the
tests, then merge to `main`. The review and the tests are never skipped.

### 5. Close
Commit the submodule, push, bump the pointer in `cpal.global`, close the ticket.

---

## Git conventions

In [`../engineering/contributing.md`](../engineering/contributing.md): branch and
commit naming, merge path, submodule pushes, feature flags.

---

## Code standards & Review Gate

In [`../engineering/code-standards.md`](../engineering/code-standards.md): the
blocking Review Gate checklist and the design, clean code, DRY, testing and OWASP
Top 10 rules behind it. App repos record where they diverge in their own docs
(for example [`cpaltracker.web/docs/design-and-standards.md`](../../cpaltracker.web/docs/design-and-standards.md)).
