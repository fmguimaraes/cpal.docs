# Documentation

Where each kind of document lives, and how documents stay in step with the code.

## Single source of truth

- Requirements, specs, acceptance criteria, edge cases, architecture and process
  docs live in the **documentation repo** — once, under a stable id (`FR3`,
  `NFR2`, `AC5`).
- Tickets and PRs **reference by id and link**; they never copy requirement text.
- A requirement changes in its source doc only, never by editing a copy.
- When a doc and a ticket or code comment disagree, the doc wins: fix the copy or
  the code. If the doc was the one that was wrong, fix the doc.

## Where things live

| What | Where |
|---|---|
| Feature specs, epics, architecture, process, these standards | the documentation repo, one folder per topic, each with a `README.md` entry point |
| How to set up, run, test and deploy an app | that app's repo: `README.md`, `CONTRIBUTING.md`, `docs/` |
| Where an app diverges from these standards, and why | that app's `docs/` |
| Secrets | nowhere in git; a gitignored `.env`, documented by a committed `.env.example` |

Every app repo has at least:

- **`README.md`** — what it is, quick start, links to architecture and setup docs.
- **`CONTRIBUTING.md`** — links to [`contributing.md`](contributing.md) and
  [`code-standards.md`](code-standards.md), plus anything repo-specific.
- **Setup and test instructions** a new developer can follow from a clean clone to
  a running app and a green test suite in under an hour.
- **`.env.example`** — every required variable, with a comment, no real values.

## Docs change with the code

A code change can make two kinds of prose wrong. Both are fixed by **the story that
caused the drift**, in the same change — never left for "a docs pass later".

- **The feature spec.** If building a story shows the spec is inaccurate,
  incomplete or overtaken by a decision, amend the spec first, then update the
  affected requirement ids and the epic's traceability table, before the story
  closes. Code never diverges silently from its spec: a divergence is either a spec
  amendment or a bug.
- **User-facing help.** When a story changes a page, flow, control, label or
  behavior that a help article describes, the story updates that article. If the
  update is genuinely separate work, file a follow-up story under the same epic,
  linked back. A story never closes leaving stale help untracked.

The reviewer checks both as part of the Review Gate.
