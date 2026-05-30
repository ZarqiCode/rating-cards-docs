---
product: rating.cards
layer: integration
domains: [shop, billing]
auto_sync: true
last_verified: 2026-05-30
---

# Shop — Stripe and payments

**Audience:** Developers, AI assistants. Product context: [overview.md](./overview.md). Fulfillment after payment: [fulfillment.md](./fulfillment.md).

## Model

- **Mode:** Stripe Checkout **Sessions** (`mode: payment`) — one-time hardware only
- **Not used for shop:** Payment Links (single-SKU), Customer Portal, subscription prices
- **Autopilot** remains on the app / landing flow — [integrations/stripe.md](../integrations/stripe.md)

Use the **same Stripe account** as Autopilot with a **separate webhook endpoint** and shop-owned edge functions so deploys in `rating-cards-shop` never touch `stripe-webhook` or subscription handlers in `rating-cards-app`.

## Checkout flow

1. Buyer builds cart in the shop (each line includes a Places-backed review URL — see [overview.md](./overview.md)).
2. Shop calls `shop-create-checkout-session` with a **server-validated** cart snapshot (never trust client subtotals alone).
3. Edge function creates a Checkout Session with:
   - `line_items` from Stripe **Price IDs** mapped to catalog slugs (`cards`, `stickers`, `stands`, …)
   - `shipping_address_collection` and configured **Shipping Rates**
   - **Stripe Tax** enabled (`automatic_tax`)
   - `success_url` / `cancel_url` under `SHOP_ORIGIN` (e.g. `/checkout/success?session_id={CHECKOUT_SESSION_ID}`, `/checkout/cancel`)
   - **Metadata** encoding cart lines and per-line Places fields for the fulfillment webhook (compact JSON; split across keys if needed — Stripe metadata value limit 500 characters per key)
4. Buyer completes **hosted** Stripe Checkout (email, shipping, payment).
5. Stripe sends `checkout.session.completed` to `shop-stripe-webhook` — see [fulfillment.md](./fulfillment.md).

## Edge functions (shop repo)

| Function | Role |
|----------|------|
| `shop-create-checkout-session` | Validate slugs/qty; map to Price IDs; create session; return `url` |
| `shop-stripe-webhook` | Verify signature; on completed checkout, trigger fulfillment (email); idempotent per `session.id` |
| `shop-health` | Plumbing / deploy check only |

Deploy by name from `rating-cards-shop` only. Do not bulk-deploy Supabase functions from either repo.

## Stripe Dashboard (shop lane)

Configure in **Test** and **Live** as needed:

| Area | Action |
|------|--------|
| **Products & Prices** | One Product per SKU; one-time Prices. Optional product metadata `slug: cards` for ops alignment with `src/lib/products.ts`. |
| **Shipping rates** | Rates used by Checkout (e.g. flat US). |
| **Tax** | Enable Stripe Tax for applicable regions. |
| **Webhooks** | Endpoint: `https://<project>.supabase.co/functions/v1/shop-stripe-webhook` |
| **Events** | At minimum `checkout.session.completed`; optional `checkout.session.expired`, `charge.refunded` later |

**Do not** point shop webhooks at `stripe-webhook` (subscription logic). The app webhook already skips Payment Link sessions without `business_id`; shop sessions must not update `subscriptions`.

## Supabase secrets

Set on the shop edge functions (not Next.js `.env.local`):

| Secret | Purpose |
|--------|---------|
| `STRIPE_SECRET_KEY` | Stripe API (may match app if same account) |
| `SHOP_STRIPE_WEBHOOK_SECRET` | Signing secret for **shop** webhook endpoint only |
| `SHOP_ORIGIN` | `https://shop.rating.cards` (or staging URL) for redirect URLs |
| Price map | Per-SKU Stripe Price IDs — e.g. `STRIPE_PRICE_ID_CARDS` or a single JSON map secret |

Reference list also appears in `rating-cards-shop/.env.local.example` (comments only).

## Server-side validation rules

- Reject unknown slugs and quantities outside allowed bounds
- Recompute totals from Stripe Price IDs, not from browser `subtotalCents`
- Attach Places data (`businessName`, `formattedAddress`, `reviewUrl`, `placeId`, …) to session metadata at session creation

## Client routes (planned / implementing)

| Route | Purpose |
|-------|---------|
| Cart → checkout action | POST cart to `shop-create-checkout-session`, redirect to returned `url` |
| `/checkout/success` | Thank-you; read `session_id`; clear cart |
| `/checkout/cancel` | Message; cart remains in `localStorage` |

## Testing

- Stripe **test mode** cards (e.g. `4242 4242 4242 4242`) — same approach as [runbooks/stripe-testing-guide.md](../runbooks/stripe-testing-guide.md)
- Stripe CLI: forward events to `shop-stripe-webhook` for local verification
- Confirm exactly **one** fulfillment email per `session.id` on webhook retry

## Deferred (not v1)

- Embedded Checkout / Payment Element on-domain
- Stripe as sole catalog source of truth (sync products from API)
- Merging hardware + Autopilot in one Checkout session
- Shop Customer Portal

## Related docs

- [fulfillment.md](./fulfillment.md) — webhook side effects after payment
- [overview.md](./overview.md) — shop scope and out-of-scope list
- [integrations/stripe.md](../integrations/stripe.md) — Autopilot only

## Changelog

- 2026-05-30: Initial shop payments spec — Checkout Sessions, shop-* functions, separate webhook
- 2026-05-19: Autopilot Stripe doc (separate lane) unchanged in integrations/stripe.md
