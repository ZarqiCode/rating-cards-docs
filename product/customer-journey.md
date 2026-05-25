---
product: rating.cards
layer: product
domains: [onboarding, billing]
auto_sync: true
last_verified: 2026-05-22
---

# Customer journey

**Audience:** Sales (Antonio), founders, AI assistants.

rating.cards is a **sales-led** business. Typical path:

## 1. Outreach

Antonio identifies and contacts local hospitality venues through direct outreach and in-person visits.

## 2. Demo

Live or in-person walkthrough: tap-to-review experience, Autopilot features, value of a higher Google rating. For multi-store prospects, highlight one account / multiple storefronts / manager SMS workflow.

## 3. Hardware purchase

Venue buys first set of physical products (cards, stickers, stands, or posters). One-time purchase, no commitment.

## 4. Device setup

rating.cards team sets up and activates devices before delivery. Owner receives devices already live — no setup on their end. See [admin-flows.md](./admin-flows.md).

## 5. Autopilot subscription

After seeing review volume increase, owner subscribes from **rating.cards** landing page. Stripe payment **before** account creation. After paying, they sign up; subscription links automatically. No dashboard login required to subscribe.

## 6. Onboarding

Guided flow (~10 minutes):

1. Connect Google — multi-account + **multi-select GBP locations** (organisation name derived from GBP account)
2. Optional **per-store manager phones** with async SMS verification (not blocking)
3. Shared brand voice for AI
4. Confirmation

Autopilot behaviour is **per storefront** once manager lines verify. See [owner-flows.md](./owner-flows.md).

## 7. Expansion

- Add GBP locations from **Settings → Locations**
- Buy more devices across stores (organisation-scoped in v1)
- Multi-store operators: one flat $20/mo account covers all storefronts in v1

## Changelog

- 2026-05-22: Onboarding starts at Google Connect; org name auto-derived from GBP
- 2026-05-19: Split from project-brief.md; multi-location onboarding and Settings expansion
