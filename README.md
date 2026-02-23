# Kalabot Website

## 1. Problem

- Small businesses (salons, restaurants, clinics, service providers, professionals) waste time answering calls/WhatsApp manually for **bookings, inquiries, and service requests**.
- Existing solutions are either:
  - Too **complex** (require Meta/Facebook onboarding, technical setup).
  - Too **generic** (multichannel CRMs not designed for SMB needs).
- Result: lost leads, missed bookings, overworked staff.


## 2. Solution

- **Frictionless WhatsApp automation platform**: pay → get a WhatsApp Business number → start. No WABA, just web version with Baileys.
- Industry-specific **templates**:
  - Booking flow (Google Calendar sync).
  - Lead qualification (lawyers, architects, consultants).
  - Service dispatch (plumbers, cleaners).
- **Dashboard**: see all conversations + bookings/leads/jobs in one place.
- **Top support**: onboarding + live help, tuned for non-technical SMB owners.

### 2.1 Booking Product

We will start with this solution.

#### Onboarding
- Registration
- Payment
  - Kalabot: Get new phone number
- Download Whatsapp business
- Whatsapp activation
  - Follow instalation
  - Use new phone number
  - Kalabot: Receive the SMS code, send to the client
  - Fill the code in the app
- Google Calendar integration: Link existing calendar in two clicks.
- Assisted setup: The professional uploads a photo of their current price list. AI (using Kimi k2.5 vision) extracts services, prices, and durations automatically.
- QR link: Share to start receiving bookings

#### Main Objectives

**A. End Customer Experience (WhatsApp user)**

- **Natural feel**: The customer should feel they are talking to a very efficient human assistant. No "Press 1" menus.
- **24/7 availability**: Ability to book around the clock without the professional having to touch their phone.
- **Reduce no-shows**: Automatic reminder and confirmation system with persuasive language.

**B. Professional Experience (Hairdresser, Physio)**

- **Instant onboarding**: Google Calendar integration in two clicks and connection of a dedicated WhatsApp number to separate personal from professional life.
- **Zero maintenance**: The professional only looks at their calendar. If they need to move an appointment, they do it in Google Calendar and the bot notifies the client and renegotiates the slot.
- **Voice/chat control**: The professional can "talk" to their own bot to block hours or add manual appointments while working.

#### Technical Architecture

**3.1 Connectivity Layer ("The Body")**

- **WhatsApp engine**: Baileys. As a reverse-engineered WhatsApp Web protocol library, it allows much higher instance density per server than Puppeteer.
- **Session isolation**: Each client has a lightweight container or isolated process managing their WhatsApp auth state, so if one session drops, it does not affect the rest of the network.

**3.2 Intelligence Layer ("The Brain")**

- **Orchestrator**: Agent Swarm using Kimi k2.5 SDK for cost efficiency.
- **Triage agent**: Analyzes incoming messages. Is it a new appointment? A cancellation? A price question?
- **Calendar agent**: Translates natural language into Google API queries (e.g. "Tuesday afternoon" → `timeMin: 2026-02-24T16:00:00Z`).
- **Closing agent**: Drafts the final response seeking explicit confirmation.

**3.3 Calendar Integration**

- **Google Calendar API**: Heavy use of `freeBusy` to check availability in real time.
- **Buffer logic**: The system automatically adds "cleanup time" between appointments (configurable by the professional) to avoid overlap.

#### Service Definition and Business Logic

The system supports a complex service matrix configured by the professional in their admin panel:

| Service            | Estimated Duration | AI Requirements                                                                 |
| ------------------ | ------------------ | ------------------------------------------------------------------------------- |
| Quick cut          | 20 min             | Immediate confirmation.                                                         |
| Colour and styling | 120 min            | Verify long slots; do not offer if close to closing.                            |
| Physio (session)   | 60 min             | Ask if first visit (requires more time for intake).                             |


## 3. Target Market

- **Initial geography**: Spain (WhatsApp penetration ~90%, but only ~27% SMB adoption).
- **Expansion**: Europe, LATAM, India, Brazil (similar SMB + WhatsApp-heavy markets).
- **Segments**:
  - Booking-based: salons, clinics, restaurants.
  - Lead-based: lawyers, architects, consultants.
  - Dispatch-based: plumbers, electricians, cleaners.


## 4. Unique Value Proposition

- "Your **24/7 receptionist** on WhatsApp — set up instantly, without tech headaches."
- Unlike competitors:
  - **Not multichannel bloat** (focus only on WhatsApp).
  - **Instant onboarding** (we provision number + templates for them).
  - **Vertical templates** (ready-to-use, no flow building needed).


## 5. Competitors

### 2.1 Booking Product
- **Booksy**: 29€ (base) + 20€ (per extra calendar)
- **Treatwell**: Cobran comisión por cada cita si viene de su marketplace. Kalabot es una cuota fija, lo que prefiere el peluquero que ya tiene su clientela fija en el pueblo.
- **Calendly**: 12€ (personal, up to 6 calendars). Or 20€/person for teams

### WABA products

- **360dialog** – frictionless WABA, but developer-focused, no SMB dashboard.
- **Tidio / Respond.io** – good UI, but multichannel, not WhatsApp-first, limited SMB flows.
- **WATI, Callbell, Gupshup** – offer inboxes, but lack easy onboarding + SMB simplicity.
- **Opportunity**: combine frictionless onboarding + templates + strong UX.


## 6. Business Model (PENDING)

- **Pricing (Monthly subscription)**
  - BASE: 19.90€ (1 phone number and calendar)
  - additional calendar: +9.90€ per calendar
  - smart Reminders: +10€ per calendar
  - smart Fill & Recovery: +10€ per calendar


- **Referals program**
  - Every referal: 5€ (while the referal continues paying)
  - The client must pay at least 5€

- **Costs**:
  - Zadarma + IA + Server ~4,90€
  - Stripe ~0,65 (€1.5% + 0.25€ sobre 19,90€)
  - Software Facturación ~0,30€ (Quaderno/Holded: 30€/mes entre 100 clientes)
  - Gestoría / Autónomos ~1,50€ (Prorrateando gastos de empresa/autónomo)
  - TOTAL GASTOS ~7,35

## 7. Go-to-Market

- **Pilot phase** (flexible client mix):
  - Personal network: 3 salons, 1 restaurant, 1 architect (or similar booking-heavy SMBs).
  - Goal: validate product fit and willingness to pay before broader launch.
- **Acquisition channels** (Spain first):
  - Direct outreach in SMB verticals.
  - Referals
  - Ads?
- **Expansion**: WhatsApp-heavy markets (Spain → LATAM → EU/India).


## 8. Risks & Mitigation (PENDING)

- **Onboarding friction (Meta/Facebook requirements)** → Use own WABA for MVP, migrate big clients to official WABA later.
- **Compliance (spam, misuse)** → Strict opt-in, monitor quality scores, enforce usage policies.
- **Competition from bigger players** → Differentiate with SMB simplicity + human support.
- **Technical / operational (Baileys stability, scaling)** → Session isolation per client; monitor session health; plan migration path to official API for high-volume clients.
- **Regulatory (GDPR, Meta policy)** → Data processing agreements, clear privacy policy; stay aligned with Meta terms of service.
