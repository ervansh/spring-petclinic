# implementation-plan (CC-7 / CC-15)

## Overview

- **Goal:** Implement a dedicated *Pet Details* page so clinic staff can view a pet's core information and its **visit history** when “viewing that pet’s information” (CC-7 AC #4 / CC-15).
- **Context:** Current UI is owner-centric; visits are primarily shown under Owner Details. This sub-task introduces an explicit pet-centric route/view.


## Scope

### In scope

- A **GET Pet Details route** (preferred/nested): `GET /owners/{ownerId}/pets/{petId}`.
- A new **Thymeleaf template**: `src/main/resources/templates/pets/petDetails.html`.
- Navigation from **Owner Details – Pet Details**.
- Pet details content includes:
  - Pet core info: **name, type, birth date**.
  - Owner reference: **owner name + link** back to Owner Details.
  - Visit history: **date + description*, **newest-first**.
  - Empty state when there are **no visits**, with a clear path to **add a visit**.

#### Out of scope

- Any new persistence model changes (assume existing Pet/Visit entities already support this).