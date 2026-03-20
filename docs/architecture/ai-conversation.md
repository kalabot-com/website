# AI Conversation Architecture

This document describes the AI layer powering Kalabot's WhatsApp conversations: the agent structure, tool-calling flow, model choice, and API constraints.

---

## 1. Agent Roles

Kalabot uses an agent swarm pattern. Each agent has a focused role:

| Agent | Responsibility |
|-------|---------------|
| **Triage** | Classifies the incoming message: new booking, cancellation, price question, FAQ, or other. Routes to the appropriate agent. |
| **Booking** | Handles booking intent. Translates natural language to Google Calendar queries (e.g. "Tuesday afternoon" → `timeMin`, `timeMax`). Checks availability via `freeBusy`, proposes slots, and confirms. |
| **Info** | Answers client questions from owner-provided content: services, prices, hours, FAQs, location. |
| **Closing** | Drafts final responses and seeks explicit confirmation when needed (e.g. booking confirmation, cancellation acknowledgment). |

---

## 2. Tool-Calling Flow

Every inbound WhatsApp message follows this sequence:

1. **Receive** — inbound message arrives via Meta Cloud API webhook.
2. **Load history** — chat service loads conversation history for the thread.
3. **First completion** — `chat.completions.create(messages, tools)` with the full tool set available. The model may call one or more tools.
4. **Execute tools** — tool results are executed (calendar queries, content lookups, etc.) and appended to the message list.
5. **Second completion** — `chat.completions.create(messages_with_tool_results)` to produce the final reply.
6. **Persist** — save all messages (user, assistant, tool calls, tool results) to the database.
7. **Send** — deliver the reply via Meta Cloud API.

---

## 3. Model and SDK

- **Model**: Kimi k2.5 (via Moonshot AI), used for cost efficiency.
- **SDK**: `openai` npm package, configured with a custom `baseURL` pointing to the Kimi-compatible endpoint.
- **API**: Chat Completions API (`/chat/completions`).

**Do not use the OpenAI Responses API (`/responses`).** It is incompatible with Kimi k2.5. All model calls must go through Chat Completions.

The model is abstracted via environment variables so the provider can be swapped without code changes:

```
AI_API_KEY=...
AI_BASE_URL=...
AI_MODEL=...
```

---

## 4. Calendar Integration

- **Availability**: Google Calendar `freeBusy` API for real-time slot checking.
- **Buffer logic**: Configurable cleanup time between appointments to avoid overlap.
- **Natural language → query**: The booking agent converts conversational time references to structured `timeMin`/`timeMax` parameters.

---

## 5. Business Info Answering

- Owner uploads a price list (image or document) during onboarding.
- Kimi k2.5 vision extracts services, prices, and durations at upload time.
- The info agent answers client questions from this extracted content plus any manually added FAQs, hours, and location.

---

## 6. References

- [Reminder System](reminder-system.md) — scheduled outbound messages using the same AI infrastructure
- [Manual Mode and Resilience](manual-mode-and-resilience.md) — how conversations hand off to humans
- [v1 Product Contract](v1-product-contract.md) — scope boundaries for the AI layer
