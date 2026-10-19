# ADR-0021: A family recipe box with parsed ingredient lines, and no meal plan

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

ADR-0015 put the kitchen (FOS-33 to FOS-36) in scope. FamilyOS has a recipe box, a weekly meal plan, today's meals on Today and an ingredients-to-shopping handoff. It keeps ingredients as free-text lines and never scales. Maple Frontier decides dinner day by day, so nobody would keep a weekly plan up to date. The family's recipes are spread across Crouton, Paprika 3, Umami and cookbooks. There's no screen in the kitchen: whoever cooks uses a phone or brings in a tablet. The family uses iPhones, and iOS Safari can't receive shares into a web app. Home HQ already owns the family's Telegram bot (ADR-0019).

## Decision

- Home HQ keeps one family **recipe box**. Whoever adds a recipe owns it. Deleting someone else's is a delete request (ADR-0016).
- There's no meal plan and no "today's meals" (FOS-34 and FOS-36 are ruled out).
- Each ingredient line is parsed when it's saved into an amount, a unit and the rest. Lines that don't parse ("salt to taste") are kept as written and never scale.
- A recipe scales by servings, or by ½× and 2×. Amounts show as kitchen fractions, and units tidy within their own system (6 tsp → 2 tbsp, 16 oz → 1 lb).
- A US/metric toggle converts ingredient amounts like for like: volume to volume and weight to weight, never volume to weight. Step text is never converted. Each device remembers the choice for every recipe. Recipes are the only place in Home HQ that converts units (ADR-0020).
- Ticked ingredients go to a shopping list as shown on screen, scaled and converted. An item already on the list is never changed, and the result reports the difference.
- Recipes come in by pasting a URL (read from the page's schema.org Recipe data), by typing them in, or by sending a link to the Telegram bot. A link sent to the bot becomes a **recipe draft** that only the sender sees until they save it.
- Nothing is imported from Crouton, Paprika or Umami.

## Alternatives Considered

### Free-text ingredient lines (FamilyOS)

- **Pros**: nothing to parse at save time, and lines can't be mangled.
- **Cons**: no scaling and no unit toggle, both of which the family wants.
- **Why not**: scaling needs amounts the code understands.

### Weekly meal plan

- **Why not**: dinner is decided on the day, so a plan would sit empty. Dinner ideas belong to the AI features.

### One-time import from the recipe apps

- **Pros**: the family's existing recipes on day one. Crouton and Paprika exports are documented by working importers.
- **Why not**: the family chose to bring recipes in as they get cooked. Umami's bulk format is unverified.

### Ongoing Paprika sync

- **Why not**: the cloud API is unofficial, and heavy use has reportedly led to IP bans.

### iOS Shortcut or a web share target

- **Why not**: iOS Safari doesn't support web share targets, and a Shortcut needs a token set up on every phone. The bot is already in everyone's share sheet.

## Consequences

### Positive

- Recipes scale and switch units without the cook doing the arithmetic.
- Sharing a link from any app works with no setup on the phone.

### Negative

- The ingredient parser is real work: fractions, ranges, unicode fractions, headings and US and metric units.
- The recipe box starts empty.

### Risks

- A misparsed line scales wrongly. Mitigation: the edit form shows how each line was read, and an unparsed line never scales.
- Some recipe sites block server fetches. Mitigation: the form opens with the URL as the source, ready to fill in by hand.
