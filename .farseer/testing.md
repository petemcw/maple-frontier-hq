# Testing Configuration

This project's test facts: framework, commands, locations, coverage and naming. TDD methodology lives in Farseer's `tdd-worker` agent, so it is not repeated here.

When exploring the codebase, read `.farseer/glossary.md` (if it exists) so test names and interface vocabulary match the project's domain language, and respect ADRs in the area you're touching. If a change contradicts an ADR, say so rather than silently overriding it.

> **Provisional.** There is no code yet. These settings copy the sibling `ha-dashboard` project (`../ha-dashboard/.farseer/testing.md`) and stand until planning settles the stack and hosting (brainstorm questions 2, 3 and 18 in `docs/brainstorming.md`). Update this file when the first plan scaffolds the app.

## Test Framework

- **Vitest** (jsdom environment) for domain logic, view models and infrastructure.
- **React Testing Library** with `@testing-library/user-event` and `@testing-library/jest-dom` for components.
- **Playwright** (headless Chromium) for critical user workflows and layouts.

## Commands

```bash
# Run all unit and component tests (Vitest runs files in parallel by default)
npm test

# Run all tests sequentially, for debugging a flaky failure
npm test -- --no-file-parallelism

# Run a specific test file or filter (scoped run, used by TDD workers)
npm test -- src/path/to/module.test.ts

# Run with coverage
npm run test:coverage

# Playwright end-to-end tests
npm run test:e2e
```

## Parallel Execution

- **Default**: always run tests in parallel unless debugging a specific failure.
- Parallel command: `npm test` (Vitest worker pool) and `npm run test:e2e` (Playwright `fullyParallel`).
- Sequential fallback: `npm test -- --no-file-parallelism`, `npm run test:e2e -- --workers=1`.

## Test File Locations

- Unit and component tests: next to the source, `src/**/<name>.test.ts(x)`.
- Shared Vitest setup and helpers: `src/test/`.
- End-to-end tests: `e2e/*.spec.ts`, with fixtures alongside. Screenshots go to `e2e/screenshots/` (gitignored).

## Coverage Requirements

- Minimum: 80% (lines, functions, branches, statements) on logic code, enforced in the Vitest config once it exists.
- New code must have tests.

## Safety: this is a real house

- Tests must never change real devices. Anything that taps a control runs against a fake Home Assistant, never the live instance.
- Tests that read the live instance are tagged `@live` and only read.
- Tokens never go into source, snapshots, screenshots or logs.

## Test Naming Convention

- Test files: `<module>.test.ts` / `<Component>.test.tsx`; Playwright `<workflow>.spec.ts`.
- Structure: `describe('<unit or behavior>')` with `it('<does something observable>')`, written as a sentence in domain language.
- Tag Playwright tests that hit the real instance with `{ tag: '@live' }`.
