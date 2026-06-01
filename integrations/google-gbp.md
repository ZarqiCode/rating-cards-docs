---
product: rating.cards
layer: integration
domains: [gbp, multi-location]
auto_sync: false
last_verified: 2026-06-01
---

# Google Business Profile integration

**Audience:** Developers, AI assistants. Test procedures: [runbooks/google-gbp-edge-function-tests.md](../runbooks/google-gbp-edge-function-tests.md).

## OAuth and import

- `google-auth-url` — start OAuth for a Google account (scopes: `business.manage`, `openid`, `email`, `profile`)
- `google-callback` — exchange code, read Google account email via userinfo, store tokens in `google_connections`; returns GBP **accounts** (with `accountName`) and **locations** for multi-select import
- `google-list-locations` — list GBP locations for all stored `google_connections` (refresh tokens); merge with DB storefronts (`available` | `active` | `paused`); per-connection `reconnect_required` errors
- `google-import-reviews` — attach selected GBP location as storefront; on first import sets `businesses.business_name` from GBP account name (fallback: location title) if unset

Supports **multiple Google OAuth connections** per business (e.g. locations under different Google logins). Each connection is keyed by **`google_account_email`** (required, non-null); OAuth must include email scopes so `google-callback` can persist the Google login identity.

## Per-storefront sync

- `sync-reviews` — pull reviews per attached GBP location (service role / scheduled)
- Reviews stored against storefront records; dashboard switcher scopes stats to active storefront

## Replies

- `respond-to-review` / GBP reply helpers post manager-approved text to the **correct** listing for that storefront

## Onboarding and Settings

- Initial import during onboarding via `GbpImportFlow` with `flow: onboarding` — always checkbox picker after OAuth
- **Settings → Storefronts** — `AddShopsModal` calls `google-list-locations` (no OAuth when connections exist); first connect uses OAuth `flow: settings`
- **Settings → Google accounts** — `flow: integrations` for linking an additional Google login only
- OAuth redirect URI: `https://<APP_HOST>/settings/google-callback` (register in Google Cloud + `GOOGLE_REDIRECT_URI`)
- OAuth `state` includes `flow` (`onboarding` | `settings` | `integrations`) and optional `google_connection_id` for per-login reconnect
- Soft-deactivate storefronts in v1 (hard-delete deferred); re-importing a paused storefront sets `is_active: true`

## Gating with SMS

AI drafts and review alert SMS only for storefronts with **verified manager line**. Unverified storefronts still sync reviews to dashboard.

## Changelog

- 2026-06-01: `google-list-locations`; OAuth `integrations` flow; Storefronts token-based add-more-shops
- 2026-05-24: OAuth scopes include `openid`, `email`, `profile`; `google-callback` validates email before upsert
- 2026-05-22: `google-callback` returns account names; `google-import-reviews` auto-sets org name on first import
- 2026-05-20: Document `/settings/google-callback` redirect URI and OAuth `flow` state
- 2026-05-19: Initial integration doc; multi-account OAuth and per-storefront sync
