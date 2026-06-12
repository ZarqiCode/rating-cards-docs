# AI Review Responder — Testing Guide

Since Google Business Profile API access is pending approval, this guide covers how to test the full AI responder pipeline using simulated data. Everything except the actual Google reply POST can be validated end-to-end.

---

## Prerequisites

Before testing, ensure the following are in place:

1. **Anthropic API key** stored in Supabase Vault:
   ```sql
   SELECT vault.create_secret('<your-anthropic-api-key>', 'anthropic_api_key');
   ```
   The edge function reads it via `Deno.env.get('ANTHROPIC_API_KEY')`.

2. **A test business** with:
   - An active subscription (`subscriptions.status = 'active'`)
   - Brand voice settings configured (`brand_voice_settings` row with required v2 fields: `business_type`, `formality`, `emoji_usage`, `criticism_style`, `signature_type`, and `signature_name` when sign-off is owner name)
   - An email notification contact (`email_notification_contacts` row with `is_active = true`)

3. **Resend API** working (for negative alert emails with AI draft).

4. **Dry-run mode enabled**: Set the `AI_RESPONDER_DRY_RUN` environment variable to `true` in your Supabase Edge Function secrets:
   ```bash
   supabase secrets set AI_RESPONDER_DRY_RUN=true
   ```
   This skips the Google reply PUT but runs everything else (AI generation, DB updates, emails).

5. **Edge functions deployed**: `respond-to-review`, `approve-reply`, `retry-failed-replies`.

---

## Finding your test business ID

```sql
SELECT b.id AS business_id, b.business_name, b.city,
       s.status AS subscription_status,
       bv.business_type, bv.formality, bv.emoji_usage, bv.criticism_style, bv.signature_type
FROM businesses b
LEFT JOIN subscriptions s ON s.business_id = b.id
LEFT JOIN brand_voice_settings bv ON bv.business_id = b.id
WHERE b.business_name IS NOT NULL
ORDER BY b.created_at DESC
LIMIT 5;
```

Replace `<BUSINESS_ID>` in all SQL below with the `business_id` from this query. --> d9c52250-8db1-4d63-b976-575791c8d873

---

## Test scenarios

### Test 1: Positive review with text (5 stars) — auto-post

**Expected**: AI generates a warm reply, status set to `posted` (dry-run skips Google PUT).

```sql
INSERT INTO reviews (business_id, google_review_id, reviewer_name, rating, review_text, review_date, is_initial_import, has_owner_reply)
VALUES (
  '<BUSINESS_ID>',
  'test-positive-5star-' || gen_random_uuid(),
  'Maria Garcia',
  5,
  'Amazing coffee and the staff was so friendly! Will definitely be back.',
  now(),
  false,
  false
);
```

### Test 2: Mid-range review with text (3 stars) — auto-post

**Expected**: AI generates a balanced reply, status set to `posted`.

```sql
INSERT INTO reviews (business_id, google_review_id, reviewer_name, rating, review_text, review_date, is_initial_import, has_owner_reply)
VALUES (
  '<BUSINESS_ID>',
  'test-mid-3star-' || gen_random_uuid(),
  'Alex Johnson',
  3,
  'Coffee was decent but the place was quite noisy. Might try again on a quieter day.',
  now(),
  false,
  false
);
```

### Test 3: Negative review with text (1 star) — draft + email alert

**Expected**: AI generates an empathetic draft, status set to `draft`, negative alert email sent WITH the AI draft and "Approve & Post" button.

```sql
INSERT INTO reviews (business_id, google_review_id, reviewer_name, rating, review_text, review_date, is_initial_import, has_owner_reply)
VALUES (
  '<BUSINESS_ID>',
  'test-negative-1star-' || gen_random_uuid(),
  'John Smith',
  1,
  'Waited 30 minutes and the coffee was cold. Very disappointing service.',
  now(),
  false,
  false
);
```

**Check**: Look in your email inbox for the negative alert. It should include the AI-drafted reply and an "Approve & Post" button.

### Test 4: Review without text (4 stars)

**Expected**: AI generates a short thank-you even though there is no review text.

```sql
INSERT INTO reviews (business_id, google_review_id, reviewer_name, rating, review_text, review_date, is_initial_import, has_owner_reply)
VALUES (
  '<BUSINESS_ID>',
  'test-notext-4star-' || gen_random_uuid(),
  'Anonymous',
  4,
  '',
  now(),
  false,
  false
);
```

### Test 5: Review with existing owner reply — should skip

**Expected**: Trigger fires but `respond-to-review` sees `has_owner_reply = true` and sets status to `skipped`.

```sql
INSERT INTO reviews (business_id, google_review_id, reviewer_name, rating, review_text, review_date, is_initial_import, has_owner_reply)
VALUES (
  '<BUSINESS_ID>',
  'test-ownerreply-' || gen_random_uuid(),
  'Lisa Brown',
  5,
  'Loved it!',
  now(),
  false,
  true
);
```

### Test 6: Initial import review — trigger should NOT fire

**Expected**: No trigger fires, no AI processing at all.

```sql
INSERT INTO reviews (business_id, google_review_id, reviewer_name, rating, review_text, review_date, is_initial_import, has_owner_reply)
VALUES (
  '<BUSINESS_ID>',
  'test-import-' || gen_random_uuid(),
  'Old Reviewer',
  5,
  'Great place!',
  '2025-01-15T10:00:00Z',
  true,
  false
);
```

### Test 7: Business without brand voice — should skip

**Expected**: `respond-to-review` detects no brand voice settings and sets status to `skipped`.

First, find or create a business WITHOUT a `brand_voice_settings` row, then insert a review for it.

### Test 8: Business without subscription — should skip

**Expected**: `respond-to-review` detects no active subscription and skips.

Create a review for a business that has no row in `subscriptions` (or status = `canceled`).

### Test 9: Negative review (2 stars) — verify email content

**Expected**: Same as Test 3 but at 2-star threshold. Validates the rating boundary.

```sql
INSERT INTO reviews (business_id, google_review_id, reviewer_name, rating, review_text, review_date, is_initial_import, has_owner_reply)
VALUES (
  '<BUSINESS_ID>',
  'test-negative-2star-' || gen_random_uuid(),
  'Carlos Mendez',
  2,
  'The latte art was nice but the espresso was way too bitter. Not worth the price.',
  now(),
  false,
  false
);
```

### Test 10: Multilingual review — reply should match language

**Expected**: AI detects the language and replies in the same language.

```sql
INSERT INTO reviews (business_id, google_review_id, reviewer_name, rating, review_text, review_date, is_initial_import, has_owner_reply)
VALUES (
  '<BUSINESS_ID>',
  'test-spanish-' || gen_random_uuid(),
  'Elena Ruiz',
  5,
  'El mejor cafe de la ciudad! El ambiente es increible y los baristas son muy amables.',
  now(),
  false,
  false
);
```

---

## Direct HTTP testing (bypass trigger)

Call `respond-to-review` directly to iterate on prompt quality without inserting rows:

```bash
curl -X POST https://<PROJECT_REF>.supabase.co/functions/v1/respond-to-review \
  -H "Authorization: Bearer <SERVICE_ROLE_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "review_id": "<UUID_FROM_REVIEWS_TABLE>",
    "business_id": "<BUSINESS_ID>",
    "reviewer_name": "Test User",
    "rating": 5,
    "review_text": "Lovely place, great vibes and amazing pastries!",
    "review_date": "2026-04-14T10:00:00Z",
    "google_review_id": "test-direct-call"
  }'
```

**Note**: When calling directly, the review must already exist in the DB (the function updates `ai_reply_text` and `ai_reply_status` on the row).

---

## Testing the approval flow

After a 1-2 star test review is processed:

1. Check your email for the negative alert
2. Click the "Approve & Post" link
3. In dry-run mode, it will update the status to `posted` without calling Google
4. Verify the status change:

```sql
SELECT id, ai_reply_text, ai_reply_status, ai_reply_sent_at
FROM reviews
WHERE business_id = '<BUSINESS_ID>'
AND rating <= 2
AND ai_reply_status = 'posted'
ORDER BY created_at DESC;
```

Alternatively, test the approval endpoint directly:

```bash
curl "https://<PROJECT_REF>.supabase.co/functions/v1/approve-reply?token=<TOKEN_FROM_EMAIL>"
```

---

## Testing the retry cron

1. Manually set a review to `failed` status:

```sql
UPDATE reviews
SET ai_reply_status = 'failed'
WHERE google_review_id LIKE 'test-%'
AND ai_reply_status = 'posted'
LIMIT 1;
```

2. Call the retry function:

```bash
curl -X POST https://<PROJECT_REF>.supabase.co/functions/v1/retry-failed-replies \
  -H "Authorization: Bearer <SERVICE_ROLE_KEY>" \
  -H "Content-Type: application/json" \
  -d '{}'
```

3. Verify the review was reprocessed:

```sql
SELECT id, ai_reply_text, ai_reply_status
FROM reviews
WHERE business_id = '<BUSINESS_ID>'
AND google_review_id LIKE 'test-%'
ORDER BY created_at DESC;
```

---

## Verification queries

### See all AI-generated replies

```sql
SELECT
  reviewer_name,
  rating,
  LEFT(review_text, 60) AS review_excerpt,
  LEFT(ai_reply_text, 80) AS reply_excerpt,
  ai_reply_status,
  ai_reply_sent_at,
  has_owner_reply,
  is_initial_import,
  created_at
FROM reviews
WHERE business_id = '<BUSINESS_ID>'
AND google_review_id LIKE 'test-%'
ORDER BY created_at DESC;
```

### Count by status

```sql
SELECT ai_reply_status, COUNT(*)
FROM reviews
WHERE business_id = '<BUSINESS_ID>'
AND google_review_id LIKE 'test-%'
GROUP BY ai_reply_status;
```

### Check email logs for negative alerts

```sql
SELECT email_type, recipient, status, metadata, created_at
FROM email_logs
WHERE business_id = '<BUSINESS_ID>'
AND email_type = 'negative_alert'
ORDER BY created_at DESC
LIMIT 5;
```

---

## Cleanup

After testing, remove test reviews:

```sql
DELETE FROM reviews
WHERE google_review_id LIKE 'test-%';
```

---

## What to test once GBP API access is granted

When Google approves the My Business API access:

1. Remove the `ai_responder_dry_run` Vault secret (or set it to `false`)
2. Insert a test review as above
3. Check the Google Business Profile to verify the reply actually appears
4. Test the approval flow for a 1-2 star review end-to-end
5. Verify that `has_owner_reply` is correctly captured by `sync-reviews` for reviews that already have manual replies on Google
