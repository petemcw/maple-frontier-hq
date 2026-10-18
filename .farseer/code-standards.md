# Code Standards

> **Provisional commands.** The stack is settled (`.farseer/adr/` ADR-0002): a pnpm monorepo, so commands run from the repo root with pnpm. Script names and paths are final once the first plan scaffolds the app.

## Style Guide

TypeScript (strict) and React function components. Formatting is Prettier: 2-space indent, single quotes, no semicolons, trailing commas, 100-column lines. Markdown is not auto-formatted.

## Linting

```bash
# Check for issues
pnpm run lint

# Auto-fix issues
pnpm exec oxlint --fix
```

oxlint with the `react`, `typescript` and `oxc` plugins.

## Formatting

```bash
pnpm run format        # write
pnpm run format:check  # verify
```

## Pre-commit Checks

Run before every commit:

```bash
pnpm run format:check && pnpm run lint && pnpm test && pnpm run build
```

- All tests must pass.
- Run `pnpm run test:e2e` when a change touches a user workflow, layout, connection or auth.
- Never commit secrets: tokens, `.env.local`, private URLs. Never put a long-lived Home Assistant token in source, the bundle, or a file served publicly (anything under HA's `/config/www` is served without sign-in).

## Naming Conventions

- Components: PascalCase, one per file, file named after it (`LightTile.tsx`).
- Hooks: `useX`, camelCase file (`useEntity.ts`).
- Functions and variables: camelCase.
- Types: PascalCase. Prefer `type` aliases; no `enum`, use string-literal unions.
- Constants: SCREAMING_SNAKE_CASE for module-level fixed values.
- Non-component modules: camelCase files.
- Home Assistant identifiers keep HA's spelling (`entity_id`, `media_player`, attribute keys). Map them in view models; don't camelCase data that comes from HA.
- Use the terms in `.farseer/glossary.md` in names, tests and issue titles.

## UI

- Touch first: no hover-only affordances, touch targets at least 44×44 CSS px.
- Accessibility is required: every control has an accessible name, real `<button>`s for actions, visible focus.
- Every family member uses Home HQ, on different devices, so layouts must work from phone to wall display.

## Comments

Explain why, not what: Home Assistant quirks, protocol behavior, and constraints a reader couldn't infer from the code.
