# TODO - Booking Product V1

All tasks from the initial plan have been implemented.

## Phase 0: Housekeeping
1. Done - Dependencies updated (openai SDK, @nestjs/schedule, Baileys). Removed AuditModule.
2. Done - AI layer uses openai SDK + Chat Completions API. Claude/Twilio/Meta providers removed.

## Phase 1: WhatsApp with Baileys
3. Done - Baileys provider with QR auth, auto-reconnect, DB-backed auth state.
4. Done - WhatsappSession model, lifecycle, API endpoints, WebSocket gateway.

## Phase 2: Business Configuration & Services
5. Done - BusinessProfile model, API, frontend settings.
6. Done - Service model and CRUD, frontend admin UI.
7. Done - AI-assisted service extraction from price list photo (Kimi k2.5 vision).

## Phase 3: AI Agent System
8. Done - Dynamic prompt system (buildSystemPrompt from BusinessProfile + Services).
9. Done - Booking tools: find_service, find_availability, create_event, cancel_event.
10. Done - Booking model, lifecycle, list/update API.

## Phase 4: Onboarding Flow
11. Done - Frontend onboarding wizard (5 steps).
12. Done - WhatsApp activation flow (QR + WebSocket status).

## Phase 5: Professional Dashboard
13. Done - Conversations view (list + thread, read-only).
14. Done - Bookings view (table, filters, cancel, no-show).
15. Done - Calendar view (weekly grid).

## Phase 6: Reminders
16. Done - Reminder cron every 15 min, 24h WhatsApp reminders, reminderSentAt.

## Phase 7: Production Readiness
17. Done - Unit tests (128 passing including ReminderService).
18. Done - Error handling: AI fallback messages, rate limiting (20/ min), duplicate detection (2s window), health endpoint.
19. Done - Docker Compose (postgres, redis, beta), docker-compose.dev.yml, beta Dockerfile.

---

## Next steps (future work)
- Review reminders:
  - use luxon
  - use start day and end day to find target dates
  - in the reminder, do not ask for an answer, do the opposite: only if the user can not attend, send a message
  - do it 2 - 3 times per day?
  - is it safe to send menssages like that? probably not (pottential Meta ban)
- review onboarding: it is not following what is defined in the readme
  - payment
  - phone numbers
- calendar:
  - maybe it is better to embeded the google calendar instead of creating our own view?
  - from calendar event click and go to conversation
- conversations and bookings: from bookings, go to conversation
- manual conversation: in case of error, we may want the client to be able to talk with the customer and the bot must not answer. Maybe we can detect that kind of situation (when the client writes)
- it just booked in a busy slot?
