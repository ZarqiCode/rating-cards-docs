---
product: rating.cards
layer: product
domains: [onboarding, sms, billing, multi-location, managers]
auto_sync: true
last_verified: 2026-05-23
---

# Owner flows

**Audience:** Developers, AI assistants — CEO/account-owner journeys in the web app and linked SMS paths.

## Account linking

1. rating.cards team creates **business profile** when first device is provisioned (no customer account yet)
2. Owner subscribes via Stripe **before** signup
3. Owner creates account; subscription and business profile link automatically — device history preserved

## Onboarding (one-time)

After signup, guided flow:

1. **Google connection** — multi-account OAuth; **multi-select GBP locations** to import as storefronts; organisation name auto-filled from GBP account on first import
2. **Per-store SMS (optional)** — manager phone numbers, TCPA consent (`sms_v2` checkbox + Privacy/Terms links); **async** verification via public link + OTP; onboarding can complete with verifications pending
3. **Brand voice** — shared Q&A profile for AI drafts across all storefronts
4. **Confirmation** — Autopilot active per storefront as manager lines verify

Welcome splash may appear before step 1 for new users. Organisation name can be edited later in **Settings → Your info**.

After `onboarding_completed_at` is set, `/onboarding` redirects to the dashboard. The wizard is not available for returning users.

Route gates: incomplete onboarding cannot open `/settings`; completed users cannot reopen `/onboarding`.

## Manager verification (no app login)

Managers receive SMS with link to **`/verify-sms/:token`**. OTP confirmation verifies the line; the page shows SMS program disclosures and links to [Privacy Policy](https://www.rating.cards/privacy) and [Terms of Service](https://www.rating.cards/terms). Automation (alerts + AI drafts) turns on for that storefront when verified.

## Day-to-day: SMS approval loop

For verified storefronts, managers receive SMS on new reviews with AI draft. Two-tap approve, write own reply, or skip. Queue handles backlog. CEOs do not need to participate in SMS thread unless they are also the verified manager.

## Overview (CEO home)

CEO logs in for a quick health check — proof metrics, unified alerts, and system status. Rarely needed for approvals — managers handle SMS.

**Navigation:** Shared header on Overview and Settings — **Overview** and **Settings** links with active-state underline; user menu (avatar + business name) with Sign out. Admin users see a distinct dark **Admin** button. Logo returns to Overview (`/dashboard`).

**Overview shows:**
- Business name + status headline (`All good`, `N things need attention`, or `Autopilot inactive`)
- **All locations** default with storefront switcher when multiple shops
- Unified tiered alerts when anything needs setup or attention
- Green system status checklist when fully healthy
- Light proof metrics: rolling 7-day reviews/rating, replies this month, total reviews

If no active storefronts exist after onboarding, Overview shows a blocking alert with a CTA to **Settings → Storefronts** (import GBP locations).

## Settings (returning users)

| Section | Purpose |
|---------|---------|
| **Billing** | View subscription, Stripe portal for payment/cancel |
| **Your info** | Edit organisation name (auto-filled from GBP on first import) |
| **Storefronts** (locations) | Add shops from Google, list active/paused storefronts, pause a shop |
| **Notifications** | Per-store manager lines; invite link + CEO inline OTP; verified lines read-only |
| **Brand voice** | Edit shared AI draft personality (does not change onboarding step) |
| **Google accounts** (integrations) | Which Google logins are linked; link another login; add shops in Storefronts |

Sections use `?section=` in the URL. Smart default opens the first incomplete area (locations → notifications → brand voice → billing).

GBP OAuth callback: `/settings/google-callback` with `flow` in OAuth state (`onboarding` or `settings`).

## Weekly pulse

Organisation-level email every Monday. SMS handles moment-to-moment; pulse handles strategic recap.

## Related docs

- [multi-location.md](./multi-location.md) — roles, gating, v1 boundaries
- [integrations/twilio.md](../integrations/twilio.md) — SMS bot and verification
- [integrations/stripe.md](../integrations/stripe.md) — billing

## Changelog

- 2026-05-23: SMS consent sms_v2 with Privacy/Terms links on onboarding, Settings, and verify-sms page
- 2026-05-22: 4-step onboarding (drop manual name step); org name from GBP; Settings **Your info**
- 2026-05-21: Overview redesign — nav label Overview, proof metrics, unified alerts, no devices on home
- 2026-05-21: Storefronts vs Google accounts labels and plain-language Settings copy
- 2026-05-20: Settings owns post-completion config; onboarding blocked after complete; OAuth callback and section routing
- 2026-05-19: Split from project-brief.md; multi-location onboarding and Settings sections
