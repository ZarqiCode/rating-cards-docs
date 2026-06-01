---
product: rating.cards
layer: runbook
domains: [shop, billing]
auto_sync: false
last_verified: 2026-05-30
---

# Shop — Stripe Dashboard setup

**Audience:** Founders, developers (Lucas). Step-by-step Stripe Dashboard configuration for **shop.rating.cards** hardware checkout.

Product context: [shop/payments.md](../shop/payments.md). Post-payment email: [shop/fulfillment.md](../shop/fulfillment.md). Autopilot billing (separate lane): [integrations/stripe.md](../integrations/stripe.md), [stripe-testing-guide.md](./stripe-testing-guide.md).

## Before you start

1. Confirm you are in the **same Stripe account** that already runs Autopilot subscriptions.
2. Toggle **Test mode** ON (switch in the Stripe Dashboard header). Complete all steps below in Test first; repeat in Live when ready to accept real orders.
3. Install the [Stripe CLI](https://stripe.com/docs/stripe-cli) for local webhook testing.
4. Link Supabase CLI to the shared project: `npx supabase link --project-ref xxvzjrbglghofaobjiuc`

**Lane isolation:** Shop checkout uses a **second webhook endpoint** (`shop-stripe-webhook`). Do **not** add shop events to the Autopilot `stripe-webhook` endpoint or modify that endpoint’s signing secret.

---

## Step 1 — Create hardware products and prices

Go to **Product catalog** → **Add product** for each SKU below. Use **one-time** pricing (not recurring).

### 1a. Cards (`slug: cards`)


| Field                           | Value                                                                                                                                                                                                                                                           |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Name**                        | Google Review NFC & QR Card                                                                                                                                                                                                                                     |
| **Description**                 | Premium NFC and QR cards for tables — customers tap or scan to leave a Google review. Pocket-sized (85mm × 55mm), dual NFC + QR, durable PVC. Pre-configured with your Google Business Profile review URL before shipping. One-time purchase — no monthly fees. |
| **Image**                       | Optional — upload product photo from marketing assets                                                                                                                                                                                                           |
| **Pricing**                     | **One time** · **$29.95 USD**                                                                                                                                                                                                                                   |
| **Product metadata** (optional) | Key: `slug` · Value: `cards`                                                                                                                                                                                                                                    |


Save and copy the **Price ID** (starts with `price_`). You will need it for `STRIPE_PRICE_ID_CARDS`.

### 1b. Stickers (`slug: stickers`)


| Field                           | Value                                                                                                                                                                                                                                                      |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Name**                        | Google Review NFC & QR Sticker                                                                                                                                                                                                                             |
| **Description**                 | Adhesive NFC and QR stickers for surfaces, packaging, and receipts — review prompts anywhere. Weather-resistant adhesive, compact round format, dual NFC + QR. Pre-linked to your Google review page before dispatch. One-time purchase — no monthly fees. |
| **Pricing**                     | **One time** · **$19.95 USD**                                                                                                                                                                                                                              |
| **Product metadata** (optional) | Key: `slug` · Value: `stickers`                                                                                                                                                                                                                            |


Copy the **Price ID** → `STRIPE_PRICE_ID_STICKERS`.

### 1c. Stands (`slug: stands`)


| Field                           | Value                                                                                                                                                                                                                                                                         |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Name**                        | Google Review NFC & QR Stand                                                                                                                                                                                                                                                  |
| **Description**                 | Countertop and table stands for high-traffic spots near the register — tap or scan to review. Weighted base, large NFC tap zone and scannable QR, premium front-of-house finish. Programmed with your Google review URL before shipping. One-time purchase — no monthly fees. |
| **Pricing**                     | **One time** · **$42.95 USD**                                                                                                                                                                                                                                                 |
| **Product metadata** (optional) | Key: `slug` · Value: `stands`                                                                                                                                                                                                                                                 |


Copy the **Price ID** → `STRIPE_PRICE_ID_STANDS`.

**Checklist**

- Three products created (Cards, Stickers, Stands)
- All prices are **one-time**, not recurring
- Dollar amounts match the shop catalog ($29.95 / $19.95 / $42.95)
- Three Price IDs recorded

---

## Step 2 — Create a free US shipping rate

Go to **Product catalog** → **Shipping rates** → **Create shipping rate**.


| Field                 | Value                             |
| --------------------- | --------------------------------- |
| **Display name**      | Free standard shipping            |
| **Type**              | Fixed amount                      |
| **Amount**            | **$0.00 USD**                     |
| **Delivery estimate** | Optional — e.g. 3–5 business days |
| **Countries**         | United States only                |


Save and copy the **Shipping rate ID** (starts with `shr_`) → `STRIPE_SHIPPING_RATE_ID`.

To charge for shipping later: create a new shipping rate with the desired amount, update the `STRIPE_SHIPPING_RATE_ID` secret — no code change required.

---

## Step 3 — Enable Stripe Tax (US)

1. Go to **Settings** → **Tax**.
2. Enable **Stripe Tax**.
3. Add a **US** tax registration (follow Stripe’s wizard for your business location).
4. Confirm tax calculation is active for **United States** addresses.

Checkout uses `automatic_tax: { enabled: true }` with US-only shipping address collection.

---

## Step 4 — Create the shop webhook endpoint

Go to **Developers** → **Webhooks** → **Add endpoint**.


| Field            | Value                                                                       |
| ---------------- | --------------------------------------------------------------------------- |
| **Endpoint URL** | `https://xxvzjrbglghofaobjiuc.supabase.co/functions/v1/shop-stripe-webhook` |
| **Description**  | Shop hardware checkout (rating-cards-shop)                                  |
| **Events**       | `checkout.session.completed`                                                |


Do **not** reuse the Autopilot endpoint URL (`…/stripe-webhook`). This is a separate endpoint with its own signing secret.

After saving, open the endpoint → **Signing secret** → Reveal → copy `whsec_…` → `SHOP_STRIPE_WEBHOOK_SECRET`.

---

## Step 5 — Set Supabase Edge Function secrets (Test)

From the `rating-cards-shop` repo root (linked to Supabase):

```bash
npx supabase secrets set STRIPE_SECRET_KEY=sk_test_YOUR_KEY
npx supabase secrets set SHOP_STRIPE_WEBHOOK_SECRET=whsec_YOUR_SHOP_WEBHOOK_SECRET
npx supabase secrets set SHOP_ORIGIN=http://localhost:3000
npx supabase secrets set STRIPE_PRICE_ID_CARDS=price_YOUR_CARDS_ID
npx supabase secrets set STRIPE_PRICE_ID_STICKERS=price_YOUR_STICKERS_ID
npx supabase secrets set STRIPE_PRICE_ID_STANDS=price_YOUR_STANDS_ID
npx supabase secrets set STRIPE_SHIPPING_RATE_ID=shr_YOUR_SHIPPING_RATE_ID
```


| Secret                       | Notes                                                                                                 |
| ---------------------------- | ----------------------------------------------------------------------------------------------------- |
| `STRIPE_SECRET_KEY`          | Test secret key (`sk_test_…`). May already exist from Autopilot — same value is fine if same account. |
| `SHOP_STRIPE_WEBHOOK_SECRET` | From **Step 4** — shop endpoint only                                                                  |
| `SHOP_ORIGIN`                | `SHOP_ORIGIN` for local dev; change to `https://shop.rating.cards` for production                     |
| `STRIPE_PRICE_ID_`*          | From **Step 1**                                                                                       |
| `STRIPE_SHIPPING_RATE_ID`    | From **Step 2**                                                                                       |
| `RESEND_API_KEY`             | Reuse existing app secret if already set (fulfillment email)                                          |


List secrets to verify: `npx supabase secrets list`

---

## Step 6 — Apply the shop database migration

Run the migration in `supabase/migrations/*_shop_checkout_fulfillment.sql` via Supabase SQL Editor or:

```bash
npx supabase db push
```

This creates schema `shop` and table `shop.checkout_fulfillment` for webhook idempotency (prevents duplicate fulfillment emails).

**Expose the schema to PostgREST** (required — without this, `shop-stripe-webhook` returns `PGRST106 Invalid schema: shop`). Run in the SQL Editor:

```sql
GRANT USAGE ON SCHEMA shop TO postgres, service_role;

GRANT ALL ON ALL TABLES IN SCHEMA shop TO postgres, service_role;
GRANT ALL ON ALL SEQUENCES IN SCHEMA shop TO postgres, service_role;
GRANT ALL ON ALL ROUTINES IN SCHEMA shop TO postgres, service_role;

ALTER DEFAULT PRIVILEGES IN SCHEMA shop
  GRANT ALL ON TABLES TO postgres, service_role;

ALTER DEFAULT PRIVILEGES IN SCHEMA shop
  GRANT ALL ON SEQUENCES TO postgres, service_role;

ALTER DEFAULT PRIVILEGES IN SCHEMA shop
  GRANT ALL ON ROUTINES TO postgres, service_role;

ALTER ROLE authenticator SET pgrst.db_schemas = 'public, graphql_public, shop';

NOTIFY pgrst, 'reload config';
```

Alternatively (or additionally): Supabase Dashboard → **Project Settings** → **API** → **Exposed schemas** → add `shop` alongside `public`.

---

## Step 7 — Deploy shop edge functions

Deploy **by name only** from `rating-cards-shop` (never bulk-deploy):

```bash
npx supabase functions deploy shop-create-checkout-session
npx supabase functions deploy shop-stripe-webhook
```

Existing `shop-health` can be redeployed separately if needed.

---

## Step 8 — Local end-to-end test

### 8a. Start the shop

```bash
npm run dev
```

Ensure `.env.local` has `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY`.

### 8b. Forward webhooks (optional for local CLI testing)

If testing against deployed functions:

```bash
stripe listen --forward-to https://xxvzjrbglghofaobjiuc.supabase.co/functions/v1/shop-stripe-webhook
```

Use the CLI’s `whsec_…` only when forwarding locally; production uses the Dashboard signing secret from Step 4.

### 8c. Happy-path checkout

1. Open [http://localhost:3000](http://localhost:3000).
2. Add a product to cart with a Google Places business selected.
3. Click **Secure checkout**.
4. On Stripe Checkout, use test card **4242 4242 4242 4242**, any future expiry, any CVC, any US address.
5. Complete payment.

**Verify:**

- Redirect to `/checkout/success?session_id=cs_…`
- Cart cleared in the shop
- Exactly **one** email to **[hello@rating.cards](mailto:hello@rating.cards)** with order details, shipping address, and review URLs per line
- Stripe Dashboard → **Payments** shows the completed Checkout Session

### 8d. Cancel flow

1. Add items to cart → checkout → click **Back** on Stripe Checkout.
2. Verify redirect to `/checkout/cancel` and cart still has items.

### 8e. Webhook idempotency

In Stripe Dashboard → **Developers** → **Webhooks** → shop endpoint → select a `checkout.session.completed` event → **Resend**.

Verify: still only **one** fulfillment email for that `session.id`.

---

## Step 9 — Test → Live rollout

When ready for real orders, **repeat Steps 1–7 in Live mode**. This is not just changing `SHOP_ORIGIN`.

1. Toggle Stripe Dashboard to **Live mode**.
2. Recreate all three products/prices (Live Price IDs differ from Test).
3. Recreate the free shipping rate (Live `shr_…`).
4. Confirm Stripe Tax US registration in Live.
5. Add a **new Live webhook** endpoint (same URL path; Live signing secret differs) → update `SHOP_STRIPE_WEBHOOK_SECRET`.
6. Update all Supabase secrets with **Live** values:

```bash
npx supabase secrets set STRIPE_SECRET_KEY=sk_live_YOUR_KEY
npx supabase secrets set SHOP_STRIPE_WEBHOOK_SECRET=whsec_YOUR_LIVE_SHOP_SECRET
npx supabase secrets set SHOP_ORIGIN=https://shop.rating.cards
npx supabase secrets set STRIPE_PRICE_ID_CARDS=price_LIVE_CARDS
npx supabase secrets set STRIPE_PRICE_ID_STICKERS=price_LIVE_STICKERS
npx supabase secrets set STRIPE_PRICE_ID_STANDS=price_LIVE_STANDS
npx supabase secrets set STRIPE_SHIPPING_RATE_ID=shr_LIVE_SHIPPING
```

1. Redeploy edge functions (same code): `shop-create-checkout-session`, `shop-stripe-webhook`.
2. Smoke test on `https://shop.rating.cards` with a real card (or keep Test keys on a staging project if you split environments later).

---

## Quick reference — env secrets


| Secret                       | Test example              | Live example                |
| ---------------------------- | ------------------------- | --------------------------- |
| `STRIPE_SECRET_KEY`          | `sk_test_…`               | `sk_live_…`                 |
| `SHOP_STRIPE_WEBHOOK_SECRET` | `whsec_…` (Test endpoint) | `whsec_…` (Live endpoint)   |
| `SHOP_ORIGIN`                | `http://localhost:3000`   | `https://shop.rating.cards` |
| `STRIPE_PRICE_ID_CARDS`      | `price_…`                 | `price_…`                   |
| `STRIPE_PRICE_ID_STICKERS`   | `price_…`                 | `price_…`                   |
| `STRIPE_PRICE_ID_STANDS`     | `price_…`                 | `price_…`                   |
| `STRIPE_SHIPPING_RATE_ID`    | `shr_…`                   | `shr_…`                     |
| `RESEND_API_KEY`             | Same across modes         | Same across modes           |


Commented reference also appears in `rating-cards-shop/.env.local.example`.

---

## Troubleshooting


| Symptom                         | Check                                                                                                               |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Checkout button errors          | Edge function logs; Price ID secrets match Test/Live mode                                                           |
| No fulfillment email / empty `shop.checkout_fulfillment` | Webhook logs for `PGRST106` → run Step 6 expose-schema SQL; resend webhook event in Stripe |
| Webhook `PGRST106 Invalid schema: shop` | Run Step 6 SQL to expose `shop` schema to PostgREST |
| Duplicate emails                | Idempotency table; webhook should insert before sending                                                             |
| Wrong redirect URL              | `SHOP_ORIGIN` matches where the browser runs                                                                        |
| Autopilot subscription affected | Confirm shop sessions hit `shop-stripe-webhook`, not `stripe-webhook`                                               |


---

## Related docs

- [shop/payments.md](../shop/payments.md) — checkout session contract
- [shop/fulfillment.md](../shop/fulfillment.md) — email content and Antonio workflow
- [stripe-testing-guide.md](./stripe-testing-guide.md) — Autopilot subscription testing (do not mix with shop setup)

## Changelog

- 2026-05-30: Step 6 SQL to expose `shop` schema (PGRST106 fix); troubleshooting for empty idempotency table
- 2026-05-30: Initial shop Stripe Dashboard runbook — products, shipping, tax, webhook, secrets, Test→Live

