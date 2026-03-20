# Kalabot Product Overview

---

## 1. Problem

- Small businesses (salons, clinics, service providers) waste time on calls and WhatsApp for **bookings** and **repeated questions** (hours, prices, services, location).
- Many SMBs have no website or cannot keep one updated — clients expect instant answers anyway.
- Existing solutions are either too complex (heavy Meta onboarding, technical setup) or too generic (multichannel CRMs not designed for SMB needs).
- Result: lost bookings, frustrated clients, overworked staff.

---

## 2. Solution

**WhatsApp-first business assistant** focused on two pillars:

1. **Bookings**: 24/7 scheduling with Google Calendar sync.
2. **Conversational Business Info**: Clients ask questions via WhatsApp (hours, prices, services, FAQs); the bot answers from content the owner provides — effectively replacing the need for a website for common inquiries.

- **Official WhatsApp Business API**: Quality, resilience, and trust. No reverse-engineered protocols. See [WhatsApp Provider Configuration](../architecture/whatsapp-provider.md).
- **Simple onboarding**: Six-step sequence from registration to live. See [Onboarding](../onboarding.md) and [v1 Product Contract](../architecture/v1-product-contract.md).
- **Dashboard**: View all conversations, bookings, and client interactions in one place.
- **Human support**: Onboarding and live help tuned for non-technical SMB owners.

### 2.1 Product Scope

**In scope**

- Bookings (Google Calendar integration)
- Conversational Business Info (owner-provided content, AI-powered answers to client questions)
- Reminders and confirmations to reduce no-shows
- Single, focused WhatsApp channel

**Out of scope**

- Lead qualification flows
- Service dispatch / field-worker management
- Multichannel inboxes
- Generic CRM features

### 2.2 Objectives

**A. End Customer Experience (WhatsApp user)**

- **Natural feel**: Conversation with an efficient human assistant. No "Press 1" menus.
- **24/7 availability**: Book and get answers at any time.
- **Reduced no-shows**: Reminders and confirmation flows with clear, persuasive language.
- **Instant answers**: Common questions (hours, prices, services) answered from owner content.

**B. Professional Experience (business owner)**

- **Instant onboarding**: Calendar + WhatsApp + business info in minutes.
- **Zero maintenance**: Manage from the calendar. Move an appointment → bot notifies the client.
- **No website needed**: Clients get info through WhatsApp; owner provides content once.
- Human override at any moment
- Operational resilience

---

## 3. Unique Value Proposition

- **"Your 24/7 receptionist on WhatsApp"** — bookings and answers to client questions, without a website.
- **WhatsApp-first**: No multichannel bloat; one channel done well.
- **Official API**: Quality, resilience, trust.
- **Conversational Business Info**: Replace a static website with a bot that answers from your content.
- **Vertical templates**: Ready-to-use flows, no flow builder required.

### Competitive Advantage: Invisibility

Kalabot does not compete on "number of buttons" but on convenience:

- **Where the user is**: ~90% of people in Spain already have WhatsApp open. No download, no login, no friction.
- **AI vs. menus**: Booksy is a database with menus. Kalabot is a conversation. The client says "Do you have anything Tuesday afternoon?" and the bot responds. That is a premium experience, not low-cost.

---

## 4. Go-to-Market

- **Pilot phase**: Personal network — 3 salons, 1 consultant (or similar booking-heavy SMBs). Goal: validate product fit and willingness to pay.
- **Acquisition (Spain first)**: Direct outreach, referrals, ads (to be defined).
- **Expansion**: WhatsApp-heavy markets (Spain → LATAM → EU / India).

Geography: Spain first (WhatsApp penetration ~90%, low SMB automation adoption), then Europe, LATAM, India, Brazil.

---

## 5. Risks & Mitigation


| Risk                                  | Mitigation                                                                                                                     |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| **Onboarding friction (Meta / WABA)** | Clear documentation, assisted onboarding. Both flows (with/without WABA) supported; client owns assets, we obtain permissions. |
| **Compliance (spam, misuse)**         | Strict opt-in, quality score monitoring, usage policies.                                                                       |
| **Competition**                       | Differentiate with SMB simplicity, bookings + info, human support. See [Competitors](competitors.md).                          |
| **Technical / operational**           | Official WhatsApp API reduces stability concerns vs. reverse-engineered solutions; monitor delivery and session health.        |
| **Regulatory (GDPR, Meta policy)**    | Data processing agreements, privacy policy, alignment with Meta ToS.                                                           |


---

## 6. References

- [Target Customer Segments](target-segments.md)
- [Pricing](pricing.md)
- [Competitors](competitors.md)
- [Onboarding](../onboarding.md)
- [v1 Product Contract](../architecture/v1-product-contract.md)
- [WhatsApp Provider Configuration](../architecture/whatsapp-provider.md)
- [Manual Mode and Resilience](../architecture/manual-mode-and-resilience.md)
- [AI Conversation Architecture](../architecture/ai-conversation.md)

