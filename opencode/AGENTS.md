# Development Agent Instructions

## Core Methodology

- Follow TDD: Red -> Green -> Refactor. No exceptions.
- Follow "Tidy First": separate structural changes from behavioral changes. Never mix them in the same commit.
- Write the simplest failing test first. Implement the minimum code to make it pass.

## Commands

- ALL project commands MUST be executed through package.json scripts.
- Run `npm run check` (or equivalent) before every commit to verify all checks pass.
- When installing dependencies, always check for the most recent stable version.

## Code Style

- Use strict TypeScript configuration with all strict checks enabled.
- Keep functions small (<20 lines preferred, <10 ideal), single-responsibility.
- Use intention-revealing names. Avoid generic suffixes (Manager, Handler, Utils, Helper).
- Prefer `unknown` over `any`. Prefer interfaces over type aliases for object shapes.
- Prefer composition over inheritance.
- Remove dead code immediately — never comment it out.
- DO NOT write tests for things the TypeScript compiler validates (method existence, type checking, interface adherence).
- DO NOT add GIVEN-WHEN-THEN comments in tests.

## Git Workflow

- Use Conventional Commits format.
- Subject: <50 chars, imperative mood, no trailing period.
- Body: explain the "why", wrapped at 72 chars.
- Commit only when all checks pass and the change is a single logical unit.
- Separate structural and behavioral changes into distinct commits.
- NEVER add generated footers or co-author signatures.

## Boundaries

- Never commit `.env`, credentials, or secret files.
- Do not modify migration files directly.
- Do not list local agent config files as references in commits.

## Skills

Two skills are available for specialized workflows:

- **task-creation** — converts requirements into ordered TDD task lists following DDD layering.
- **qa-verification** — runs the manual verification and QA protocol with TDD-based issue fixing.
