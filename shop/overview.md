---
product: rating.cards
layer: product
domains: [shop]
auto_sync: true
last_verified: 2026-06-25
---

# Shop overview

**Audience:** Founders, sales (Antonio), developers, AI assistants.

The **shop** is a separate front-end at **`shop.rating.cards`** (`rating-cards-shop` repo) for self-serve hardware purchase. It complements — does not replace — the sales-led path in [customer-journey.md](../product/customer-journey.md).

## Status (2026-05-30)

**Catalog, cart, Places picker, and Stripe checkout shipped** in `rating-cards-shop`: Next.js 16, product pages, `localStorage` cart with per-line Google Places data, hosted Stripe Checkout, success/cancel routes, and shop edge functions.

**Homepage (`/`)** — marketing landing with hero, benefit cards, client results, Autopilot teaser, FAQ, and closing CTA; includes the shoppable product grid (`#shop`) on the same page. `/hardware` and `/software` routes are planned; not shipped in this repo yet.

**Stripe Dashboard + secrets** — follow [runbooks/shop-stripe-setup.md](../runbooks/shop-stripe-setup.md) to configure Test/Live products, webhooks, and Supabase secrets before first live order.

## Scope (v1)

| Area | Behavior | Doc |
|------|----------|-----|
| Catalog | Cards, stickers, stands (posters when added); prices in app catalog | [product-capabilities.md](../product/product-capabilities.md) |
| Cart | Multi-SKU, quantities, persist across reloads; one Places selection per line | — |
| Business picker | Google Places Autocomplete (New API) on product pages — `placeId`, name, address, write-review URL per line | [integrations/google-gbp.md](../integrations/google-gbp.md) (app OAuth); shop uses Places only |
| Checkout | Stripe Checkout Sessions (hosted), tax + shipping, guest only | [payments.md](./payments.md) |
| After payment | Internal email to hello@rating.cards; Antonio fulfills from device stock | [fulfillment.md](./fulfillment.md) |
| Thank-you | Buyer confirmation; sets expectation for team follow-up | [fulfillment.md](./fulfillment.md) |

Autopilot subscription stays on the landing / app Stripe flow — see [integrations/stripe.md](../integrations/stripe.md).

## Out of scope (v1)

- Shop user accounts (guest checkout only)
- Subscription sales on the shop
- Buyer self-serve device activation (admin-provisioned per [admin-flows.md](../product/admin-flows.md))
- Admin panel order queue or `shop_orders` database
- Auto-create / link `businesses` rows from checkout
- Inventory tracking, abandoned-cart emails, discount codes

## Architecture summary

- **Subdomain:** `shop.rating.cards` (separate Vercel project)
- **Stack:** Next.js App Router, Tailwind v4, shadcn/ui, `@supabase/ssr`
- **Supabase:** shared project with `rating-cards-app`; shop deploys only `shop-*` edge functions (per-function deploy — never bulk-deploy)
- **Stripe:** shop lane separate from Autopilot — see [payments.md](./payments.md)

## Related docs

- [payments.md](./payments.md) — Stripe Checkout, edge functions, Dashboard setup
- [fulfillment.md](./fulfillment.md) — post-payment email and Antonio workflow
- [runbooks/shop-stripe-setup.md](../runbooks/shop-stripe-setup.md) — Stripe Dashboard step-by-step (products, webhooks, secrets)
- [customer-journey.md](../product/customer-journey.md) — sales-led path (still primary)
- [admin-flows.md](../product/admin-flows.md) — device setup from unclaimed stock
- [integrations/stripe.md](../integrations/stripe.md) — Autopilot billing only

## Changelog

- 2026-06-25: Homepage repurposed as marketing landing + embedded shop grid (benefit cards, pilot results, Autopilot teaser, FAQ, closing CTA).
- 2026-05-30: Stripe checkout, edge functions, and fulfillment idempotency shipped
- 2026-05-30: Split from `product/shop.md`; payments and fulfillment moved to dedicated shop docs
- 2026-05-27: GBP picker on product pages; catalog and cart shipped
- 2026-05-26: Initial shop placeholder
