---
product: rating.cards
layer: product
domains: [onboarding, sms, billing, multi-location, managers]
auto_sync: true
last_verified: 2026-06-12
---

# Owner flows

**Audience:** Developers, AI assistants — CEO/account-owner journeys in the web app and linked SMS paths.

## Account linking

1. rating.cards team creates **business profile** when first device is provisioned (no customer account yet)
2. Owner subscribes via Stripe Payment Link **before** signup
3. System sends setup email to checkout address; browser path stores signed claim token through signup
4. Owner creates account (any email); subscription and business profile link automatically — device history preserved
5. If checkout email ≠ signup email, onboarding shows an informational billing notice (receipts vs login)

## Onboarding (one-time)

After signup, guided flow:

1. **Google connection** — OAuth once; **checkbox list of GBP storefronts** (always shown, even for a single location); organisation name auto-filled from GBP account on first import. A second Google login is optional later under **Settings → Google accounts** (info icon on step 1 explains this).
2. **Brand voice (required)** — single-page form: business type, formality, emoji preference, criticism handling, and sign-off; optional “sound even more like you” block (voice samples, phrases, words to avoid); shared profile for AI drafts across all storefronts; optional **auto-respond** toggle (4–5 star reviews auto-post when enabled; 1–3 star still SMS). Cannot skip.
3. **Per-store SMS activation** — per-location checklist with status badges (Not started / Pending verification / Verified / Set up later); manager phone numbers, TCPA consent (`sms_v2` checkbox + Privacy/Terms links); **async** verification via public link + OTP; CEO must decide per storefront before continuing; verification not required to finish onboarding
4. **Review & finish** — honest status headline (4 tiers) plus per-storefront SMS summary; Autopilot live per storefront as manager lines verify

Progress bar shows 3 segments (Google → Brand voice → Manager SMS); finish screen has no progress segment.

Welcome splash may appear before step 1 for new users. Organisation name can be edited later in **Settings → Your info**.

After `onboarding_completed_at` is set, `/onboarding` redirects to the dashboard. The wizard is not available for returning users.

Route gates: incomplete onboarding cannot open `/settings`; completed users cannot reopen `/onboarding`.

## Manager verification (no app login)

Managers receive SMS with link to **`/verify-sms/:token`**. OTP confirmation verifies the line; the page shows SMS program disclosures and links to [Privacy Policy](https://www.rating.cards/privacy) and [Terms of Service](https://www.rating.cards/terms). When verified, that storefront gets **manager SMS alerts** and the **negative-review approval loop** (AI draft → manager approves via SMS).

**Verification gating (code truth):** Unverified storefronts still sync reviews. Manager SMS alerts and negative-review drafts require a verified line. When **auto-respond** is enabled in Brand voice, **4–5 star** reviews can get an AI reply posted to Google **without** a verified manager line — implementation allows auto-post when `rating >= 4` and `auto_respond_enabled` is true.

## Day-to-day: SMS approval loop

For verified storefronts, managers receive SMS on **1–3 star** reviews (and on all reviews when auto-respond is off). Two-tap approve, write own reply, or skip. Queue handles backlog. CEOs do not need to participate in SMS thread unless they are also the verified manager.

When **auto-respond** is enabled in Brand voice (Settings or onboarding), **4–5 star** reviews get an AI reply posted to Google automatically — no SMS. **1–3 star** reviews always go through the SMS approval loop and require a verified manager line.

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
| **Storefronts** (locations) | **Add more shops** modal — pick unimported GBP locations from already linked Google logins (no re-OAuth); list active/paused storefronts; pause a shop; post-import prompt to set manager phones |
| **Notifications** | Per-store manager lines; invite link + CEO inline OTP; verified lines read-only |
| **Brand voice** | Single scrollable panel with five cards (your business, your voice, handling reviews, autopilot, optional advanced); read-only summary strip; incomplete-setup callout lists missing required fields; **Save changes** / **Discard** sticky footer when edited; hash anchors (`#business`, `#voice`, `#reply-style`, `#autopilot`, `#advanced`) for deep links from Overview alerts |
| **Google accounts** (integrations) | Edge case: link an **additional** Google login (separate owner email); success modal → add shops in **Storefronts** |

Sections use `?section=` in the URL. Smart default opens the first incomplete area (locations → notifications → brand voice → billing).

GBP OAuth callback: `/settings/google-callback` with `flow` in OAuth state (`onboarding`, `settings`, or `integrations`). Listing locations for add-more-shops uses `google-list-locations` (stored tokens, no OAuth).

## Weekly pulse

Organisation-level email every Monday. SMS handles moment-to-moment; pulse handles strategic recap.

## Related docs

- [multi-location.md](./multi-location.md) — roles, gating, v1 boundaries
- [integrations/twilio.md](../integrations/twilio.md) — SMS bot and verification
- [integrations/stripe.md](../integrations/stripe.md) — billing

## Changelog

- 2026-06-10: Copy alignment — verification gating doc matches code (4–5 star auto-post without verified SMS when auto-respond on); manager SMS vs negative-review drafts wording
- 2026-06-12: Brand voice v2 — expression-focused fields (business type, formality, emoji, criticism, sign-off); optional voice samples/phrases/avoid list; removed personality tags and response length
- 2026-06-07: Onboarding reorder — Google → brand voice (required) → SMS activation checklist → honest finish; 3-segment progress bar
- 2026-06-07: Hybrid auto-respond — opt-in 4–5 star auto-post; 1–3 star SMS approval unchanged
- 2026-06-09: Payment Link claim tokens, setup email recovery, billing email notice on onboarding
- 2026-06-01: Storefronts add-more-shops modal (token-based picker); Google accounts only for extra logins; onboarding always shows location checkbox
- 2026-05-23: SMS consent sms_v2 with Privacy/Terms links on onboarding, Settings, and verify-sms page
- 2026-05-22: 4-step onboarding (drop manual name step); org name from GBP; Settings **Your info**
- 2026-05-21: Overview redesign — nav label Overview, proof metrics, unified alerts, no devices on home
- 2026-05-21: Storefronts vs Google accounts labels and plain-language Settings copy
- 2026-05-20: Settings owns post-completion config; onboarding blocked after complete; OAuth callback and section routing
- 2026-05-19: Split from project-brief.md; multi-location onboarding and Settings sections
