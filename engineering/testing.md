# Testing

What to test, at which level, and how. Each app repo documents its own commands
(how to run the suite, where tests live) in its README or `docs/`; this page is
the strategy they all follow.

> **c-PAL today:** unit tests live in each app repo (for example PHPUnit in
> `cpaltracker.web`, see its [`docs/testing-and-cicd.md`](../../cpaltracker.web/docs/testing-and-cicd.md)).
> There is no shared end-to-end repo yet; when one is created, the end-to-end rules
> below apply to it.

## Unit tests

- Every change ships with unit tests for its new behavior; no code merges without them.
- Keep logic in pure functions outside UI/framework components, and test those
  directly.
- Deterministic, independent, fast (milliseconds). Mock external dependencies at
  the adapter/gateway boundary, never deep inside the code under test.
- Cover edge cases: empty, null/undefined, boundary values, error conditions.
- Name each test after the behavior it checks, and reference the acceptance
  criterion it verifies where there is one (`AC3 — rejects expired token`).
- Test code meets production standards; delete obsolete tests.

## End-to-end tests — scenario first, automate against running code

End-to-end verification drives the real app (browser, routes, live backend) and
is **automated by default**.

**Order of work for a story:**

| Step | What | Artifact |
|---|---|---|
| Design | Fix the test **contract**: routes, `data-testid`/ARIA roles, API shapes the scenarios will target | design note |
| Scenarios | Write Given/When/Then scenarios from the story's acceptance criteria; tag each one (see below). No runnable test yet. | scenario file |
| Develop + automate | Implement, then write the automated spec against the **running** slice until green | spec file under the story key |
| Testing | Run the suite headless against the local stack; walk the `manual` scenarios by hand and record the result | test report |
| Merge | Under the PR path, the suite runs in CI and gates the merge | green check |

**Why the end-to-end test isn't written before the code.** A unit test can be
written red against an interface. A browser test can't: written before any code
exists, a failing spec is ambiguous (missing feature, or a wrong selector guess?).
What *can* be fixed before code is the scenario (what must be observably true) and
the contract (routes, test ids, API shapes). The executable spec is written against
the first running version, from what is actually on the page — never from imagination.

**Scenario tags:**

| Tag | Meaning |
|---|---|
| `automation: e2e` | Automated by a committed end-to-end spec. The default. |
| `manual` | A human could observe it, but the harness can't drive it yet (platform gap, missing fixture) or it needs human judgement (visual feel, demo script). The reason is written next to the tag. |
| `unit-only` | No browser- or API-observable surface at all (a pure function, a config rule); the named unit tests are the complete test. |

A `manual` reason that names a gap is a fact about *today*. Whoever closes that gap
converts the scenarios that cite it. Before an epic is accepted, every `manual` and
`unit-only` reason in it is re-checked against the current code.

**Existing scenarios migrate lazily.** A manual scenario is automated when a story
touches its area, not in a bulk back-port against code nobody is running.

**Spec conventions:**

- One folder per epic; spec files prefixed with the story key (`KAN-123-*.spec.ts`).
- Every test title carries the acceptance-criteria IDs it verifies
  (`'AC3 AC4 — filter persists across reload'`), so traceability is greppable.
- Fixtures reference real sample files from the repo, never hypothetical ones.
- The suite runs against the local stack by default and takes `BASE_URL` for
  deployed environments.

## Calls to paid external services

Tests never call a paid or rate-limited external service (LLM APIs, SMS, payment)
with a real key. They run on recorded or stubbed responses. A missing recording is
a test failure, not a pass. A real call needs an explicit go from the project owner.
