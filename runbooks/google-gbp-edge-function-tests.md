# Google Business Profile (GBP) — Edge Function tests (Supabase Dashboard)

Use the **built-in test UI** for Edge Functions so you do not need `curl` or local env vars in the terminal. Supabase sends the request from your browser and shows the response in the same panel.

These functions implement **OAuth + listing locations + importing reviews** for GBP. Ongoing **review sync** uses a separate function (`sync-reviews`); see [email-bot-edge-function-tests.md](./email-bot-edge-function-tests.md) for that.

---

## Where to open the test UI

1. Open your project in the [Supabase Dashboard](https://supabase.com/dashboard).
2. Go to **Edge Functions**.
3. Click the function name (e.g. `google-auth-url`).
4. Open **Test** / **Invoke** (wording may vary) to open the modal with **HTTP Method**, **Request Body**, **Headers**, **Query Parameters**, **Role**, and **Send request**.

---

## Auth: user JWT (not service role)

These three functions validate the caller with **`supabase.auth.getUser()`** using the **`Authorization`** header and the **anon** client. They expect a **normal logged-in user’s access token** (the same JWT your app sends after `supabase.auth.signIn`).

| Role / header | Works? |
|---------------|--------|
| **`Authorization: Bearer <user access_token>`** | Yes — use this for real tests. |
| **Service role** JWT in **Role** (Dashboard) | **No** — `getUser()` will not treat this as an end user; you will usually get **401 Unauthorized**. |
| **Anon** key only, no user JWT | **No** — missing or invalid user session → **401**. |

### How to get a user `access_token`

1. Sign in to your app in the browser.
2. In **DevTools → Application → Local Storage** (or **Session Storage**), find your Supabase project key and copy **`access_token`** from the stored session JSON, **or**
3. In **Network**, trigger any authenticated Supabase request and copy the **`Authorization: Bearer …`** value (without `Bearer `).

In the Dashboard test UI, add a header:

| Header | Value |
|--------|--------|
| `Authorization` | `Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...` |

(Your full user JWT; the UI may also let you paste only the token—follow what your Dashboard version expects.)

**Important:** The user you impersonate must **own** the `business_id` you send (same `user_id` as in `businesses`), or you may hit RLS or confusing results when rows are read from the client elsewhere. The edge functions use **service role** only for writes to `google_connections` / `reviews` where applicable.

---

## Prerequisites (Edge Function secrets)

Ensure these are set under **Project Settings → Edge Functions → Secrets** (names must match what the code reads):

| Secret | Used by |
|--------|---------|
| `GOOGLE_CLIENT_ID` | `google-auth-url`, `google-callback` |
| `GOOGLE_CLIENT_SECRET` | `google-callback`, `google-import-reviews` (refresh) |
| `GOOGLE_REDIRECT_URI` | `google-callback` (must match Google Cloud OAuth client **Authorized redirect URIs**; `google-auth-url` can fall back to request body `redirect_uri` for the authorize URL) |

Also: **`SUPABASE_URL`**, **`SUPABASE_ANON_KEY`**, **`SUPABASE_SERVICE_ROLE_KEY`** are required for the Supabase clients inside the functions (usually provided by the platform).

---

## Test 1 — `google-auth-url`

Builds the Google OAuth URL (scopes `business.manage`, `openid`, `email`, `profile`; offline access; encoded `state` with `business_id`, `user_id`, and optional `flow`).

| Field | Value |
|--------|--------|
| **HTTP Method** | `POST` |
| **Headers** | `Authorization: Bearer <user access_token>` (see above) |
| **Role** | Does not replace the user JWT; ensure the **Authorization** header is set manually with a **user** token. |
| **Request Body** | JSON (replace UUIDs and URL with your values) |

```json
{
  "business_id": "YOUR_BUSINESS_UUID",
  "redirect_uri": "https://YOUR_APP_HOST/settings/google-callback",
  "flow": "settings"
}
```

- `business_id` must exist in **`businesses`** for the logged-in user you are testing as (recommended).
- `redirect_uri` should match what you use in the app; the function prefers **`GOOGLE_REDIRECT_URI`** env when building the authorize URL if set.
- `flow` is optional: `onboarding` (default) or `settings` — encoded in OAuth `state` for post-callback routing.

### Register redirect URI in Google Cloud

1. [Google Cloud Console](https://console.cloud.google.com/) → **APIs & Services** → **Credentials** → your OAuth 2.0 Client ID.
2. Under **Authorized redirect URIs**, add:
   - `https://<PRODUCTION_HOST>/settings/google-callback`
   - `http://localhost:5173/settings/google-callback` (or your Vite dev port)
3. Set Supabase secret **`GOOGLE_REDIRECT_URI`** to the production callback URL above.
4. After deploy, smoke-test onboarding step 2 and Settings → Locations import.
5. Remove legacy `/onboarding` redirect URI once migration is verified (optional).

**Send request.**

**Expected (success):** `200` and JSON like:

```json
{ "url": "https://accounts.google.com/o/oauth2/v2/auth?..." }
```

**Expected (failure):** `400` with `Missing business_id or redirect_uri`; `401` if the user JWT is missing or invalid; `500` if `GOOGLE_CLIENT_ID` is missing.

**Full OAuth:** Copy `url` into a browser, complete Google consent, and continue with **Test 2** using the `code` and `state` from the redirect (or test the app flow end-to-end).

---

## Test 2 — `google-callback`

Exchanges the OAuth **`code`** for Google tokens, upserts **`google_connections`**, then calls Google **Account Management** + **Business Information** APIs and returns **`locations`**.

| Field | Value |
|--------|--------|
| **HTTP Method** | `POST` |
| **Headers** | `Authorization: Bearer <same user access_token as Test 1>` |
| **Request Body** | JSON |

**2a — Smoke test (invalid code, no Google setup required)**

Use a dummy code to confirm the function runs and Google returns an error (proves client id/secret/redirect are exercised).

Build **`state`** the same way the app does: Base64 (standard) of `{"business_id":"<uuid>","user_id":"<uuid>"}` with **your** real IDs. In the browser console: `btoa(JSON.stringify({ business_id: "…", user_id: "…" }))`.

```json
{
  "code": "invalid_test_code",
  "state": "PASTE_BASE64_FROM_STEP_ABOVE"
}
```

If `state` is not valid Base64 or JSON, you may get **500** from `atob` / `JSON.parse` before Google is called.

**Expected:** `400` with a body containing Google’s OAuth error (e.g. `invalid_grant`). That confirms the handler, token endpoint, and env wiring are reachable.

**2b — Full test (real OAuth code)**

After completing OAuth in the browser, copy **`code`** and **`state`** from the redirect query string (same user session). Body:

```json
{
  "code": "PASTE_AUTHORIZATION_CODE",
  "state": "PASTE_STATE_FROM_REDIRECT"
}
```

**Expected (success):** `200` and JSON:

```json
{
  "locations": [
    {
      "accountId": "...",
      "locationId": "...",
      "name": "Location title",
      "address": "..."
    }
  ],
  "google_connection_id": "..."
}
```

Verify in **Table Editor → `google_connections`**: the row for your `business_id` has a non-null **`google_account_email`** matching the Google account you authorized. If email is missing, `google-callback` returns **400** with a user-facing message (not a raw Postgres constraint error).

**Expected (no GBP):** `200` and `{ "locations": [] }` if the Google account has no Business Profile locations (or APIs return empty).

**Expected (failure):** `400` for invalid/expired `code` or token response errors; `401` for bad user JWT.

---

## Test 3 — `google-import-reviews`

Persists **`gbp_account_id`**, **`gbp_location_id`**, optional **`gbp_location_name`**, then calls **`mybusiness.googleapis.com/v4/.../reviews`**, upserts **`reviews`**, returns counts.

| Field | Value |
|--------|--------|
| **HTTP Method** | `POST` |
| **Headers** | `Authorization: Bearer <user access_token>` |
| **Request Body** | JSON |

Use IDs from **Test 2** response or from **`google_connections`** / your app after a successful connect:

```json
{
  "business_id": "YOUR_BUSINESS_UUID",
  "gbp_account_id": "NUMERIC_OR_STRING_ACCOUNT_ID",
  "gbp_location_id": "NUMERIC_OR_STRING_LOCATION_ID",
  "gbp_location_name": "My Shop Name"
}
```

- `gbp_location_name` is optional; if omitted, existing `gbp_location_name` in the row is left unchanged.

**Send request.**

**Expected (success):** `200` and JSON like:

```json
{
  "reviewCount": 12,
  "averageRating": 4.5
}
```

**Expected (Google / API error):** `502` with JSON including **`error`** (human-readable), and often **`googleHttpStatus`**, **`googleError`** (e.g. permission denied if **Google My Business API** is not enabled or not approved for your Cloud project).

**Expected (failure):** `400` for missing fields; `401` for bad user JWT; `404` if there is no **`google_connections`** row for `business_id`.

---

## Related: `sync-reviews` (not part of “connect”, but GBP data)

After a connection exists with **`gbp_account_id`** and **`gbp_location_id`**, scheduled or manual sync uses **`sync-reviews`**, which expects **service role** auth. See **Test 1** in [email-bot-edge-function-tests.md](./email-bot-edge-function-tests.md).

---

## Quick checklist

| Function | Method | Auth | Body summary |
|----------|--------|------|----------------|
| `google-auth-url` | POST | User JWT | `business_id`, `redirect_uri` |
| `google-callback` | POST | User JWT | `code`, `state` |
| `google-import-reviews` | POST | User JWT | `business_id`, `gbp_account_id`, `gbp_location_id`, optional `gbp_location_name` |
| `sync-reviews` | POST | Service role | `{}` (see other doc) |

---

## Troubleshooting

| Symptom | What to check |
|---------|----------------|
| **401** on all three | User **`access_token`** expired or wrong; not using service role for these functions—use a **user** JWT. |
| **`google-auth-url` 500** | Missing **`GOOGLE_CLIENT_ID`**. |
| **`google-callback` 400** from Google | **`GOOGLE_REDIRECT_URI`** mismatch vs the redirect used to obtain `code`; wrong `client_id` / `client_secret`; expired `code` (codes are single-use and short-lived). |
| **`google-import-reviews` 502** with Google message | Reviews API denied or misconfigured—enable **Google My Business API** / complete Google’s Business Profile API access request; confirm OAuth scopes include business management. |
| **Test UI shows generic error** | Open **Edge Functions → [function] → Logs** for stack traces; confirm response body in the panel when available. |

Redeploy after code changes, for example:

`npx supabase functions deploy google-auth-url google-callback google-import-reviews`

`verify_jwt = false` is set in [`supabase/config.toml`](../../supabase/config.toml) for these functions; auth is enforced inside each handler as described above.

---

## Optional: same tests from a terminal

You can `curl` these endpoints with `Authorization: Bearer <USER_ACCESS_TOKEN>` and a JSON body. Do not commit tokens or secrets; use env vars or CI secrets only.
