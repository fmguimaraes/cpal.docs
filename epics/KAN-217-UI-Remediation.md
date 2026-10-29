# UI Remediation - cpaltracker-react onto the visual guideline

## Tracker

| Field | Value |
| --- | --- |
| Key | `KAN-217` |

## Subject

UI Remediation - cpaltracker-react onto the visual guideline

## Business Case

The 2026-10-28 audit found 9 Major-layout, 32 Major-UI and 8 Minor findings, and 0 of 9 pages conform to the guideline. The app has an off-brand palette, no kit, no guideline lint, no navigation on a phone and a production policy that blocks every map tile. Moving the app onto the reference gives the product owner screens that match what was approved in KAN-200, and gives reviewers a blocking lint instead of opinions.

## Description

Close the reference kit gaps, upgrade the stack, port tokens, theme, kit and lint into cpaltracker-react, then move every page onto the reference patterns inside one `AppShell`. The map-tile policy fix is a hotfix that can ship on its own. The existing unit and E2E suites keep passing, with the spec adjustments listed in the feature spec (C1 to C9). Each page keeps the language it has in the monolith; new strings are written in the language of the page they appear on.

Delivery order (feature spec section 7): APP-CSP can ship on its own; REF-SHELL, REF-DATA and REF-CHARTSMAP, then APP-STACK, then APP-FOUNDATION land in order; APP-SHELL, APP-LIVE, APP-HISTORY, APP-PILOTAGE and APP-AUTH can then run in parallel; APP-CLEANUP closes the feature.

## Requirement References

Feature spec: [`IN-PROGRESS-UI-Remediation.md`](../features/IN-PROGRESS-UI-Remediation.md) - 16 FRs, 5 NFRs, 6 ACs, 10 kit gaps (K-1 to K-10), 9 constraints (C1 to C9), audit findings F-1 to F-48 in scope. This epic references requirements by ID only; requirement text lives solely in the feature spec.

## KPIs

- Conforming pages: 9/9 for structure (F-45 home content and F-49 copy excepted).
- Major findings open: 0 in scope (F-37 confirmation step out).
- Raw interactive elements outside `src/ui`: 0.
- Inline `style` outside `src/ui`: 0.
- Hard-coded colors in `src/`: 0.
- Guideline lint violations: 0, blocking.

## Dependencies

- Story order: APP-SHELL depends on APP-FOUNDATION (KAN-223). Feature areas 1 to 3 (REF-*, APP-STACK, APP-FOUNDATION) land in order before the rest; APP-SHELL, APP-LIVE, APP-HISTORY, APP-PILOTAGE and APP-AUTH can run in parallel; APP-CLEANUP closes the feature. APP-CSP is independent.
- Predecessor: [KAN-200](KAN-200-UI-Bootstrap.md) (reference UI and guideline), all stories Done.
- Third-party: React 19, react-leaflet 5, shadcn/ui (Radix), Plotly, Leaflet, OpenStreetMap tiles.
- Developer Epic Context: not bound for c-PAL, skipped.

## Child Stories

| Key | Story | Pts | Covered Requirements |
| --- | --- | --- | --- |
| KAN-218 | [REF-SHELL] Reference kit gaps: shell, theme before paint, theme toggle, guideline alignment | 8 | FR1 (K-3, K-4, K-10), FR2 (K-9), NFR1, AC1, AC2, AC5, C6, C9 |
| KAN-219 | [REF-DATA] Reference kit gaps: status types, typed date entry, table virtualization | 8 | FR1 (K-1 status/health-grid/timeline part, K-5, K-6), NFR1, AC1, AC2, AC5, C9 |
| KAN-220 | [REF-CHARTSMAP] Reference kit gaps: chart series overflow, map playback, map decoupling | 8 | FR1 (K-1 map part, K-7, K-8), NFR1, AC1, AC2, AC5, C7, C9 |
| KAN-221 | [APP-STACK] Stack upgrade: React 19, react-leaflet 5, kit dependencies (no visual change) | 5 | FR3 (K-2), NFR1, AC1, AC5 |
| KAN-222 | [APP-CSP] Production CSP allows OpenStreetMap tiles | 1 | FR5, F-18, NFR1, AC1, AC4 (production part), AC5 |
| KAN-223 | [APP-FOUNDATION] Foundation: tokens, theme, kit and guideline lint ported into the app | 13 | FR4, F-10 (lint in report mode), F-11, F-12, F-13 (token), F-14, F-15, F-16 (token parts), F-17 (theme mechanism), F-20, F-42, F-43 (token), NFR1, NFR4, NFR5, AC1, AC5, C3, C5 |
| KAN-224 | [APP-SHELL] One shell: AppShell layout route, sidebar, mobile drawer, theme toggle, user menu | 8 | FR6 (F-1, F-2, F-3, F-17 toggle, F-25 shell, F-26, F-27), FR7 (shell part), F-4, F-5, NFR1, NFR2, NFR3, NFR5, AC1, AC4, AC5, C1, C6 |
| KAN-225 | [APP-LIVE] Live tracking on PAT-SPLIT | 13 | FR8, FR13, FR14, FR15 (for these files), FR5 (tile notice), F-6, F-19, F-23, F-24 (live), F-47, F-22 (live), F-36 (live files), F-28, F-29, F-34, F-31, F-32, F-38 (own files), NFR1, NFR2, NFR3, NFR5, AC1, AC4, AC5, C1, C7, C8 |
| KAN-226 | [APP-HISTORY] History on the reference History page | 13 | FR9, FR13, FR14, FR15 (for these files), FR5 (tile notice), F-7, F-13, F-21, F-24 (timeline), F-30 (history), F-39, F-40, F-22 (history), F-33 (lot), F-36 (history files), F-28, F-29, F-34, F-31, F-32, F-38 (own files), NFR1, NFR2, NFR3, NFR5, AC1, AC4, AC5, C1, C2, C7, C8 |
| KAN-227 | [APP-PILOTAGE] Pilotage on PAT-LIST and PAT-FORM | 8 | FR10, FR13, FR14, FR15 (for these files), F-8, F-22, F-30 (transfer), F-33 (device, transfer), F-36 (device list), F-37 (layout part), F-28, F-29, F-34, F-31, F-32, F-38 (own files), NFR1, NFR2, NFR3, NFR5, AC1, AC4, AC5, C1 |
| KAN-228 | [APP-AUTH] Auth pages on PAT-AUTH and account on PAT-SETTINGS | 8 | FR11, FR12, FR13, FR14 (for these files), F-9, F-25 (auth), F-46, F-41, F-28, F-29, F-34, F-31, F-32, F-38 (own files), NFR1, NFR2, NFR3, NFR5, AC1, AC4, AC5, C1 |
| KAN-229 | [APP-CLEANUP] Cleanup: blocking lint, remaining controls and states, minor batch | 13 | FR13 (close, lint blocking), FR14, FR15 (leftovers), FR16, FR4 (alias removal), F-10 (close), F-14, F-15, F-16 (control parts), F-22 (close), F-28, F-29, F-34, F-31, F-32, F-38, F-35, F-36 (close), F-43, F-44, F-45 (title part), F-48, NFR1, NFR2, NFR3, NFR4, NFR5, AC1, AC3, AC5, AC6, C1, C2, C4, C5 |

*Total: 106 points across 12 stories.*

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |
| FR1 | KAN-218, KAN-219, KAN-220 |
| FR2 | KAN-218 |
| FR3 | KAN-221 |
| FR4 | KAN-223, KAN-229 |
| FR5 | KAN-222, KAN-225, KAN-226 |
| FR6 | KAN-224 |
| FR7 | KAN-224, KAN-225, KAN-226, KAN-227, KAN-228 |
| FR8 | KAN-225 |
| FR9 | KAN-226 |
| FR10 | KAN-227 |
| FR11 | KAN-228 |
| FR12 | KAN-228 |
| FR13 | KAN-225, KAN-226, KAN-227, KAN-228, KAN-229 |
| FR14 | KAN-225, KAN-226, KAN-227, KAN-228, KAN-229 |
| FR15 | KAN-225, KAN-226, KAN-227, KAN-229 |
| FR16 | KAN-229 |
| NFR1 | KAN-218, KAN-219, KAN-220, KAN-221, KAN-222, KAN-223, KAN-224, KAN-225, KAN-226, KAN-227, KAN-228, KAN-229 |
| NFR2 | KAN-224, KAN-225, KAN-226, KAN-227, KAN-228, KAN-229 |
| NFR3 | KAN-224, KAN-225, KAN-226, KAN-227, KAN-228, KAN-229 |
| NFR4 | KAN-223, KAN-229 |
| NFR5 | KAN-223, KAN-224, KAN-225, KAN-226, KAN-227, KAN-228, KAN-229 |
| AC1 | KAN-218, KAN-219, KAN-220, KAN-221, KAN-222, KAN-223, KAN-224, KAN-225, KAN-226, KAN-227, KAN-228, KAN-229 |
| AC2 | KAN-218, KAN-219, KAN-220 |
| AC3 | KAN-229 |
| AC4 | KAN-222, KAN-224, KAN-225, KAN-226, KAN-227, KAN-228 |
| AC5 | KAN-218, KAN-219, KAN-220, KAN-221, KAN-222, KAN-223, KAN-224, KAN-225, KAN-226, KAN-227, KAN-228, KAN-229 |
| AC6 | KAN-229 |
| K-1 | KAN-219, KAN-220 |
| K-2 | KAN-221 |
| K-3 | KAN-218 |
| K-4 | KAN-218 |
| K-5 | KAN-219 |
| K-6 | KAN-219 |
| K-7 | KAN-220 |
| K-8 | KAN-220 |
| K-9 | KAN-218 |
| K-10 | KAN-218 |
| F-1 | KAN-224 |
| F-2 | KAN-224 |
| F-3 | KAN-224 |
| F-4 | KAN-224 |
| F-5 | KAN-224 |
| F-6 | KAN-225 |
| F-7 | KAN-226 |
| F-8 | KAN-227 |
| F-9 | KAN-228 |
| F-10 | KAN-223, KAN-229 |
| F-11 | KAN-223 |
| F-12 | KAN-223 |
| F-13 | KAN-223, KAN-226 |
| F-14 | KAN-223, KAN-229 |
| F-15 | KAN-223, KAN-229 |
| F-16 | KAN-223, KAN-229 |
| F-17 | KAN-223, KAN-224 |
| F-18 | KAN-222 |
| F-19 | KAN-225 |
| F-20 | KAN-223 |
| F-21 | KAN-226 |
| F-22 | KAN-225, KAN-226, KAN-227, KAN-229 |
| F-23 | KAN-225 |
| F-24 | KAN-225, KAN-226 |
| F-25 | KAN-224, KAN-228 |
| F-26 | KAN-224 |
| F-27 | KAN-224 |
| F-28 | KAN-225, KAN-226, KAN-227, KAN-228, KAN-229 |
| F-29 | KAN-225, KAN-226, KAN-227, KAN-228, KAN-229 |
| F-30 | KAN-226, KAN-227 |
| F-31 | KAN-225, KAN-226, KAN-227, KAN-228, KAN-229 |
| F-32 | KAN-225, KAN-226, KAN-227, KAN-228, KAN-229 |
| F-33 | KAN-226, KAN-227 |
| F-34 | KAN-225, KAN-226, KAN-227, KAN-228, KAN-229 |
| F-35 | KAN-229 |
| F-36 | KAN-225, KAN-226, KAN-227, KAN-229 |
| F-37 (layout) | KAN-227 |
| F-38 | KAN-225, KAN-226, KAN-227, KAN-228, KAN-229 |
| F-39 | KAN-226 |
| F-40 | KAN-226 |
| F-41 | KAN-228 |
| F-42 | KAN-223 |
| F-43 | KAN-223, KAN-229 |
| F-44 | KAN-229 |
| F-45 (title) | KAN-229 |
| F-46 | KAN-228 |
| F-47 | KAN-225 |
| F-48 | KAN-229 |
| C1 | KAN-224, KAN-225, KAN-226, KAN-227, KAN-228, KAN-229 |
| C2 | KAN-226, KAN-229 |
| C3 | KAN-223 |
| C4 | KAN-229 |
| C5 | KAN-223, KAN-229 |
| C6 | KAN-218, KAN-224 |
| C7 | KAN-220, KAN-225, KAN-226 |
| C8 | KAN-225, KAN-226 |
| C9 | KAN-218, KAN-219, KAN-220 |

Findings not listed are out of scope (feature spec section 6): F-37 confirmation step, F-45 home content, F-49 copy translation.

*Coverage: all 16 FRs, 5 NFRs, and 6 ACs mapped to at least one story, plus all 10 kit gaps, 48 in-scope finding groups and 9 constraints. 100% structural coverage.*

## Related Documentation

- Feature spec: [`IN-PROGRESS-UI-Remediation.md`](../features/IN-PROGRESS-UI-Remediation.md)
- Audit: [`2026-10-28-audit.md`](../ui/audits/2026-10-28-audit.md)
- Predecessor epic: [`KAN-200-UI-Bootstrap.md`](KAN-200-UI-Bootstrap.md)

## Tracker Link

- Epic: [KAN-217](https://c-pal.atlassian.net/browse/KAN-217) - sprint S2, 2026-10-11 to 2026-10-22
