# Kalabot Business Plan

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

- **Official WhatsApp Business API**: Quality, resilience, and trust. No reverse-engineered protocols.
- **Simple onboarding**: Registration -> payment -> connect WhatsApp -> connect calendar -> upload business info (price list, FAQs) -> go live/start receiving clients.
- **Dashboard**: View all conversations, bookings, and client interactions in one place.
- **Human support**: Onboarding and live help tuned for non-technical SMB owners.

### 2.1 Platform Model (Channel Ownership & Control)

Kalabot operates as a **Business Solution** (per Meta WhatsApp Provider Configuration):

- **WABA and Business Manager** always belong to the client. In both onboarding flows (with or without existing WABA), the client owns these assets.
- **Kalabot's role**: Obtain partner permissions and connect the client's WABA and phone number(s) to the Cloud API.
- **Number policy**: One dedicated number per client. If needed, Kalabot provisions the number through Telnyx.
- **Numbers** operate exclusively via Cloud API. The WhatsApp Business mobile app cannot be active at the same time.
- **Onboarding flows**:
  - **With WABA**: Client grants access to their Business Manager → connect their WABA to our app via Embedded Signup.
  - **Without WABA**: Client completes Embedded Signup → Meta creates a Business Manager and WABA for them → we receive tokens and connect the Cloud API.

This model ensures:

- Predictable conversations
- Single, consistent flow for all clients
- Resilience and fallback built in
- Cost control (client or Kalabot pays for messaging per agreement)

### 2.2 Product Scope

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

### 2.3 Onboarding

Locked contract reference: [v1 Product Contract](architecture/v1-product-contract.md)

| Step | Action |
|------|--------|
| 1 | Registration |
| 2 | Payment |
| 3 | **WhatsApp Business API**: Connect via Embedded Signup — existing number or new one. Client owns WABA and Business Manager; we obtain permissions. |
| 4 | **Google Calendar**: Link existing calendar in two clicks |
| 5 | **Business Info**: Upload price list (photo or document); AI (Kimi k2.5 vision) extracts services, prices, durations. Optionally add FAQs, hours, location. |
| 6 | **Go Live**: Activate assistant after required onboarding data is complete |

### 2.4 Objectives

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

## 3. Technical Architecture

### 3.1 Connectivity Layer

- **WhatsApp Business API**: Official Meta Cloud API for reliability, compliance, and scalability.
- **Session and number lifecycle**: Each client connects their own WABA and number; auth and delivery handled through the official stack.

### 3.2 Intelligence Layer

- **Orchestrator**: Agent swarm using Kimi k2.5 for cost efficiency.
- **Triage agent**: Classifies incoming messages — new booking, cancellation, price question, FAQ, other.
- **Booking agent**: Natural language → Google Calendar queries (e.g. "Tuesday afternoon" → `timeMin`, `timeMax`).
- **Info agent**: Answers client questions from owner-provided content (services, prices, hours, FAQs).
- **Closing agent**: Drafts final responses and seeks explicit confirmation when needed.

### 3.3 Calendar Integration

- **Google Calendar API**: Real-time availability via `freeBusy`
- **Buffer logic**: Configurable cleanup time between appointments to avoid overlap

---

## 4. Resilience & Fallback Architecture

### Human Override Mode

- Owner can respond manually to any conversation
- Pause the bot per chat
- `bot_enabled` flag per conversation (`true` / `false`)

### Degraded Mode

- Webhooks continue receiving messages
- Secure queue for message storage
- Owner notification: "Assistant in manual mode"
- Conversations automatically switch to human handling

### Minimum Infrastructure

- Stable webhook
- Message queue (Redis / SQS)
- Safety timeout (>X seconds → hand off to human)
- Health monitoring and alerts

---

## 5. Service Definition

The system supports a service matrix configured by the owner:

| Service | Estimated Duration | AI Requirements |
|---------|-------------------|-----------------|
| Quick cut | 20 min | Immediate confirmation |
| Colour and styling | 120 min | Verify long slots; avoid close to closing |
| Physio (session) | 60 min | Ask if first visit (needs more time for intake) |

---

## 6. Target Market

*Aligned with [Target Customer Segments](clientes.md)*

### Geography

Spain first (WhatsApp penetration ~90%, low SMB automation adoption), then Europe, LATAM, India, Brazil.

### Segments

| Segment | Avg. Calendars | Notes |
|---------|----------------|-------|
| **Beauty & Aesthetics** | 2.5 | Barber shops, hair salons, nail salons, tattoo studios. Primary Booksy territory; high no-show impact. |
| **Health & Wellness** | 1.8 | Physiotherapists, psychologists, nutritionists, dentists, veterinarians. High-trust vertical; WhatsApp reminders highly valued. |
| **Professional Services** | 3.2 | Mechanics, tutors, padel courts, real estate. Underserved; many discover the need when WhatsApp saves 2+ hours of phone time per day. |

**Overall average**: ~2.5 calendars per client.

*Restaurants are out of scope — the use case (table management, shifts) is too different from appointment-based scheduling.*

### Ideal Profile

Businesses that receive many "what are your hours?" and "how much is X?" queries — and want to offload both bookings and info to WhatsApp.

---

## 7. Unique Value Proposition

- **"Your 24/7 receptionist on WhatsApp"** — bookings and answers to client questions, without a website.
- **WhatsApp-first**: No multichannel bloat; one channel done well.
- **Official API**: Quality, resilience, trust.
- **Conversational Business Info**: Replace a static website with a bot that answers from your content.
- **Vertical templates**: Ready-to-use flows, no flow builder required.

### Competitive Advantage: Invisibility

Kalabot does not compete on "number of buttons" but on convenience:

- **Where the user is**: ~90% of people in Spain already have WhatsApp open. No download, no login, no friction.
- **AI vs. menus**: Booksy is a database with menus. Kalabot (with Kimi k2.5) is a conversation. The client says "Do you have anything Tuesday afternoon?" and the bot responds. That is a premium experience, not low-cost.

---

## 8. Competitors

### Booking Products

| Competitor | Pricing |
|------------|---------|
| **Booksy** | €29 base + €20 per extra calendar |
| **Treatwell** | Commission per booking from marketplace. Kalabot is fixed fee — better for businesses with an existing client base. |
| **Calendly** | €12 personal (up to 6 calendars); €20/person for teams |

### WhatsApp Business API Products

- **360dialog**: Developer-focused, no SMB dashboard
- **Tidio, Respond.io**: Multichannel, not WhatsApp-first, limited SMB flows
- **WATI, Callbell, Gupshup**: Inboxes and automation, heavier onboarding, less SMB simplicity
- **Opportunity**: Simple onboarding + bookings + conversational info + strong UX on the official API

### Strategic Note (from Target Segments)

At 3–4 staff, Booksy's volume discounts become more attractive.

---

## 9. Business Model

### Pricing (Monthly Subscription)

Locked pricing contract reference: [v1 Product Contract](architecture/v1-product-contract.md)

| Tier | Price | Calendars |
|------|-------|-----------|
| Tier 1 | €25 per calendar + reminders | Up to 2 calendars |
| Tier 2 | €20 per calendar + reminders | Up to 10 calendars |
| Tier 3 | Contact us | — |

**Add-ons**

- Smart fill & recovery: +€15 per 100 messages (cost: 100 × €0.0509 = €5.09)
- Marketing campaign: +€15 per 100 messages (cost: 100 × €0.0509 = €5.09)

**Strategic implication**: Most clients need ~2 calendars → ~€50/month per customer, doubling revenue per acquisition vs. single-calendar pricing.

### Referral Program

- €5 per referral (while the referred client continues paying)
- Referred clients pay a minimum of €10 per calendar (discounted from standard tiers)
- Add-ons not included

### Costs (Per Client)

| Item | Cost |
|------|------|
| Telnyx (numbers) | €1 |
| VPS | €0.25 |
| AI (Kimi k2.5) | ~€0.50 |
| Stripe | ~€0.48 (approx. 1.5% + €0.25 on €15) |
| Invoicing (Holded) | ~€0.30 |
| Others | ~€0.20 |
| **Total** | **~€3** |

---

## 10. Go-to-Market

- **Pilot phase**: Personal network — 3 salons, 1 consultant (or similar booking-heavy SMBs). Goal: validate product fit and willingness to pay.
- **Acquisition (Spain first)**: Direct outreach, referrals, ads (to be defined).
- **Expansion**: WhatsApp-heavy markets (Spain → LATAM → EU / India).

---

## 11. Risks & Mitigation

| Risk | Mitigation |
|------|-------------|
| **Onboarding friction (Meta / WABA)** | Clear documentation, assisted onboarding. Both flows (with/without WABA) supported; client owns assets, we obtain permissions. |
| **Compliance (spam, misuse)** | Strict opt-in, quality score monitoring, usage policies. |
| **Competition** | Differentiate with SMB simplicity, bookings + info, human support. |
| **Technical / operational** | Official WhatsApp API reduces stability concerns vs. reverse-engineered solutions; monitor delivery and session health. |
| **Regulatory (GDPR, Meta policy)** | Data processing agreements, privacy policy, alignment with Meta ToS. |
