---
product: rating.cards
layer: product
domains: [shop]
auto_sync: true
last_verified: 2026-05-30
---

# Shop — Post-payment fulfillment

**Audience:** Antonio, founders, developers, AI assistants. Payments: [payments.md](./payments.md). Admin device setup: [admin-flows.md](../product/admin-flows.md).

## Principle

Shop v1 does **not** write orders to Postgres or surface an admin “orders” UI. Payment success triggers an **internal fulfillment email** to **hello@rating.cards**. Antonio configures devices from **unclaimed stock** already registered in admin, sets each device’s destination URL to the buyer’s Google review link, and ships.

This matches the existing ops model: devices are pre-registered, not claimed, until the team sets the dynamic URL and hands them off.

## Trigger

Only **`shop-stripe-webhook`** on Stripe event `checkout.session.completed` sends fulfillment mail — not the Next.js checkout button. The handler must be **idempotent** (same `session.id` must not send duplicate emails on webhook retries).

Implementation uses table **`shop.checkout_fulfillment`** (`session_id`, `emailed_at`) in a dedicated `shop` Postgres schema. Insert-on-conflict-skip before sending email.

## Internal email (hello@rating.cards)

Send via **Resend** (same provider as app weekly pulse — `RESEND_API_KEY` on the webhook function).

### Suggested subject

`[Shop order] {summary}` — e.g. quantity + top product + business name + paid total.

### Body sections

**Order**

- Stripe Checkout Session ID (support / refunds in Dashboard)
- Amount paid (from session)
- Customer email (Stripe `customer_details`)

**Ship to**

- Full shipping address from Checkout (`collected_information.shipping_details` on the session)

**Line items** (one block per cart line)

For each line from session metadata / expanded line items:

- Product name and slug
- Quantity and unit price (from Stripe line items on the retrieved Checkout Session)
- **Business name** (Places)
- **Formatted address** (Places)
- **Review / destination URL** — `https://search.google.com/local/writereview?placeid=...` (Antonio uses this as `destination_url` in admin)

Optional footer: link to Stripe Dashboard payment or session.

## Buyer-facing communication

| Channel | Content |
|---------|---------|
| **Stripe** | Payment receipt to email entered at Checkout |
| **Thank-you page** (`/checkout/success`) | Order received; team will configure devices and follow up (align with sales-led messaging in [customer-journey.md](../product/customer-journey.md)) |
| **Buyer email from rating.cards** | Not required in v1 (optional later: copy of fulfillment summary) |

## Antonio workflow (after email)

1. Read internal email — review URLs, SKUs, quantities, ship-to address.
2. Sign in to **app.rating.cards** admin.
3. **Select or create business** if needed (same as [admin-flows.md](../product/admin-flows.md)).
4. For each unit sold, pick an **unclaimed device** from stock, set **destination URL** to the line’s review link, activate.
5. Ship to the Checkout shipping address.

No automatic `businesses` row creation from the shop in v1. No linking Stripe Customer to a Supabase user.

## Multi-line / multi-venue carts

Each cart line can reference a **different** Places selection. The email must list **each line with its own review URL** so Antonio can configure the correct destination per device.

## Edge cases (v1)

| Case | Behavior |
|------|----------|
| Abandoned / failed payment | No fulfillment email |
| Webhook retry | Idempotency — at most one email per session |
| Refund | Manual in Stripe Dashboard; no automation |
| Cross-device checkout → signup | N/A — shop is guest-only, no `claim-checkout-session` |

## Out of scope (v1)

- `shop_orders` / admin order queue
- Auto-provision devices or businesses from webhook
- Inventory deduction in software
- Buyer order status portal

Upgrade path when volume grows: persist orders + admin list; optional buyer confirmation email template.

## Related docs

- [payments.md](./payments.md) — Checkout Session and webhook contract
- [overview.md](./overview.md) — shop boundaries
- [admin-flows.md](../product/admin-flows.md) — device activation steps

## Changelog

- 2026-05-30: Webhook retrieves full session for shipping (`collected_information`) and per-line unit prices
- 2026-05-30: Idempotency table documented as `shop.checkout_fulfillment` in `shop` schema
- 2026-05-30: Initial fulfillment spec — Resend to hello@rating.cards; no DB orders; Antonio unclaimed-stock workflow
- 2026-05-27: Prior plan (deferred): admin order UI and business auto-link from shop.md
