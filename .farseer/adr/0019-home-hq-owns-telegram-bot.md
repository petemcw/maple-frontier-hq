# ADR-0019: The Home HQ server owns the Maple Frontier Telegram bot

**Date**: 2026-10-18
**Status**: accepted
**Deciders**: Pete McWilliams

**Note (2026-10-20)**: assistant chat isn't built, so the bot carries quick add and recipe links only. Members link their chat with a one-time code from Home HQ (ADR-0030). The decision is unchanged.

## Context

Quick add (FOS-41) and assistant chat (FOS-44) take messages from a Telegram bot, and FOS-50 adds approve and reject buttons to Telegram notices. The household already has @MapleFrontierBot. Telegram hands a bot's incoming messages to one reader only: a second `getUpdates` poller gets `409 Conflict` or silently loses messages, and a webhook blocks polling entirely. Sharing a bot between icloudpd and another poller broke icloudpd's re-auth replies, so icloudpd now has its own bot. HA's `telegram_bot` integration can be that reader, and its events are on the non-admin subscribe allowlist, but an attachment arrives only as a `file_id` that `telegram_bot.download_file` saves to HA's own disk, and HA ignores any chat missing from its `allowed_chat_ids`.

## Decision

- The Home HQ server is the only reader of @MapleFrontierBot's incoming messages. It polls the Bot API with `getUpdates`; there is no webhook, since Home HQ isn't publicly reachable (ADR-0014).
- The bot token is a server secret, kept like the HA token: never in source, the bundle or a served file.
- HA may send through the same bot for its own automations only with the `broadcast` platform, which never reads messages. HA never uses `polling` or `webhooks` for this bot.
- Push notifications stay with the Companion app (ADR-0012). Whether Telegram also carries FOS-50 notices is left to the planning of those features.

## Alternatives Considered

### HA owns the bot

- **Pros**: one integration path through the service user (ADR-0004), no new secret, and HA automations get the full bot.
- **Cons**: quick add's photos, PDFs and voice notes land on HA's disk where Home HQ can't read them, and each member's chat needs an HA admin to add it to `allowed_chat_ids`.
- **Why not**: FOS-41 attachments are the main reason to have a Telegram way in.

### Webhook instead of polling

- **Pros**: no long-poll loop.
- **Cons**: needs a public HTTPS endpoint, such as Tailscale Funnel.
- **Why not**: exposes Home HQ to the internet for no gain at one household's message volume.

### A second bot for Home HQ

- **Why not**: @MapleFrontierBot already carries the household's name, and HA's broadcast mode removes the conflict that a second bot would avoid.

## Consequences

### Positive

- Attachments reach Home HQ directly, and member linking needs no HA config change.
- Telegram quick add and chat keep working while HA is down.

### Negative

- The server runs and supervises its own polling loop.
- HA automations can send to the bot but can't react to replies.

### Risks

- Another program polls the token, and messages vanish. Mitigation: the server shows a conflict status in admin settings when `getUpdates` returns 409, and every other service gets its own bot.
- The token leaks. Mitigation: `/revoke` in BotFather and replace the server secret.
