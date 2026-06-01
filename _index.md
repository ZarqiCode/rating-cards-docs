# Documentation index

Central map for the rating.cards doc system. AI agents: read this file during every doc impact pass.

## 1. Doc registry

| Doc | Layer | Domains | auto_sync | skip_if |
|-----|-------|---------|-----------|---------|
| [product/business-context.md](./product/business-context.md) | product | — | true | refactor-only |
| [product/business-model.md](./product/business-model.md) | product | billing | true | refactor-only |
| [product/product-capabilities.md](./product/product-capabilities.md) | product | devices, sms, gbp, email, onboarding | true | refactor-only |
| [product/multi-location.md](./product/multi-location.md) | product | multi-location, managers, billing | true | refactor-only |
| [product/customer-journey.md](./product/customer-journey.md) | product | onboarding, billing | true | refactor-only |
| [product/owner-flows.md](./product/owner-flows.md) | product | onboarding, sms, billing, multi-location, managers | true | refactor-only |
| [product/admin-flows.md](./product/admin-flows.md) | product | admin, devices | true | refactor-only |
| [product/end-customer-flow.md](./product/end-customer-flow.md) | product | redirects, devices | true | refactor-only |
| [shop/overview.md](./shop/overview.md) | product | shop | true | refactor-only |
| [shop/payments.md](./shop/payments.md) | integration | shop, billing | true | refactor-only |
| [shop/fulfillment.md](./shop/fulfillment.md) | product | shop | true | refactor-only |
| [architecture/system-overview.md](./architecture/system-overview.md) | architecture | admin, multi-location | true | refactor-only |
| [architecture/devices-nfc-qr.md](./architecture/devices-nfc-qr.md) | architecture | devices, redirects | true | refactor-only |
| [integrations/stripe.md](./integrations/stripe.md) | integration | billing | false | unless Stripe contract changes |
| [integrations/twilio.md](./integrations/twilio.md) | integration | sms, managers | false | unless Twilio contract changes |
| [integrations/google-gbp.md](./integrations/google-gbp.md) | integration | gbp, multi-location | false | unless GBP API contract changes |
| [sales/sales-brief.md](./sales/sales-brief.md) | sales | * | true | never for user-facing changes |
| [brand/brand-guidelines.md](./brand/brand-guidelines.md) | brand | — | false | tone/visual only |
| [legal/privacy-policy.md](./legal/privacy-policy.md) | legal | sms, billing, gbp, email | false | unless legal/contract terms change |
| [legal/terms-of-service.md](./legal/terms-of-service.md) | legal | billing, sms, gbp | false | unless legal/contract terms change |
| [runbooks/shop-stripe-setup.md](./runbooks/shop-stripe-setup.md) | runbook | shop, billing | false | env/config-only edits |
| [runbooks/*](./runbooks/) | runbook | — | false | env/config-only edits |

## 2. Domain → doc trigger matrix

| Domain | Code paths (indicative) | Docs to check |
|--------|-------------------------|---------------|
| `multi-location` | `src/components/settings/**`, `src/pages/VerifySmsPage`, `src/components/dashboard/LocationSwitcher`, `src/contexts/ActiveLocationContext`, `google-import-reviews`, location DB types | **multi-location.md**, owner-flows, product-capabilities, integrations/google-gbp, integrations/twilio, sales-brief |
| `managers` | `verify-sms-public`, `create-sms-verification`, `send-otp-sms`, `handle-sms-reply`, `ApprovalPage` | multi-location, owner-flows, integrations/twilio, sales-brief |
| `devices` | `src/components/admin/**`, `supabase/functions/*device*`, `card-redirect` | admin-flows, end-customer-flow, devices-nfc-qr, product-capabilities, multi-location |
| `redirects` | `supabase/functions/card-redirect/**` | devices-nfc-qr, end-customer-flow |
| `onboarding` | `src/components/onboarding/**`, `src/hooks/useOnboarding*` | owner-flows, product-capabilities, multi-location, sales-brief |
| `sms` | `supabase/functions/*sms*`, `handle-sms-reply` | owner-flows, multi-location, integrations/twilio, product-capabilities, sales-brief |
| `gbp` | `supabase/functions/google-*` (incl. `google-list-locations`), `sync-reviews` | integrations/google-gbp, product-capabilities, multi-location, sales-brief |
| `billing` | `src/components/settings/Billing*`, `stripe-webhook`, `create-checkout-session` | business-model, owner-flows, integrations/stripe, sales-brief |
| `admin` | `src/pages/AdminPage`, `src/components/admin/**`, `admin-*` functions | admin-flows, product-capabilities |
| `email` | `send-weekly-pulse`, `_shared/emails/**` | owner-flows, product-capabilities, sales-brief |
| `shop` | `rating-cards-shop/**` (separate repo), `supabase/functions/shop-*` | **shop/overview.md**, **shop/payments.md**, **shop/fulfillment.md**, **runbooks/shop-stripe-setup.md**, customer-journey, integrations/stripe, integrations/google-gbp, sales-brief |

## 3. Auto-sync policy

| Change type | Docs to update |
|-------------|----------------|
| Any user-facing behavior | Affected `product/*` + **sales/sales-brief.md always** |
| Multi-location / storefront / manager | **product/multi-location.md always** + owner-flows, product-capabilities, relevant integrations |
| Admin / device / billing logic | admin-flows, owner-flows, product-capabilities, architecture as needed |
| Edge function / redirect / schema | architecture/*, relevant integrations/* |
| Pricing / GTM | business-model, customer-journey, sales-brief |
| Pure refactor, no behavior change | Skip — note "No doc impact" |
| Runbook env steps | runbooks/* only if API contract or env var names change |

## 4. Authority hierarchy

1. **Code** (implementation truth)
2. **Architecture + integration docs**
3. **Product docs**
4. **Sales brief** (derived — never invent features here)
5. **Brand guidelines** (tone/visual only)

## 5. Maintenance checklist

Before marking a task complete (if `src/**`, `supabase/**`, or product behavior changed):

1. **Inventory** — list changed files; infer domains from the matrix above
2. **Select candidates** — union docs for matched domains; add sales-brief for user-facing changes
3. **Read and diff** — does documented behavior, steps, pricing, or architecture need updating?
4. **Apply** — edit docs directly; update `last_verified` in frontmatter; add a `## Changelog` line
5. **Report** — end task summary with `### Doc updates` (files changed or "No doc impact")

Tag deferred scope only in `product/multi-location.md` as `(deferred v1)`.
