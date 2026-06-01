---
product: rating.cards
layer: architecture
domains: [ui, onboarding, admin]
auto_sync: true
last_verified: 2026-06-01
---

# Frontend loading states

**Audience:** Developers and AI assistants working on the React app.

## Purpose

The app uses a single taxonomy for loading UX: skeleton placeholders for content with a known layout, spinners for actions and indeterminate processes, and a full-screen loader only before the app shell is available.

## When to use each pattern

| Pattern | Use when | Examples |
|---------|----------|----------|
| **Skeleton presets** | First load of a page, section, table, or form with predictable layout | Dashboard boot, settings tabs, admin lists |
| **Stale busy overlay** | Re-fetch while content is already visible (anti-flicker) | Admin filter change, settings section reload |
| **Full-screen loader** | Pre-auth, route guards, OAuth/callback pages, lazy route bootstrap before layout | Login guard, `/auth/callback`, Suspense at app root |
| **Loader2 (shadcn Button)** | Public-app button submit states | Settings forms, onboarding actions |
| **Custom Spinner** | Admin buttons, inline admin actions, autocomplete, indeterminate waits | AdminButton, Google Places search, PDF generation |

## Components

All shared loading UI lives under `src/components/ui/loading/`:

- **`Skeleton`** — base pulse block; `appearance="light"` (public app) or `"dark"` (admin)
- **`LoadingRegion`** — wraps content with `aria-busy`, optional stale overlay on re-fetch
- **`TableSkeleton`**, **`CardListSkeleton`**, **`StatGridSkeleton`**, **`FormSkeleton`**, **`SettingsSectionSkeleton`**, **`DashboardSkeleton`**, **`DetailPanelSkeleton`**, **`PageContentSkeleton`** — layout presets
- **`FullScreenLoader`** — centered amber spinner + message; pre-auth and callbacks only
- **`Spinner`** — custom SVG spinner for admin and indeterminate flows

## `useLoadingPhase`

Hook at `src/hooks/useLoadingPhase.ts`:

- **`initial`** — show skeleton (`showSkeleton`)
- **`refetch`** — keep stale content, show busy overlay (`showStaleBusy`)
- Pass a **`resetKey`** (e.g. `businessId`) to treat context changes as a fresh initial load

## Route-level behavior

- **Inside `AppLayout`:** header stays visible; content area uses skeletons (`PageContentSkeleton` for nested Suspense, page-specific skeletons for data boot)
- **Outside app shell:** `FullScreenLoader` only

## Accessibility

- Skeleton screens include sr-only “Loading…” text
- `LoadingRegion` sets `aria-busy` and `aria-live="polite"` during transitions
- Skeleton pulse disabled when `prefers-reduced-motion: reduce`

## Out of scope

- TanStack Query migration
- Sonner toast loading icon
- Onboarding step ProgressBar

## Changelog

- 2026-06-01 — Initial doc: unified skeleton/spinner taxonomy, `useLoadingPhase`, and shared loading components.
