# Meta WhatsApp Provider Configuration

This document describes Meta's provider ecosystem, the differences between provider types, onboarding flows with and without an existing WABA, and the steps required to operate as a Business Solution.

For implementation decisions, the locked contract is defined in [v1 Product Contract](architecture/v1-product-contract.md).

---

## 1. Provider Types

Meta offers three provider tiers in the WhatsApp ecosystem:

v1 posture for Kalabot: operate as a Business Solution on official Cloud API. Tech Provider/BSP status is not required for v1 delivery.


| Type                                   | Description                                                                                                                                                    | Typical Use Case                                                                    |
| -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| **Business Solution**                  | Standard partner using Cloud API. You integrate with Meta's official APIs; the customer interacts with their own Business Manager during setup.                | SaaS products, booking platforms, CRM tools serving SMBs and mid-market             |
| **Tech Provider**                      | Advanced partner with deeper integration. You benefit from embedded signup flows, reduced friction, multi-tenant management, and official partner recognition. | B2B platforms serving many clients with higher volume and operational control needs |
| **BSP** *(Business Solution Provider)* | Top-tier partner program. Reserved for very large companies with dedicated Meta support and specialized terms.                                                 | Enterprises with millions of conversations and custom integrations                  |


---

## 2. Tech Provider vs. Business Solution

The distinction is **not technical at the API level** — both use the same Cloud API. The difference lies in **user experience, scalability, and partner standing**.

### Business Solution

- **Onboarding:** The customer interacts directly with their own Business Manager.
- **Setup steps:** Customer must create or verify their business, accept permissions, add payment method when required, and grant you partner access — or complete standard Embedded Signup.
- **Visibility:** Customers see Meta screens and understand they are connecting their own Meta account.
- **Automation:** You can automate parts of the flow, but Meta's UI remains central.
- **Control:** Limited — you depend on the customer completing Meta flows.

### Tech Provider

- **Onboarding:** More integrated and smooth.
- **Embedded Signup:** Deeper integration, fewer separate steps.
- **Permissions:** Lower friction in consent and approval flows.
- **Multi-tenancy:** Better tooling and control for managing many clients.
- **Recognition:** Appear as an approved provider in Meta's partner ecosystem.
- **Experience:** Feels like a single flow inside your product rather than several Meta hops.

### Summary


| Aspect               | Business Solution         | Tech Provider       |
| -------------------- | ------------------------- | ------------------- |
| API access           | Cloud API                 | Cloud API (same)    |
| Embedded Signup      | Standard                  | Deeper / smoother   |
| Meta visibility      | High (customer sees Meta) | Lower (more in-app) |
| Multi-client control | Basic                     | Advanced            |
| Partner status       | Standard                  | Recognized provider |


---

## 3. Business Solution Setup Steps

### 3.1 Complete Business Verification

- Meta may request company documents during verification.
- For Cloud API with **owned numbers** and avoiding stricter limits, business verification is required.
- Without verification: you can test with a personal account in testing mode, but you cannot scale or create production numbers for real customers.

### 3.2 Configure Your Meta App

- Create a WhatsApp Cloud API app in Meta Developer.
- Generate access token and webhook.
- Configure permissions: `business_management` and `whatsapp_business_management`.

### 3.3 Connect Number(s)

- **Own numbers:** Create a WABA in your Business Manager or use an existing number you control.
- Verify the number via SMS or phone call.
- Connect the number to your Cloud API app via webhook.

> **Note:** While a number is connected to the API, it cannot be used in the WhatsApp mobile app.

### 3.4 Client Onboarding

Two scenarios:


| Scenario                     | Process                                                                                                                   |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| **Client has existing WABA** | Client grants you access to their Business Manager → you connect their WABA to your app via Embedded Signup or Graph API. |
| **Client has no WABA**       | Client completes Embedded Signup → Meta creates Business Manager and WABA for them → you receive tokens and connect Cloud API. In both cases, WABA and Business Manager belong to the client. |


### 3.5 Backend & Dashboard

- Webhook to receive messages
- Queue for resilience and retries
- Human override / manual mode
- Bot logic, confirmations, and calendar integration

---

## 4. Onboarding Flows: With vs. Without WABA

### 4.1 Without WABA (Client Has No Existing Account)

Client does not yet have a WABA. You create everything via Embedded Signup.

#### User-Facing Flow


| Step | Action                                     |
| ---- | ------------------------------------------ |
| 1    | Click "Connect WhatsApp"                   |
| 2    | Log in with Facebook / Meta                |
| 3    | Enter basic business info (name, category) |
| 4    | Verify number (SMS or phone call)          |
| 5    | Accept terms                               |
| 6    | Done                                       |


**What the user does *not* see or do:**

- Does not see Business Manager
- Does not create anything manually
- Does not touch technical settings

#### Internal / Technical Flow

1. Embedded Signup is launched from your app.
2. If the user has no Business Manager:
  - Meta creates a basic one automatically.
3. A new WABA is created under that Business Manager.
4. The number is created or registered:
  - Can be a new number (if you purchased it beforehand).
  - Or an existing number not yet on WhatsApp.
5. Meta:
  - Registers the number in Cloud API.
  - Links it to your app.
  - Returns Phone Number ID and WABA ID.
6. Your backend receives:
  - Access token
  - WABA ID
  - Phone Number ID
  - Business ID
7. You configure:
  - Webhook
  - Basic templates (e.g., reminder, confirmation)
  - Internal usage limits
  - Payment method (your card if you pay for messaging)

---

### 4.2 With WABA (Client Has Existing Account)

Client already has a WABA in their Business Manager. You connect it to your app.

#### User-Facing Flow


| Step | Action                         |
| ---- | ------------------------------ |
| 1    | Click "Connect WhatsApp"       |
| 2    | Log in with Facebook / Meta    |
| 3    | Select their existing business |
| 4    | Confirm permissions            |
| 5    | Done                           |


Single flow — no navigating across multiple Meta sites.

#### Internal / Technical Flow

1. Embedded Signup detects that a WABA already exists in their Business Manager.
2. The user selects:
  - Their business
  - Their existing WABA
  - The number to connect
3. Meta:
  - Registers that number in Cloud API.
  - Unlinks it from the WhatsApp Business app if it was there.
4. Your backend receives:
  - WABA ID
  - Phone Number ID
  - Access token
5. You add your card as the payment method (if you pay for messaging on their behalf).

---

## 5. Business Checklist: Achieving Business Solution Status

From a **business and operations** perspective, these items are required to operate as a proper Business Solution:

### Meta / Meta Business

- Complete Meta business verification (submit requested documents)
- Create and configure the WhatsApp Cloud API app in Meta Developer
- Request and obtain `business_management` and `whatsapp_business_management` permissions
- Set up webhook URL and verify webhook subscription
- Generate and securely store app access token(s)

### Numbers and WABA

In both flows (Section 4), the **WABA and Business Manager belong to the client**. You only obtain permissions and connect the Cloud API.

Number contract for v1:

- One dedicated number per client (no shared numbers).
- If the client does not have an eligible number path, provision a dedicated Telnyx number during onboarding.

- Support both onboarding flows: with WABA and without WABA
- Obtain partner access / permissions to the client's Business Manager
- Connect the client's WABA and phone number(s) to your Cloud API app
- Verify numbers via SMS or phone call (client completes this during Embedded Signup)
- Connect numbers to Cloud API via webhook
- Define number lifecycle (provisioning, release, migration)

### Payment and Billing

- Add payment method in Business Manager (your card or client’s)
- Decide who pays for messaging (you vs. client)
- Monitor usage and costs to avoid surprises

### Product and UX

- Implement Embedded Signup flow (with and without existing WABA)
- Handle both flows: client has WABA vs. client does not
- Provide clear UI for “Connect WhatsApp” and permission steps
- Communicate to the user that the number cannot be used in the mobile app while on API

### Technical Backend

- Webhook endpoint to receive incoming messages and status updates
- Message queue for reliability and retries
- Human override / manual mode for support
- Bot logic, confirmations, and calendar (or booking) integration
- Template management (reminder, confirmation, etc.)

### Operations and Support

- Document internal flows (with/without WABA) for support and engineering
- Process for handling Meta policy or verification issues
- Plan for scaling and limits (verified vs. unverified business)
