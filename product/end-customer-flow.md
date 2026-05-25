---
product: rating.cards
layer: product
domains: [redirects, devices]
auto_sync: true
last_verified: 2026-05-19
---

# End-customer flow

**Audience:** Developers, sales, AI assistants — the venue **guest** experience (tap/scan to review).

## Overview

A customer at the venue taps (NFC) or scans (QR) a physical product. They are redirected to leave a Google review. One tap, one action. Every tap is logged.

## Active device

When device status is **active**:

1. Customer tap/scan opens short URL with device `public_id`
2. Edge function resolves device and reads `destination_url` (typically the venue's Google Reviews page for that storefront's GBP listing)
3. Customer redirects to Google — no app install, no search, no friction
4. Tap logged asynchronously (user agent, country) for dashboard analytics

## Device states

| Status | Customer experience |
|--------|---------------------|
| **active** | Redirect to `destination_url` |
| **unclaimed** | Redirect to legacy `/setup/:publicId` (fallback — primary flow is admin-driven provisioning) |
| **deactivated** / missing | Redirect to device error page |

## Physical products

Same flow for cards, stickers, stands, and posters — each carries unique NFC/QR pointing at the redirect URL.

## Multi-location note (v1)

Devices are organisation-scoped, not tied to storefront rows. The **destination URL** configured at admin setup determines which GBP listing the guest reaches. Operators with multiple locations may use different devices or URLs per storefront.

## Related docs

- [admin-flows.md](./admin-flows.md) — how devices are provisioned
- [architecture/devices-nfc-qr.md](../architecture/devices-nfc-qr.md) — redirect logic

## Changelog

- 2026-05-19: Split from project-brief.md; documented legacy unclaimed path
