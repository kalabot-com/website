# Client Onboarding Process Definition

This document defines the end-to-end onboarding process for Kalabot clients.

---

## 1. Overview and Objectives

### Goal

Enable a client to go from signup to receiving clients on WhatsApp in minutes.

### Target User

Non-technical SMB owners in appointment-based businesses, especially:

- Beauty and aesthetics
- Health and wellness
- Professional services

### Core Principles

- Low-friction onboarding
- Clear and guided UX
- Human support available when needed
- Business-owner control over data and channels

### Out of Scope

Restaurants are out of scope because table management and shift logic are materially different from appointment-based scheduling.

---

## 2. End-to-End Onboarding Sequence

Kalabot onboarding follows a locked six-stage order:

`Registration -> Payment -> Connect WhatsApp -> Connect Calendar -> Business Info -> Go Live`

This sequence is mandatory and is defined in the [v1 Product Contract](architecture/v1-product-contract.md).

---

## 3. Step-by-Step Definition

### Step 1: Registration

- Create account (email, password, business name)
- Select business segment for onboarding defaults:
  - Beauty and aesthetics
  - Health and wellness
  - Professional services

### Step 2: Payment

- Select subscription tier:
  - Tier 1: up to 2 calendars, EUR 25/calendar
  - Tier 2: up to 10 calendars, EUR 20/calendar
  - Tier 3: contact sales
- If referred, apply referral pricing floor:
  - Minimum EUR 10/calendar
  - Add-ons are not included
- Add payment method (Stripe)

### Step 3: Connect WhatsApp

Kalabot must support both onboarding flows. The path depends on whether the client already has a WABA.

#### 3A) Client Has Existing WABA

| Client Action | Kalabot/System Action |
| --- | --- |
| Click "Connect WhatsApp" | Launch Embedded Signup |
| Log in with Facebook/Meta | Await authorization |
| Select existing Business Manager, WABA | Validate selected assets |
| Select existing number or we provide one | Provision a dedicated number through Telnyx for this client when needed |
| Verify phone number (SMS or call) | If it is a new Telnyx number, receive OTP and guide the client to complete verification |
| Confirm permissions | Receive `WABA ID`, `Phone Number ID`, `Access token`; connect Cloud API |

#### 3B) Client Has No WABA

| Client Action | Kalabot/System Action |
| --- | --- |
| Click "Connect WhatsApp" | Launch Embedded Signup |
| Log in with Facebook/Meta | Await authorization |
| Confirm "I need a new number" | Provision a dedicated number through Telnyx for this client |
| Enter business info (name, category) | Await Meta provisioning |
| Verify phone number (SMS or call) | If it is a new Telnyx number, receive OTP and guide the client to complete verification |
| Accept terms | Meta creates Business Manager + WABA for the client; Kalabot receives tokens and connects Cloud API |

#### Critical Rules to Communicate

See [WhatsApp Provider Configuration](architecture/whatsapp-provider.md) for ownership rules, number policy, and provider constraints.

### Step 4: Connect Google Calendar

- Connect existing Google Calendar in a simple guided flow
- Support multiple calendars based on plan limits
- Use segment-based guidance for expected setup:
  - Beauty: commonly 1 to 5 calendars
  - Health: commonly 1 to 4 calendars
  - Professional services: commonly 1 to 8 calendars

### Step 5: Business Info Setup

- Upload price list (image or document)
- Use AI extraction (Kimi k2.5 vision) for:
  - Services
  - Prices
  - Estimated durations
- Add optional structured info:
  - FAQs
  - Business hours
  - Location
- Configure service matrix and duration rules
- Publish onboarding and go live

---

## 4. Post-Onboarding State

After onboarding is complete:

- WhatsApp bot is active for client conversations
- Booking and Q&A flows are available immediately
- Owner can use dashboard to monitor conversations and bookings
- Human override is available:
  - Manual response takeover
  - Per-conversation bot pause (`bot_enabled` true/false)

---

## 5. Segment-Specific Considerations

Use segment defaults to suggest calendar counts during setup (guidance, not hard limits). See [Target Customer Segments](product/target-segments.md) for the full breakdown by business type.

---

## 6. Assisted Onboarding and Support

- Provide live human support for non-technical business owners
- Offer clear help content for Meta permission and verification steps
- Define operational process for Meta policy/verification issues
- Escalate onboarding blockers quickly (verification, token permissions, number migration)

---

## 7. References

- [Product Overview](product/overview.md)
- [Target Customer Segments](product/target-segments.md)
- [WhatsApp Provider Configuration](architecture/whatsapp-provider.md)
- [v1 Product Contract (Locked)](architecture/v1-product-contract.md)
