# SDLC — c-PAL

How work is organised, tracked, built and reviewed across the c-PAL repositories.
This document is the reference; other repos point here rather than restating it.

Relative links below assume the repos are checked out together inside `cpal.global`
(see [Repository topology](#repository-topology)).

---

## Repository topology

All projects are git submodules of one umbrella repository, `cpal.global`. Work is
done from a `cpal.global` checkout so every repo, the shared tooling and the docs
are side by side.

```
cpal.global/                 umbrella repo — CLAUDE.md, .gitmodules, shared .env (gitignored)
├── cpal.docs/               documentation — single source of truth (this repo)
├── cto-tools/               shared scripts & dev rules used by more than one project
├── c-PAL.web/               public website (static HTML/CSS/JS)
└── cpaltracker.web/         c-PAL tracker web app (PHP)
```

| Repo | Remote | Role |
|---|---|---|
| `cpal.global` | `git@github.com:fmguimaraes/cpal.global.git` | Umbrella; pins a commit of every submodule |
| `cpal.docs` | `git@github.com:fmguimaraes/cpal.docs.git` | Specs, requirements, process docs (SSOT) |
| `cto-tools` | `git@github.com:fmguimaraes/cto-tools.git` | Cross-project scripts, code standards, Claude Code hooks |
| `c-PAL.web` | `git@github.com:GCO147/c-PAL.web.git` | Product / marketing website |
| `cpaltracker.web` | `git@github.com:GCO147/cpaltracker.web.git` | Tracker application |

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
- Layout: one folder per topic (`sdlc/`, and e.g. `specs/`, `architecture/` as they
  appear), each with a `README.md` as its entry point.

---

## Shared tooling: `cto-tools`

Anything used by more than one project goes in `cto-tools` rather than being
duplicated per repo. Today it holds:

| Path | What it is |
|---|---|
| [`CLAUDE.md`](../../cto-tools/CLAUDE.md) | Always-on dev rules: commit conventions, PR flag, blocking Review Gate |
| [`docs/code-standards.md`](../../cto-tools/docs/code-standards.md) | Full code standards behind the Review Gate (design, clean code, DRY, testing, OWASP Top 10) |
| [`config/sdlc.json`](../../cto-tools/config/sdlc.json) | Process toggles (currently `pr`) |
| [`scripts/jira/`](../../cto-tools/scripts/jira/) | Jira REST CRUD scripts |
| [`scripts/hooks/context-guard.py`](../../cto-tools/scripts/hooks/context-guard.py) + [`.claude/settings.json`](../../cto-tools/.claude/settings.json) | Token-saving Claude Code `PreToolUse` hook |
| [`.env.default`](../../cto-tools/.env.default) | Template for the shared `.env` |

### Credentials (`.env`)

One gitignored `.env` at the **`cpal.global` root** is shared by every submodule.
Create it from the template:

```bash
cp cto-tools/.env.default .env   # from cpal.global/, then fill in the values
```

| Key | Required | Notes |
|---|---|---|
| `JIRA_URL` | yes | Atlassian site URL |
| `JIRA_EMAIL` | yes | Account email for the API token |
| `JIRA_TOKEN` | yes | Atlassian API token — never committed, never passed as a literal shell arg |
| `PROJECT_KEY` | yes | Jira project key for c-PAL |
| `STORY_POINTS_FIELD` | no | Per-site custom field id (default `customfield_10016`); find it via `GET /rest/api/3/field` |

### Jira scripts

```bash
pip install -r cto-tools/scripts/jira/requirements.txt   # requests
cd cto-tools/scripts/jira
```

| Script | Usage |
|---|---|
| `epics.py` | `list [--status S]` · `create <summary> [-d DESC] [-l LABELS…]` · `read <key>` · `update <key> [-s SUMMARY] [-d DESC] [--status S] [-a ASSIGNEE] [-l LABELS…]` · `delete <key>` |
| `user_stories.py` | `list <epic>` · `create <epic> <summary> [-d DESC]` · `read <key>` · `update <key> [-s] [-d] [--status] [-a]` · `delete <key>` |
| `get_description.py` | `<key>` — description rendered as Markdown |
| `get_comments.py` | `<key>` — all comments, oldest first |
| `add_comment.py` | `<key> "<text>"` |
| `update_status.py` | `<key> "<status name>"` — transitions the issue |
| `update_assignee.py` | `<key> <email>` · `<key> --unassign` |
| `add_labels.py` | `<key> <label…>` · `<key> --remove <label…>` · `<key> --list` (suggested labels: `infra`, `back`, `front`) |
| `adf_to_markdown.py` | Library + stdin CLI converting Jira ADF JSON to Markdown |

Prefer these scripts for repeatable work; the Atlassian MCP connector in Claude Code
reaches the same site and is fine for ad-hoc lookups or operations the scripts
don't cover.

### Claude Code context guard

`context-guard.py` runs before `Read` and `Agent` tool calls and blocks:

- whole-file reads of files over `CONTEXT_GUARD_MAX_LINES` (default 300) without `offset`/`limit`;
- re-reading a file already read whole in the same session;
- subagent spawns that would silently inherit the session model — pass
  `model: "haiku"` for search/lookup, `"sonnet"` for multi-step work.

`CONTEXT_GUARD_OFF=1` disables it for a session. The hook is registered in
`cto-tools/.claude/settings.json` with a `$CLAUDE_PROJECT_DIR/scripts/...` path, so
it is active only when the Claude Code session is opened in `cto-tools`. To enforce
it across the whole umbrella, register it in `cpal.global/.claude/settings.json`
pointing at `$CLAUDE_PROJECT_DIR/cto-tools/scripts/hooks/context-guard.py`.

---

## Delivery workflow

### 1. Specify
Write or update the requirement in `cpal.docs` and give it a stable id.

### 2. Track
Create the Epic / User Story in Jira (`epics.py`, `user_stories.py`). The ticket
links to the `cpal.docs` id; it does not restate the requirement. Label by area
(`front`, `back`, `infra`).

### 3. Build
Work in the relevant submodule. Move the ticket with `update_status.py`.

### 4. Review & merge
The merge path is set by `pr` in [`cto-tools/config/sdlc.json`](../../cto-tools/config/sdlc.json)
(current value: **`false`**):

| `pr` | Path |
|---|---|
| `true` | branch → PR → review → CI → merge |
| `false` | review the local diff against the Review Gate → merge directly to `main` |

Read the flag once per unit of work; don't switch path mid-task. New process
toggles go into `config/sdlc.json`, not into prose.

### 5. Close
Commit the submodule, push, bump the pointer in `cpal.global`, close the ticket.

---

## Git conventions

- Commit message: `<JIRA-KEY> - <message>` when a ticket exists. Maintenance without
  a ticket uses `docs:` / `chore:` / `ci:` / `fix(ci):`.
- **No `Co-Authored-By` or AI-attribution lines** in commits or PR descriptions;
  commits are authored by the person who approved them.
- Every merged branch is deleted locally and remotely; `main` is never deleted.
- Check `git status` / `git diff` before switching or deleting branches; stash or
  branch unrelated uncommitted work, never check out over it.
- Secrets never enter git; `.env` is gitignored at every level.

---

## Code standards & Review Gate

Every change must pass the blocking Review Gate in
[`cto-tools/CLAUDE.md`](../../cto-tools/CLAUDE.md#review-gate-must--block-on-any-failure):
short focused methods, shallow nesting, SRP, injected dependencies, no business
logic in UI components, unit tests for new behaviour, no committed secrets,
parameterized queries, server-side access control and validation.

The elaboration (GoF patterns, SOLID, naming, error handling, DRY, testing rules,
OWASP Top 10 mapping) is in
[`cto-tools/docs/code-standards.md`](../../cto-tools/docs/code-standards.md).
That file is the authority for code standards; this doc does not duplicate it.

---

## Deliberately not in place yet

Added only when a project actually needs them:

- **Multi-agent dispatch** (feature-writer / story-developer / code-reviewer agents,
  a team-lead orchestrator).
- **Workflow/status beacon** (per-item step tracking + append-only journal).
- **Sprint/board automation** — only if c-PAL runs Jira sprints.
