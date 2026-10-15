1. Feature Name

Technical Feature Discovery — cpaltracker.web

2. Problem / Pain Point

The planned React UI rebuild (KAN-60) and the ongoing REST API extraction
(KAN-116) both need a complete, trustworthy inventory of what
`cpaltracker.web` actually does today. No such inventory exists: the
per-repo docs (`architecture.md`, `security.md`, `design-and-standards.md`)
describe the stack and cross-cutting concerns, but no document walks every
page, lists everything it does, and flags what's missing or inconsistent.
Building a new frontend without that inventory risks silently dropping a
capability a client currently relies on, or carrying forward an
inconsistency (e.g. a rule enforced on one page but not another) into the
new UI.

3. Goal / Outcome (Success Definition)

Every existing page/capability in `cpaltracker.web` is documented as a
**Technical-Feature** (reverse-engineered current behavior, gaps, and
suggested improvements — see `cpal.docs/sdlc/product/TECH-FEATURE.md`),
grouped by feature area, with 100% of the codebase's visible behavior
cited to at least one Technical-Feature doc. The final deliverable is a
single consolidated coverage list (current behavior + gaps + suggested
improvements, across all areas) ready to be used directly as the
requirement input for KAN-60's story-splitting.

4. User Stories (Primary Scenarios)

- As the engineer planning KAN-60, I want a complete list of what every
  existing page does, so that the new UI doesn't silently drop a capability.
- As the engineer planning KAN-60, I want a list of gaps and inconsistencies
  in the current implementation, so that the new UI can fix them instead of
  carrying them forward.
- As a technical reviewer, I want each reverse-engineered behavior claim
  cited to the file/function it came from, so that the inventory is
  verifiable against the actual code, not guessed.
- As the product owner, I want suggested improvements clearly separated
  from current-behavior facts, so that I can decide what to accept without
  confusing "what it does" with "what someone recommends".

5. In Scope (What WILL be delivered)

- One Technical-Feature doc per feature area (see §11a-equivalent grouping
  below), each following `product/TECH-FEATURE.md`: Current Behavior
  (CB-n, cited), Current Data Model & Contracts, Current Integration
  Points, Edge Cases Already Handled, Gaps (GAP-n), Suggested Improvements
  (IMP-n), Open Questions, Coverage Checklist.
- Feature areas, covering every file in the repo's web root:
  1. Auth & Session (`login.php`, `logout.php`, `auth.php`)
  2. Password Recovery & Account Management (`forgot_password.php`,
     `reset_password.php`, `account.php`, `password_policy.php`)
  3. Live Tracking (`index.php`, `map_timeline.js` live mode)
  4. History Dashboard (`dashboard.php`, `api_dashboard.php`, chart/timeline
     rendering)
  5. Device Pilotage (`pilotage.php`)
  6. Client Branding & Assets (`client_logo.php`, `auth.css`/`styles.css`
     shared conventions)
  7. Device Ingestion (`insert_data.php`) — machine-facing (TTN webhook
     target), included for completeness, explicitly flagged as non-UI in
     its own doc.
- A final **consolidated coverage list** (one doc) rolling up every area's
  Current Behavior, Gaps and Suggested Improvements into a single
  cross-referenced inventory, with an explicit "ready for KAN-60" framing.

6. Out of Scope (What WILL NOT be delivered)

- Any code change. This epic produces documentation only; `pr: false`'s
  "review the local diff" path applies to the docs commits, not to any
  application behavior.
- Implementing any Suggested Improvement — that's for KAN-60 (UI) or
  follow-up epics to pick up, story by story, after this epic's sign-off.
- The TTN bridge scripts, `config.php`-level deployment specifics, or
  anything already covered in depth by `security.md`/`deployment.md` — this
  epic cites and cross-references those docs rather than re-deriving them.
- Analysis of KAN-116's new `/api/v1` layer — that layer is already
  documented as it's built (see `cpaltracker.web/docs/architecture.md`);
  this epic covers the pre-existing pages KAN-116 extracts *from*.

7. User Flow / UX Breakdown

Not applicable in the usual sense (this is an analysis epic, not a feature
with a user-facing flow). The "flow" is the analysis procedure, specified
per-story: read every file in the area's scope, extract CB-n with
citations, cross-check against `security.md`/`design-and-standards.md`/
`architecture.md` for already-known gaps, identify new gaps, propose
improvements, fill the Coverage Checklist.

8. Functional Requirements (FRs)

- FR1. A Technical-Feature doc SHALL exist for "Auth & Session", covering
  every rule in `login.php`, `logout.php` and `auth.php`.
- FR2. A Technical-Feature doc SHALL exist for "Password Recovery &
  Account Management", covering every rule in `forgot_password.php`,
  `reset_password.php`, `account.php` and `password_policy.php`.
- FR3. A Technical-Feature doc SHALL exist for "Live Tracking", covering
  `index.php` and `map_timeline.js`'s live-mode behavior.
- FR4. A Technical-Feature doc SHALL exist for "History Dashboard",
  covering `dashboard.php`, `api_dashboard.php` and the chart/timeline
  rendering they drive.
- FR5. A Technical-Feature doc SHALL exist for "Device Pilotage", covering
  every rule in `pilotage.php`.
- FR6. A Technical-Feature doc SHALL exist for "Client Branding & Assets",
  covering `client_logo.php` and the shared CSS conventions.
- FR7. A Technical-Feature doc SHALL exist for "Device Ingestion", covering
  `insert_data.php`, explicitly flagged as non-UI-facing.
- FR8. Every Technical-Feature doc's Current Behavior (CB-n) item SHALL
  cite the specific file/function it was read from.
- FR9. A single consolidated coverage-list doc SHALL exist, rolling up
  every area's CB-n/GAP-n/IMP-n into one cross-referenced inventory.
- FR10. The consolidated doc SHALL include a traceability check: every file
  in the web root (per `cpaltracker.web/docs/architecture.md`'s Components
  table) is cited by at least one CB-n in some area's doc.

9. Non-Functional Requirements (NFRs)

- NFR1. **Zero code changes**: this epic touches no `.php`/`.js` application
  file; its diffs are documentation-only (`cpal.docs/features/technical/`
  and the consolidated doc).
- NFR2. **Citation discipline**: no Current Behavior (CB-n) claim without a
  file/function citation; no Gap (GAP-n) asserted without evidence (a
  missing check, an inconsistency between two cited behaviors, or a
  documented rule from `security.md`/`design-and-standards.md` that the
  code doesn't follow).
- NFR3. **Separation of fact and recommendation**: Suggested Improvements
  (IMP-n) are never phrased as current behavior; a reader must be able to
  tell, from the section alone, that §8 is the author's recommendation.
- NFR4. **Full coverage**: every file in scope (§5) is cited by at least
  one CB-n in its area's doc (the Coverage Checklist in
  `TECH-FEATURE.md` §10 is filled and checked for every doc before its
  story is Done).

10. Data Inputs & Outputs

- Input: the existing `cpaltracker.web` source tree (read-only) and its
  existing docs (`architecture.md`, `security.md`, `design-and-standards.md`,
  `testing-and-cicd.md`, `deployment.md`, `local-setup.md`).
- Output: 7 Technical-Feature docs + 1 consolidated coverage-list doc, all
  under `cpal.docs/features/technical/`.

11. Integration Points

- None (read-only analysis of the existing codebase and its docs).

11a. Software Items (IEC 62304 §5.3)

Not applicable — see `cpal.docs/sdlc/README.md#applicability-to-c-pal-today`.

12. Edge Cases & Constraints

- A file whose behavior depends on server-only config (`config.php`
  env-var defaults aside) or a gitignored script (TTN bridge) is cited as
  an Open Question (§9 of `TECH-FEATURE.md`), not guessed.
- Where two pages implement the same rule slightly differently (e.g. a
  validation check present on one page but not another), this is a GAP-n
  in whichever area's doc covers the inconsistency, cross-referenced from
  the other area's doc rather than duplicated.
- A behavior already fully documented in `security.md`/
  `design-and-standards.md` is still cited (CB-n/GAP-n with a pointer to
  that doc) rather than silently omitted — the Technical-Feature doc must
  be self-contained enough to seed KAN-60 without requiring the reader to
  cross-reference three other documents first.

13. Metrics / KPIs

- 100% of the web root's files (per `architecture.md`'s Components table,
  excluding KAN-116's new `/api/v1` layer) cited by at least one CB-n.
- 0 Current Behavior claims without a file/function citation (NFR2).
- 1 consolidated coverage-list doc produced, explicitly framed as ready
  for KAN-60 story-splitting.

14. Risks & Mitigations

- Risk: reverse-engineered "current behavior" drifts from actual behavior
  if read carelessly. Mitigation: NFR2's citation discipline, plus a
  review pass (Opus) cross-checking each doc's CB-n claims against the
  cited file.
- Risk: scope creep into actually implementing Suggested Improvements.
  Mitigation: §6 Out of Scope is explicit; IMP-n items are recommendations
  for KAN-60/follow-up epics to pick up, not this epic's work.
- Risk: double-documenting what `security.md` already covers in depth.
  Mitigation: cite and cross-reference rather than re-derive (§12).

15. Acceptance Criteria (Definition of Done)

- [ ] Technical-Feature doc for Auth & Session exists and its Coverage
      Checklist is fully checked (FR1).
- [ ] Technical-Feature doc for Password Recovery & Account Management
      exists and its Coverage Checklist is fully checked (FR2).
- [ ] Technical-Feature doc for Live Tracking exists and its Coverage
      Checklist is fully checked (FR3).
- [ ] Technical-Feature doc for History Dashboard exists and its Coverage
      Checklist is fully checked (FR4).
- [ ] Technical-Feature doc for Device Pilotage exists and its Coverage
      Checklist is fully checked (FR5).
- [ ] Technical-Feature doc for Client Branding & Assets exists and its
      Coverage Checklist is fully checked (FR6).
- [ ] Technical-Feature doc for Device Ingestion exists and its Coverage
      Checklist is fully checked (FR7).
- [ ] Every CB-n across all docs cites a file/function (FR8, NFR2).
- [ ] Consolidated coverage-list doc exists, cross-referencing every
      area's CB-n/GAP-n/IMP-n (FR9).
- [ ] Consolidated doc's file-coverage check passes 100% (FR10, NFR4).
- [ ] All FRs and NFRs validated.

16. Future Extensions (Optional, Not Required Now)

- Using this epic's output directly as the requirement-traceability input
  for KAN-60's Workflow 1 (feature processing), so KAN-60's stories map
  back to specific CB-n/IMP-n ids instead of being re-derived from scratch.
- A follow-up epic picking up specific IMP-n items that aren't React-UI
  specific (e.g. a backend-only gap fix).
