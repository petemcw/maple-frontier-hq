# ADR-0032: Every family member gets their own HA user and the Companion app, with location While Using

**Date**: 2026-10-27
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

ADR-0005 has every family member sign in to Home HQ with their own HA user, and ADR-0012 sends every push through the HA Companion app. Both assumed every member already had an HA account. The HA inventory found that only Peter's person entity is linked to an HA user, and that only Peter's iPhone and an iPad mini have the Companion app. The other five are home or away from UniFi Wi-Fi alone, and four of their phones use private Wi-Fi addresses. HA has no Home Assistant Cloud, so away from home a phone reaches HA only over Tailscale (ADR-0007, ADR-0014). Task reminders (ADR-0023), Time to leave and leave-by (ADR-0024) and Garage left open (ADR-0027) all wait on this. Research: "Inventory Maple Frontier's Home Assistant" (`research/ha-inventory`) and "Finding which room a family member is in" (`research/room-location`).

## Decision

- **HA users.** Jessica, Ryan, Molly, Claire and Leah each get their own non-admin HA user, linked to their person entity. Peter stays the only HA admin. Jessica is an admin in Home HQ (ADR-0016) but not in HA. ADR-0005 stands.
- **TOTP for the admins only.** Peter and Jessica turn on HA's TOTP. The other four sign in with a password alone, since HA is reachable only on the LAN and the tailnet. Kiosk HA users have no TOTP.
- **The Companion app on every phone.** All six iPhones install it, signed in as their owner, with notifications on and location set to **While Using**. ADR-0012 stands.
- **One notify target each.** Each family member maps to exactly one phone. Their pushes never go to an iPad or a Mac. When someone replaces a phone, an admin re-maps them (ADR-0012's risk).
- **Tailscale on every phone.** All six iPhones join the tailnet with On Demand (ADR-0014), so location updates and push actions such as Close garage reach HA from anywhere. Nothing is exposed to the internet.
- **A stale location means home.** Leave-by and Time to leave use a member's location only when it is under 30 minutes old and Wi-Fi doesn't place them at home. Otherwise they measure from home. This sets ADR-0024's "stale".
- **Private Wi-Fi addresses.** Setup checks that each phone's Private Wi-Fi Address is Fixed or Off on the house networks. Fixed is iOS's default. Rotating would make a new UniFi tracker every two weeks, and with While Using, Wi-Fi does most of the home-and-away work.
- **The wall panel is the only kiosk.** The iPad mini isn't part of Home HQ. Home HQ maps it to nobody and doesn't treat it as a kiosk, and admin settings list its notify target as unmatched.

## Alternatives Considered

### Home HQ's own sign-in for those without HA users

- **Why not**: ADR-0005 rejected a second credential per person, and creating HA users costs less.

### Push through the Telegram bot for those without the Companion app

- **Pros**: Home HQ already owns a bot (ADR-0019), and nobody would need the app.
- **Cons**: two push paths, no time-sensitive level, and actions like Close garage would need building again.
- **Why not**: everyone will install the Companion app.

### Location Always

- **Pros**: leave-by would measure from where the member really is, and Time to leave would fire from anywhere.
- **Cons**: background location on every phone.
- **Why not**: the family chose While Using. Leave-by falls back to home more often.

### Home Assistant Cloud for away access

- **Pros**: no VPN on any phone.
- **Cons**: about $65 a year, and HA becomes reachable from the internet.
- **Why not**: ADR-0007 keeps everything off the internet, and Tailscale On Demand already exists for iPhones.

### TOTP for everyone

- **Why not**: the family accepted a password alone for the four non-admins, with HA on the LAN and the tailnet only.

### The iPad mini as a second kiosk or someone's tablet

- **Why not**: it isn't part of this. The entry panel is the one kiosk.

## Consequences

### Positive

- Task reminders, Time to leave and Garage left open reach every member who should get them.
- Revoking a person still happens in HA's user list (ADR-0005).
- Location stays on the phone unless its owner has the Companion app open.

### Negative

- Each new HA user can see and control every device in the Companion app, because HA has no per-entity permissions.
- With While Using, an away member's location is often stale, so leave-by and Time to leave usually measure from home.
- On iOS, a link in a push opens Safari, not the installed Home HQ app. The notification work has to decide where each push's tap leads.
- Every phone needs the Companion app and Tailscale set up and kept signed in, which is household setup to agree with each person first.

### Risks

- A phone's Private Wi-Fi Address set to Rotating breaks its HA presence. Mitigation: setup checks it, and an admin re-links any new UniFi tracker to the person.
- Tailscale On Demand switched off on a phone stops its push actions and location from reaching HA while away. Mitigation: pushes still arrive through HA's push relay, and the setup checklist covers On Demand.
- A non-admin member's HA password is guessed. Mitigation: HA is reachable only on the LAN and the tailnet, and revoking happens in HA's user list.
