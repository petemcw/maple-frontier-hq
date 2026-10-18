# ADR-0007: A LAN domain behind Caddy, with Tailscale for away access

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

Home HQ runs on 10.10.20.98 in the iot Compose stack next to HA. It must work at home and away, and only Tailscale may expose it. The PWA, service worker and wake lock need a secure context (HTTPS) on every device, the LAN included. HA itself is already reachable at its `ts.net` URL.

## Decision

- Caddy in the iot stack serves `hq.<domain>` for Home HQ only. Its certificate comes from Let's Encrypt via DNS-01 with Caddy's Cloudflare DNS module and a scoped Cloudflare API token.
- A public A record points `hq.<domain>` at 10.10.20.98. The host's `tailscaled` advertises 10.10.20.98/32 as a subnet route, so away devices reach the same URL and certificate over Tailscale. Nothing is exposed to the internet.
- Kiosks also run the Tailscale app, so they can reach HA's `ts.net` login page (ADR-0005). After sign-in they use the LAN domain, so a Tailscale outage doesn't break them.

## Alternatives Considered

### Tailscale everywhere via `tailscale serve`

- **Pros**: no domain or proxy; how `ha-dashboard` already runs.
- **Cons**: every device must run Tailscale for every request; `ts.net` URL only.
- **Why not**: home devices should keep working when Tailscale doesn't.

### Plain HTTP on the LAN, Tailscale away

- **Why not**: no secure context on the LAN, so no service worker, wake lock or offline queue.

### Split DNS with no public record

- **Pros**: hides even the private IP.
- **Cons**: UniFi local DNS plus Tailscale split DNS to configure; DNS breaks if UniFi is down.
- **Why not**: a private IP in public DNS is harmless.

### Put HA behind the same Caddy

- **Why not**: HA's existing access already works; kiosks use Tailscale for the one-time sign-in instead.

## Consequences

### Positive

- One URL and one valid certificate for every device, at home and away.

### Negative

- A Cloudflare API token and a Caddy build with the DNS module to maintain.
- Away access needs the Tailscale app on every member's phone; six users may exceed Tailscale's free tier unless some devices sit under a parent's login.

### Risks

- Port 443 on 10.10.20.98 may already be taken (not verified). Check before the first deploy.
- Subnet routers SNAT by default, so away clients appear to come from the host's LAN IP. Don't use client IP for any access decision.
