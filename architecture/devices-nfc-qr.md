---
product: rating.cards
layer: architecture
domains: [devices, redirects]
auto_sync: true
last_verified: 2026-05-19
---

# Devices, NFC & QR

**Audience:** Developers, AI assistants.

## Concept

Every physical product embeds a URL — via QR code (dynamic redirect through QRTiger) and/or NFC chip (URL written to hardware). Customer tap/scan opens that URL; our server decides the final destination based on device state in Postgres.

## Redirect flow (`card-redirect`)

Edge function accepts `public_id` (or legacy `card_id` query param):

1. Look up `devices` row by `public_id`
2. Branch on `status`:
   - **missing / error** → `APP_ORIGIN/device-error`
   - **unclaimed** → `APP_ORIGIN/setup/:publicId` *(legacy fallback — primary flow is admin-driven activation)*
   - **deactivated** → device error
   - **active** + `destination_url` → log tap to `device_taps` (async), 302 to `destination_url`

Tap logging captures `user_agent` and `ip_country` (Cloudflare header) without blocking redirect.

## Device statuses

| Status | Meaning |
|--------|---------|
| `unclaimed` | Provisioned but not yet activated with destination URL |
| `active` | Live — redirects to Google Reviews (or configured URL) |
| `deactivated` | Intentionally offline |

## Admin provisioning

Antonio activates devices via admin panel before shipping. Sets internal name and `destination_url` (Google Reviews link). See [product/admin-flows.md](../product/admin-flows.md).

## QR vs NFC

| | QR (QRTiger) | NFC |
|--|--------------|-----|
| **Storage** | QRTiger redirect URL | URL written to chip |
| **Updates** | Change destination via QRTiger API without reprint | URL always points to our app; we control redirect server-side |
| **Typical URL** | `qr1.io/...` → our redirect | `app.rating.cards` redirect endpoint with `public_id` |

## Multi-location (v1)

Devices are **organisation-scoped** — not foreign-keyed to storefront rows. Which GBP listing a guest reaches is determined by `destination_url` set at provisioning. Different devices can point to different storefront review URLs within the same account.

## Related docs

- [product/end-customer-flow.md](../product/end-customer-flow.md)
- [architecture/system-overview.md](./system-overview.md)

## Changelog

- 2026-05-19: Replaced stale nfc_qr_architecture.md; aligned with card-redirect and admin-driven flow
