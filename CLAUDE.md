# Home HQ

The family and home hub for Maple Frontier, our home. All six family members will use it from their own devices, on top of Home Assistant. It is in the research stage: no code yet, and the details are being decided in planning.

## Feature Development

For any feature or change beyond a simple fix, use the `/farseer:planner` skill to trigger the autonomous development workflow. Never use Claude Code's built-in plan mode. After writing a plan, ask the user if they want to execute it, and provide the command to run it later with the `/farseer:orchestrate` skill.

Use this workflow for: new features, multi-file changes, anything requiring multiple steps or tests.

Skip for: quick bug fixes, single-line changes, questions, documentation. When skipped, still ensure appropriate tests exist for any added functionality.

## Asking the User

**Every question put to the user goes through the `AskUserQuestion` tool**. When presenting options, provide genuine alternatives; the recommended one always comes first, labelled "(Recommended)". An open-ended question still goes through the tool, whose built-in free-text path carries the user's own words. If a question is worth stopping for, it is worth making answerable in one click.

## Tech Stack (provisional)

Matches the sibling `../ha-dashboard` project until planning decides otherwise:

- **Language**: TypeScript
- **UI**: React on Vite
- **Testing**: Vitest + React Testing Library, Playwright
- **Linting**: oxlint, Prettier

## Core Principles

- Home Assistant is the source of truth for the house. Home HQ never keeps its own copy of device state.
- This is a real house with a real family. Ask before anything that changes devices, Home Assistant config, or family data.
- Every family member, every device: layouts work from phone to wall display, touch first, accessible.
- Decisions that settle a brainstorm question get an ADR in `.farseer/adr/`.

## Project Structure

- `docs/brainstorming.md` - Research and requirements with stable IDs (`TODAY-06`, `FOS-19`, `RED-40`); section 18 lists the open questions for planning.
- `docs/family-os-inventory.md` - FamilyOS feature inventory.
- `docs/references/reddit/` - Screenshots from researched Reddit threads.

## Key Rules

- Cite brainstorm requirement IDs in plans and issues; never renumber them.
- Never commit tokens or secrets, and never put a long-lived Home Assistant token in source, the bundle, or a publicly served file.
- Tests never change real devices; they run against a fake Home Assistant.
- Use the terms in `.farseer/glossary.md` in names, tests and issue titles.

## Agent Skills

Project configuration files are in `.farseer/`:

- `architecture.md` - Technical patterns and structure (mostly to be decided in planning)
- `code-standards.md` - Coding conventions (provisional)
- `testing.md` - Test configuration and commands (provisional)

### Glossary

The household: Maple Frontier, Home HQ, and family members. See `.farseer/glossary.md`.

### Issue Tracker

Local markdown files under `.farseer/plans/{plan-name}/issues/`; the repo has no remote. See `.farseer/issue-tracker.md`.

### Triage Labels

Default Farseer roles, each label string equal to the role name, written as the issue file's `Status:` line. See `.farseer/issue-labels.md`.
