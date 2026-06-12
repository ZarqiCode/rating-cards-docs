---
product: rating.cards
layer: product
domains: [devices, sms, gbp, email, onboarding]
auto_sync: true
last_verified: 2026-06-12
---

# Product capabilities

**Audience:** Developers, sales, AI assistants — feature inventory for rating.cards.

## Physical products

| Product | Description |
|---------|-------------|
| **Cards** | Premium NFC + QR cards on tables — tap/scan to Google Reviews |
| **Stickers** | Adhesive NFC + QR for surfaces, packaging, receipts |
| **Stands** | Countertop/table stands near register or high-traffic areas |
| **Posters** | Wall-mounted QR for windows, walls, waiting areas |

Every touchpoint where a happy customer sits, stands, or waits should have a review prompt nearby.

## Autopilot — software layer

Philosophy: hospitality owners should rarely log into a dashboard. Day-to-day runs on SMS for managers; CEOs use the web app for oversight.

### AI review responder + SMS approval

- Drafts replies **per storefront** using shared brand voice + **storefront name**
- **Hybrid auto-respond (opt-in):** when enabled, **4–5 star** reviews are AI-drafted and posted to Google automatically; **1–3 star** reviews always go through SMS approval
- Surfaces drafts to **verified manager** via SMS for reviews that need human approval (no app login)
- **Gating:** unverified storefronts sync reviews only — no AI drafts or alert SMS (positive auto-post does not require SMS contact when auto-respond is on)
- Manager approves, writes own reply, or skips via SMS
- Approved replies post to correct GBP listing; `posted_via` tracks auto vs SMS-approved vs SMS-custom
- Sentiment and star rating calibrate tone
- **Human-voice guardrails (hardcoded):** every draft follows anti-slop rules (plain owner tone, no corporate stock phrases, no em/en dashes, no AI filler words); optional owner **voice samples** in brand voice settings replace generic few-shot examples; sign-off is the owner or business name on its own line

**(planned)** Claude sentiment guardrail to replace fixed star-rating threshold; dashboard edit/retract of auto-posted replies; intercept delay; owner digest of auto-posted positives.

### SMS notification bot

For each **verified** storefront, **1–3 star** reviews (or all reviews when auto-respond is off) trigger SMS to that location's manager with review content and AI draft when enabled.

### Auto-queue

Multiple pending reviews queue oldest-first. After each resolution, bot prompts for next. Duplicates (already replied on GBP) skipped silently. "All caught up!" when empty.

### Weekly pulse email

Monday morning organisation-level summary: new reviews, average rating, best review, negative flags. Per-location rollups may evolve.

## Google Business Profile integration

- Connect Google; **checkbox-select** GBP storefronts (onboarding always shows the list)
- **Add more shops** from Settings → Storefronts using stored tokens (no re-OAuth for the same login)
- Optional **additional Google logins** under Settings → Google accounts (edge case)
- Sync reviews **per storefront**
- AI + SMS **per verified location**
- Post replies to correct listing
- Feed weekly pulse from review activity

Details: [integrations/google-gbp.md](../integrations/google-gbp.md).

## Dashboard and settings (CEO)

Web app at **app.rating.cards**:

- **Dashboard** — stats scoped to active storefront; **storefront switcher** when multiple locations
- **Settings → Billing** — Stripe portal, subscription management
- **Settings → Storefronts** — add/pause storefronts; **Settings → Google accounts** — extra logins only
- **Settings → Notifications** — per-store manager lines, resend verification SMS
- **Devices** — organisation-scoped list and tap counts (v1)

Dashboard is secondary to SMS for day-to-day approvals. See [multi-location.md](./multi-location.md).

## Changelog

- 2026-06-12: Brand voice v2 — configurable business type, formality, emoji; optional voice samples replace hardcoded few-shot examples; dropped personality tags and response length
- 2026-06-12: AI review responder — hardcoded human-voice rules, rewritten few-shot examples, newline sign-off (no em dash)
- 2026-06-07: Hybrid auto-respond — opt-in 4–5 star auto-post; SMS only for 1–3 star when enabled
- 2026-06-01: Storefronts token-based add-more-shops; Google accounts for additional logins only
- 2026-05-19: Split from project-brief.md; per-storefront Autopilot and multi-GBP import
