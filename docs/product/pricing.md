# Pricing

Single source of truth for Kalabot pricing. The canonical locked contract is in [v1 Product Contract](../architecture/v1-product-contract.md).

---

## Subscription Tiers (Monthly)

| Tier | Price | Calendar Limit |
|------|-------|----------------|
| Tier 1 | EUR 25 per calendar | Up to 2 calendars |
| Tier 2 | EUR 20 per calendar | Up to 10 calendars |
| Tier 3 | Contact sales | Custom |

**Strategic implication**: Most clients need ~2 calendars → ~€50/month per customer, doubling revenue per acquisition vs. single-calendar pricing.

---

## Referral Program

- EUR 5 per referral (while the referred client continues paying).
- Referred clients have a minimum price floor of EUR 10 per calendar.
- Add-ons are excluded from referral floor pricing.

---

## Add-ons

| Add-on | Price | Cost to Kalabot |
|--------|-------|-----------------|
| Smart fill & recovery | EUR 15 per 100 messages | 100 × €0.0509 = €5.09 |
| Marketing campaigns | EUR 15 per 100 messages | 100 × €0.0509 = €5.09 |

---

## Cost Structure (Per Client)

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

## Change Control

Any change to pricing tiers, referral terms, or add-on pricing requires explicit product + engineering approval and an update to [v1 Product Contract](../architecture/v1-product-contract.md) first.
