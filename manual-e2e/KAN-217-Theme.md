# KAN-217 / KAN-223 - Theme before first paint (manual E2E scenarios)

Spec: `cpaltracker-react/e2e/KAN-223-theme-before-first-paint.spec.ts` (2 scenarios, all automated).

| ID | Tag | Scenario | automation |
|----|-----|----------|------------|
| AC1 | @SI-223-1 | A stored light theme is on `<html>` before React mounts, with no CSP violation. | playwright |
| AC2 | @SI-223-2 | Without a stored choice the default dark theme is applied before React mounts. | playwright |
