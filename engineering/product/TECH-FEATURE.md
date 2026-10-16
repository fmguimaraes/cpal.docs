# Technical-Feature Template

> **What this is, vs. `FEATURE.md`:** a `FEATURE.md` specifies *new* behavior
> from a product ask — the requirement comes first, the code follows. A
> **Technical-Feature** is the reverse: the code already exists, and this
> document is what's extracted *from* it — reverse-engineered current
> behavior, not a forward-looking spec. Used when the lifecycle's job is to
> produce an inventory of an existing system (e.g. to seed a rebuild), not to
> implement something new.
>
> Same SSoT discipline applies: this is still the one place this content is
> written. An epic/story built from this doc references it by ID, never
> copies its text.

## Authoring Rules

Write as a Technical Leader auditing existing, running code — not as a
Product Manager proposing something new.

- Every "Current Behavior" item (CB-n) must cite the file(s)/function(s) it
  was read from. No current-behavior claim without a code citation.
- Keep "Current Behavior" descriptive (what the system does today, including
  its quirks) and "Gaps"/"Suggested Improvements" clearly separated from it —
  don't blend "what it does" with "what it should do" in the same sentence.
- A Gap is something the current code is missing, inconsistent about, or
  does unsafely/incorrectly relative to its own stated intent (e.g. a
  documented rule that one page enforces and another doesn't). A Suggested
  Improvement is the author's own recommendation — label it as such, don't
  present it as a current-behavior fact.
- Be technical and specific; avoid vague terms like "should be better".

## 1. Feature Area

<!-- Short, precise name — the grouping this doc covers (one area per doc). -->

## 2. Scope (files / pages covered)

<!-- Every file this doc's analysis is based on. If a file is deliberately
     excluded from this area's scope, say so and why. -->
-

## 3. Current Behavior (CB-n)

<!-- Numbered, sequential, unique IDs (CB1, CB2, ...) — used for traceability
     the same way FR/NFR ids are. Each item: what the system does today,
     with a file/function citation. Binary and testable, like an FR, but
     descriptive of existing behavior, not prescriptive of new behavior.
     Example: "CB3. login.php rejects a login attempt when
     cpal_ip_bloquee($pdo) returns true (auth.php), independent of which
     account is targeted." -->
-

## 4. Current Data Model & Contracts

<!-- Tables/fields touched, request/response shapes already in use (even if
     undocumented elsewhere), session/cookie state. Cite source. -->

## 5. Current Integration Points

<!-- External systems/services this area already talks to (DB, mailer,
     HIBP, TTN, CDN assets, etc.), with direction (read/write) and file. -->

## 6. Edge Cases Already Handled

<!-- Edge cases the current code visibly handles (with citation) — not
     hypothetical ones; those go under Gaps if missing. -->
-

## 7. Gaps (GAP-n)

<!-- Numbered, sequential, unique IDs (GAP1, GAP2, ...). Something missing,
     inconsistent, or unsafe relative to the codebase's own stated rules or
     documented intent — not a wishlist item (that's §8). Cite evidence. -->
-

## 8. Suggested Improvements (IMP-n)

<!-- Numbered, sequential, unique IDs (IMP1, IMP2, ...). The author's own
     recommendations — explicitly labeled as recommendations, not current
     fact or confirmed scope. May address a Gap (reference its GAP-n) or be
     a standalone suggestion (e.g. for the future React UI). -->
-

## 9. Open Questions

<!-- Anything that couldn't be resolved by reading the code alone (e.g.
     server-only files, deployment config not in this repo). -->
-

## 10. Coverage Checklist

<!-- Binary checklist: every file in §2 has at least one CB-n citing it;
     every documented business rule found has either a CB-n or a GAP-n. -->
- [ ] Every file in Scope (§2) cited by at least one CB-n.
- [ ] Every authz/validation rule found is captured (as CB-n or GAP-n).
