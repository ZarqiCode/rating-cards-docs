---
product: rating.cards
layer: product
domains: [multi-location, managers, billing]
auto_sync: true
last_verified: 2026-05-20
---

# Multi-location v1

**Audience:** Developers, sales, AI assistants — **boundary doc** for what is shipped vs deferred in multi-store support.

## Roles

| Role | Surface | Responsibilities |
|------|---------|------------------|
| **CEO / account owner** | Web app (`app.rating.cards`) | Onboarding, Settings, dashboard, storefront switcher, billing |
| **Store manager** | SMS only — no app login | Verify phone via public link; approve/write/skip review replies |

## Storefront model

- One subscriber account connects **multiple GBP locations** (across one or more Google OAuth logins)
- Reviews, SMS alerts, and AI drafts behave **per storefront**
- **Verification gating:** unverified storefronts still **sync reviews** to the dashboard but receive **no AI drafts or alert SMS** until a manager verifies their number
- **Shared brand voice** across all storefronts in v1

## Devices in v1

Devices remain **organisation-scoped** — not tied row-for-row to storefront records. Redirect URLs point to GBP review links configured at provisioning time.

## Billing in v1

**Flat** Autopilot subscription per account. No per-location Stripe quantity.

## Shipped (v1)

- Multiple GBP locations per account; multiple Google OAuth connections per business
- Dashboard **storefront switcher** with stats scoped to active storefront
- **Settings → Locations** — shared `GbpImportFlow` (OAuth via `/settings/google-callback`) and soft-deactivate storefronts
- **Settings → Notifications** — per-store manager lines + resend verification SMS
- Public **`/verify-sms/:token`** — manager verification (SMS link + OTP) without app login
- CEO-only product surface; managers approve on SMS only

## Deferred (v1)

Tag these `(deferred v1)` elsewhere — do not document as shipped:

- Manager or staff application logins and RBAC beyond the CEO
- Stripe per-location metering or seat billing
- Different brand voice presets per storefront
- Cross-store aggregate analytics in the dashboard
- Hard-delete of storefront rows (deactivate only in v1)
- Returning city, brand color, and Google Places picker to **web** onboarding

## Related docs

- [owner-flows.md](./owner-flows.md) — onboarding and Settings flows
- [product-capabilities.md](./product-capabilities.md) — Autopilot per storefront
- [integrations/google-gbp.md](../integrations/google-gbp.md) — multi-location import
- [integrations/twilio.md](../integrations/twilio.md) — manager SMS verification

## Changelog

- 2026-05-20: Settings Locations uses shared GBP import; onboarding not used post-completion
- 2026-05-19: Split from project-brief.md Multi-location v1 section
