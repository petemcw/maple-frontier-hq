# ADR-0030: AI features are quick add and dinner ideas on the Claude API, and the AI only suggests

**Date**: 2026-10-20
**Status**: accepted
**Deciders**: Pete McWilliams

## Context

ADR-0015 put quick add, assistant chat, family search by meaning and dinner ideas in scope (FOS-41 to FOS-45, RED-10). FamilyOS runs these through one OpenAI-compatible client and has every AI change wait as a draft for approval. Its Supervised mode, where an adult approves, was dropped for this household of adults (ADR-0016). A recipe draft is already visible only to its sender until they save it (ADR-0021). Home HQ owns the Telegram bot so it can receive attachments (ADR-0019), its calendars are read-only (ADR-0024), and there's no meal plan (ADR-0021). The research found that only a direct API call gives Home HQ its own propose tools, photo and PDF input, streaming, caching and a key HA users can't spend. Claude takes no audio. Jev, from TypeSafe AI, answers typed questions with probabilities but can't write prose, is waitlist-only, and was five weeks old when we looked. Settles brainstorm section 18 question 16. Research: "LLM route for Home HQ's AI features" (`research/llm-route`).

## Decision

- **Claude API direct.** The server calls Claude with `@anthropic-ai/sdk` and runs Sonnet 5.5 for both features. The key is a server secret, kept like the HA token. Home HQ doesn't use HA's AI Task or conversation agent for these features.
- **The AI only suggests.** The model holds propose tools and no write tools. Everything it reads from a capture becomes a **suggestion** that only the sender sees, starting ticked. **Add selected** saves the ticked ones as the sender, through the same path as their own tap, with one Undo for the batch. No other member approves. Nothing saves on its own, so FOS-43's auto-approval isn't built.
- **Quick add** takes text, photos and PDFs from the Telegram bot and from the in-app add box on a member's own device. Voice notes, a forwarding mailbox and a PWA share target aren't ways in. A member links their Telegram chat by sending the bot a one-time code from Home HQ, and the bot turns away unlinked chats.
- **What a capture can suggest**: tasks (ADR-0023), shopping items (ADR-0017), and recipes. A recipe in a capture becomes a recipe draft (ADR-0021). An event becomes a suggestion with an **Add to Google Calendar** button. The button opens Google's pre-filled event page in the sender's own account, and Home HQ keeps read-only scopes.
- **Reviewing.** The bot replies with the suggestions and an Add selected button. The same suggestions wait in the sender's **quick add tray** in the app, where they can be edited. A suggestion that matches something already there starts unticked and says so ("Already on Shopping"). Suggestions expire after 7 days, or once their date has passed if that comes first. The capture's photo or PDF is deleted once its suggestions are added, dismissed or expired. A recipe draft keeps its photo.
- **Dinner ideas** run only when someone asks, on a phone or a kiosk. Each run shows three ideas, mostly from the recipe box plus one new dish. The model weighs today's weather, the last 14 days of dinners and the household's **food notes**, one short text any member edits. **Other ideas** runs again. **Make this** counts the dish as today's dinner and opens it. A new dish also has **Save to recipe box**, owned by whoever saved it, or by the household on a kiosk. A recipe opened in cook mode also counts as that day's dinner.
- **Kiosks** get dinner ideas but not quick add, since a kiosk's suggestions would have no sender (ADR-0006).
- **Spending.** A monthly AI cap in Home HQ, $5 by default and changed by admins, pauses both features when reached. Admins see this month's spend. The Anthropic Console workspace limit is the backstop. When the cap is hit or the model is down, the bot replies that AI is paused and drops the message. The add box keeps the typed text, and dinner ideas say they're unavailable.
- **Not built**: assistant chat (FOS-44), search by meaning (FOS-45) with or without Jev, and a read-only "ask Home HQ" box. Search is plain full-text search with no AI.

## Alternatives Considered

### HA's AI Task or conversation agent

- **Pros**: one integration path through the service user, and HA already has it.
- **Cons**: no custom tools, no photos from the service user, and a key any HA user can spend outside Home HQ's roles and cap.
- **Why not**: quick add needs photos and propose tools.

### An OpenAI-compatible client, as FamilyOS has

- **Pros**: any provider, including a local model.
- **Why not**: no native tools or caching, for flexibility nobody asked for.

### AI writes directly, with Undo

- **Pros**: fastest, with no review step.
- **Why not**: a misread photo would scatter wrong tasks and items before anyone looked.

### An admin approves what the AI suggests

- **Why not**: everyone is an adult, and ADR-0016 dropped supervised approval. The sender knows what they meant to send.

### Auto-approval for shopping items or tasks (FOS-43)

- **Why not**: Add selected is one tap, and a setting would add a second rule to explain.

### Assistant chat (FOS-44) or an "ask Home HQ" box

- **Pros**: plain-language questions over the family's data.
- **Why not**: the family wanted quick add and dinner ideas, and chat would cost the most. Plain search and the screens answer "what's on the list".

### Jev to re-rank search by meaning

- **Pros**: no embeddings or vector store.
- **Cons**: a second AI vendor, waitlist-only and weeks old, with the candidate records leaving the house on every search.
- **Why not**: plain search is enough for six people. Jev can be looked at again once it's out of early access.

### A write path to the Family calendar

- **Why not**: it reopens ADR-0024's read-only scopes for a few events a month. Google's own event page adds them as the sender.

### Voice notes through Whisper or a hosted speech API

- **Why not**: Siri and Assist already put spoken shopping items in the voice inbox (ADR-0017). Another service, or a second vendor, isn't worth it for the rest.

### Hold a capture and retry while AI is paused

- **Pros**: nothing is lost.
- **Why not**: the family preferred a plain refusal. The sender still has the message in Telegram to send again.

### A meal log or "we had this" tap for recent dinners

- **Why not**: picking an idea and opening cook mode already mark most dinners, with no extra step.

## Consequences

### Positive

- Every AI change goes through the same rules and records as a person's tap, so roles and delete requests need nothing new.
- Expected spend is about $1.30 a month, well under the cap.
- Calendars stay read-only, and no new vendor or mailbox is added.

### Negative

- A capture's text and images, and the parts of the household's data the model needs to match them, go to Anthropic.
- Telegram voice notes are refused.
- Quick add can't add an event straight away. The sender adds it in Google.
- With no chat and no search by meaning, a question like "when did we last service the car?" means opening Records.

### Risks

- Sonnet misreads handwriting or a busy photo. Mitigation: every suggestion is reviewed. An eval on real captures re-checks the model choice before build.
- Recent dinners miss meals cooked without cook mode or dinner ideas. Mitigation: Other ideas, and food notes such as "had tacos twice this week".
- A $5 cap trips in an unusually busy month. Mitigation: admins raise it, and the bot says why it's paused.
- Google's event page opens in a browser signed in to the wrong account. Mitigation: the button names the account it expects. Each member's Workspace address is already known (ADR-0024).
