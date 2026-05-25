---
product: rating.cards
layer: product
domains: []
auto_sync: true
last_verified: 2026-05-22
---

# Business context

**Audience:** Founders, AI assistants, anyone making product or strategy decisions.

## The problem

Hospitality businesses live and die by their online reputation. The difference between a 4.6 and a 4.8 on Google Maps translates directly into foot traffic, reservations, and revenue. Owners know they need more reviews — but they have no system to ask for them.

A cafe owner works from 5 AM to close, managing staff, suppliers, and customers. Asking every happy guest to leave a Google review is unrealistic. Most don't think of it; those who do forget once they leave. Responding to reviews consistently falls to the bottom of the pile.

The result: great businesses with mediocre online reputations, losing customers to competitors who simply have more reviews.

## The solution

rating.cards is a physical + software product that turns every customer touchpoint in a hospitality venue into a frictionless review prompt — and then manages the venue's entire online reputation automatically.

**Physical:** NFC and QR-enabled cards, stickers, stands, and posters placed throughout each venue — or scaled across locations for multi-store operators. A customer taps or scans and lands on **that storefront's** Google Reviews page when devices are routed per GBP link.

**Software — Autopilot:** Syncs reviews from Google per **storefront**, drafts replies in the account's shared brand voice, and texts a **verified manager** at that location when a new review lands. Managers approve via SMS **without an app login**. CEOs use the web dashboard to switch between storefronts. A weekly pulse email summarises organisation-level activity.

See [multi-location.md](./multi-location.md) for the v1 multi-store scope.

## Team and location

rating.cards is a two-person startup based in **Carlsbad, California**.

- **Lucas** — Technical cofounder. Builds the product, platform, and infrastructure.
- **Antonio** — Marketing and sales cofounder. Outreach, demos, and customer relationships with local hospitality venues.

## Positioning

### Premium angle

Hospitality is a premium industry. Every product we put into a venue must reflect that standard — designed to belong there, not like an afterthought taped to the counter.

### Multi-store operators

Beyond single-venue owners, **multi-store operators** — small chains and franchise-style groups — are supported in v1: one Autopilot account ties together multiple storefronts without duplicating billing profiles for each store.

### Brand customization

Web onboarding collects a shared **brand voice** for AI drafts; organisation name is **auto-derived from the GBP account** on first location import (editable in **Settings → Your info**). Legacy DB fields (`city`, `brand_color`, `google_place_id`) remain for admin provisioning but are not used in the owner app. Physical products use a consistent premium look. Software uses **organisation and storefront names** with brand voice so drafts feel location-specific while staying on-brand at the account level.

### Unboxing vision (planned)

Long-term: premium kit packaging, welcome card, and a product suite owners want to show their team. Not yet in place — reflects brand direction.

## Competitive advantage

The moat is the **complete system**:

- Physical products designed to look premium in a venue environment
- Frictionless tap-to-review that actually gets customers to leave reviews
- SMS-first response workflow — managers on their phones, not another dashboard
- Software that manages the full review lifecycle: collection, AI drafting, approval, posting to Google

Purpose-built for hospitality venues that care about their brand and don't have time for another login.

## Changelog

- 2026-05-22: Org name auto-derived from GBP; owner app name-only; Settings **Your info**
- 2026-05-19: Split from project-brief.md; includes multi-location v1 positioning
