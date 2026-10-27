# Testing Configuration

This project's test facts: framework, commands, locations, coverage and naming. TDD methodology lives in Farseer's `tdd-worker` agent, so it is not repeated here.

When exploring the codebase, read `.farseer/glossary.md` (if it exists) so test names and interface vocabulary match the project's domain language, and respect ADRs in the area you're touching. If a change contradicts an ADR, say so rather than silently overriding it.

> **Provisional commands.** The stack is settled (`.farseer/adr/` ADR-0002): a pnpm monorepo with `apps/web`, `apps/server` and `packages/shared`, each with its own tests. Script names and paths are final once the first plan scaffolds the app.

## Test Framework

- **Vitest** (jsdom environment) for domain logic, view models and infrastructure.
- **React Testing Library** with `@testing-library/user-event` and `@testing-library/jest-dom` for components.
- **Playwright** (headless Chromium) for critical user workflows and layouts.

## Commands

```bash
# Run all unit and component tests (Vitest runs files in parallel by default)
pnpm test

# Run all tests sequentially, for debugging a flaky failure
pnpm test -- --no-file-parallelism

# Run a specific test file or filter (scoped run, used by TDD workers)
pnpm test -- src/path/to/module.test.ts

# Run with coverage
pnpm run test:coverage

# Playwright end-to-end tests
pnpm run test:e2e
```

## Parallel Execution

- **Default**: always run tests in parallel unless debugging a specific failure.
- Parallel command: `pnpm test` (Vitest worker pool) and `pnpm run test:e2e` (Playwright `fullyParallel`).
- Sequential fallback: `pnpm test -- --no-file-parallelism`, `pnpm run test:e2e -- --workers=1`.

## Test File Locations

- Unit and component tests: next to the source, `src/**/<name>.test.ts(x)`.
- Shared Vitest setup and helpers: `src/test/`.
- End-to-end tests: `e2e/*.spec.ts`, with fixtures alongside. Screenshots go to `e2e/screenshots/` (gitignored).

## Coverage Requirements

- Minimum: 80% (lines, functions, branches, statements) on logic code, enforced in the Vitest config once it exists.
- New code must have tests.

## Exhaustive rule tests

Rule-driven content is tested over every combination of its inputs, not only examples (RED-03, ADR-0031). That covers the backdrop rules (weather code × half hour × month × holiday), the weather advice line (TODAY-03), and the kiosk idle and night timers. Each run asserts that every combination gets a sensible result, such as a backdrop cell that exists in the set.

## Safety: this is a real house

- Tests must never change real devices. Anything that taps a control runs against a fake Home Assistant, never the live instance.
- Tests that read the live instance are tagged `@live` and only read.
- Tokens never go into source, snapshots, screenshots or logs.

## Test Naming Convention

- Test files: `<module>.test.ts` / `<Component>.test.tsx`; Playwright `<workflow>.spec.ts`.
- Structure: `describe('<unit or behavior>')` with `it('<does something observable>')`, written as a sentence in domain language.
- Tag Playwright tests that hit the real instance with `{ tag: '@live' }`.
