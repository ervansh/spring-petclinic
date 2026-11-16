# Implementation plan

## Overview
- **Story**: CC-7 — As a clinic staff member, I want to manage pet owners, their pets, and visit records so that I can keep each patient’s information and care history in one place.
- **Goal**: Confirm (and, if needed, close gaps in) the existing Owner/Pet/Visit management flows so clinic staff can find owners, add/update owner and pet details, record visits, view visit history, and receive clear validation feedback. This plan is based on the current `main` branch state and focuses only on what CC-7 acceptance criteria explicitly require.
- **Repo baseline (read)**: `main` @ `500158f732419217507c7656904b8e6aa1bcc0d6`
- **Jira**: https://ervanshsingh.atlassian.net/browse/CC-7

## Scope
### In scope (traces to ACs)
- Owner search and owner detail view (pets listed for an owner). (AC-1)
- Create/update owner contact details. (AC-2)
- Create/update pet information for an owner. (AC-2)
- Record a visit for a pet with visit date and description. (AC-3)
- Show a pet’s visit history when viewing pet information (via owner detail page). (AC-4)
- Ensure required/invalid form entries show clear validation feedback (server-side validation behavior + view rendering). (AC-5)

### Out of scope
- Role-based access control / authentication & authorization (not mentioned in story).
- API-only (JSON) endpoints for owner/pet/visit management (not mentioned in story).
- Database migrations / schema redesign (not mentioned in story; repo already provides schema and seed data).
- UI redesign / new wireframes (not mentioned in story).

## Requirements
| AC | Requirement (from story) |
|---|---|
| AC-1 | Staff can find an owner and view the pets associated with that owner. |
| AC-2 | Staff can add or update an owner’s contact details and pet information. |
| AC-3 | Staff can record a visit for a pet, including the visit date and description. |
| AC-4 | A pet’s visit history is available when viewing that pet’s information. |
| AC-5 | Required or invalid form entries are identified with clear validation feedback. |

## Current-state findings (main @ 500158f732419217507c7656904b8e6aa1bcc0d6)
- **EXISTS**: Owner search and pagination via `GET /owners/find` and `GET /owners` in `src/main/java/org/springframework/samples/petclinic/owner/OwnerController.java`.
- **EXISTS**: Owner create/update via `GET/POST /owners/new` and `GET/POST /owners/{ownerId}/edit` in `src/main/java/org/springframework/samples/petclinic/owner/OwnerController.java`, using `@Valid` and `BindingResult`.
- **EXISTS**: Owner detail page renders pets and each pet’s visits in `src/main/resources/templates/owners/ownerDetails.html` (iterates `${owner.pets}` and `${pet.visits}`).
- **EXISTS**: Pet create/update under an owner via `src/main/java/org/springframework/samples/petclinic/owner/PetController.java` (`/owners/{ownerId}/pets/new` and `/owners/{ownerId}/pets/{petId}/edit`) with validation (custom `PetValidator` + additional checks).
- **EXISTS**: Visit create via `src/main/java/org/springframework/samples/petclinic/owner/VisitController.java` (`/owners/{ownerId}/pets/{petId}/visits/new`) with validation for date and `@Valid`.
- **EXISTS**: Controller-level web MVC tests exist for owner/pet/visit flows: `src/test/java/org/springframework/samples/petclinic/owner/OwnerControllerTests.java`, `.../PetControllerTests.java`, `.../VisitControllerTests.java`.
- **EXISTS**: Project uses Spring Boot WebMVC + Thymeleaf + Validation via `pom.xml` dependencies (`spring-boot-starter-webmvc`, `spring-boot-starter-thymeleaf`, `spring-boot-starter-validation`).

## Proposed approach (Proposal)
- **Proposal**: Treat CC-7 as a “verification + gap-closure” story: the repo already has end-to-end MVC flows for owners/pets/visits; implement only what is missing to meet the ACs and add/adjust tests to prove coverage.
  - **Reason**: The controllers, templates, and tests already exist for all described behaviors; the lowest-risk implementation is to confirm existing behavior meets ACs and tighten validation feedback where unclear.
- **Proposal**: Use controller tests (MockMvc) to encode AC expectations and adjust templates only if tests show validation feedback is not visible/clear.
  - **Reason**: The codebase already uses `@WebMvcTest`-based tests for these controllers (e.g., `PetControllerTests`, `VisitControllerTests`), making it consistent to validate ACs at the MVC boundary.

**Alternatives considered**
- End-to-end UI tests (e.g., Selenium/Playwright): not proposed because no tooling is referenced in CC-7 or verified in the repo.
- Repository/service layer refactor: not proposed because CC-7 does not request architectural changes and existing controllers already encapsulate the flows.

## Milestones
- **M1**: Confirm current behavior against ACs and identify gaps (tests-first).
- **M2**: Implement any required adjustments in controllers/templates to satisfy ACs.
- **M3**: Stabilize and ensure coverage via tests.

## Task checklist

T-01: Extract AC-focused test cases for owner find and owner details (pets + visits)
- Description: Add/adjust MockMvc tests to explicitly cover owner search and owner details rendering of pets and their visit history.
- Files: EXISTS: `src/test/java/org/springframework/samples/petclinic/owner/OwnerControllerTests.java`
- AC: AC-1, AC-4
- Depends on: none
- Definition of done: Tests pass proving that an owner can be found and the owner details page includes pets and their visits.

T-02: Verify owner create/update validation feedback is visible and clear
- Description: Add/adjust MockMvc tests for invalid owner create/update submissions to ensure field errors are produced and the correct form view is rendered.
- Files:
  - EXISTS: `src/test/java/org/springframework/samples/petclinic/owner/OwnerControllerTests.java`
  - EXISTS: `src/main/java/org/springframework/samples/petclinic/owner/OwnerController.java`
  - UNVERIFIED: `src/main/resources/templates/owners/createOrUpdateOwnerForm.html` (template existence not confirmed in this plan; verify on branch)
- AC: AC-2, AC-5
- Depends on: T-01
- Definition of done: Tests demonstrate invalid owner submissions return the owner form with binding errors.

T-03: Verify pet create/update and validation feedback
- Description: Ensure pet create/update flows (including invalid inputs such as blank name, missing type, invalid birthDate) produce clear validation errors; extend tests only if gaps exist.
- Files:
  - EXISTS: `src/main/java/org/springframework/samples/petclinic/owner/PetController.java`
  - EXISTS: `src/test/java/org/springframework/samples/petclinic/owner/PetControllerTests.java`
- AC: AC-2, AC-5
- Depends on: T-01
- Definition of done: Existing + any new tests pass for valid and invalid pet create/update scenarios.

T-04: Verify visit recording and validation feedback
- Description: Ensure visit creation requires valid visit date and description per model constraints; add any missing tests to assert validation feedback for invalid input.
- Files:
  - EXISTS: `src/main/java/org/springframework/samples/petclinic/owner/VisitController.java`
  - EXISTS: `src/test/java/org/springframework/samples/petclinic/owner/VisitControllerTests.java`
- AC: AC-3, AC-5
- Depends on: T-01
- Definition of done: Tests pass for successful visit creation and for invalid inputs returning the visit form with field errors.

T-05: Close any UI validation feedback gaps found by tests
- Description: If tests reveal errors are not displayed (e.g., missing error blocks in Thymeleaf templates), update the relevant templates to render field/global errors clearly.
- Files:
  - EXISTS: `src/main/resources/templates/owners/ownerDetails.html`
  - UNVERIFIED: `src/main/resources/templates/owners/createOrUpdateOwnerForm.html`
  - UNVERIFIED: `src/main/resources/templates/pets/createOrUpdatePetForm.html`
  - UNVERIFIED: `src/main/resources/templates/pets/createOrUpdateVisitForm.html`
- AC: AC-5
- Depends on: T-02, T-03, T-04
- Definition of done: Manual spot-check + tests confirm validation errors are visible on each form view.

## Testing and QA
- **Controller (MockMvc) tests**:
  - Owner find + details: `src/test/java/org/springframework/samples/petclinic/owner/OwnerControllerTests.java` (AC-1, AC-4)
  - Pet create/update + validation: `src/test/java/org/springframework/samples/petclinic/owner/PetControllerTests.java` (AC-2, AC-5)
  - Visit create + validation: `src/test/java/org/springframework/samples/petclinic/owner/VisitControllerTests.java` (AC-3, AC-5)
- **Manual smoke (dev run)**:
  - Navigate to `/owners/find`, search for an owner, open owner details, confirm pets + visits visible. (AC-1, AC-4)
  - Create/update owner and pet; submit invalid values to see validation messages. (AC-2, AC-5)
  - Add visit with valid/invalid date and description. (AC-3, AC-5)

## Rollout
Not specified in story.

## Observability and operations
Not specified in story.

## Risks and mitigations
| Risk | Mitigation |
|---|---|
| CC-7 overlaps existing Petclinic baseline functionality; risk of adding redundant tests or unnecessary changes. | Focus on AC-driven tests first; only change production code/templates when a concrete gap is proven. |
| Validation feedback “clear” is subjective. | Default to: field-level errors visible near inputs + global error summary if present; capture via tests that errors exist and the form re-renders. |
| Some templates referenced by controllers may differ from assumed names/paths. | Verify template paths on branch before implementing changes; mark unverified paths as such until confirmed. |

## Assumptions
- **ASSUMPTION**: CC-7 is intended to confirm baseline Petclinic owner/pet/visit management flows rather than introduce new features beyond what exists.
- **ASSUMPTION**: “Clear validation feedback” can be satisfied by existing Spring/Thymeleaf error rendering patterns (field errors on form views).

## Open questions
- Does “viewing that pet’s information” (AC-4) mean a dedicated pet details page, or is the owner details page sufficient since it displays each pet and its visits?
  - Default: Treat the existing owner details page (`owners/ownerDetails`) as the “pet’s information” view because it shows per-pet info + visit history in `src/main/resources/templates/owners/ownerDetails.html`.
- What constitutes “clear” validation feedback for CC-7: field-level messages only, or also top-level alerts?
  - Default: Field-level errors for invalid/required fields must be present; optional global message may remain as-is (e.g., flash attributes in `OwnerController`).

## Traceability table
| AC | Tasks | Status |
|---|---|---|
| AC-1 | T-01 | Covered |
| AC-2 | T-02, T-03 | Covered |
| AC-3 | T-04 | Covered |
| AC-4 | T-01 | Covered |
| AC-5 | T-02, T-03, T-04, T-05 | Covered |
