---
product: rating.cards
layer: product
domains: [admin, devices]
auto_sync: true
last_verified: 2026-06-12
---

# Admin flows

**Audience:** Internal team (Lucas, Antonio), developers, AI assistants.

Device setup and business management happen in the **admin panel** — not by venue owners.

## Access

Antonio (or Lucas) signs into the admin panel with an admin account at **app.rating.cards** (admin route).

**Navigation:** Admin uses a separate dark shell. Logo returns to Dashboard; **Dashboard** link and shared user menu (avatar + business name, Sign out). From the CEO app, admin users reach Admin via a dark-styled button in the shared header.

## Device setup (admin-driven)

When a customer purchases hardware, the team provisions devices **before delivery**:

1. **Sign in to admin**
2. **Select or create business** — search existing profile or create new. Requires business name; city and brand color optional. Business profiles exist without customer accounts.
3. **Name the device** — internal label (e.g. "Front counter", "Patio table 5")
4. **Enter Google Reviews link** — destination URL for tap/scan. Pre-filled from most recent active device if business already has devices.
5. **Activate** — device goes live immediately
6. **Confirmation** — success screen with device name, business name, destination URL; option to add another device or return to admin

**Principle:** owners receive devices already live — no setup, no account creation, no friction.

## Other admin capabilities

- Business list and detail views
- Device list, detail, bulk provision wizard
- Print assets generation (cards, stickers, etc.)
- Subscription overview (admin view)
- **AI Demo** — test how a business's saved brand voice responds to a pasted review (admin-only; no DB writes, no Google posting)
- Test mode toggle for internal QA

## AI Demo (internal)

**Navigation:** Admin panel → **AI Demo** tab.

**Purpose:** Let Lucas or Antonio preview AI-generated review replies using a business's saved brand voice — without inserting reviews, triggering SMS, or posting to Google.

**Flow:**

1. **Search and select a business** — searchable list (same data as Businesses tab).
2. **View brand voice snapshot** — read-only summary of saved `brand_voice_settings` (formality, emoji, criticism handling, sign-off, voice samples, etc.). If no settings exist, generation is blocked with a clear message.
3. **Enter a test review** — star rating (1–5), reviewer name, review text.
4. **Generate reply** — calls the `demo-respond` edge function (admin-gated). Returns the AI reply and a collapsible view of the exact system prompt sent to Claude.

**Access:** Admin accounts only (same gate as other admin endpoints). No subscription or SMS requirements — this is an internal tuning tool.

## Devices and multi-location (v1)

Devices are **organisation-scoped** — not assigned row-for-row to storefront records in v1. Destination URL determines which GBP listing the customer reaches. See [multi-location.md](./multi-location.md).

## Related docs

- [end-customer-flow.md](./end-customer-flow.md) — what happens when a customer taps
- [architecture/devices-nfc-qr.md](../architecture/devices-nfc-qr.md) — redirect implementation

## Changelog

- 2026-06-12: Added AI Demo tab — admin-only brand voice preview with `demo-respond` edge function
- 2026-05-20: Admin header shares user menu with CEO app; dark Admin entry from CEO header
- 2026-05-19: Split from project-brief.md device setup section
