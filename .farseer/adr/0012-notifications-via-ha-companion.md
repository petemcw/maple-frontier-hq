# ADR-0012: Push notifications go through the HA Companion app

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams
**Note (2026-10-27)**: only Peter had the Companion app. All six phones install it, each member maps to one phone, and links in a push open Safari on iOS (ADR-0032).

## Context

Home HQ needs to reach phones with chore reminders, leak alerts and list changes. Every family member has an HA account. Web push on iOS only works for a PWA installed to the home screen and has no critical-alert level.

## Decision

The server sends notifications by calling `notify.mobile_app_<device>` through the HA service user (ADR-0004). Home HQ maps each member to their Companion app notify targets. There is no web push.

## Alternatives Considered

### Web push from the PWA (VAPID)

- **Pros**: works without the Companion app.
- **Cons**: iOS needs an installed PWA; no critical alerts; push keys and subscriptions to manage.
- **Why not**: the Companion app already does this better on iOS.

### Both, chosen per notice

- **Why not**: two push systems to maintain.

## Consequences

### Positive

- Critical alerts (water leak) can break through Focus on iOS.
- No push infrastructure to build.

### Negative

- Every member's phone needs the HA Companion app.
- Tapping a notification opens the Companion app unless the notice carries a Home HQ URL.

### Risks

- A member's notify target changes when they replace their phone. Mitigation: show unmatched targets in admin settings.
