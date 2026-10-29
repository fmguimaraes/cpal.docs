# UI Bootstrap — reference UI and visual guideline

## Tracker

| Field | Value |
| --- | --- |
| Key | `KAN-200` |

## Subject

UI Bootstrap — reference UI and visual guideline

## Business Case

cpaltracker-react has no shared visual system: colors, spacing and components drift per screen, dark mode breaks piece by piece, and reviewers have nothing to check a screen against. A single reference UI and guideline give the product owner something to approve and give reviewers a yardstick before the look spreads through the app.

## Description

Deliver a standalone reference UI (`cpal.global/cpaltracker.ui-reference`) on the same stack as cpaltracker-react: tokens in light and dark, a themed kit, the page patterns with mock data, a showcase, the guideline doc, lint rules, a Review Gate item and Playwright snapshots. It does not change cpaltracker-react and uses mock data only.

## Requirement References

Feature spec: [`UI-Bootstrap.md`](../features/UI-Bootstrap.md) — 13 FRs, 5 NFRs, 10 ACs, 7 ECs, 3 KPIs. This epic references requirements by ID only; requirement text lives solely in the feature spec.

## KPIs

- 100% of the inventory items in FR3–FR7 present in the showcase.
- 0 lint violations.
- Every token pair shown on Foundations passes its contrast threshold in both themes.

## Dependencies

- Pattern stories (OVERVIEW, LIVE, HISTORY, LIST, DETAIL, FORMS, STATES) depend on SHELL and the kit stories they use; GUIDELINE depends on SHELL, KIT-*, CHARTS and MAP (see the keys in the table below).
- Third-party: shadcn/ui (Radix), Plotly, Leaflet, OpenStreetMap tiles.
- Shared files owned by the lead: `App.tsx`, `nav.ts`, `tokens.css`, `index.css`, `mocks/`, `package.json`.

## Child Stories

| Key | Story | Pts | Covered Requirements |
| --- | --- | --- | --- |
| KAN-201 | [FOUNDATIONS] Foundations showcase and reference index | 5 | FR1, FR2, FR8, NFR1, NFR3, AC1, AC2, EC1, KPI-showcase, KPI-contrast, D7 |
| KAN-202 | [SHELL] App shell and navigation chrome | 8 | FR6, FR7, FR8, NFR1, NFR2, AC3, AC4, EC6, D1 |
| KAN-203 | [KIT-FORMS] Kit: actions and form controls | 8 | FR3, FR4, FR8, NFR1, AC3 |
| KAN-204 | [KIT-OVERLAYS] Kit: overlays and feedback | 8 | FR3, FR5, FR8, NFR1, AC3 |
| KAN-205 | [KIT-DATA] Kit: data display and navigation | 13 | FR3, FR6, FR8, NFR1, AC3, EC4, EC5, EC7, D2, D3, D5, D6 |
| KAN-206 | [CHARTS] Plotly chart kit | 8 | FR6, FR8, FR12, NFR1, AC3, AC9, EC2, D4, D11 |
| KAN-207 | [MAP] Leaflet map kit | 8 | FR6, FR8, FR13, NFR1, AC3, AC9, EC2, EC3, D8, D9 |
| KAN-208 | [AUTH] Login and forgot-password pattern | 3 | FR7, FR8, NFR2, AC4, EC6 |
| KAN-209 | [GUIDELINE] Guideline, lint, Review Gate and snapshots | 8 | FR9, FR10, FR11, NFR4, NFR5, AC5, AC6, AC7, AC8, AC10, EC1, KPI-lint |
| KAN-210 | [OVERVIEW] Fleet overview pattern | 5 | FR7, FR8, NFR2, AC4, D5, D6 |
| KAN-211 | [LIVE] Live tracking pattern | 8 | FR7, FR8, FR12, FR13, NFR2, AC4, D8, D9 |
| KAN-212 | [HISTORY] History dashboard pattern | 5 | FR7, FR8, FR12, NFR2, AC4, D4, D6 |
| KAN-213 | [LIST] Device list pattern | 5 | FR7, FR8, NFR2, AC4, EC4, EC5 |
| KAN-214 | [DETAIL] Device detail pattern | 5 | FR7, FR8, NFR2, AC4 |
| KAN-215 | [FORMS] Settings and transfer dialog forms | 5 | FR4, FR7, FR8, NFR1, NFR2, AC4 |
| KAN-216 | [STATES] Empty, error, loading and 404 states | 3 | FR5, FR7, FR8, NFR2, AC4 |

*Total: 105 points across 16 stories.*

All 16 stories are Done. KAN-201 went to the operator after three review bounces. Its
last blocker was the light-theme orange text, which was 4.14:1 over its soft fill. The
operator approved the fix (`#915923` → `#85511F`, focus ring included) on 2026-10-29.

## Requirement Traceability

| Requirement | Story key(s) |
| --- | --- |
| FR1 | KAN-201 |
| FR2 | KAN-201 |
| FR3 | KAN-203, KAN-204, KAN-205 |
| FR4 | KAN-203, KAN-215 |
| FR5 | KAN-204, KAN-216 |
| FR6 | KAN-205, KAN-202, KAN-206, KAN-207 |
| FR7 | KAN-202, KAN-208, KAN-210, KAN-211, KAN-212, KAN-213, KAN-214, KAN-215, KAN-216 |
| FR8 | KAN-201, KAN-202, KAN-203, KAN-204, KAN-205, KAN-206, KAN-207, KAN-208, KAN-210, KAN-211, KAN-212, KAN-213, KAN-214, KAN-215, KAN-216 |
| FR9 | KAN-209 |
| FR10 | KAN-209 |
| FR11 | KAN-209 |
| FR12 | KAN-206, KAN-212, KAN-211 |
| FR13 | KAN-207, KAN-211 |
| NFR1 | KAN-201, KAN-203, KAN-204, KAN-205, KAN-202, KAN-206, KAN-207, KAN-215 |
| NFR2 | KAN-202, KAN-208, KAN-210, KAN-211, KAN-212, KAN-213, KAN-214, KAN-215, KAN-216 |
| NFR3 | KAN-201 |
| NFR4 | KAN-209 |
| NFR5 | KAN-209 |
| AC1 | KAN-201 |
| AC2 | KAN-201 |
| AC3 | KAN-203, KAN-204, KAN-205, KAN-202, KAN-206, KAN-207 |
| AC4 | KAN-202, KAN-208, KAN-210, KAN-211, KAN-212, KAN-213, KAN-214, KAN-215, KAN-216 |
| AC5 | KAN-209 |
| AC6 | KAN-209 |
| AC7 | KAN-209 |
| AC8 | KAN-209 |
| AC9 | KAN-206, KAN-207 |
| AC10 | KAN-209 |
| EC1 | KAN-209, KAN-201 |
| EC2 | KAN-206, KAN-207 |
| EC3 | KAN-207 |
| EC4 | KAN-205, KAN-213 |
| EC5 | KAN-205, KAN-213 |
| EC6 | KAN-208, KAN-202 |
| EC7 | KAN-205 |
| KPI-showcase | KAN-201 |
| KPI-lint | KAN-209 |
| KPI-contrast | KAN-201 |
| D1 | KAN-202 |
| D2 | KAN-205 |
| D3 | KAN-205 |
| D4 | KAN-206, KAN-212 |
| D5 | KAN-205, KAN-210 |
| D6 | KAN-205, KAN-210, KAN-212 |
| D7 | KAN-201 |
| D8 | KAN-207, KAN-211 |
| D9 | KAN-207, KAN-211 |
| D11 | KAN-206 |

*Coverage: all 13 FRs, 5 NFRs, and 10 ACs mapped to at least one story. 100% structural coverage.*

## Related Documentation

- Feature spec: [`UI-Bootstrap.md`](../features/UI-Bootstrap.md)

## Tracker Link

- Epic: [KAN-200](https://c-pal.atlassian.net/browse/KAN-200) — sprint S2, 2026-10-12 → 2026-10-23
