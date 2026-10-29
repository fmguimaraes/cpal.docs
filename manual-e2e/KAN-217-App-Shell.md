# KAN-217 / KAN-224 - One app shell (manual E2E scenarios)

Spec: `cpaltracker-react/e2e/KAN-224-app-shell.spec.ts` (7 scenarios, all automated).

| ID | Tag | Scenario | automation |
|----|-----|----------|------------|
| AC1 | @SI-224-1 | At 1440 px, an authenticated route shows one `main`, a skip link that takes focus on the first Tab, `app-sidebar`, `app-nav` and `cpal-brand-mark`. | playwright |
| AC2 | @SI-224-2 | Nav labels are unchanged (Accueil, Suivi en direct, Historique, Pilotage) with testids `nav-*`; the active item has `aria-current="page"`. | playwright |
| AC3 | @SI-224-3 | At 390 px the sidebar is a closed drawer opened from the toolbar; the nav is inside it and `cpal-brand-mark` stays visible once outside it. | playwright |
| AC4 | @SI-224-4 | `theme-toggle` flips the theme in one click; `theme-system` follows the system scheme. | playwright |
| AC5 | @SI-224-5 | The user menu opens and `logout-button` signs out with one POST, then redirects to `/login`. | playwright |
| AC6 | @SI-224-6 | `/change-password` renders inside the shell (sidebar visible) in normal mode (FR12). | playwright |
| AC7 | @SI-224-7 | `/change-password` stays shell-less in first-connection mode (`mustChangePassword` and `?premiere`). | playwright |
