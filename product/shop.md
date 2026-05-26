---
product: rating.cards
layer: product
domains: [shop]
auto_sync: true
last_verified: 2026-05-26
---

# Shop (planned)

**Audience:** Founders, sales (Antonio), AI assistants — scope and intended behavior for the self-serve hardware shop.

The **shop** is a separate front-end at **`shop.rating.cards`** for self-serve hardware purchase. It complements — does not replace — the sales-led path described in [customer-journey.md](./customer-journey.md). Antonio still drives outreach and demos; the shop gives prospects a path to buy hardware online when they're ready, without waiting on a call.

## Status (2026-05-26)

**Scaffold only.** Repo `rating-cards-shop` exists with Next.js 16 + Tailwind v4 + shadcn/ui + `@supabase/ssr`, linked to the shared Supabase project, with one stub edge function (`shop-health`) and a placeholder home page. No product catalog, cart, checkout, or admin integration yet.

## Scope (planned)

- **Catalog** — browse the four product types from [product-capabilities.md](./product-capabilities.md): cards, stickers, stands, posters. Premium photography per SKU. (planned)
- **Cart** — add multiple SKUs, adjust quantities, persist across reloads. (planned)
- **Checkout** — Stripe Checkout for payment (Stripe Tax + Shipping Rates enabled), shipping address collection. One-time purchases only — Autopilot subscription stays on the landing page flow. (planned)
- **GBP picker (pre-checkout)** — buyer searches and selects their Google Business Profile location(s) so the rating.cards team can configure devices to point at the right Google Reviews destination before shipping. Reuses the `google-auth-url` / `google-callback` OAuth flow from [integrations/google-gbp.md](../integrations/google-gbp.md). (planned)
- **Admin handoff** — on `checkout.session.completed`, a `shop-stripe-webhook` edge function records the order, creates or links a `businesses` row, and surfaces it in the admin panel for the team to provision and ship devices per [admin-flows.md](./admin-flows.md). (planned)
- **Thank-you page** — order confirmation; messaging mirrors the sales-led journey: "Antonio will email you when your devices are set up and shipped." (planned)

## Out of scope (v1)

- User accounts on the shop (guest checkout only).
- Subscription sales (Autopilot stays on the landing page Stripe flow per [integrations/stripe.md](../integrations/stripe.md)).
- Self-serve device activation by the buyer (devices remain admin-provisioned per [admin-flows.md](./admin-flows.md)).
- Inventory tracking, abandoned-cart emails, discount codes.

## Architecture (planned)

- **Subdomain:** `shop.rating.cards` (separate Vercel project; DNS deferred).
- **Stack:** Next.js (App Router) + Tailwind v4 + shadcn/ui + `@supabase/ssr`. Brand tokens copied from the main app's `src/theme.ts` (light-mode subset).
- **Backend:** shares the existing Supabase project. Shop-owned edge functions live in `rating-cards-shop/supabase/functions/` (currently only `shop-health`); per-function deploy discipline — never bulk-deploy. Existing 27 functions in `rating-cards-app` are not touched from the shop repo.
- **Stripe:** new shop checkout uses a separate flow from Autopilot. Both Stripe customers can be reconciled later if needed.

## Related docs

- [customer-journey.md](./customer-journey.md) — sales-led path (still primary)
- [product-capabilities.md](./product-capabilities.md) — physical product list
- [admin-flows.md](./admin-flows.md) — provisioning path the shop hands off to
- [integrations/stripe.md](../integrations/stripe.md) — current Autopilot Stripe contract
- [integrations/google-gbp.md](../integrations/google-gbp.md) — GBP OAuth the picker reuses

## Changelog

- 2026-05-26: Initial placeholder doc. Scaffold-only status; full scope marked (planned).
