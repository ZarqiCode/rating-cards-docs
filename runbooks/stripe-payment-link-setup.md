# Stripe Payment Link Setup Guide

This guide walks through everything you need to do — once — to wire up the **pay-first** signup flow for rating.cards.

Customers click the CTA on the landing page, pay via a Stripe Payment Link, and only then create an account. The app provisions the business + subscription after signup via the `claim-checkout-session` Edge Function. This document explains how to configure Stripe for that flow.

For test card numbers and end-to-end testing, see [`stripe-testing-guide.md`](./stripe-testing-guide.md).

---

## What you'll set up

1. A Stripe product + recurring price (if you don't already have one)
2. A Stripe Payment Link that points at that price
3. The Payment Link's success URL pointing at `/checkout/success`
4. The landing page CTA href pointing at the Payment Link URL
5. A confirmation that the webhook endpoint is unchanged

When you finish, every CTA click on the landing page will route into Stripe Checkout, and the rating.cards app will provision the user's account after they sign up.

---

## 1. Confirm the Stripe product + price

You should already have a product configured (it's the same one the in-app `create-checkout-session` Edge Function uses for re-subscribes).

1. In the [Stripe Dashboard](https://dashboard.stripe.com/products), go to **Products**.
2. Confirm you have a product named something like **rating.cards Autopilot** with a recurring **$20/month** price.
3. Copy its **Price ID** (starts with `price_...`).
4. Confirm that this same Price ID is set as the `STRIPE_PRICE_ID` secret in Supabase. It must match — both the Payment Link and the in-app re-subscribe flow charge the same price:

   ```bash
   supabase secrets list | grep STRIPE_PRICE_ID
   ```

If the Price IDs ever diverge, churned users re-subscribing from `/settings` would be charged a different amount than new users from the landing page. Keep them in sync.

---

## 2. Create the Payment Link

1. In the [Stripe Dashboard](https://dashboard.stripe.com/payment-links), go to **Payment Links → New**.
2. **Product**: select the Autopilot price from step 1.
3. **Type**: confirm **Subscription** (auto-selected for recurring prices).
4. **Quantity**: fixed, 1.
5. Leave the rest of the defaults; the important options are in step 3 below.

Click **Create link** — Stripe will generate a URL like `https://buy.stripe.com/xxxxx`. Don't share it yet; we need to configure the success URL first.

---

## 3. Configure the success URL

This is the **single most important** setting for the pay-first flow. Without it, paying customers can't be linked back to their new accounts.

1. Open the Payment Link you just created (**Payment Links → your link → Edit**).
2. Under **After payment**, select **Don't show confirmation page → redirect to your website**.
3. Set the **URL** to:

   ```
   https://app.rating.cards/checkout/success?session_id={CHECKOUT_SESSION_ID}
   ```

   `{CHECKOUT_SESSION_ID}` is a literal placeholder — Stripe substitutes the real session id at redirect time. Keep the braces.

4. Save.

The `/checkout/success` route in the app reads `session_id` from the URL, stores it in `localStorage`, and routes the user to `/signup` (or claims immediately if they happened to already be logged in).

> **Note:** Payment Links do **not** support a `cancel_url`. If a customer abandons checkout, Stripe just leaves them on the Payment Link page — there is no redirect back to our app. This only affects the new-user flow; the in-app `create-checkout-session` (used by churned users from `/settings`) still has both `success_url` and `cancel_url` configured.

---

## 4. Point the landing page CTA at the Payment Link

The landing page lives outside this repository (at `rating.cards`). Update its primary CTA `href` to the Payment Link URL from step 2:

```html
<a href="https://buy.stripe.com/xxxxx" class="cta-button">
  Get Started
</a>
```

A few notes:

- Use a regular `<a>` tag, not a JS handler. Payment Links are static URLs and need no special integration.
- It's a good idea to use **`target="_blank" rel="noopener"`** if you want to keep the landing page in its own tab, but the typical pattern is a same-tab navigation since the user is committing to paying.
- The Payment Link works for both desktop and mobile — Stripe Checkout is fully responsive.

---

## 5. Confirm webhook configuration

Your existing `stripe-webhook` endpoint **does not need to change**. It already handles:

- `checkout.session.completed` — now gracefully skips when `business_id` is absent from metadata (Payment Link checkouts), since provisioning happens via `claim-checkout-session` after signup.
- `customer.subscription.updated` / `customer.subscription.deleted` — works because `claim-checkout-session` writes `business_id` into the Stripe **subscription** metadata at claim time.
- `invoice.payment_failed` — already keyed by `stripe_subscription_id`, no metadata needed.

To verify, in the [Webhooks dashboard](https://dashboard.stripe.com/webhooks):

1. Confirm there's an endpoint pointing at `https://<your-project>.supabase.co/functions/v1/stripe-webhook`.
2. Confirm it listens to at least these events:
   - `checkout.session.completed`
   - `customer.subscription.updated`
   - `customer.subscription.deleted`
   - `invoice.payment_failed`

No other changes are needed on the Stripe side.

**Edge Functions note:** Stripe-related functions (`stripe-webhook`, `claim-checkout-session`, `create-checkout-session`, `create-portal-session`) share a single client built from the official Stripe package with `target=denonext`, a pinned Stripe API version, `createFetchHttpClient()`, and—for webhook signature verification—`constructEventAsync` with `Stripe.createSubtleCryptoProvider()`. That layout matches Supabase’s recommended Stripe-on-Edge wiring and avoids Node-only crypto/microtask paths on Deno Deploy.

---

## 6. Test mode vs live mode

Payment Links are scoped to whichever Stripe mode you create them in.

- **Test mode**: the URL starts with `https://buy.stripe.com/test_...` and only accepts test cards. Use this for staging or any non-production landing page.
- **Live mode**: the URL starts with `https://buy.stripe.com/...` and charges real cards. Only use this on the production landing page.

The simplest workflow is to create **two Payment Links** (one in test, one in live) using the same product price (Stripe will surface both test and live versions of the price). Then point each environment's landing CTA at the matching link.

---

## Considerations and gotchas

- **localStorage required**: the pay-first flow relies on `localStorage` to carry the `session_id` from `/checkout/success` through signup. If a user has third-party cookies blocked in a way that also disables `localStorage` for this origin, the claim won't fire. In practice this is rare on first-party domains, but worth knowing.
- **Cross-device flow is not supported (yet)**: if a user pays on their phone and then signs up on their laptop, the `session_id` doesn't travel. The plan's out-of-scope section mentions an email-matching fallback as a future enhancement; for now, those users would land on `/setup-error` and need to contact support.
- **Email mismatch is fine**: the user enters their email on Stripe, but they can sign up later with any email or Google account. The claim function ties the subscription to whatever Supabase user holds the JWT when `claim-checkout-session` is called — the Stripe email isn't checked.
- **Re-subscribe still uses the Edge Function**: churned users re-subscribing from `/settings` are routed through `create-checkout-session` (which requires auth and embeds `business_id` in session metadata). That flow is unaffected by the Payment Link and continues to work as before.
- **Idempotency**: `claim-checkout-session` returns `409 already_subscribed` if the user already has an active subscription, so a stale `session_id` in `localStorage` after the first claim is safe — it just no-ops and gets cleared.

---

## Quick verification checklist

After setting everything up, confirm:

- [ ] Stripe Payment Link exists in **live mode** and points at the correct $20/month price.
- [ ] The Payment Link's success URL is `https://app.rating.cards/checkout/success?session_id={CHECKOUT_SESSION_ID}` (note the literal braces).
- [ ] The landing page CTA href matches the Payment Link URL.
- [ ] The `STRIPE_PRICE_ID` secret in Supabase matches the price the Payment Link is selling.
- [ ] The webhook endpoint listens to `checkout.session.completed`, `customer.subscription.updated`, `customer.subscription.deleted`, and `invoice.payment_failed`.
- [ ] A test purchase in **test mode** completes, redirects to `/checkout/success`, prompts for signup, and lands the user on `/onboarding` with a subscription row in the database.
