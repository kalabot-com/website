# Booking Product V1 - Plan

## Key Decisions

- **Stack**: NestJS + Sequelize (backend), Nuxt 4 + Nuxt UI (frontend)
- **AI model**: Kimi k2.5 primary (~5x cheaper than GPT-4o). Test Spanish quality early; switch to GPT-4o-mini if needed. Provider abstraction kept.
- **AI SDK**: Rewrite AI layer with official `openai` npm package + Chat Completions API. Current Responses API code is incompatible with Kimi.
- **WhatsApp**: Replace Meta Graph API + Twilio with Baileys (`@whiskeysockets/baileys`)
- **Approach**: Rewrite AI/chat/WhatsApp layers from scratch rather than patching existing code

## What We Keep

- Auth module (JWT, signup/login, refresh tokens, guards)
- Google Calendar integration (OAuth, FreeBusy, event creation)
- Redis module
- Sequelize ORM + migrations
- Frontend auth flow + Google Calendar connection UI

## What We Remove

- Claude provider
- Twilio provider + npm dependency
- Meta WhatsApp provider
- `js-tiktoken` dependency
- OpenAI Responses API code (`/responses`, `/conversations` endpoints)
- Hardcoded `HAIR_SALON_INSTRUCTIONS`
- Hardcoded 30-min slot logic

## Architecture

```
WhatsApp Customer
  |
  v
Baileys (per-business session)
  |
  v
Chat Service (conversation history in DB)
  |
  v
openai SDK -> Kimi k2.5 (Chat Completions API + tool calling)
  |
  v
Booking Tools (find_service, find_availability, create_event, cancel_event)
  |
  v
Google Calendar API (FreeBusy, events)
  |
  v
Booking Model (local DB tracking)
```

## Phases

### Phase 0: Housekeeping
1. Update all dependencies to latest versions
2. Remove unused code, rewrite AI layer with openai SDK

### Phase 1: WhatsApp with Baileys
3. Baileys provider (QR auth, reconnect, message routing)
4. Multi-tenant session management (DB model, lifecycle, WebSocket for QR)

### Phase 2: Business Configuration & Services
5. Business profile model (name, working hours, timezone, buffer)
6. Service model and CRUD
7. AI-assisted service extraction from price list photo (vision)

### Phase 3: AI Agent System
8. Dynamic prompt system (per-business from config + services)
9. Rewrite booking tools (service-aware, cancel, reschedule)
10. Booking model and lifecycle tracking

### Phase 4: Onboarding Flow
11. Frontend onboarding wizard (5 steps)
12. WhatsApp activation flow (QR + WebSocket status)

### Phase 5: Professional Dashboard
13. Conversations view (list + thread, read-only)
14. Bookings view (table, filters, actions)
15. Calendar view (day/week visual)

### Phase 6: Reminders
16. Reminder system (cron, 24h WhatsApp, confirmation handling)

### Phase 7: Production Readiness
17. End-to-end and integration testing
18. Error handling and edge cases
19. Deployment setup (Docker Compose, env, persistence)
