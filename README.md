# dotfiles

Reusable engineering standards, coding guidelines, and AI agent configurations. This repository provides a single source of truth for development practices that can be symlinked or copied into any project.

## Directory Structure

### `automation-excellence/`

Language-specific automation pipeline standards for achieving full CI/CD quality gates. Each file defines the complete toolchain, configuration, verification commands, and developer fix guides for a language ecosystem.

- **`typescript.md`** — The Ironclad TypeScript Automation Pipeline: Biome, dependency-cruiser, Knip, jscpd, Vitest, Testcontainers, ls-lint, ts-reset, Stryker. Includes `npm run verify` setup, pre-commit hooks, and violation fix guides.

### `opencode/`

Agent configuration and skill definitions for OpenCode projects.

- **`AGENTS.md`** — Repository-context template for project-specific structure, build commands, and key files.
- **`skills/task-creation/`** — Converts requirements into TDD task lists following Domain-Driven Design layering.
- **`skills/qa-verification/`** — Manual verification and QA protocol: browser testing, database validation, TDD-based issue fixing.
