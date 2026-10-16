# Contributing

How code gets written, reviewed and merged. Applies to every contributor, human
or automated.

## Trunk-based workflow

- One long-lived branch, `main`. Every change lands on a short-lived branch
  cut from `main` and merges back within a day or two — never left open across
  several work sessions.
- Keep changes small: one story per branch. Splitting a story into several small
  PRs is fine; bundling several stories into one PR is not.
- Merge to `main` often. A branch that outlives its story is a signal to either
  merge what's done behind a [feature flag](#feature-flags) or split the remaining
  work into its own story — not to keep accumulating commits.

## Branch and commit naming

- **Branch name**: `KAN-<number>-<slug>` — the issue key the work is tracked
  under, plus a short kebab-case description.
- **Commit message**: lead with the issue key — `KAN-123 - <message>`.
  Maintenance with no tracked issue uses a prefix instead: `docs:`, `chore:`,
  `ci:`, `fix(ci):`.
- **No `Co-Authored-By` or tool-attribution lines** in commits or PR descriptions.
  A commit is authored by the person who approved it.
- Delete the branch (local + remote) once merged; `main` is never deleted.
- Check `git status` / `git diff` before switching or deleting branches; stash or
  branch unrelated uncommitted work, never check out over it.

## Merge path

Every change is reviewed against the
[Review Gate](code-standards.md#review-gate-blocking) and passes its tests before
it reaches `main`. The project uses one of two merge paths:

| Path | Steps |
|---|---|
| **Pull request** (default) | branch → PR → review → CI green → merge |
| **Reviewed direct merge** (small teams) | branch → review of the local diff → tests pass locally → merge to `main` |

**c-PAL uses the reviewed direct merge.** Either way, the review
and the tests are never skipped; the path only changes *where* they happen.

## Working in the umbrella repo

Every c-PAL repo is a git submodule of `cpal.global`
([topology](../sdlc/README.md#repository-topology)). A submodule is its own repo:

1. Commit and push **inside** the submodule first.
2. Then, from `cpal.global`, commit the new pointer (`git add <submodule>`,
   message `chore: bump <submodule>` or `KAN-123 - bump <submodule>`) and push.

Never push `cpal.global` pointing at a submodule commit that isn't pushed: it breaks
every other checkout. A change isn't finished while `git status` in `cpal.global`
shows a modified submodule pointer.

## PR rules

- **Review required.** A failed Review Gate item blocks the merge until fixed.
- **CI must pass** (lint, build, tests) before merge, when the repo has CI.
- **No direct pushes to `main`** outside the reviewed merge path above. Enable
  branch protection where the platform allows it; until then this is a process
  rule, and a direct push is a Review Gate violation.

## Feature flags

Use a feature flag whenever a branch needs to merge before the feature it
contains is ready to ship — so merging stays frequent without exposing
half-built behavior.

- **Where**: one committed flag file per app, `config/flags.json` (not `.env` —
  flags are a deploy-time toggle, not a secret). One boolean per flag:

  ```json
  { "newOnboardingFlow": false }
  ```

- **Naming**: `camelCase`, named for the capability, not the ticket
  (`newOnboardingFlow`, not `proj123Flag`) — the flag outlives the ticket.
- **Default**: `false` on merge. The code path behind a `false` flag must be
  inert — merging it changes nothing a user or caller observes.
- **Lifecycle**: flip to `true` when the feature is ready; remove the flag and
  the dead branch it guarded once it has been `true` in production for a full
  release with no rollback. Track the removal as a follow-up task.
- **Scope**: flags gate *incomplete features*, not environment differences
  (those belong in `.env` / `.env.example`).
