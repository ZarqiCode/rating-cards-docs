---
product: rating.cards
layer: architecture
domains: [admin, multi-location]
auto_sync: true
last_verified: 2026-05-20
---

# System overview

**Audience:** Developers, AI assistants.

## Domains

| Domain | Purpose |
|--------|---------|
| **rating.cards** | Landing page and marketing site |
| **app.rating.cards** | Web application — dashboard, onboarding, settings, admin, device management |

## Stack

- **Frontend:** React + Vite + TypeScript
- **Backend:** Supabase (Postgres, Auth, Edge Functions on Deno)
- **Payments:** Stripe (checkout, webhooks, customer portal)
- **SMS:** Twilio (review bot, OTP, manager verification)
- **Email:** Resend (weekly pulse, alerts)
- **Reviews:** Google Business Profile APIs (OAuth, sync, reply)

## Auth model

- **CEO / account owner:** Supabase auth; full app access
- **Managers:** No app login — verify via public `/verify-sms/:token` and operate entirely over SMS
- **Admin:** Separate admin route and service-role-backed edge functions for internal provisioning

## Storefront data model (summary)

- **Business** — organisation; links to user account after subscription
- **Storefront / location** — GBP location imported per business; soft-deactivate in v1
- **Devices** — organisation-scoped in v1; `public_id`, `status`, `destination_url`
- **Reviews** — synced per storefront from GBP
- **Manager verification** — per-storefront phone lines with async OTP flow

See [product/multi-location.md](../product/multi-location.md) for v1 boundaries.

## Edge functions (index)

| Function | Role |
|----------|------|
| `card-redirect` | Device tap → redirect + log |
| `activate-device`, `admin-devices`, `admin-businesses`, `bulk-provision` | Admin provisioning |
| `google-auth-url`, `google-callback`, `google-import-reviews` | GBP OAuth and location import |
| `sync-reviews` | Scheduled/manual review sync |
| `send-review-sms`, `handle-sms-reply`, `approve-reply`, `respond-to-review` | SMS review workflow |
| `create-sms-verification`, `verify-sms-public`, `send-otp-sms`, `verify-otp-sms` | Manager phone verification |
| `create-checkout-session`, `claim-checkout-session`, `stripe-webhook`, `create-portal-session` | Billing |
| `send-weekly-pulse`, `send-negative-alert` | Email |
| `retry-failed-replies`, `handle-unsubscribe` | Reliability / compliance |

Runbooks: [runbooks/](../runbooks/).

## Key routes (app)

CEO routes `/dashboard` and `/settings` share an **AppLayout** shell: header with primary nav, user menu, and optional Admin button.

| Route | Purpose |
|-------|---------|
| `/onboarding` | CEO guided setup |
| `/dashboard` | Overview — org-default proof metrics, alerts, system status (AppLayout) |
| `/settings` | Billing, Locations, Notifications (AppLayout) |
| `/verify-sms/:token` | Public manager verification |
| `/setup/:publicId` | Legacy unclaimed device setup |
| `/admin` | Internal admin panel (separate dark shell) |

## Changelog

- 2026-05-21: Overview page redesign at `/dashboard` (nav label Overview)
- 2026-05-19: Initial architecture doc from project-brief Domains + codebase
