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

### Optional: 7-day free trial (card required)

To offer a trial before the first charge:

1. In the Payment Link editor, enable **Include a free trial** and set **7 days**.
2. Leave **Let customers start trial without payment method** **unchecked** — customers must enter a card; Stripe charges when the trial ends.
3. No app code changes beyond what is already deployed: pending checkout accepts `payment_status = no_payment_required`, and subscriptions are stored as **`trialing`** until the first invoice.

Do **not** enable no-card trials unless you also configure Stripe trial end behavior and email reminders — the app does not collect payment methods outside Checkout.

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

The `/checkout/success` route calls **resolve-checkout-session** to exchange `session_id` for a signed claim token, stores the token in `localStorage`, and routes the user to `/signup` (or claims immediately if already logged in).

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

Your existing `stripe-webhook` endpoint **does not need a new URL**. For Payment Link checkouts it now:

- `checkout.session.completed` (no `business_id`) — upserts `pending_checkouts`, sends setup email via Resend (once per checkout)
- `checkout.session.completed` (with `business_id`) — in-app re-subscribe; upserts `subscriptions` as before
- `customer.subscription.updated` / `customer.subscription.deleted` — works because `claim-checkout-session` writes `business_id` into Stripe subscription metadata at claim time
- `invoice.payment_failed` — keyed by `stripe_subscription_id`

Set the claim signing secret (generate a random 32+ byte string):

```bash
npx supabase secrets set CLAIM_CHECKOUT_SECRET=your_random_secret_here
```

Confirm `APP_ORIGIN` is set (used for setup email links), e.g. `https://app.rating.cards`.

To verify, in the [Webhooks dashboard](https://dashboard.stripe.com/webhooks):

1. Confirm there's an endpoint pointing at `https://<your-project>.supabase.co/functions/v1/stripe-webhook`.
2. Confirm it listens to at least these events:
   - `checkout.session.completed`
   - `customer.subscription.updated`
   - `customer.subscription.deleted`
   - `invoice.payment_failed`

No other changes are needed on the Stripe side.

**Edge Functions note:** Stripe-related functions (`stripe-webhook`, `resolve-checkout-session`, `claim-checkout-session`, `create-checkout-session`, `create-portal-session`) share a single client built from the official Stripe package with `target=denonext`, a pinned Stripe API version, `createFetchHttpClient()`, and—for webhook signature verification—`constructEventAsync` with `Stripe.createSubtleCryptoProvider()`. That layout matches Supabase’s recommended Stripe-on-Edge wiring and avoids Node-only crypto/microtask paths on Deno Deploy.

---

## 6. Test mode vs live mode

Payment Links are scoped to whichever Stripe mode you create them in.

- **Test mode**: the URL starts with `https://buy.stripe.com/test_...` and only accepts test cards. Use this for staging or any non-production landing page.
- **Live mode**: the URL starts with `https://buy.stripe.com/...` and charges real cards. Only use this on the production landing page.

The simplest workflow is to create **two Payment Links** (one in test, one in live) using the same product price (Stripe will surface both test and live versions of the price). Then point each environment's landing CTA at the matching link.

---

## Considerations and gotchas

- **localStorage required**: the pay-first flow stores a **claim token** (not raw `session_id`) in `localStorage` through signup/OAuth round-trips. If localStorage is disabled for this origin, the user can still recover via the setup email link.
- **Cross-device supported**: if a user pays on one device and signs up on another, the setup email to the **checkout address** contains `/signup?claim=…` with the same token.
- **Email mismatch is fine**: checkout email (receipts, setup email) can differ from signup email. Onboarding shows an informational billing notice when they differ.
- **Re-subscribe still uses the Edge Function**: churned users re-subscribing from `/settings` use `create-checkout-session` (auth required, `business_id` in metadata). Unaffected by Payment Link flow.
- **Idempotency**: `claim-checkout-session` returns `409 already_subscribed` if the user already has an active subscription; stale tokens are cleared safely. A subscription cannot be claimed by two different accounts.

---

## Quick verification checklist

After setting everything up, confirm:

- [ ] Stripe Payment Link exists in **live mode** and points at the correct $20/month price.
- [ ] The Payment Link's success URL is `https://app.rating.cards/checkout/success?session_id={CHECKOUT_SESSION_ID}` (note the literal braces).
- [ ] The landing page CTA href matches the Payment Link URL.
- [ ] The `STRIPE_PRICE_ID` secret in Supabase matches the price the Payment Link is selling.
- [ ] The webhook endpoint listens to `checkout.session.completed`, `customer.subscription.updated`, `customer.subscription.deleted`, and `invoice.payment_failed`.
- [ ] `CLAIM_CHECKOUT_SECRET` and `APP_ORIGIN` are set in Supabase secrets.
- [ ] A test purchase in **test mode** completes, redirects to `/checkout/success`, prompts for signup, and lands the user on `/onboarding` with a subscription row in the database (`active`, or **`trialing`** if the Payment Link includes a free trial).
- [ ] Abandoned-pay test: pay, close tab, open setup email on another device, complete signup via claim link.
