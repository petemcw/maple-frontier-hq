# ADR-0013: Evergreen browsers and a fluid layout; the wall panel is Android with Fully Kiosk

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

The prototype targets one 10.1" 1280x800 photo frame running Android WebView 106, using a fixed stage scaled to fit (NFR-01) and CSS limited for that engine (NFR-02). Home HQ runs on phones, tablets and a 21.5" touch panel. Settles brainstorm section 18 question 12.

## Decision

- Support current Safari (iOS and iPadOS 17+) and current Chrome only.
- Layouts are fluid and responsive from phone to 1920x1080 landscape. The fixed stage and the Chromium 106 CSS limits are dropped.
- The 21.5" panel is an Android all-in-one running Fully Kiosk Browser (current Chromium, HA integration for screen on/off and motion wake, remote reload), with Tailscale (ADR-0007).

## Alternatives Considered

### Keep the Chromium 106 floor

- **Pros**: the old frame keeps working.
- **Cons**: no `color-mix()`, `dvh`, CSS nesting or 8-digit hex anywhere.
- **Why not**: no device in the plan needs it.

### Mini PC or Raspberry Pi with Chromium kiosk

- **Why not**: screen wake and remote reload become our job; Fully Kiosk does both and talks to HA.

## Consequences

### Positive

- Modern CSS and one layout system for every device.

### Negative

- NFR-01 and NFR-02 no longer apply; the prototype's frame isn't supported.

### Risks

- Fully Kiosk's WebView updates with Android System WebView. Mitigation: keep the panel's WebView auto-updating.
