---
name: qa-verification
description: >
  Execute manual verification and quality assurance for a feature increment.
  Validates functionality via Browser MCP, checks database state via Postgres
  MCP, and fixes all issues through TDD. Use when the user says "verify",
  "QA", "manual test", "check the feature", or after completing a set of
  implementation tasks. Do NOT use for writing new features or planning.
---

# Manual Verification & Quality Assurance Protocol

**Role:** Lead QA & Implementation Engineer.
**Objective:** Verify all features, fix detected issues via TDD, and produce a final summary. Do not stop until a full verification pass completes with zero issues.

## Phase 1: Environment & Tool Validation

1. **Validate Browser MCP** — navigate to the project URL and confirm the MCP reads content correctly.
2. **Launch Backend** — start the local dev server in the background. Monitor terminal output for errors throughout testing.
3. **Launch Application** — open the local server URL (e.g., `localhost:3000`) via Browser MCP and confirm the app is running.

## Phase 2: Verification Loop (Recursive)

If *any* issue is found and fixed, restart testing of **all** use cases to catch regressions.

### A. Manual Testing

For each feature/use case in the increment:

1. **Visual & Functional Check** via Browser MCP.
   * *Visual:* Does it match the design intent?
   * *Functional:* Does it work end-to-end?
2. **Snapshot** source code and browser state during execution.
3. **Database Validation** (recommended when state changes occur):
   * Query via Postgres MCP. Verify expected state.
   * You may update user state via DB (using local `.env` credentials) to test specific scenarios.
4. **Backend Monitoring** — check server logs for silent errors.

### B. Issue Management

If anything fails:

1. Create or update `issues.md` in the project root.
2. Log: Use Case, Expected vs. Actual, Error Log snippet.
3. **Do not fix immediately.** Finish the full manual pass to collect all issues first.

## Phase 3: TDD Fix Protocol

For each issue in `issues.md`:

1. **Write a failing test** at the Application Service or API Service level. Confirm it **fails** before proceeding.
2. **Implement the fix** following project conventions.
3. **Run the test again.** Confirm it is **green** before proceeding.
4. Mark the issue as resolved in `issues.md`.

> **CRITICAL:** After all fixes, return to Phase 2 and re-run all use cases from the beginning.

## Phase 4: Final Deliverable

Done only when all use cases pass in a single run with zero issues.

**Output a Final Summary Report:**
* **Verified Use Cases** — checklist of what was tested.
* **Issue Log** — what broke, the regression test created, and how it was fixed.
* **Final Status** — confirmation that every use case is functional and visually verified.
