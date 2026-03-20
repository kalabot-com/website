# Kalabot Documentation

All feature documentation lives here. Every implemented feature has a corresponding doc.

---

## Product

| Doc | Description |
|-----|-------------|
| [Overview](product/overview.md) | Problem, solution, UVP, go-to-market, risks |
| [Target Segments](product/target-segments.md) | Customer segments with calendar counts and strategic notes |
| [Pricing](product/pricing.md) | Subscription tiers, referral program, add-ons, cost structure |
| [Competitors](product/competitors.md) | Competitive landscape: booking products and WhatsApp API players |

---

## Onboarding

| Doc | Description |
|-----|-------------|
| [Onboarding](onboarding.md) | End-to-end client onboarding: 6-step sequence, both WABA flows, segment defaults |

---

## Architecture

| Doc | Description |
|-----|-------------|
| [v1 Product Contract](architecture/v1-product-contract.md) | Canonical locked contract: scope, onboarding order, pricing, WhatsApp ownership |
| [WhatsApp Provider Configuration](architecture/whatsapp-provider.md) | Meta provider types, Business Solution setup, checklist |
| [AI Conversation](architecture/ai-conversation.md) | Agent swarm, tool-calling flow, model (Kimi k2.5), API constraints |
| [Reminder System](architecture/reminder-system.md) | Cron schedule, send window, rate limiting, timezone rules |
| [Manual Mode and Resilience](architecture/manual-mode-and-resilience.md) | bot_enabled semantics, queue/retry strategy, timeout-to-human |
| [Multi-Calendar Booking Rules](architecture/multi-calendar-booking-rules.md) | Booking logic across multiple calendars |
| [Telnyx Provisioning v1](architecture/telnyx-provisioning-v1.md) | Phone number provisioning via Telnyx |
