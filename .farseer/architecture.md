# Architecture

Home HQ is the family and home hub for Maple Frontier. All six family members will use it, from various devices. It builds on Home Assistant for the house and adds family features (calendar, chores, shopping, photos and more).

> **To be decided in planning.** There is no code yet, and the directory structure, patterns and integrations are deliberately left open. The research behind those decisions is in `docs/brainstorming.md`; its open questions (section 18) include whether to build on FamilyOS, how Home HQ is hosted, where shared family state lives, and how each device signs in. Record each decision as an ADR in `.farseer/adr/` and fill in the sections below as plans land.

## Directory Structure

To be decided in planning.

Existing content:

- `docs/brainstorming.md` - Requirements from the photo-frame prototype, FamilyOS candidate features, and candidate features from Reddit dashboards, with requirement IDs (`TODAY-06`, `FOS-19`, `RED-40`).
- `docs/family-os-inventory.md` - Code-grounded inventory of FamilyOS.
- `docs/references/reddit/` - Screenshots from the Reddit threads, one folder per thread.

## Patterns Used

To be decided in planning. The provisional stack (React, Vite, TypeScript) matches the sibling `ha-dashboard` project, whose patterns (`../ha-dashboard/.farseer/architecture.md`) are the starting point to accept, adapt or reject.

## Conventions

- Requirement IDs from `docs/brainstorming.md` are stable. Refer to them in plans and issues; don't renumber them.
- Decisions that settle a brainstorm question get an ADR.

## Key Integrations

To be decided in planning. Home Assistant is the source of truth for the house.

## Business-Critical Paths

Areas where a change carries the highest test and review burden:

- **Anything that changes the real house**: locks, garage, alarm, climate, water valve, lights. A bug can unlock a door or turn off the heat.
- **Auth and tokens**: how each family member and each device signs in. Tokens must never reach source, the bundle, logs, git, or a publicly served file.
- **Family data and privacy**: presence, location, children's details, photos.

Before changing one of these, check `.farseer/adr/` for a decision covering that area.
