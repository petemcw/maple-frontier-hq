# ADR-0009: Build images in GitHub Actions and deploy from GHCR

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

Home HQ runs in the existing Docker Compose stack at `/home/prm/iot` on 10.10.20.98 (x86_64), next to HA. The repo has no remote yet. `ha-dashboard` already builds in GitHub Actions and pushes to GHCR.

## Decision

Add a private GitHub remote. GitHub Actions builds a `linux/amd64` image and pushes `ghcr.io/petemcw/home-hq`, and the iot Compose stack pulls it. Secrets (the HA service token, the Cloudflare token, the session signing key) live in Docker secrets or a host `.env`, never in the repo or the image.

## Alternatives Considered

### Build on the host

- **Pros**: no remote or registry.
- **Cons**: builds use the HA box's CPU; source cloned onto the server.
- **Why not**: the HA host should run things, not build them.

### Build locally and push by hand

- **Why not**: no CI, so nothing checks a build before it reaches the house.

## Consequences

### Positive

- The same flow as `ha-dashboard`; CI runs the pre-commit checks before an image exists.

### Negative

- Depends on GitHub for builds.

### Risks

- The GitHub remote does not move the issue tracker; issues stay in local markdown (`.farseer/issue-tracker.md`) until we decide otherwise.
