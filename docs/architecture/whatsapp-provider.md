# WhatsApp Provider Configuration

This document describes Meta's provider ecosystem, the differences between provider types, the steps required to operate as a Business Solution, and the Business Solution checklist.

For ownership rules, number policy, and the locked contract, see [v1 Product Contract](v1-product-contract.md).
For onboarding flows (with and without WABA), see [Onboarding](../onboarding.md).

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

### 3.4 Backend & Dashboard

- Webhook to receive messages
- Queue for resilience and retries
- Human override / manual mode
- Bot logic, confirmations, and calendar integration

---

## 4. Business Checklist: Achieving Business Solution Status

From a **business and operations** perspective, these items are required to operate as a proper Business Solution:

### Meta / Meta Business

- Complete Meta business verification (submit requested documents)
- Create and configure the WhatsApp Cloud API app in Meta Developer
- Request and obtain `business_management` and `whatsapp_business_management` permissions
- Set up webhook URL and verify webhook subscription
- Generate and securely store app access token(s)

### Numbers and WABA

The **WABA and Business Manager belong to the client** in both onboarding flows. Kalabot only obtains permissions and connects the Cloud API.

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

- Add payment method in Business Manager (your card or client's)
- Decide who pays for messaging (you vs. client)
- Monitor usage and costs to avoid surprises

### Product and UX

- Implement Embedded Signup flow (with and without existing WABA)
- Handle both flows: client has WABA vs. client does not
- Provide clear UI for "Connect WhatsApp" and permission steps
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

