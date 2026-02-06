# The Ironclad TypeScript Automation Pipeline

## Executive Summary

This standard enforces a "Zero Broken Windows" policy. We utilize a best-in-class, automated toolchain to guarantee architectural integrity, code quality, and system reliability before any code reaches production.

**The Golden Rule:**
The `npm run verify` command must pass 100% locally before you push or ask for a Code Review.

## The Toolchain

| Category | Tool | Why? |
|---|---|---|
| Lint/Format | Biome | 100x faster than ESLint/Prettier. Enforces modern TS best practices (no `any`, no dead variables). |
| Architecture | dependency-cruiser | Strict boundary enforcement. Prevents circular deps and illegal cross-layer imports (e.g., UI -> DB). |
| Dead Code | Knip | Detects unused exports, files, and dependencies. If it's not used, it doesn't belong here. |
| Duplication | jscpd | Detects copy-pasted logic across the codebase to enforce DRY. |
| Testing | Vitest | Fast, Vite-native test runner compatible with Astro. |
| Integration | Testcontainers | Spins up real Docker containers (Postgres, Redis) for tests. No mocks for infrastructure. |
| Naming | ls-lint | Enforces file naming conventions (kebab-case vs PascalCase) to match Astro routing rules. |
| Safety | ts-reset | Patches TypeScript's standard library to be safer (e.g., safer `JSON.parse`). |
| Verification | Stryker | Mutation testing. Deliberately breaks code to ensure tests actually fail when logic is broken. |

## Installation & Setup

### Dependencies

```bash
npm install -D @biomejs/biome dependency-cruiser jscpd knip typescript vitest @vitest/coverage-v8 testcontainers @ls-lint/ls-lint @total-typescript/ts-reset @stryker-mutator/core @stryker-mutator/vitest-runner npm-run-all husky lint-staged
```

### Global Type Safety (`src/reset.d.ts`)

Create this file to automatically make TypeScript smarter/safer project-wide.

```typescript
// src/reset.d.ts
import "@total-typescript/ts-reset";
```

## Configuration Specifications

### A. Code Quality (`biome.json`)

Enforces: No `any`, no unused vars, organized imports.

```json
{
  "$schema": "https://biomejs.dev/schemas/1.5.3/schema.json",
  "organizeImports": { "enabled": true },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "correctness": {
        "noUnusedVariables": "error",
        "noUnusedImports": "error"
      },
      "suspicious": {
        "noExplicitAny": "error",
        "noConsoleLog": "error"
      }
    }
  },
  "javascript": {
    "formatter": { "quoteStyle": "single", "lineWidth": 100 }
  }
}
```

### B. Architecture & Boundaries (`.dependency-cruiser.js`)

Enforces: Unidirectional flow. Features can import core, but core cannot import features.

```javascript
/** @type {import('dependency-cruiser').IConfiguration} */
module.exports = {
  forbidden: [
    {
      name: 'no-circular',
      severity: 'error',
      from: {},
      to: { circular: true }
    },
    {
      name: 'no-orphans',
      severity: 'error',
      from: { orphan: true },
      to: {}
    },
    {
      name: 'core-is-sacred',
      comment: 'Core logic should never depend on UI or Features',
      severity: 'error',
      from: { path: '^src/core' },
      to: { path: ['^src/features', '^src/ui'] }
    },
    {
      name: 'ui-cannot-touch-db',
      severity: 'error',
      from: { path: '^src/ui' },
      to: { path: '^src/db' }
    }
  ],
  options: {
    tsPreCompilationDeps: true,
    exclude: 'node_modules'
  }
};
```

### C. Dead Code Elimination (`knip.json`)

Enforces: Zero unused files, exports, or dependencies.

```json
{
  "$schema": "https://unpkg.com/knip@latest/schema.json",
  "entry": [
    "src/pages/**/*.{astro,ts,tsx,js}",
    "src/content/config.ts",
    "astro.config.mjs",
    "src/index.ts"
  ],
  "project": ["src/**/*.{ts,tsx,astro,js,mjs}"],
  "ignore": ["**/*.d.ts", "src/reset.d.ts"],
  "ignoreDependencies": ["lint-staged", "husky", "@astrojs/check", "typescript"],
  "rules": {
    "files": "error",
    "classMembers": "error",
    "dependencies": "error",
    "exports": "error",
    "types": "error",
    "unlisted": "error",
    "duplicates": "error"
  }
}
```

### D. File Naming (`.ls-lint.yml`)

Enforces: Consistent file casing.

```yaml
ls:
  src/pages:
    .astro: kebab-case
    .ts: kebab-case
  src/components:
    .astro: PascalCase
    .tsx: PascalCase
  src/core:
    .ts: kebab-case
ignore:
  - .git
  - node_modules
```

### E. Integration Testing (`vitest.config.ts`) & Testcontainers

Enforces: Real infrastructure testing.

**`test/global-setup.ts`**

```typescript
import { PostgreSqlContainer } from '@testcontainers/postgresql';

export async function setup() {
  const container = await new PostgreSqlContainer().start();
  process.env.DATABASE_URL = container.getConnectionUri();

  return async () => {
    await container.stop();
  };
}
```

**`vitest.config.ts`**

```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globalSetup: './test/global-setup.ts',
    hookTimeout: 30000,
  },
});
```

### F. Mutation Testing (`stryker.config.json`)

Enforces: High quality tests by checking if code changes cause failures.

```json
{
  "$schema": "./node_modules/@stryker-mutator/core/schema/stryker-schema.json",
  "packageManager": "npm",
  "reporters": ["html", "progress"],
  "testRunner": "vitest",
  "mutate": ["src/core/**/*.ts"],
  "thresholds": { "high": 80, "low": 60, "break": 50 }
}
```

## Workflow & Commands

### `package.json` scripts

```json
{
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "preview": "astro preview",

    "test": "vitest",
    "test:mutation": "stryker run",

    "verify": "npm-run-all --continue-on-error --parallel verify:*",
    "verify:lint": "biome check src",
    "verify:naming": "ls-lint",
    "verify:duplication": "jscpd src --threshold 0 --ignore \"**/*.test.ts\"",
    "verify:arch": "depcruise src --config .dependency-cruiser.js",
    "verify:deadcode": "knip",
    "verify:test": "vitest run",
    "verify:build": "astro build",

    "prepare": "husky install"
  }
}
```

### Pre-Commit Hook

We use Husky to ensure you don't commit code that breaks basic rules.

**`.husky/pre-commit`**

```sh
#!/bin/sh
. "$(dirname -- "$0")/_/husky.sh"

npx lint-staged
```

**`package.json` lint-staged config**

```json
{
  "lint-staged": {
    "*.{ts,tsx,astro}": [
      "biome check --write",
      "vitest related --run"
    ]
  }
}
```

## Developer Guide: How to Fix Violations

### "Knip failed: Unused exports"

**The Issue:** You exported a function/class that is never imported.
**The Fix:**
1. Delete it (preferred).
2. If it is a page or config file, add it to `entry` in `knip.json`.
3. If it is a library function meant for future use, export it from `src/index.ts` (public API).

### "Dep-cruiser failed: core-is-sacred"

**The Issue:** Your Core logic file imported something from `src/features` or `src/ui`.
**The Fix:** Core logic must be pure. Move the dependency down into Core, or inject the dependency using an interface (Dependency Inversion).

### "JSCPD failed: Duplication found"

**The Issue:** You copy-pasted more than 5 lines of code.
**The Fix:** Extract the common logic into a utility function in `src/shared` or `src/utils`.

### "Biome failed: noExplicitAny"

**The Issue:** You used `: any`.
**The Fix:** Define the type. If you truly don't know the shape, use `unknown` and validate it with Zod before using it.

## Extensibility

### Adding New Architectural Rules

Modify `.dependency-cruiser.js`. Example — prevent Utils from importing React:

```javascript
{
  name: 'utils-are-pure',
  from: { path: '^src/utils' },
  to: { path: 'react' }
}
```

### Ignoring Files

- **Linting:** Add to `biome.json` > `files.ignore`.
- **Dead Code:** Add to `knip.json` > `ignore`.
- **Duplication:** Add `--ignore` flag in `package.json` script.
