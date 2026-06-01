# rating.cards — Sales Enablement Brief

**Audience:** Antonio (sales cofounder) and anyone pitching rating.cards to a venue owner.
**Purpose:** Understand what customers get, why it matters, and how to sell each feature. Derived from [product/](../product/) docs — keep in sync on every feature change.

---

## The 30-Second Pitch

> "Your Google rating moves your business more than almost anything else. rating.cards gets happy customers to actually leave reviews — then manages your whole reputation on autopilot. Premium tap-to-review cards on the tables, AI that replies in your voice, and managers who approve from a text thread without logging into anything. One account can cover every location you run."

**What we sell:** Physical products (one-time) + Autopilot (**$20/month flat per account** in v1).
**Who we sell to:** Hospitality venues — cafes, restaurants, bars, salons — and **multi-store operators** managing several Google listings.

---

## Roles (multi-location v1)

| Who | How they use rating.cards |
|-----|---------------------------|
| **Owner / CEO** | Web app: one-time onboarding, then dashboard + **Settings** for all ongoing config (locations, SMS, brand voice, billing) |
| **Store manager** | SMS only — verify phone once, approve review replies from text. No app login. |

**Sales line:** *"Your managers don't need another app. They get a text when a review comes in, tap approve, done. You keep oversight from the dashboard across every store."*

---

## Onboarding (~10 minutes, once)

| Step | What happens | Sales angle |
|------|--------------|-------------|
| **1. Connect Google** | OAuth once; **checkbox pick storefronts**; add more later from Settings without signing in again | One login usually covers every location — no manual name step |
| **2. Manager phones (optional)** | Per-store manager numbers; async SMS verification | Managers verify on their own time — setup doesn't block |
| **3. Brand voice** | Shared Q&A teaches AI how the brand talks | One voice profile; drafts use each **storefront name** |
| **4. Confirmation** | Autopilot live per store as managers verify | Automation turns on location by location |

**Objection:** *"I don't have time to set this up."* → Under 10 minutes once. We also set up physical cards before they ship.

---

## Overview (CEO)

- **All locations** default with storefront switcher when multiple shops
- Light proof metrics: new reviews (7-day rolling), average rating, replies posted, total reviews
- Unified alerts when setup or billing needs attention; green system status when healthy
- Header nav: **Overview** and **Settings**; user menu shows business name and Sign out
- **Sales line:** *"Open it when you want the numbers. Day-to-day approvals happen on your managers' phones."*

---

## Autopilot — the subscription value

### Per storefront (when manager verified)

- New review → SMS to that store's manager with AI draft
- Manager approves, writes own reply, or skips — all via SMS
- Queue clears overnight backlogs in one thread
- Replies post to the **correct** Google listing

### Gating (important for honest selling)

Stores **without** a verified manager still sync reviews to the dashboard — but no SMS alerts or AI drafts until someone verifies. Protects cost and trust.

### Weekly pulse email

Monday organisation-level summary: new reviews, average rating, highlights, negative flags.

**Sales line:** *"$20 a month for the whole account in v1 — every location, flat. Less than a dollar a day to never miss a review again."*

---

## Physical products (entry point)

Cards, stickers, stands, posters — NFC + QR, one-time purchase. We configure and activate **before delivery**. Customer taps → Google Reviews. Every tap logged.

**Sales line:** *"You unbox it, put it on the table, it works. We did the setup before it left our hands."*

---

## Expansion

- Add GBP locations from **Settings → Locations**
- More devices across stores (org-scoped in v1)
- Multi-store operators: one subscription, multiple storefronts, manager SMS per location

---

## Billing

- **$20/month** flat per account (v1 — not per store)
- Stripe-powered; cancel anytime from Settings
- Pay before account creation on landing page

**Sales line:** *"Cancel anytime. Stripe-secured. We earn your month."*

---

## Objection handling

| Objection | Response |
|-----------|----------|
| *"I run multiple locations."* | One account, import every GBP location, assign a manager phone per store. Flat $20/mo in v1. |
| *"My managers won't use software."* | They don't. SMS only — verify once, approve from text forever. |
| *"AI replies sound generic."* | Brand voice Q&A + storefront name in every draft. Sounds like you, not a bot. |
| *"What if AI says something wrong?"* | Managers approve every reply from SMS before it posts. |
| *"Is $20/month worth it?"* | One extra customer from a higher rating pays for it. Most venues see dozens more reviews in month one. |
| *"Can I cancel?"* | Anytime, one click, Stripe portal. Access through end of paid period. |

---

## Complete value loop

1. Physical products → more taps → more reviews
2. Reviews sync per storefront from Google
3. AI drafts in brand voice → manager approves via SMS → posts to Google → better ranking
4. Weekly pulse keeps CEO informed without effort
5. Dashboard for oversight across all locations

**Remember:** We're a reputation **system**, not a dashboard company. The less the CEO opens the app, the better we did our job.

## Changelog

- 2026-06-01: Pitch/onboarding — add more storefronts from Settings without another Google sign-in
- 2026-05-22: 4-step onboarding; org name from GBP; Settings **Your info**
- 2026-05-21: Overview redesign — nav label Overview, proof metrics, unified alerts
- 2026-05-20: Settings as post-setup config hub; onboarding not revisited after completion
- 2026-05-19: Full rewrite for multi-location v1, manager SMS workflow, flat account pricing, simplified onboarding
