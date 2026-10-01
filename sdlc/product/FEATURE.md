1. Feature Name

(Give the feature a short, precise, unambiguous name.
Avoid vague terms. Choose a name the whole team can instantly recognize.)

2. Problem / Pain Point

(Describe the real problem this feature solves.
Who experiences this pain?
What happens today without this feature?
Why is this important for the user AND for the product?)/

3. Goal / Outcome (Success Definition)

(What does success look like when this feature is working perfectly?
Describe the outcome from the user’s perspective, not the internal implementation.
This section defines “what good looks like.”)

4. User Stories (Primary Scenarios)

(List 3–5 user stories representing real workflows this feature must support.
Use the structure:
“As a [user persona], I want [action], so that [benefit].”
Keep them concrete and realistic.)

5. In Scope (What WILL be delivered)

(List exactly what is included in this feature.
Use bullet points.
Each item must be measurable, testable, and unambiguous.
Include UI screens, backend behaviors, validations, transformations, and integrations.)

6. Out of Scope (What WILL NOT be delivered)

(List explicitly what is not included in this iteration of the feature.
This protects against scope creep and sets clear expectations.
Be explicit: formats not supported, workflows not included, UX not implemented, etc.)

7. User Flow / UX Breakdown

*(Describe step-by-step what the user does and what the system does.
This should read like a screenplay:

User does X

System responds with Y

User selects Z
This is the section developers rely on most.)*

8. Functional Requirements (FRs)

(Numbered list of behaviors the system MUST support.
Each FR should describe a single requirement in a binary, testable way.
Each FR MUST have a unique sequential ID (FR1, FR2, ...) — these IDs are used for traceability into Epics and User Stories. Every FR listed here MUST appear in at least one User Story.
Example: "FR3. System SHALL validate .h5ad files and reject invalid metadata.")

9. Non-Functional Requirements (NFRs)

(Requirements related to performance, security, compliance, reliability, usability.
Each NFR MUST have a unique sequential ID (NFR1, NFR2, ...) — these IDs are used for traceability into Epics and User Stories. Every NFR listed here MUST appear in at least one User Story.
Examples:
– NFR1. Upload must support files up to 100 MB
– NFR2. All data must remain in EU/France cloud regions
– NFR3. UI must respond within <300 ms
These protect you from "implicit expectations.")

10. Data Inputs & Outputs

(List the input formats, schemas, fields, and the expected output formats or structures.
Describe what data comes in, how it’s transformed, and what comes out.
Critical for anything touching analytics or ML.)

11. Integration Points

(List all services, APIs, components, or external systems this feature interacts with.
Include read/write behavior, data flow direction, and dependencies.
This avoids hidden complexity.)

11a. Software Items (IEC 62304 §5.3 — MANDATORY)

(List the software items this feature touches, by UID from the Software Item Registry
`04 - architecture/software-items.md` — IDs and a one-line "what changes in it", never a
copy of the registry row. A feature is a requirements-axis object and maps many-to-many
onto items; do NOT invent "one item per feature".
Format:
– SI-015 (Evidence & governance) — new evidence-type field sourcing rules
– SI-034 (Evidence UI) — declaration form gains per-field scope selector
If the feature needs an item that does not exist yet, add it to the registry FIRST (same
SSoT rule as requirements), then reference it here. Stories created from this feature
each declare the subset of these items they change; unit-test IDs and E2E tags carry the
item UID; the review gate checks the diff touched only declared items.)

12. Edge Cases & Constraints

(Describe unusual or extreme conditions the feature must handle safely.
Examples: corrupted files, missing metadata, empty datasets, timeouts, user cancellation.
This prevents bugs and rework later.)

13. Metrics / KPIs

(Define how success will be measured.
Examples:
– Upload success rate > 95%
– Median UMAP compute time < 15s
– AI responses with citations > 90%
These make the feature measurable and accountable.)

14. Risks & Mitigations

(List potential risks (technical, UX, compliance, performance) and how you plan to mitigate them.
Shows foresight and reduces surprises during development.)

15. Acceptance Criteria (Definition of Done)

(Binary checklist indicating exactly when this feature is considered complete.
Every item must be yes/no.
Example:
[ ] Supports upload of .mtx and .h5ad
[ ] Displays UMAP
[ ] Handles missing metadata gracefully
[ ] All FRs and NFRs validated)

16. Future Extensions (Optional, Not Required Now)

(List enhancements intentionally postponed.
Keeps the MVP focused while documenting the long-term vision.
Prevents stakeholders from trying to inject future features into the current scope.)





