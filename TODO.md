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
- Run migrations before starting beta in Docker (or add to CI/CD).
- Add confirmation/cancellation response handling for reminders.
- Spanish quality validation with Kimi k2.5.
- Stripe payment integration for onboarding.
- QR link sharing page for end customers.
