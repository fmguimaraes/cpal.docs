# KAN-217 / KAN-229 - Cleanup: 404, document title, auth-loading (manual E2E scenarios)

Spec: `cpaltracker-react/e2e/KAN-229-cleanup-not-found-titles.spec.ts` (5 scenarios, all automated).

| ID | Tag | Scenario | automation |
|----|-----|----------|------------|
| AC1 | @SI-229-1 | Anonymous on an unknown URL: standalone 404 ("404 — Page not found", no sidebar) with `not-found-home` ("Home") linking to `/`. | playwright |
| AC2 | @SI-229-2 | Signed in on an unknown URL: the 404 renders inside the shell (`app-sidebar` visible) with `not-found-home` linking to `/`. | playwright |
| AC3 | @SI-229-3 | `document.title` on known routes is "<page> · cpaltracker-react", each page name in its own language (Suivi en direct, Historique). | playwright |
| AC4 | @SI-229-4 | `document.title` on an unknown route is "404 — Page not found · cpaltracker-react". | playwright |
| AC5 | @SI-229-5 | While the session resolves, `auth-loading` (Spinner) is visible, then disappears. | playwright |
