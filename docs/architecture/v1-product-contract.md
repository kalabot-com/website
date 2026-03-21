# Kalabot v1 Product Contract (Locked)

This document is the canonical v1 contract for product and engineering. If any other document conflicts with this one, this contract takes precedence.

---

## 1. Scope Contract

### In scope (v1)

- WhatsApp-first assistant for:
  - Bookings (Google Calendar integration)
  - Conversational business information (hours, prices, services, FAQs, location)
- Official Meta WhatsApp Cloud API onboarding and messaging
- Human/manual override and resilience fallback

### Out of scope (v1)

- Restaurants
- Lead qualification flows
- Dispatch / field-worker management
- Generic CRM suite
- Multichannel inbox

---

## 2. Mandatory Onboarding Order

The onboarding order is fixed and cannot be reordered:

`Registration -> Payment -> Connect WhatsApp -> Connect Calendar -> Business Info -> Go Live`

Contract rules:

- Users cannot skip mandatory steps.
- Payment must be valid before WhatsApp connection.
- Go-live is blocked until required business information is completed.

---

## 3. Pricing Contract

### Subscription (monthly)

| Price | Calendars |
| --- | --- |
| EUR 25 per calendar | Unlimited |

No tiers. One flat price regardless of calendar count.

### Referral

- EUR 5 per referral per month, while the referred client continues paying.
- Add-ons are excluded from referral earnings.

### Add-ons

- Smart fill and recovery: EUR 15 per 100 messages.
- Marketing campaigns: EUR 15 per 100 messages.

---

## 4. WhatsApp Ownership and Provider Constraints

### Ownership (non-negotiable)

- The client always owns their Meta Business Manager and WABA.
- This applies to both onboarding flows:
  - Client with existing WABA
  - Client without existing WABA (created during Embedded Signup)

### Kalabot role

- Kalabot operates on the official Meta Cloud API using Embedded Signup.
- Kalabot obtains delegated permissions and connects webhook/token infrastructure.
- Kalabot does not take ownership of client Business Manager or WABA assets.

### Number policy

- One dedicated number per client; no shared numbers.
- For clients without an eligible number/WABA path, Kalabot provisions a dedicated number through Telnyx.
- While connected to Cloud API, a number cannot be active in the WhatsApp Business mobile app.

### Provider posture for v1

- v1 operates as a Meta Business Solution implementation on Cloud API.
- Tech Provider/BSP status is not required for v1 scope delivery.

---

## 5. Compliance and Operational Baseline

- Use least-privilege permissions required for Embedded Signup and messaging flows.
- Store and rotate access tokens securely.
- Keep an audit trail of onboarding and number assignment events.
- Enforce idempotent webhook handling for payment and WhatsApp events.

---

## 6. Change Control

Any change to onboarding order, pricing contract, or WhatsApp ownership/provider constraints requires explicit product + engineering approval and an update to this document first.
