---
product: rating.cards
layer: integration
domains: [sms, managers]
auto_sync: false
last_verified: 2026-05-23
---

# Twilio integration

**Audience:** Developers, AI assistants. Ops detail: [runbooks/twilio-setup-guide.md](../runbooks/twilio-setup-guide.md).

## Roles

| Flow | Actor | Channel |
|------|-------|---------|
| Manager verification | Store manager (no app login) | SMS link → `/verify-sms/:token` + OTP |
| Review alerts | Verified manager per storefront | SMS/MMS with review + AI draft |
| Review approval | Verified manager | Two-way SMS conversation |

CEOs configure manager lines in Settings → Notifications; managers never log into the app.

## Manager verification

1. CEO enters manager phone + `sms_v2` consent checkbox (links to [Privacy Policy](https://www.rating.cards/privacy) and [Terms](https://www.rating.cards/terms)) during onboarding or Settings
2. `create-sms-verification` sends SMS with token link
3. Manager opens `/verify-sms/:token` (SMS program disclosure + legal links on page), completes OTP via `verify-sms-public` / `send-otp-sms` / `verify-otp-sms`
4. Storefront marked verified — AI drafts and alert SMS enabled for that location

Verification is **async** — onboarding can complete with lines pending.

## Review bot

- `send-review-sms` — notify manager on new review (verified storefronts only)
- `handle-sms-reply` — parse inbound SMS for approve / custom reply / skip / queue navigation
- `approve-reply` / `respond-to-review` — post approved text to GBP

**Gating:** unverified storefronts do not receive bot traffic. See [product/multi-location.md](../product/multi-location.md).

## Queue

When multiple reviews pending, bot presents oldest next after each resolution. Duplicates already replied on GBP are skipped.

## Changelog

- 2026-05-23: sms_v2 consent copy; Privacy/Terms links on opt-in and verify-sms surfaces
- 2026-05-19: Initial integration doc; per-store manager verification and SMS-only approvers
