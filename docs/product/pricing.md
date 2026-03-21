# Pricing

Single source of truth for Kalabot pricing. The canonical locked contract is in [v1 Product Contract](../architecture/v1-product-contract.md).

---

## Subscription (Monthly)

| Price | Calendars |
|-------|-----------|
| EUR 25 per calendar | Unlimited |

No tiers, no volume discount. One flat price per calendar, regardless of how many calendars the account has.

---

## Referral Program

- EUR 5 per referral per month (while the referred client continues paying).
- Add-ons are excluded from referral earnings.

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
