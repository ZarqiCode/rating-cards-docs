# Email bot — Edge Function tests (Supabase Dashboard)

Use the **built-in test UI** for Edge Functions so you do not need `curl` or local env vars in the terminal. Supabase sends the request from your browser and shows the response in the same panel.

## Where to open the test UI

1. Open your project in the [Supabase Dashboard](https://supabase.com/dashboard).
2. Go to **Edge Functions**.
3. Click the function name (e.g. `sync-reviews`).
4. Open **Test** / **Invoke** (wording may vary) to open the modal with **HTTP Method**, **Request Body**, **Headers**, **Query Parameters**, **Role**, and **Send request**.

The screenshot you have matches this flow: method at the top, body, headers, query params, **Role** at the bottom, then **Send request**.

## Role: service role vs anon

These functions expect the **Authorization** header to match your project’s **service role** JWT (same check as `Bearer <SUPABASE_SERVICE_ROLE_KEY>` in code).

- For **`sync-reviews`**, **`send-weekly-pulse`**, and **`send-negative-alert`**: set **Role** to **`service role`**. The test UI injects the correct bearer token so you do **not** need to paste the key into **Headers** manually.
- For **`handle-unsubscribe`**: the function does **not** use that check; use **GET** and pass **`token`** as a query parameter. You can leave Role as **anon** or **service role**; it does not change auth for this handler.

---

## Test 1 — `sync-reviews`

| Field | Value |
|--------|--------|
| **HTTP Method** | `POST` |
| **Request Body** | `{}` |
| **Role** | `service role` |
| **Headers** | None required (UI adds auth when Role is service role) |
| **Query Parameters** | None |

**Send request.**

**Expected (success):** JSON such as `{ "synced": <number>, "businesses": <number> }`, or `{ "synced": 0, "businesses": 0, "message": "No active connections" }` if no Google Business Profile connections exist yet.

**If you see `401`:** Check the response body for a `stage` field. `wrong_role` means the JWT's `role` claim is not `service_role` — set **Role** to **`service role`** in the test UI. `missing_header` or `bad_scheme` means the `Authorization: Bearer` header was not attached.

---

## Test 2 — `send-weekly-pulse`

| Field | Value |
|--------|--------|
| **HTTP Method** | `POST` |
| **Request Body** | `{}` |
| **Role** | `service role` |
| **Headers** | None required |
| **Query Parameters** | None |

**Send request.**

**Expected:** JSON like `{ "sent": <number>, "failed": <number> }`, or `{ "sent": 0, "message": "No active contacts" }` if no row in `email_notification_contacts` with `is_active = true`.

Also check **Table Editor → `email_logs`** after a successful send, and your inbox if Resend is configured.

---

## Test 3 — `send-negative-alert`

| Field | Value |
|--------|--------|
| **HTTP Method** | `POST` |
| **Role** | `service role` |
| **Request Body** | JSON object below (replace IDs with real values from your database) |
| **Headers** | `Content-Type: application/json` is usually set automatically when you send JSON body; add it if the UI does not. |

**Example body** (use a **new** `review_id` UUID each time you want a real email; reuse the same `review_id` to verify deduplication):

```json
{
  "review_id": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
  "business_id": "YOUR_BUSINESS_UUID",
  "reviewer_name": "Test Reviewer",
  "rating": 1,
  "review_text": "Manual test from Supabase UI.",
  "review_date": "2026-03-27T12:00:00.000Z"
}
```

- `business_id` must exist in `businesses`.
- For an email to actually send, that business needs **`email_notification_contacts`** with **`is_active = true`** and **`RESEND_API_KEY`** set on the function.

**Expected:**

- `{ "sent": true, "resend_id": "..." }` — email path worked.
- `{ "skipped": true, "reason": "No active email contact" }` — function works; fix contact row to test delivery.
- `{ "skipped": true, "reason": "Alert already sent for this review" }` — same `review_id` already logged in `email_logs` (dedup).

---

## Test 4 — `handle-unsubscribe`

This function only accepts **GET** and reads **`token`** from the query string.

### 4a — Smoke test (invalid / missing token)

| Field | Value |
|--------|--------|
| **HTTP Method** | `GET` |
| **Query Parameters** | Leave empty **or** add a dummy `token` = `invalid` |
| **Role** | `anon` is fine |
| **Request Body** | Empty (not used for GET) |

**Send request.**

**Expected:** Status **400** and HTML containing a message like invalid/expired link. Confirms the function is deployed and returns HTML errors correctly.

### 4b — Full test (valid token)

You need a **valid** HMAC token (same algorithm as in `handle-unsubscribe` / email templates). Easiest path: send yourself a **Weekly Pulse** or **negative alert** email and copy the unsubscribe URL from the footer, then either:

- Open that URL in a **browser**, or  
- In the test UI: **GET**, **Query Parameters** → `token` = paste **only** the token value from the URL (the part after `token=`; URL-decode if needed).

**Expected:** Status **200** and HTML “You’ve been unsubscribed.” Confirm in **Table Editor** that `email_notification_contacts.is_active` is `false` for that business.

---

## Quick checklist

| Function | Method | Role | Body / query |
|----------|--------|------|----------------|
| `sync-reviews` | POST | service role | `{}` |
| `send-weekly-pulse` | POST | service role | `{}` |
| `send-negative-alert` | POST | service role | JSON with `review_id`, `business_id`, … |
| `handle-unsubscribe` | GET | anon OK | Query: `token` (optional for 4a; required valid for 4b) |

---

## If `send-weekly-pulse` returns 500

The test UI sometimes only shows **“Edge function returned an error”** without the JSON body. Use both of these:

1. **Response body (if visible)** — The handler returns JSON with a **`stage`** field:
   - **`contacts_query`** — Postgres/PostgREST error loading `email_notification_contacts` (e.g. missing `is_active` column). The body may include **`code`**, **`details`**, **`hint`** from Supabase.
   - **`env`** — Missing **`RESEND_API_KEY`**, **`UNSUBSCRIBE_SECRET`**, or **`SUPABASE_URL`** in Edge Function secrets.
   - **`unhandled`** — Unexpected error; see logs.

2. **Edge Function logs** — **Edge Functions** → **`send-weekly-pulse`** → **Logs**. Look for lines that are JSON objects with `"fn":"send-weekly-pulse"` and a **`stage`** (`contacts_query`, `business_iteration`, `email_logs_insert_failed`, `unhandled`, etc.). **`business_iteration`** includes **`message`** and **`stack`** for per-business failures (Resend, template render, missing secret at runtime).

Redeploy the function after code changes: `npx supabase functions deploy send-weekly-pulse` (the `verify_jwt = false` setting is declared in `supabase/config.toml`).

**Prettier / `createRequire` errors in logs:** Email HTML is built with **`react-dom/server`** `renderToStaticMarkup` in [`renderEmail.ts`](../../supabase/functions/_shared/emails/renderEmail.ts), not `@react-email/render` (that package bundles Prettier and breaks on Edge). Redeploy **`send-weekly-pulse`** and **`send-negative-alert`** after template changes.

---

## Optional: same tests from a terminal

If you prefer scripts or CI, you can still use `curl` with `Authorization: Bearer <SERVICE_ROLE_KEY>` for POST functions and a GET URL with `?token=` for unsubscribe. Do not commit secrets; use env vars or CI secrets only.
