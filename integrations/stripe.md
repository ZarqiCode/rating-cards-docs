---
product: rating.cards
layer: integration
domains: [billing]
auto_sync: false
last_verified: 2026-06-09
---

# Stripe integration

**Audience:** Developers, AI assistants — conceptual layer. Ops detail: [runbooks/stripe-payment-link-setup.md](../runbooks/stripe-payment-link-setup.md), [runbooks/stripe-testing-guide.md](../runbooks/stripe-testing-guide.md).

## Model

- **Autopilot:** $20/month recurring, **flat per account** in v1 (not per storefront)
- **Physical products (shop):** one-time Checkout Sessions on `shop.rating.cards` — see [shop/payments.md](../shop/payments.md). Sales-led path still supported; Autopilot unchanged here.

Per-location Stripe quantity is **deferred v1**. See [product/multi-location.md](../product/multi-location.md).

## Pay-before-signup (Payment Link)

1. Owner completes Stripe Checkout from landing page (Payment Link)
2. **Webhook** (`checkout.session.completed`) records a `pending_checkouts` row and sends a **setup email** to the checkout address (idempotent)
3. Browser redirect lands on `/checkout/success?session_id=…` → **resolve-checkout-session** exchanges the session for a signed **claim token** (stored in localStorage)
4. Owner creates account at app.rating.cards (any email or Google — **no match required** with checkout email)
5. **claim-checkout-session** validates the claim token, provisions `businesses` + `subscriptions`, marks the pending row claimed

Cross-device recovery: the setup email link (`/signup?claim=…`) carries the same claim token.

In-app re-subscribe (`create-checkout-session`) is unchanged: authenticated user, `business_id` in metadata, webhook provisions immediately.

## Claim tokens

- HMAC-signed tokens (`CLAIM_CHECKOUT_SECRET`); format `{checkout_session_id}.{signature}`
- Reusable until successfully claimed; 30-day expiry on pending rows
- One Stripe subscription → one business (enforced in claim + DB unique constraints)

## Webhooks

`stripe-webhook` handles subscription lifecycle: active, updated, canceled, payment failed. For Payment Link checkouts (no `business_id` in metadata), writes `pending_checkouts` + setup email instead of provisioning. In-app checkouts upsert `subscriptions` directly.

## Customer portal

Settings → Billing opens Stripe Customer Portal for payment method updates, cancellation, re-subscribe. Backed by `create-portal-session`. Onboarding shows a billing notice when checkout email ≠ signup email.

## Changelog

- 2026-06-09: Payment Link claim tokens, pending_checkouts, setup email recovery, resolve-checkout-session
- 2026-05-30: Note shop hardware lane in shop/payments.md (separate webhook)
- 2026-05-19: Initial integration doc; flat account billing for v1
