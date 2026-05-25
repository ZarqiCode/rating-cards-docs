---
product: rating.cards
layer: integration
domains: [billing]
auto_sync: false
last_verified: 2026-05-19
---

# Stripe integration

**Audience:** Developers, AI assistants — conceptual layer. Ops detail: [runbooks/stripe-payment-link-setup.md](../runbooks/stripe-payment-link-setup.md), [runbooks/stripe-testing-guide.md](../runbooks/stripe-testing-guide.md).

## Model

- **Autopilot:** $20/month recurring, **flat per account** in v1 (not per storefront)
- **Physical products:** sold outside this app's Stripe flow (one-time, sales-led)

Per-location Stripe quantity is **deferred v1**. See [product/multi-location.md](../product/multi-location.md).

## Pay-before-signup

1. Owner completes Stripe Checkout from landing page (Payment Link or session)
2. Owner creates account at app.rating.cards
3. `claim-checkout-session` links Stripe customer/subscription to new user and existing business profile

No dashboard login required to subscribe.

## Webhooks

`stripe-webhook` handles subscription lifecycle: active, updated, canceled, payment failed. Updates local subscription state; gates app access via subscription check.

## Customer portal

Settings → Billing opens Stripe Customer Portal for payment method updates, cancellation, re-subscribe. Backed by `create-portal-session`.

## Changelog

- 2026-05-19: Initial integration doc; flat account billing for v1
