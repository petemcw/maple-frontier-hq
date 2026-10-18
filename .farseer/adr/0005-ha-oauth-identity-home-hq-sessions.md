# ADR-0005: Sign in with Home Assistant; Home HQ owns sessions and roles

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

Every family member already has an HA account. Devices are rarely shared: each phone belongs to one person, and kiosks don't pick a person. HA's user model has no idea of kids or displays. Settles brainstorm section 18 question 20 and part of question 11.

## Decision

- Family members and kiosks sign in through HA's OAuth flow (client ID is the Home HQ URL). The server exchanges the code, reads the HA user with `auth/current_user`, maps it to a Home HQ member, and discards the HA tokens.
- Home HQ issues its own sliding session of about 90 days: an HTTP-only cookie for the web, a bearer token for future native clients (ADR-0003).
- Each kiosk has its own HA user (e.g. "Kitchen Panel"), so one kiosk is revoked by disabling its HA user.
- Roles (admin, member, display; the kid role was dropped by ADR-0016) are stored in Home HQ. The first HA admin to sign in becomes the Home HQ admin.
- HA commands still go through the service user (ADR-0004), whoever is signed in.

## Alternatives Considered

### Also act as the signed-in user

- **Pros**: the HA logbook names the person.
- **Cons**: six or more HA refresh tokens to encrypt and guard.
- **Why not**: Home HQ records attribution itself.

### Per-user long-lived HA tokens

- **Why not**: clunky to paste, never expire, and add nothing over OAuth.

### Home HQ's own accounts (passkeys and password)

- **Pros**: passkeys, no dependency on HA's login page.
- **Cons**: a second credential per person to manage.
- **Why not**: one sign-in per device makes HA's password-plus-TOTP login good enough.

### One shared "Home" HA user, or pairing codes for kiosks

- **Why not**: a shared user can't revoke one kiosk; pairing codes mean more code to build when HA users already work.

## Consequences

### Positive

- No passwords stored in Home HQ; HA MFA applies.
- Revoking a person or kiosk happens in HA's user list.

### Negative

- No passkeys. Sign-in depends on HA's login page being reachable from the device (kiosks run Tailscale for this, ADR-0014).

### Risks

- An HA user with no Home HQ mapping signs in. Mitigation: unmapped users get no access until an admin assigns a member and role.
