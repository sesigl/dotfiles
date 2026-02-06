---
name: task-creation
description: >
  Convert requirements, specifications, or feature ideas into an actionable
  TDD task list following Domain-Driven Design layering. Use when the user
  says "create tasks", "plan feature", "break down", "task list", or asks
  to turn a requirement into implementation steps. Do NOT use for executing
  tasks or writing code — only for planning.
---

# Task Creation Excellence

Convert requirements into granular, ordered tasks split into **Test (Red)** and **Implementation (Green)** pairs. Never create generic tasks like "Implement Feature X."

## Execution Order

### 1. Architecture Check (Pre-Code)

* **Trigger:** New Bounded Context or Domain Concept.
* **Task:** Update `architecture/contexts.mermaid` and context overviews.

### 2. Domain Layer — Start Here

* **Target:** Aggregate Roots (AR). If no AR exists, use a Domain Service.
* **Value Objects:** Include in the AR task unless they require complex validation.
* **Format:**
    1. `[TEST]` Write unit test for AR logic.
    2. `[IMPL]` Implement AR to make the test pass.

### 3. Application Layer — After Domain Is Stable

* **Target:** Use Cases / Application Services.
* **Constraint:** If you discover missing Domain logic, **PAUSE** and insert a Domain task pair first.
* **Format:**
    1. `[TEST]` Write test for Use Case (mock infrastructure interfaces).
    2. `[IMPL]` Implement Use Case orchestration.

### 4. Infrastructure Layer — The Wiring

* **Base** (transaction managers, referenced by App layer) go before Application tasks.
* **Implementations** (repositories, adapters) go after Application tasks.
* **Format:** `[IMPL]` for concrete classes + `[IMPL]` for registration in `di.ts`.

### 5. Interface Layer — Last

* **Target:** Astro Actions / API endpoints.
* Keep logic minimal. Manual verification is often sufficient if the Application layer is tested.

## Required Output Format

```markdown
- [ ] **Architecture**: Update `contexts.mermaid` with new context.
- [ ] **Domain (Order AR)**:
    - [ ] `[TEST]` Spec: Order total calculation logic.
    - [ ] `[IMPL]` Implement `Order` aggregate.
- [ ] **Application (PlaceOrder)**:
    - [ ] `[TEST]` Spec: Service orchestrates domain and repo.
    - [ ] `[IMPL]` Implement `PlaceOrder` use case.
- [ ] **Infrastructure**:
    - [ ] `[IMPL]` Implement `SqlOrderRepository`.
    - [ ] `[IMPL]` Update `di.ts`.
- [ ] **Interface**: Connect Astro Action.
- [ ] **QA**: Run QA verification skill.
```
