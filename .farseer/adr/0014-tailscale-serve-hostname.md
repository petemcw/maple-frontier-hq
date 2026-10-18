# ADR-0014: Serve Home HQ at maplefrontier.alpine-ling.ts.net through a Tailscale sidecar

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

ADR-0007 put Home HQ on a LAN domain behind Caddy, so home devices could work without Tailscale. But sign-in goes through HA's login page at its `ts.net` URL (ADR-0005), so every phone and kiosk runs Tailscale anyway. The LAN domain then only helped signed-in devices while Tailscale was off or down, at the cost of a domain, Caddy with a DNS module, a Cloudflare token, a public record and a subnet route. HA and `ha-dashboard` already use Tailscale sidecars with `tailscale serve`.

## Decision

- A `tailscale/tailscale` sidecar with hostname `maplefrontier` runs `tailscale serve` (HTTPS, Funnel disabled) and proxies to the Home HQ container, which uses `network_mode: service:<sidecar>` and publishes no host ports, matching `ha-dashboard`'s compose setup.
- The URL is `https://maplefrontier.alpine-ling.ts.net`, which is also the HA OAuth client ID (ADR-0005).
- The server reaches HA at its LAN address from inside the stack. Browsers still never talk to HA except for the sign-in page.
- `ha-dashboard`'s sidecar currently holds the `maplefrontier` name. It is renamed before Home HQ's first deploy.
- Key expiry is disabled for the Home HQ sidecar and every kiosk node. Kiosks run Tailscale as Android always-on VPN; iPhones use the On Demand setting.

## Alternatives Considered

### LAN domain behind Caddy (ADR-0007)

- **Pros**: signed-in home devices keep working while Tailscale is off or down; the URL survives a tailnet rename.
- **Cons**: domain, Caddy build with a DNS module, Cloudflare token, public record, subnet route, port 443 on the host.
- **Why not**: every device needs Tailscale for sign-in anyway, so the extra infrastructure buys little.

### Another hostname, keeping `maplefrontier` for `ha-dashboard`

- **Why not**: Home HQ is the household's main app and gets the household's name.

## Consequences

### Positive

- No domain, proxy, DNS token or subnet route to run. The third service on the same sidecar pattern.
- HTTPS everywhere from `tailscale serve`, so the service worker and offline queue (ADR-0010) work unchanged.

### Negative

- Every request needs Tailscale on the device, at home too. A device with the VPN switched off can't reach Home HQ.
- The URL is tied to the tailnet name. Renaming the tailnet breaks installed PWAs and the OAuth client ID.
- Anyone using `ha-dashboard` at the old URL has to switch to its new one.

### Risks

- A kiosk cold-booting during an internet outage may not reconnect until Tailscale's coordination server is reachable. Mitigation: the cached app shell (ADR-0010) still renders, with house data greyed out until it reconnects.
- Node keys expiring silently. Mitigation: key expiry disabled on the sidecar and kiosks, as above.
