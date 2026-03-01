# Kalabot Website

## 1. Problem

- Small businesses (salons, restaurants, clinics, service providers) waste time answering calls and WhatsApp for **bookings** and **repeated questions** (hours, prices, services, location).
- Many SMBs have no website or cannot keep one updated — clients expect instant answers anyway.
- Existing solutions are either too complex (heavy Meta onboarding, technical setup) or too generic (multichannel CRMs not designed for SMB needs).
- Result: lost bookings, frustrated clients, overworked staff.


## 2. Solution

- **WhatsApp-first business assistant** focused on two pillars:
  1. **Bookings**: 24/7 scheduling with Google Calendar sync.
  2. **Conversational Business Info**: clients ask questions via WhatsApp (hours, prices, services, FAQs); the bot answers from content the owner provides — effectively replacing the need for a website for common inquiries.

- **Official WhatsApp Business API**: quality, resilience, and trust. No reverse‑engineered protocols.

- **Simple onboarding**: registration → payment → connect calendar → upload your info (price list, FAQs) → start receiving clients.
- **Dashboard**: view all conversations, bookings, and client interactions in one place.
- **Human support**: onboarding and live help tuned for non-technical SMB owners.


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


#### Onboarding
- Registration
- Payment
- WhatsApp Business API: connect your existing WhatsApp Business number or get a new one through our provisioning
- Google Calendar integration: link existing calendar in two clicks
- **Business Info setup**: upload price list (photo or document); AI (Kimi k2.5 vision) extracts services, prices, durations. Optionally add FAQs, hours, location, etc.
- QR link or WhatsApp link: share to start receiving bookings and questions

#### Main Objectives

**A. End Customer Experience (WhatsApp user)**
- **Natural feel**: the customer feels they are talking to an efficient human assistant. No "Press 1" menus.
- **24/7 availability**: book and get answers at any time.
- **Reduced no-shows**: reminders and confirmation flows with clear, persuasive language.
- **Instant answers**: common questions (hours, prices, services) answered immediately from owner content.

**B. Professional Experience (business owner)**
- **Instant onboarding**: calendar + WhatsApp + business info in minutes.
- **Zero maintenance**: manage everything from the calendar. Move an appointment → the bot notifies the client.
- **No website needed**: clients get info through WhatsApp; the owner provides content once.

#### Technical Architecture

**3.1 Connectivity Layer**
- **WhatsApp Business API**: official Meta API for reliability, compliance, and scalability.
- **Session and number lifecycle**: each client gets a dedicated WhatsApp Business number; auth and delivery handled through the official stack.

**3.2 Intelligence Layer**
- **Orchestrator**: agent swarm using Kimi k2.5 for cost efficiency.
- **Triage agent**: classifies incoming messages — new booking, cancellation, price question, FAQ, other.
- **Booking agent**: natural language → Google Calendar queries (e.g. "Tuesday afternoon" → `timeMin`, `timeMax`).
- **Info agent**: answers client questions using owner-provided content (services, prices, hours, FAQs).
- **Closing agent**: drafts final responses and seeks explicit confirmation when needed.

**3.3 Calendar Integration**
- **Google Calendar API**: real-time availability via `freeBusy`.
- **Buffer logic**: configurable cleanup time between appointments to avoid overlap.

#### Service Definition and Business Logic

The system supports a service matrix configured by the owner:

| Service            | Estimated Duration | AI Requirements                                |
| ------------------ | ------------------ | ---------------------------------------------- |
| Quick cut          | 20 min             | Immediate confirmation                         |
| Colour and styling | 120 min            | Verify long slots; avoid close to closing      |
| Physio (session)   | 60 min             | Ask if first visit (needs more time for intake) |


## 3. Target Market

- **Geography**: Spain first (WhatsApp penetration ~90%, low SMB automation adoption), then Europe, LATAM, India, Brazil.
- **Segments**: booking-heavy SMBs — salons, clinics, restaurants, wellness studios, consultants.
- **Ideal profile**: businesses that get lots of "what are your hours?" and "how much is X?" — and want to offload both bookings and info to WhatsApp.


## 4. Unique Value Proposition

- "Your **24/7 receptionist** on WhatsApp — bookings and answers to client questions, without a website."
- **WhatsApp-first**: no multichannel bloat; one channel done well.
- **Official API**: quality, resilience, trust.
- **Conversational Business Info**: replace a static website with a bot that answers from your content.
- **Vertical templates**: ready-to-use flows, no flow builder required.


## 5. Competitors

### Booking products
- **Booksy**: 29€ base + 20€ per extra calendar
- **Treatwell**: commission per booking from their marketplace. Kalabot is fixed fee — better for businesses with an existing client base.
- **Calendly**: 12€ personal (up to 6 calendars); 20€/person for teams

### WhatsApp Business API products
- **360dialog**: developer-focused, no SMB dashboard
- **Tidio, Respond.io**: multichannel, not WhatsApp-first, limited SMB flows
- **WATI, Callbell, Gupshup**: inboxes and automation, but heavier onboarding, less SMB simplicity
- **Opportunity**: simple onboarding + bookings + conversational info + strong UX on the official API


## 6. Business Model (PENDING)

- **Pricing (monthly subscription)**
  - BASE: 15€ (1 phone number + calendar)
  - Additional calendar: +10€ per calendar
  - Smart reminders: +10€ per calendar (cost: 16 books per day x 21 days x 0.0166€ = 5.5776€)
  - Smart fill & recovery: +10€ per 100 messages (cost: 100 messages x 0.0509€ = 5.09€)

- **Referral program**
  - 5€ per referral (while the referred client continues paying)
  - The client must pay at least 5€

- **Costs**
  - Telnyx (numbers) + AI + server: ~2.90€
  - Stripe: ~0.65€ (1.5% + 0.25€ on 19.90€)
  - Invoicing (Quaderno/Holded): ~0.30€
  - Admin / self-employed overhead: ~1.50€
  - **Total costs**: ~5.35€


## 7. Go-to-Market

- **Pilot phase**: personal network — 3 salons, 1 restaurant, 1 consultant (or similar booking-heavy SMBs). Goal: validate product fit and willingness to pay.
- **Acquisition (Spain first)**: direct outreach, referrals, ads (to be defined).
- **Expansion**: WhatsApp-heavy markets (Spain → LATAM → EU / India).


## 8. Risks & Mitigation (PENDING)

- **Onboarding friction (Meta / WABA requirements)**: clear documentation, assisted onboarding; provision numbers where possible.
- **Compliance (spam, misuse)**: strict opt-in, quality score monitoring, usage policies.
- **Competition**: differentiate with SMB simplicity, bookings + info, human support.
- **Technical / operational**: official WhatsApp API reduces stability concerns vs. reverse‑engineered solutions; monitor delivery and session health.
- **Regulatory (GDPR, Meta policy)**: data processing agreements, privacy policy, alignment with Meta ToS.
