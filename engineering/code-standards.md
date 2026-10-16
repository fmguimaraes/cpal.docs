# Code standards

Applies when writing, refactoring or reviewing production code.

## Review Gate (blocking)

Every change is reviewed against this checklist before it reaches `main`. Any
failed item blocks the merge.

- [ ] Every method short and focused; extract when it does more than one thing.
- [ ] Shallow nesting; guard clauses / early returns over deep `if`/`else`.
- [ ] One reason to change per class/module/function (SRP).
- [ ] Dependencies injected, never `new`-ed/constructed internally (DIP).
- [ ] No business logic in UI components — extract to functions/hooks and test those.
- [ ] Unit tests written for new behavior; external deps mocked at the boundary.
- [ ] No secrets committed; parameterized queries only.
- [ ] Server-side access control on every endpoint; deny-by-default; server-side
      input validation.

The sections below explain each rule.

## Design

- Prefer **composition over inheritance**. Apply a GoF pattern where it removes a
  conditional chain or decouples change (Strategy for interchangeable algorithms,
  Factory/Abstract Factory for runtime-chosen types, Adapter to wrap third-party
  libs, Facade to hide subsystem complexity, Decorator to add behavior without
  subclassing, Observer for one-to-many notification, State for state-dependent
  behavior) — never pattern for its own sake.
- **OCP**: add a new implementation instead of editing an existing switch/if-else.
- **ISP**: many small focused interfaces over one fat one.
- **LSP**: subtypes fully substitutable; no surprise exceptions in overrides.
- Keep third-party deps behind adapters so they can be swapped.
- Enforce layer dependency direction (presentation → application → domain → infrastructure).
- Keep modules as independent as possible; communicate only through well-defined
  interfaces, never by reaching into another module's internals.

## Clean code

- **Names**: intention-revealing, pronounceable, searchable. Classes = nouns,
  methods = verbs, booleans = assertions (`isActive`, `hasPermission`). Ban
  generic names (`data`, `info`, `temp`, `result`, `item`, `obj`, `val`). One
  word per concept across the codebase; no Hungarian notation.
- **Functions**: one thing, single level of abstraction, ≤ 2 args ideally, no flag
  arguments (split into two functions), no hidden side effects.
- **Comments**: self-documenting code first; comment **why**, never **what**;
  doc-comments on public API. Delete commented-out code and un-ticketed TODOs.
- **Errors**: exceptions over error codes; never return/pass `null` (prefer empty
  collection / Optional / throw); informative messages with context; don't swallow.
- **Formatting**: lines ≤ 120 chars; public before private; related functions close together.
- **Smells to kill**: dead code, duplication, long param lists, feature envy,
  god class/method, magic numbers/strings, primitive obsession, shotgun surgery.

## DRY

- Every piece of knowledge has one authoritative representation. Extract repeated
  logic — but don't force an abstraction over two things that merely look alike and
  evolve independently. Name and place the abstraction so its purpose is clear.

## Testing

Summary; the full strategy is in [`testing.md`](testing.md).

- Every change ships with unit tests; no code merges without them.
- Favor pure functions outside UI/framework components; test those directly.
- Deterministic, independent, fast (ms). Mock at the adapter/gateway boundary.
- Cover edge cases: empty, null/undefined, boundary values, error conditions.
- Treat test code with production-quality standards; delete obsolete tests.

## Security — OWASP Top 10

Never-recoverable rule (also in the Review Gate):

- **Never commit** secrets, API keys, or credentials. Keep them in a gitignored
  `.env` / secrets manager and load them at runtime; never pass a secret as a
  literal argument to a shell command or CI step (it ends up in process lists and
  logs).

Then:

- **A01** deny-by-default access control, object-level authorization on every op.
- **A02** hash passwords (bcrypt/argon2); TLS in transit.
- **A03** parameterized queries/ORM only; server-side validation; context-aware output encoding.
- **A04/A05** least privilege; validate business flows server-side; no default accounts/verbose prod errors.
- **A06** audit deps, pin versions, drop unused deps.
- **A07** rate-limit/lock auth endpoints; invalidate sessions on logout/password change.
- **A09/A10** log auth + access-control + validation failures without PII; allowlist
  user-supplied URLs; block internal/metadata endpoints.
