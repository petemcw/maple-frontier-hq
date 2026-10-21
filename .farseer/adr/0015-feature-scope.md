# ADR-0015: Feature scope for Home HQ

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

**Note (2026-10-20)**: of the AI features, only quick add and dinner ideas ship. Assistant chat (FOS-44), search by meaning (FOS-45) and auto-approval (FOS-43) are out of scope (ADR-0030).

## Context

The brainstorm collects features from the prototype, FamilyOS (`FOS-xx`) and Reddit dashboards (`RED-xx`), many sized for other households. Maple Frontier is two adults and four children aged 17 to 21, and the house has fewer devices than the Reddit examples. Settles brainstorm section 18 questions 7 and 19, and the Agents part of question 1.

## Decision

Beyond the prototype's core screens, these areas are in scope: tasks (FOS-19), kitchen (FOS-33 to FOS-36), household records (FOS-37 to FOS-39), quick add, assistant chat and dinner ideas (FOS-41 to FOS-45, RED-10), kitchen timers and appliances (RED-07, RED-09), an illustrated weather hero (RED-01), a verse of the day, flights overhead (RED-33), synced lyrics (RED-50), guest and vacation modes, and parcels (RED-19).

Out of scope: the Agents screen (section 11), the family board (FOS-40), energy (RED-59), cars (RED-41, RED-62), the robot vacuum (RED-35), network health (RED-34), irrigation (RED-60), the daily quote (RED-31), the watch pick (RED-36), and MCP access for outside agents (FOS-46).

An in-scope area can still drop features the house can't support.

## Alternatives Considered

### Everything in the brainstorm

- **Pros**: nothing is ruled out early.
- **Cons**: much of it serves other households and their devices.
- **Why not**: planning time goes to features this family won't use.

### Prototype screens only

- **Pros**: smallest backlog.
- **Cons**: misses tasks, kitchen and records, which the family wants.
- **Why not**: too narrow for a family hub.

## Consequences

### Positive

- Planning stays on what this family will use.

### Negative

- Bringing back an out-of-scope area needs a new ADR.

### Risks

- Device-dependent features in scope (timers, appliances, parcels) may turn out unbuildable with today's hardware. Mitigation: the area tickets in the brainstorm-to-backlog map decide this per feature.
