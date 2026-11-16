# Overview

**Story:** CC-7 — As a clinic staff member, I want to manage pet owners, their pets, and visit records so that I can keep each patient’s information and care history in one place.

**Goal:** Verify and, if needed, close gaps in existing Spring Petclinic owner/pet/visit workflows so clinic staff can manage owners, pets, and visits with clear validation feedback. Primary deliverable is explicit, AC-mapped automated test coverage and any minimal MVC/template fixes required to satisfy CC-7.

**Jira:** https://ervanshsingh.atlassian.net/browse/CC-7

**Repo/branch inspected:** https://github.com/ervansh/spring-petclinic (branch `main`)

**Commit SHA inspected:** `500158f732419217507c7656904b8e6aa1bcc0d6c`

---

# Scope

## In scope

- (AC-1) Staff can find an owner and view the pets associated with that owner.
- (AC-2) Staff can add or update an owner’s contact details and pet information.
- (AC-3) Staff can record a visit for a pet, including the visit date and description.
- (AC-4) A pet’s visit history is available when viewing that pet’s information.
- (AC-5) Required or invalid form entries are identified with clear validation feedback.

## Out of scope

- Authentication/authorization and role management (not mentioned in story).
hune ="
- New UI flows beyond existing owner/pet/visit pages (not mentioned in story).
hune ="
- Database vendor/schema redesign or migrations (not requested).

# Requirements

| AC | Requirement (from story) |
|---|---|
| AC-1 | Staff can find an owner and view the pets associated with that owner. |
| AC-2 | Staff can add or update an owner’s contact details and pet information. |
| AC-3 | Staff can record a visit for a pet, including the visit date and description. |
| AC-4 | A pet’s visit history is available when viewing that pet's information. |
| AC-5 | Required or invalid form entries are identified with clear validation feedback. |

# Current-state findings

- **EXISTS**: Owner MVC flow: `src/main/java/org/springframework/samples/petclinic/owner/OwnerController.java` and templates `src/main/resources/templates/owners/findOwners.html`, `ownersList.html`, `ownerDetails.html` .
  - (paths: `src/main/java/org/springframework/samples/petclinic/owner/OwnerController.java`, `src/main/resources/templates/owners/findOwners.html`, `ownersList.html , `src/main/resources/templates/owners/ownerDetails.html)
- **EXISTS**: Pet MVC flow: `src/main/java/org/springframework/samples/petclinic/owner/PetController.java` and template `src/main/resources/templates/pets/createOrUpdatePetForm.html.
- **EXISTS**: Visit MVC flow: `src/main/java/org/springframework/samples/petclinic/owner/VisitController.java` and template `src/main/resources/templates/pets/createOrUpdateVisitForm.html.
- **EXISTS**: Domain models: `src/main/java/org/springframework/samples/petclinic/owner/Owner.java`, `Pet.java`, `Visit.java`.
 
- **EXISTS**: Validation helper: `src/main/java/org/springframework/samples/petclinic/owner/PetValidator.java`.
 
- **EXISTS**: Thymeleaf fragments for consistent field + validation display: `src/main/resources/templates/fragments/inputField.html`, `selectField.html`.
- **EXISTS**: MVC tests exist for controllers: `src/test/java/org/springframework/samples/petclinic/owner/OwnerControllerTests.java`, `PetControllerTests.java`, `VisitControllerTests.java` .
 
- **EXISTS**: Build files present: `pom.xml`, `build.gradle`.

# Proposed approach (Proposal)

- **Proposal:** Treat CC-7 as “¨oуu...