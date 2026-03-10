# Telnyx Number Provisioning — v1

## Context

Kalabot provisions one Telnyx phone number per client during onboarding (Step 3: Connect WhatsApp). That number is registered in Meta's Cloud API and serves as the exclusive WhatsApp channel for that business. This document defines the complete lifecycle of that number and error handling at each phase.

---

## 1. Number Lifecycle
```
AVAILABLE → RESERVED → PURCHASED (purchase_pending) → ACTIVE → [SUSPENDED] → DELETED → RECYCLED
```

### 1.1 Search and Reservation

**When:** during client onboarding, before launching Meta's Embedded Signup.

**API:**
- `GET /v2/available_phone_numbers?filter[country_code]=ES&filter[features][sms]=true`
- Returns numbers with `cost_information` (monthly recurring cost).
- Numbers with `filter[reservable]=true` can be held for 30 minutes before purchase, providing time to complete Embedded Signup.
- Use `filter[exclude_held_numbers]=true` to avoid numbers recently released from other accounts.

**Kalabot internal state:** `searching`

**Selection criteria:**
- Country: ES (Spain) for initial market
- Type: local (preferred) or mobile with SMS capability
- Verify `eligible_messaging_products` includes A2P

---

### 1.2 Purchase

**When:** immediately before launching Embedded Signup, once the client has confirmed they need a new number.

**API:**
- `POST /v2/number_orders`
```json
{
  "phone_numbers": [{ "phone_number": "+34XXXXXXXXX" }],
  "messaging_profile_id": "<kalabot_ES_profile_id>"
}
```

- The order creates a `number_order` with status `purchase_pending`.
- Telnyx sends a `number_order.updated` webhook when the order is processed.

**Telnyx number states:**

| State | Description |
|---|---|
| `1` — Purchase Pending | Order being processed |
| `2` — Purchase Failed | Purchase failed |
| `4` — Active | Number active and usable |
| `5` — Deleted | Number deleted |

**Kalabot internal state:** `provisioning`

**Activation time:**
- Numbers with `quickship=true`: active immediately (recommended for Spain whenever available).
- Standard numbers: up to 2 business days.

> **Note:** charges are final. No refunds for accidental purchases.

**Telnyx Inventory validation:**

With quickship=true: activation is essentially instant — these are pre-provisioned numbers ready to go immediately. For Spain this should be the common case, and I'd recommend always filtering for quickship numbers first. Without quickship: up to 2 business days. This would be a completely broken onboarding experience — a client clicks "Connect WhatsApp" and has to wait 2 days before continuing.

Spin up a Telnyx test account, search for filter[country_code]=ES&filter[quickship]=true and see what inventory looks like. If availability is thin, the pre-purchased pool idea mentioned in section 6 becomes much more important rather than just a nice-to-have.

---

### 1.3 Assignment to Messaging Profile

**When:** as soon as the order reaches `Active` status.

**API:**
- `PATCH /v2/phone_numbers/{id}/messaging`
```json
{
  "messaging_profile_id": "<kalabot_ES_profile_id>"
}
```

The Messaging Profile defines:
- Webhook URL for inbound SMS (used to receive Meta's verification code during Embedded Signup).
- Allowed destination countries (required since March 2024).
- Default Alphanumeric Sender ID (required for international sends from Spain).

**Kalabot internal state:** `assigned`

---

### 1.4 Meta Verification (Embedded Signup)

**When:** after assignment, during Embedded Signup.

Meta sends an SMS or call to the Telnyx number to verify ownership. Kalabot receives the inbound SMS via the Messaging Profile webhook, extracts the OTP code, and can automatically display it to the client in the onboarding dashboard.

**Kalabot internal state:** `pending_meta_verification`

---

### 1.5 Number Active in Production

Once Meta completes verification:
- The number is registered in Meta's Cloud API.
- The number **cannot** be used simultaneously in the WhatsApp Business mobile app.
- Kalabot records in its DB: `telnyx_number_id`, `phone_number_e164`, `waba_id`, `meta_phone_number_id`.

**Kalabot internal state:** `active`

---

### 1.6 Temporary Suspension

**When:** client pauses their subscription, payment dispute, or explicit request.

Options:
- Keep the purchased number (Telnyx MRC continues) — recommended for short pauses (<30 days).
- Delete the number (cost stops, but a 15-day recovery window exists for accidental deletions).

**Kalabot internal state:** `suspended`

---

### 1.7 Deletion and Recycling

**When:** client churn, prolonged non-payment, or explicit request.

**API:**
- `DELETE /v2/phone_numbers/{id}`
- Recurring charges stop immediately (prorated).
- Telnyx holds the number for up to 15 days (visible with `filter[exclude_held_numbers]=false`).
- After 15 days, the number returns to the general inventory.

**Required steps before deletion:**
1. Deregister the number in Meta Cloud API.
2. Confirm no active conversations in Kalabot.
3. Delete via Telnyx API.
4. Update Kalabot DB: `status = deleted`, `deleted_at = now()`.

**Kalabot internal state:** `deleted`

---

## 2. Kalabot Internal States

| State | Description |
|---|---|
| `searching` | Searching for an available Telnyx number |
| `provisioning` | Purchase order sent, awaiting confirmation |
| `assigned` | Number active on Telnyx and assigned to Messaging Profile |
| `pending_meta_verification` | Waiting for Meta to complete number verification |
| `active` | Number in production, WhatsApp operational |
| `suspended` | Client on pause; number retained |
| `deleted` | Number deleted on Telnyx and deregistered on Meta |

---

## 3. Relevant Telnyx Webhooks

| Event | When fired | Action in Kalabot |
|---|---|---|
| `number_order.updated` | Order state change (pending → active / failed) | Update internal state; continue onboarding if `active` |
| `message.received` | Inbound SMS to the number (e.g. Meta OTP code) | Extract OTP and display to client in the onboarding wizard |
| `number_order.failed` | Number purchase failed | Trigger retry flow or notify team |

**Recommended configuration:**
- Account-level webhook for `number_order.*` (notification profile in Mission Control).
- Messaging Profile webhook for `message.received` (to receive Meta's OTP).

---

## 4. Error Handling

### 4.1 Number Purchase Failure (`purchase_failed`)

**Possible causes:**
- Number no longer available at time of order (inventory race condition).
- Insufficient balance on the Telnyx account.
- Pending regulatory requirements (more likely in markets outside Spain).

**Response:**
1. Retry search and purchase with another number from the same pool (up to 3 automatic attempts).
2. If it persists: pause onboarding, notify support team, contact client for manual assistance.
3. Log the error with `purchase_failure_reason` from the Telnyx payload.

---

### 4.2 Activation Timeout Exceeded

**Threshold:** number stuck in `purchase_pending` for more than 10 minutes (expected: <2 minutes with quickship).

**Response:**
1. Internal alert to the team.
2. Query status via `GET /v2/number_orders/{id}`.
3. If the number has not activated within 30 minutes: contact support@telnyx.com with the `order_id`.
4. Offer the client the option to resume onboarding later.

---

### 4.3 Meta OTP Not Received

**Possible causes:**
- Messaging Profile webhook not configured or not responding.
- Number not fully active on the network when Meta sends the SMS.
- Meta's SMS arrives after the Embedded Signup timeout window.

**Response:**
1. Verify the number is in `active` state on Telnyx before launching Embedded Signup.
2. Implement an OTP wait timeout (max. 3 minutes); after that, show a "resend code" option.
3. If the SMS does not arrive: offer verification by phone call (Meta also supports voice call verification).
4. Log all inbound messages to the number during the onboarding window.

---

### 4.4 Number Disconnected While Active in Production

**Symptoms:** Meta delivery errors; no Telnyx webhook arriving.

**Response:**
1. Check number status: `GET /v2/phone_numbers/{id}`.
2. Check Messaging Profile health (`health.success_ratio` field).
3. If the number appears as `deleted` unexpectedly: check for accidental deletion. Telnyx allows recovery within the first 15 days.
4. If the issue persists: enable manual mode on the conversation (`bot_enabled = false`), notify the client.

---

### 4.5 Number Deletion Failure

**Possible causes:**
- Number was already deleted (idempotency: treat as success).
- Network or Telnyx API error.

**Response:**
1. Retry `DELETE` up to 3 times with exponential backoff.
2. If it persists: mark in DB as `pending_deletion` and queue for async retry.
3. Always deregister from Meta first, regardless of the Telnyx result.

---

## 5. Relationship with the Onboarding Flow

| Onboarding step | Telnyx action |
|---|---|
| Client confirms "I need a new number" | Search for available ES number → reserve |
| Client clicks "Connect WhatsApp" | Purchase number → assign to Messaging Profile |
| Meta launches Embedded Signup | Wait for inbound SMS/OTP → display code to client |
| Meta confirms verification | Record `meta_phone_number_id` → state `active` |
| Client cancels or churns | Deregister from Meta → delete number on Telnyx |

---

## 6. Operational Considerations

- **One number per client:** Kalabot provisions exactly one Telnyx number per active WABA. Numbers are never shared between clients.
- **Costs:** the number's MRC (~€1/month per business plan) activates from the moment of purchase, not from when Meta verifies. Minimise the time between purchase and verification.
- **Pre-purchased number pool:** to reduce onboarding latency at scale, consider maintaining a small pool of pre-purchased, profile-assigned ES numbers that are transferred to clients in real time. Requires expiry monitoring and a minimum stock threshold.
- **Required logs:** store `telnyx_number_id`, `order_id`, `phone_number_e164`, timestamps for each state transition, and `purchase_failure_reason` where applicable.
