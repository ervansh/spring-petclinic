# Implementation plan

## Overview
***Story:** CC-7 — As a clinic staff member, I want to manage pet owners, their pets, and visit records so that I can keep each patient’s information and care history in one place.  
***ira:** https://ervanshsingh.atlassian.net/browse/CC-7  
***Repo / base:** https://github.com/ervansh/spring-petclinic (branch `main` @ commit `500158f732419217507c7656904b8e6aa1bcc0d6`)

**Goal:** Implement GAP-01 (proposal): add a dedicated “Pet details” view that shows a single pet’s information along with its visit history, so staff can review patient history without going through the owner aggregate view.

**Note on approval:** Latest user approval selected **GAP-01** (after an initial “no gapr” approval). This plan proceeds with GAP-01 as the approved gap.

---

## Scope

### In scope
- (AC-4) Provide a dedicated pet information view where a pet’s visit history is visible.
- (AC-1) Provide a navigation path for staff to reach the pet details view from an owner’s page.

### Out of scope
- Any new fields for pets/visits/owners not described in CC-7.
- New authentication/authorization behavior (not described in CC-7).
- API/JSON endpoints for pet details (not described in CC-7).

---

## Requirements

  | AC | Requirement (from story) |
  |---|---|
  | AC-1 | Staff can find an owner and view the pets associated with that owner. |
  | AC-2 | Staff can add or update an owner’s contact details and pet information. |
  | AC-3 | Staff can record a visit for a pet, including the visit date and description. |
  | AC-4 | A pet’s visit history is available when viewing that pet’s information. |
  | AC-5 | Required or invalid form entries are identified with clear validation feedback. |

---

## Current-state findings (main @ 500158f732419217507c7656904b8e6aa1bcc0d6)

- **EXISTS** Owner details page exists and is the current hub for pet and visit info: `src/main/resources/templates/owners/ownerDetails.html`.
- **EXISTS** Pet create/edit endpoints exist under owner context: `src/main/java/org/springframework/samples/petclinic/owner/PetController.java`.
- **EXISTS** Visit create endpoint exists under owner+pet context: `src/main/java/org/springframework/samples/petclinic/owner/VisitController.java`.
 - **EXISTS** Domain models for Pet and Visit exist: `src/main/java/org/springframework/samples/petclinic/owner/Pet.java`, `src/main/java/org/springframework/samples/petclinic/owner/Visit.java`.
 - **UNVERIFIED** A dedicated pet details template or controller method for `GET /pets/{petId}` does not appear in the inspected directory listing; will confirm during implementation.

---

## Proposed approach (Proposal)

- **Proposal:** Add a new GET endpoint for a dedicated pet details page (e.g. `GET /pets/{petId}`) in `PetController`.  
  - **Reason:** Keeps pet-related UI in one controller; existing edit route already uses `/pets/{petId}/edit` (in `src/main/java/org/springframework/samples/petclinic/owner/PetController.java`).

- **Proposal:** Create a new Thymeleaf template (e.g. `src/main/resources/templates/pets/petDetails.html`) to render pet info + visits.  
  - **Reason:** Existing templates are organized by feature area (e.g. `templates/owners/*`).

- **Proposal:** Add a link from owner details to the pet details page.  
  - **Reason:** Preserves the existing “find owner → owner details” flow (AC-1) while enabling direct pet view.

- **Alternatives considered:**
  - Keep pet+visit display only on owner details page (`templates/owners/ownerDetails.html`). Rejected because GAP-01 explicitly requests a dedicated pet details view.
  - Add endpoint under owner context (e.g. `/owners/{ownerId}/pets/{petId}`). Viable, but would require ensuring owner/pet relationship validation; `/pets/{petId}` aligns with existing edit route.

## Milestones

- M1: UI + routing for pet details page
- M2: Tests updated/added

---

## Task checklist

```
M-01: Confirm existing routes and template structure for pets/visits
- Description: Verify whether any pet details view already exists and confirm how ownerDetails renders pet visits.
- Files: src/main/java/org/springframework/samples/petclinic/owner/PetController.java (EXISTS), src/main/resources/templates/owners/ownerDetails.html (EXISTS)
- AC: AC-4
- Depends on: none
- Definition of done: Notes added to PR description or plan comments; implementation uses confirmed findings (no UNVERIFIED assumptions left impacting tasks)

T-02: Add pet details endpoint
- Description: Add a GET handler to show a single pet’s information, including its visit history, and return a dedicated view.
- Files: src/main/java/org/springframework/samples/petclinic/owner/PetController.java (EXISTS)
- AC: AC-4
- Depends on: T-01
- Definition of done: GET /pets/{petId} renders successfully in a controller test

T-03: Add pet details Thymeleaf template
- Description: Create a new template to display pet attributes and a list/table of visits (date + description).
- Files: src/main/resources/templates/pets/petDetails.html (NEW)
- AC: AC-4
- Depends on: T-02
- Definition of done: Template renders in controller test and includes visit history content

T-04: Add navigation link from owner details to pet details
- Description: Update ownerDetails to include a per-pet link to the dedicated pet details page.
- Files: src/main/resources/templates/owners/ownerDetails.html (EXISTS)
- AC: AC-1, AC-4
- Depends on: T-03
- Definition of done: Link is visible on owner details page and targets the pet details route

T-05: Add/extend MVC tests for pet details view
- Description: Add tests for the new endpoint and verify model attributes and view name; verify out-of-scope behavior is unchanged.
- Files: src/test/java/org/springframework/samples/petclinic/owner (EXISTS, exact test class TBD)
- AC: AC-4
- Depends on: T-02, T-03
- Definition of done: Tests pass and fail if visits are not included in pet details
```

---

## Testing and QA

- **MVC/controller tests:** Add a new test or extend existing tests under `src/test/java/org/springframework/samples/petclinic/owner` (EXISTS) to cover:
  - AC-4: pet details view shows visit history
  - AC-1: owner details includes link to pet details

- **Manual smoke:** Find owner → open owner details → click a pet → confirm pet details shows visits.

---

## Rollout
*Not specified in story.*

## Observability and operations
*Not specified in story.*

---

## Risks and mitigations

| Risk | Impact | Mitigation |
|---|---|---|
"virkd"}