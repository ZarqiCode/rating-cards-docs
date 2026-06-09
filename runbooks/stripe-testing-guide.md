# Stripe Subscription — Testing Guide

## Prerequisites

1. **Stripe Account** — Create a Stripe account at [dashboard.stripe.com](https://dashboard.stripe.com) if you don't have one.
2. **Stripe CLI** — Install for local webhook testing: [stripe.com/docs/stripe-cli](https://stripe.com/docs/stripe-cli)

## Step 1: Stripe Dashboard Setup (Test Mode)

Make sure you're in **Test Mode** (toggle in the Stripe Dashboard header).

### Create the Product and Price

1. Go to **Products** > **Add product**
2. Name: `Autopilot Subscription`
3. Price: `$20.00 USD`, **Recurring**, **Monthly**
4. Save and note the **Price ID** (starts with `price_`)

### Configure Customer Portal

1. Go to **Settings** > **Billing** > **Customer portal**
2. Enable:
  - **Invoices**: allow customers to view invoices
  - **Payment methods**: allow customers to update payment methods
  - **Subscriptions**: allow customers to cancel subscriptions
  - Set cancellation to "Cancel at end of billing period"
3. Save

### Create Webhook Endpoint

1. Go to **Developers** > **Webhooks** > **Add endpoint**
2. URL: `https://<your-project>.supabase.co/functions/v1/stripe-webhook`
3. Events to listen for:
  - `checkout.session.completed`
  - `customer.subscription.updated`
  - `customer.subscription.deleted`
  - `invoice.payment_failed`
4. Save and note the **Signing secret** (starts with `whsec_`)

## Step 2: Set Environment Variables

Run these commands (replace with your actual values):

```bash
npx supabase secrets set STRIPE_SECRET_KEY=sk_test_your_test_key_here
npx supabase secrets set STRIPE_WEBHOOK_SECRET=whsec_your_signing_secret_here
npx supabase secrets set STRIPE_PRICE_ID=price_your_price_id_here
npx supabase secrets set APP_ORIGIN=https://your-app.vercel.app
npx supabase secrets set CLAIM_CHECKOUT_SECRET=your_random_secret_here
```

## Step 3: Run Database Migration

Run `supabase/migrations/stripe_subscriptions.sql` and `supabase/migrations/pending_checkouts.sql` in the Supabase SQL Editor.

## Step 4: Deploy Edge Functions

```bash
npx supabase functions deploy create-checkout-session
npx supabase functions deploy resolve-checkout-session
npx supabase functions deploy claim-checkout-session
npx supabase functions deploy stripe-webhook
npx supabase functions deploy create-portal-session
npx supabase functions deploy admin-subscriptions
npx supabase functions deploy sync-reviews
npx supabase functions deploy send-weekly-pulse
npx supabase functions deploy send-negative-alert
npx supabase functions deploy activate-device
```

## Step 5: End-to-End Testing

### Test 1: Happy Path — Successful Subscription

1. Sign in to the app
2. Navigate to `/billing`
3. Verify: You see the subscription pitch page with pricing, features, FAQ
4. Click **"Subscribe to Autopilot"**
5. You're redirected to Stripe Checkout
6. Use test card: `4242 4242 4242 4242`, any future expiry, any CVC
7. Complete checkout
8. You're redirected to `/billing?success=true`
9. Verify: Success banner appears
10. Verify in Supabase: `subscriptions` table has a row with `status = 'active'`
11. Navigate to `/dashboard`
12. Verify: Full Autopilot dashboard is shown (stats, features, etc.)

### Test 2: Checkout Canceled

1. Navigate to `/billing` while not subscribed
2. Click **"Subscribe to Autopilot"**
3. On Stripe Checkout, click the back/cancel button
4. You're redirected to `/billing?canceled=true`
5. Verify: "Checkout was canceled" message appears

### Test 3: Failed Payment

1. Navigate to `/billing` while not subscribed
2. Click **"Subscribe to Autopilot"**
3. Use test card: `4000 0000 0000 0341` (attaches but fails on charge)
4. Complete checkout
5. Wait for webhook to fire
6. Verify in Supabase: `subscriptions.status` becomes `past_due`
7. Verify in app: Dashboard shows yellow warning banner, Billing page shows payment failure notice

### Test 4: Subscription Cancellation

1. While subscribed, navigate to `/billing`
2. Click **"Manage subscription"**
3. In Stripe Customer Portal, click **Cancel subscription**
4. Confirm cancellation
5. You're redirected back to `/billing`
6. Verify in Supabase: `subscriptions.cancel_at_period_end = true`
7. Verify in app: Billing page shows "Cancellation scheduled" with end date

### Test 5: Subscription Deletion (Period End)

Use Stripe CLI to simulate period end:

```bash
stripe trigger customer.subscription.deleted
```

Or in Stripe Dashboard, find the subscription and cancel immediately.

Verify in Supabase: `subscriptions.status = 'canceled'`
Verify in app: Dashboard shows locked Autopilot panel, Billing shows re-subscribe option

### Test 6: Portal — Update Payment Method

1. While subscribed, navigate to `/billing`
2. Click **"Manage subscription"**
3. In portal, update payment method
4. Verify redirect back to `/billing`

### Test 7: Admin Panel

1. Sign in as admin user
2. Navigate to `/admin`
3. Click **Subscriptions** tab
4. Verify: All businesses are listed with correct subscription statuses
5. Verify: "View in Stripe" links work for subscribed businesses

### Test 8: Edge Function Gating

1. Create two businesses: one subscribed, one not
2. Trigger `sync-reviews` (via cron or manual call)
3. Verify: Only the subscribed business's reviews are synced
4. Trigger `send-weekly-pulse`
5. Verify: Only the subscribed business receives the email
6. Insert a low-rating review for the unsubscribed business
7. Verify: `send-negative-alert` skips it with `"reason": "Business does not have an active subscription"`

### Test 9: Payment Link — pay-first signup

1. Complete checkout via Payment Link (see [stripe-payment-link-setup.md](./stripe-payment-link-setup.md))
2. Verify redirect to `/checkout/success` → signup with test account
3. Verify `pending_checkouts` row exists, then `status = claimed` after signup
4. Verify setup email received at checkout address (Resend dashboard)
5. **Cross-device:** pay, close browser, open setup email link on another device → signup → onboarding

### Test 10: Payment Link — double claim rejected

1. Complete Payment Link checkout and note claim token (or use setup email link)
2. Sign up as User A and complete claim
3. Sign out; attempt same claim link as User B
4. Verify claim fails with subscription already linked; User B lands on `/setup-error`

## Stripe Test Cards Reference


| Card Number           | Scenario                                         |
| --------------------- | ------------------------------------------------ |
| `4242 4242 4242 4242` | Succeeds                                         |
| `4000 0000 0000 0341` | Attaches to customer, but fails payment          |
| `4000 0000 0000 9995` | Always fails with `insufficient_funds`           |
| `4000 0000 0000 3220` | Requires 3D Secure authentication                |
| `4000 0025 0000 3155` | Requires 3DS2 authentication on all transactions |


All test cards use any future expiry date and any 3-digit CVC.

## Local Development with Stripe CLI

For testing webhooks locally:

```bash
# Forward Stripe events to your local Supabase
stripe listen --forward-to http://localhost:54321/functions/v1/stripe-webhook

# In another terminal, trigger test events
stripe trigger checkout.session.completed
stripe trigger customer.subscription.updated
stripe trigger customer.subscription.deleted
stripe trigger invoice.payment_failed
```

## Going Live

1. Switch Stripe Dashboard to **Live Mode**
2. Create the same Product/Price in Live Mode
3. Create webhook endpoint pointing to production URL
4. Update Supabase secrets with live keys:

```bash
npx supabase secrets set STRIPE_SECRET_KEY=sk_live_your_live_key
npx supabase secrets set STRIPE_WEBHOOK_SECRET=whsec_your_live_signing_secret
npx supabase secrets set STRIPE_PRICE_ID=price_your_live_price_id
```

1. Re-deploy edge functions
2. Test one real $20 charge, then refund it from Stripe Dashboard

