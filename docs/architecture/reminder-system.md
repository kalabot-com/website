# Reminder System

This document describes how Kalabot sends scheduled appointment reminders and confirmations to reduce no-shows.

---

## 1. Schedule

The reminder cron runs **every 5 minutes** via `@nestjs/schedule`.

---

## 2. Send Window

Reminders are only sent between **8:00 AM and 6:00 PM** in the account's local timezone.

- Timezone source: `BusinessProfile.timezone` (set during business info setup).
- All timezone-aware date math uses **Luxon**.
- A reminder due outside the send window is held and delivered at the start of the next eligible window.

---

## 3. Rate Limiting

To mitigate Meta messaging quality score degradation and ban risk:

- **Maximum 30 sends per cron run**.
- **10-second delay between sends** within a single run.

These limits apply per run across all accounts, not per account.

---

## 4. Reminder Types

- **Appointment reminder**: sent to the client before their booking (timing configurable per business).
- **Confirmation request**: sent after booking to confirm the appointment is still on.

---

## 5. Message Templates

Reminders are sent as Meta-approved message templates (required for business-initiated messages outside the 24-hour customer service window). Template management is handled during onboarding.

---

## 6. References

- [AI Conversation Architecture](ai-conversation.md) — the same AI infrastructure handles conversational replies to reminder responses
- [Manual Mode and Resilience](manual-mode-and-resilience.md) — reminder sends are subject to the same `bot_enabled` and global manual mode checks
- [WhatsApp Provider Configuration](whatsapp-provider.md) — template requirements and Meta messaging rules
