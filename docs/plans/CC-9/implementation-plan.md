# Implementation Plan

## Overview
* **Story**: CC-9 — Define and implement consistent “Find Owners” search behavior (rules, UX, pagination)
* **Goal**: Define and implement a consistent “Find Owners” search behavior: matching and sorting rules, no-results UX, and (if enabled) pagination behavior so that it can be implemented and tested reliably. This plan focuses on the CC-9 story requirements and the selected sub-task CC-16 for matching behavior.
* **Jira**: https://ervanshsingh.atlassian.net/browse/CC-9
* **Sub-task used**: CC-16 — Fix owner last-name search to support partial, case-insensitive matching (GAP-01) (https://ervanshsingh.atlassian.net/browse/CC-16)

## Scope

### In scope
- AC-1: Implement partial, case-insensitive last-name match behavior for owner search.
- AC-4: Confirm and implement the empty input behavior rule (either “Enter a last name” validation OR list all owners with pagination) — **requires clarification**.
- AC-5: Pagination behavior when enabled: retain search term and stable paging — only insofar as it is needed to meet AC-4/AC-5 behavior already present in controller/template (page parameter) and to ensure it remains stable after matching/sorting updates.

### Out of scope
- AC-3 no-results UX changes (“No owners found” copy, explicit CTA in no-results screen, retaining term) — would require controller + template updates and is not included in this sub-task-focused plan.
- Search fields beyond last name (first name/phone/email/address) — not specified as a requirement; remains an open question in CC-9.

## Requirements

| AC | Requirement (from story) |
|---|---|
| AC-1 | Partial, case-insensitive last-name match: searching for “Dav” returns owners with last names “Davis” and “Davison”; searching for “davis” returns “Davis”. |
| AC-2 | Sorting: results are sorted by last name, then first name. |
| AC-3 | No results UX: show “No owners found”, provide an “Add Owner” action/CTA, and retain the entered search term. |
| AC-4 | Empty input behavior: follow the agreed rule (validate “Enter a last name” OR list all owners with pagination). |
| AC-5 | Pagination (if enabled): page through results while retaining the search term and ensuring stable paging. |

## Current-state findings (main branch)
- **EXISTS**: Owner repository currently uses prefix match: `OwnerRepository.findByLastNameStartingWith(String lastName, Pageable pageable)` (`src/main/java/org/springframework/samples/petclinic/owner/OwnerRepository.java`).
- **EXISTS**: Find Owners controller path shows the form at `GET /owners/find` and processes search at `GET /owners` (`src/main/java/org/springframework/samples/petclinic/owner/OwnerController.java`).
- **EXISTS**: Empty input is treated as “broadest possible search” by setting lastName to empty string in `OwnerController.processFindForm(...)` (`OwnerController.java`).
- **EXISTS**: Pagination uses a fixed page size of 5 via `PageRequest.of(validatedPage - 1, pageSize)` (`OwnerController.java`).
- **EXISTS**: Pagination links currently do **not** include the search term: `@{'/owners?page=' + ${i}}` etc (`src/main/resources/templates/owners/ownersList.html`).
- **EXISTS**: No-results currently rejects the `lastName` field with message “not found” and returns to `owners/findOwners` (`OwnerController.java`). The template already has an “Add Owner” link (`src/main/resources/templates/owners/findOwners.html`), but the “No owners found” copy and term-retention are not explicitly implemented.

## Proposal
- **Matching rule (AC-1)**: Implement a repository query that performs *case-insensitive* and *partial* matching on last name without relying on database collation. CC-16 default assumes “partial” = contains — **needs confirmation** (contains vs starts-with).
- **Sorting (AC-2)**: Add explicit ordering by last name then first name (via query or `PageRequest` sort) to satisfy AC-2 and ensure stable paging.
- **Pagination (AC-5)**: Update pagination links to preserve the current `lastName` query param.
- **Empty input behavior (AC-4)**: Keep current behavior (list all owners) if it is the agreed rule; otherwise switch to validation error “Enter a last name”.

## Milestones
- M1: Clarify partial-match + empty-input rules.
- M2: Implement repository/controller changes for matching + sorting and fix pagination-link term retention.
- M3: Add tests for AC-1/2/4/5 behaviors.

## Task checklist

```
T-01: Confirm product rules for “partial” and empty input
- Description: Confirm whether partial match means contains vs starts-with, and whether empty input should validate or list all owners (with pagination).
- Files: (none)
- AC: AC-1, AC-4
- Depends on: none
- Definition of done: Decisions recorded in CC-9/CC-16 comments (or linked decision doc) and reflected in updated tasks below.
```

```
T-02: Update OwnerRepository to support partial + case-insensitive last-name search
- Description: Replace/extend the existing query method so searching for “Dav” matches “Davis” and “Davison” and “davis” matches “Davis”, with behavior not depending on DB collation.
- Files: EXISTS src/main/java/org/springframework/samples/petclinic/owner/OwnerRepository.java
- AC: AC-1
- Depends on: T-01
- Definition of done: Repository method returns expected Page results for both case-insensitive and partial scenarios.
```

```
T-03: Ensure deterministic sorting for stable paging
- Description: Apply explicit ordering by last name then first name to satisfy AC-2 and ensure stable paging.
- Files: EXISTS src/main/java/org/springframework/samples/petclinic/owner/OwnerController.java; EXISTS src/main/java/org/springframework/samples/petclinic/owner/OwnerRepository.java
- AC: AC-2, AC-5
- Depends on: T-02
- Definition of done: Paged query returns results in lastName/firstName order and results don’t move between pages across repeated calls.
```

```
T-04: Align empty input behavior with agreed rule
- Description: If the agreed rule is “validate”, add validation error for empty/blank last name; otherwise keep current “list all owners” behavior and ensure it stays covered by tests.
- Files: EXISTS src/main/java/org/springframework/samples/petclinic/owner/OwnerController.java; EXISTS src/main/resources/templates/owners/findOwners.html
- AC: AC-4
- Depends on: T-01
- Definition of done: Empty-input behavior matches the agreed rule and is verified via an automated test.
```

```
T-05: Retain search term across pagination links
- Description: Update owners list pagination links to preserve current lastName term in the query string.
- Files: EXISTS src/main/resources/templates/owners/ownersList.html
- AC: AC-5
- Depends on: T-04
- Definition of done: Paging includes lastName parameter and results remain filtered by the same term.
```

```
T-06: Add/adjust automated tests for Find Owners search behavior
- Description: Add tests for (1) case-insensitive matching, (2) partial matching, (3) sorting order, (4) empty-input rule, and (5) pagination retains lastName.
- Files: UNVERIFIED src/test/java/... (identify existing Owner/OwnerController tests and extend)
- AC: AC-1, AC-2, AC-4, AC-5
- Depends on: T-02, T-03, T-04, T-05
- Definition of done: Tests pass in CI; tests fail if behavior regresses.
```

## Testing and QA
- Repository-level tests: add/update JPA repository tests for matching (AC-1) and ordering (AC-2). (Test paths UNVERIFIED until `src/test/java` is inspected.)
- MVC tests: add controller tests for `GET /owners` verifying empty-input behavior (AC-4) and that pagination URLs preserve `lastName` (AC-5).

## Rollout
Not specified in story.

## Observability and operations
Not specified in story.

## Risks and mitigations
| Risk | Mitigation |
|---|---|
| “Partial” match interpretation differs (contains vs starts-with), causing wrong implementation. | T-01 clarify and document rule before implementation. |
| Case-insensitive behavior varies by DB collation. | Normalize comparisons in query (e.g., lowercasing) rather than relying on collation (per CC-16). |
| Paging becomes unstable after matching/sorting changes. | Deterministic ordering (AC-2) + stability tests (T-03/T-06). |

## Assumptions
- ASSUMPTION: “Partial match” means contains (substring), per CC-16 default, unless clarified otherwise.
- ASSUMPTION: Supported DB for automated tests includes at least H2 (per CC-16 DoD); others unspecified.

## Open questions
- Does “partial match” mean contains or starts-with?
  - Default: contains (substring) per CC-16.
- What is the agreed empty-input rule: validate “Enter a last name” OR list all owners with pagination?
  - Default: keep current behavior (list all owners).
- Is pagination required, and if yes what page size?
  - Default: keep current page size of 5 (`OwnerController.findPaginatedForOwnersLastName`).
- Which databases must behave identically for case-insensitive matching?
  - Default: ensure at least H2 is consistent; expand as clarified.

## Traceability table
| AC | Tasks | Status |
|---|---|---|
| AC-1 | T-01, T-02, T-06 | Covered |
| AC-2 | T-03, T-06 | Covered |
| AC-3 | none | UNCOVERED |
| AC-4 | T-01, T-04, T-06 | Covered |
| AC-5 | T-03, T-05, T-06 | Covered |
