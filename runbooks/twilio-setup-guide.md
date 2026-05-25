# Twilio Setup Guide

This guide walks through everything you need to do — once — to get the rating.cards SMS review bot live in production for US customers.

The SMS bot uses **Twilio** to send and receive MMS messages from a single shared phone number. Owners get notified of every new Google review, and approve / write / skip replies entirely via SMS.

---

## Registration path: Sole Proprietor

We register as a **Direct Customer** using Twilio's **Sole Proprietor** A2P 10DLC path. This fits a pre-LLC startup: one US-based cofounder operating without a federal business Tax ID (no US EIN / Canadian Business Number on this registration).

| | Sole Proprietor (this guide) | Standard brand (future) |
|---|---|---|
| Primary profile | **Individual** (Trust Hub) | **Business** + EIN |
| Business identity | **Direct Customer** — not ISV | Direct Customer |
| Brand type | Sole Proprietor | Standard or Low-Volume Standard |
| Campaign use case | **Sole Proprietor** (only option) | Customer Care, Mixed, etc. |
| Phone numbers per campaign | **1** | Multiple |
| Daily US volume cap | ~3,000 segments (~1,000/day to T-Mobile) | Much higher (Trust Score) |
| One-time fees | ~$4 brand + ~$15 campaign vetting | ~$44 brand + vetting |

When rating.cards forms an **LLC and obtains an EIN**, [transition to a Standard brand](https://www.twilio.com/docs/messaging/compliance/a2p-10dlc/transition-sole-proprietor-to-standard-brand) before volume or carrier limits become a problem.

References: [Sole Proprietor onboarding](https://www.twilio.com/docs/messaging/compliance/a2p-10dlc/direct-sole-proprietor-registration-overview), [Primary compliance profiles](https://www.twilio.com/docs/trust-hub/profiles/primary-compliance-profiles), [Brand type comparison](https://help.twilio.com/hc/en-us/articles/4407882914971).

---

## What you'll set up

1. A Twilio account
2. A **primary compliance profile** (Individual / sole proprietor)
3. A2P 10DLC **Sole Proprietor brand + campaign** (includes a Messaging Service)
4. **One** US 10DLC long-code phone number (SMS + MMS) attached to that Messaging Service
5. The webhook that points inbound messages back at our `handle-sms-reply` Edge Function
6. The four secrets that the Edge Functions read from Supabase

When you finish, every new review will trigger an MMS to the verified business owner, and their replies will route back to our handler.

---

## 1. Create the Twilio account

1. Go to [https://www.twilio.com/try-twilio](https://www.twilio.com/try-twilio) and sign up.
2. Verify your email and personal phone number when prompted.
3. From the onboarding questionnaire, choose:
   - "Send a notification" / "Send messages"
   - "With code"
   - Language: **Node.js** (the closest Deno-compatible answer; this only affects the docs Twilio surfaces — our code uses raw `fetch`).
4. You'll land on the [Twilio Console](https://console.twilio.com/). Note down two values from the **Account Info** card:
   - **Account SID** (starts with `AC...`)
   - **Auth Token** (click "Show" to reveal)

Keep these somewhere safe — they go into Supabase secrets later.

> **Use a fresh account if needed.** Each account supports only one primary compliance profile. If an earlier attempt locked you into the wrong **Business / ISV** path, start over on a new Twilio account rather than stacking conflicting profiles.

---

## 2. Create the primary compliance profile (Individual)

This is the Trust Hub identity step. For sole proprietor A2P, create an **Individual** primary profile — not a Business profile (Business requires EIN, business type, and ISV vs Direct choices that don't apply here).

1. Console: **Products & services → Trust Hub → [Profiles](https://console.twilio.com/us1/develop/trusthub/compliance-profiles/primary)** → **Primary profile**.
2. Click **Individual Profile** (not Business Profile).
3. **General information:** profile-friendly name, email, phone.
4. **Individual details:** legal first/last name, date of birth, physical **US or Canadian** address (Carlsbad, CA address for the US-based cofounder is appropriate if that's where the proprietor operates).
5. Upload **government-issued photo ID** and complete **selfie verification**.
6. Submit for review. Status moves to **In Review** (typically up to ~48 hours). You can often continue A2P onboarding while this processes, but some Console steps may wait on approval.

**Authorized representative fields** (if shown during A2P onboarding rather than Trust Hub): the sole proprietor is the representative. Use a real **business title** (e.g. `Founder`, `Owner`, `CEO`) and **job position** `CEO` or `Other` as appropriate.

> **Do not select ISV, Reseller, or Partner** anywhere in this flow. rating.cards sends messages as **our own business** to **our subscribing customers** — that is **Direct Customer** behavior. ISV is for embedding Twilio in a platform where end-customer brands register separately.

---

## 3. Register A2P 10DLC (Sole Proprietor brand + campaign)

US carriers require every business that sends Application-to-Person SMS to register a **brand** and at least one **campaign**. Unregistered traffic gets filtered or dropped.

Console entry point: **Messaging → Regulatory Compliance → A2P 10DLC** (or the **A2P 10DLC** tile), then follow the [Sole Proprietor onboarding wizard](https://console.twilio.com/us1/develop/sms/regulatory-compliance/a2p-onboarding).

### 3a. Eligibility confirmation

When prompted:

- **Located in US or Canada:** Yes
- **Business Tax ID (EIN / Canadian BN):** **No** — required for this path. If you already have an EIN for this business, you must use [Standard onboarding](https://www.twilio.com/docs/messaging/compliance/a2p-10dlc/direct-standard-onboarding) instead.
- Confirm you want to continue as **Sole Proprietor** (one-time ~$4 brand fee).

### 3b. Register the Sole Proprietor brand

- **Brand type:** Sole Proprietor (only option).
- **Brand name:** the proprietor's **legal first + last name**, or a **DBA** you actually operate under (e.g. `rating.cards` if that's your registered or commonly used trade name). Must align with your ID and website.
- **Business vertical (optional):** e.g. `TECHNOLOGY` or `PROFESSIONAL`.
- **Mobile number for brand OTP:** a personal **US or Canadian mobile** belonging to the sole proprietor. **Not** a Twilio number or other CPaaS line. Twilio sends a one-time SMS; **reply within 24 hours** or re-trigger OTP from the Console. This mobile can be used at most **3 times** across all TCR Sole Proprietor registrations.

Brand approval is usually minutes after OTP confirmation.

### 3c. Register the campaign

Sole Proprietor brands have **one campaign use case only:** **Sole Proprietor** (not "Customer Care" — describe the actual use case in the free-text fields).

- **Messaging Service:** if you **already created one in step 4** while buying your number, choose **Select existing Messaging Service** and pick that service. Otherwise choose **Create new Messaging Service** here (Twilio creates it on submit). Either way, use **one service only** — do not create a second one.
- **Campaign description** (be thorough; single-word answers get rejected):

  > rating.cards sends transactional SMS/MMS to store managers per storefront who opted in during onboarding or in Settings → Notifications at app.rating.cards. Messages include new Google review alerts, AI-drafted reply previews, OTP verification codes, and two-way approval prompts (reply 1/2/3). Managers can opt out anytime by replying STOP.

- **Sample messages** — paste these (must match what we send; include brand + opt-out):

  **Sample 1 (review alert):**
  ```
  rating.cards: New 4-star review for Joe's Cafe

  "Great coffee and lovely atmosphere!" — Sarah M.

  Reply:
  1 - Generate AI reply
  2 - Write your own
  3 - Skip

  Reply STOP to opt out.
  ```

  **Sample 2 (verification code):**
  ```
  rating.cards: Your verification code is 482917. Reply STOP to opt out.
  ```

- **Message flow / how end-users consent** (required, 40+ characters):

  > End users are store managers at hospitality businesses using rating.cards. Opt-in happens at https://app.rating.cards during onboarding (step 3) or in Settings → Notifications. For each storefront, the subscribing account owner enters the manager's US mobile number and checks a box confirming the recipient agrees to receive transactional SMS/MMS from rating.cards for Google review alerts (message frequency varies; message and data rates may apply; reply STOP to opt out). Before any review alerts are sent, the phone must be verified: (1) the account owner verifies inline with a 6-digit OTP in the app, or (2) the account owner sends the manager a one-time SMS link to https://app.rating.cards/verify-sms/[token], where the manager enters a 6-digit OTP. Consent timestamp and consent text version are stored per storefront. Terms of Service: https://www.rating.cards/terms. Privacy Policy: https://www.rating.cards/privacy. Users opt out by replying STOP; replying START re-enables alerts.

- **Opt-in keywords:** leave blank if opt-in is web-only, or `START`, `UNSTOP`, `YES` if you support SMS opt-in.
- **Opt-out / help:** use Twilio's **default or Advanced Opt-out** (recommended) — keywords `STOP`, `HELP`, etc. are handled automatically.
- **Embedded links / phone numbers:** **Yes** if review text may contain URLs or phone numbers.

Submit the campaign. **Manual vetting** applies (~$15 one-time vetting fee; approval can take **days to weeks**). Messages may send while **Pending** but carriers can still filter until status is **Verified / Approved**.

---

## 4. Buy a 10DLC phone number and create the Messaging Service

Sole Proprietor campaigns allow **exactly one** US 10DLC number on **one** Messaging Service. Our architecture uses one shared number for all customers — this fits.

You can reach this step from **Phone Numbers → Buy a number** or from the **A2P 10DLC onboarding wizard** after brand registration. Both paths end with the same **“Select or create a messaging service”** screen.

### 4a. Search for a number

1. Console: **Phone Numbers → Manage → Buy a number** (or continue the A2P onboarding flow if it brought you here).
2. Filters:
   - Country: **United States**
   - Capabilities: check **SMS** and **MMS**
   - Type: **Local** (US 10-digit long code)
3. Pick a number and proceed to checkout (~$1.15/month).

If purchase is blocked pending compliance, ensure step 2 (primary profile) and step 3b (brand + OTP) are submitted first.

### 4b. Select or create a Messaging Service *(you are here)*

After choosing a number, Twilio shows **“Select or create a messaging service”** — this links the long code to A2P 10DLC.

Read the notice on screen:

> A messaging service can only be linked to a **single** A2P campaign. Once linked to a campaign + brand, it cannot be repurposed — create a **new** service if you ever register a second campaign.

**What to choose:**

| Situation | Action |
|---|---|
| First time setup (no Messaging Service yet) | Leave **Create New Messaging Service** selected |
| You already created a service in step 3c | Switch dropdown to **Select existing Messaging Service** and pick that one — do **not** create a duplicate |

**Messaging service friendly name:** use something you'll recognize in the Console, e.g.

```
rating-cards-a2p
```

(Cosmetic label only — customers never see it.)

Click **Complete** to finish the purchase. Twilio creates the Messaging Service (if new) and adds this number as its **only sender**.

### 4c. Confirm the setup

1. **Phone Numbers → Manage → Active numbers** — your new `+1…` number should appear.
2. **Messaging → Services → `rating-cards-a2p`** (or whatever you named it):
   - **Sender Pool** should list **exactly one** phone number.
   - Do **not** add a second number — Sole Proprietor campaigns only verify one 10DLC route.

### 4d. Tie the service to your A2P campaign

If you **haven't submitted step 3c yet**, go back to **Messaging → Regulatory Compliance → A2P 10DLC** and register the campaign:

- **Messaging Service:** select the service you just created (`rating-cards-a2p`) — not a second new one.
- Fill in campaign description, sample messages, and message flow (see step 3c), then submit.

If you **already submitted the campaign** with a **different** Messaging Service, either:

- Re-use that original service and move your number to it (**Messaging → Services → Add Senders** on the campaign's service, remove it from the duplicate), or
- Delete the empty duplicate service and re-register the campaign against the service that holds your number.

Only the Messaging Service linked to your **Approved / Pending** Sole Proprietor campaign counts for carrier delivery.

### 4e. Copy the number for later

Note the number in **E.164** format (e.g. `+17605551234`) — you'll need it for step 6 (`TWILIO_PHONE_NUMBER`).

> **Why a local 10DLC number and not toll-free / short code?**
> Toll-free is fine but slower to register. Short codes are weeks of approval and overkill for our volume. 10DLC is the standard choice and supports SMS + MMS.

---

## 5. Configure the inbound webhook

This is what makes the SMS bot two-way: every reply from a business owner becomes a POST to our Edge Function.

1. Console: **Phone Numbers → Manage → Active numbers → click your number**.
2. Scroll to the **Messaging Configuration** section.
3. **A message comes in**:
   - Webhook
   - URL: `https://<your-supabase-project-ref>.supabase.co/functions/v1/handle-sms-reply`
     (find your project ref under **Supabase Dashboard → Project Settings → General**)
   - HTTP method: **HTTP POST**
4. Leave **Primary handler fails** as **HTTP POST** with no fallback URL (Twilio will retry automatically).
5. Click **Save configuration**.

Inbound messages will now hit `handle-sms-reply`, which validates the `X-Twilio-Signature` header (HMAC-SHA1 over the URL + form fields) using `TWILIO_AUTH_TOKEN` to confirm Twilio actually sent the request.

---

## 6. Set Edge Function secrets in Supabase

Our Edge Functions read four secrets at runtime. Set them in **Supabase Dashboard → Project Settings → Edge Functions → Secrets**:

| Secret | Value |
|---|---|
| `TWILIO_ACCOUNT_SID` | Account SID from step 1 (starts with `AC...`) |
| `TWILIO_AUTH_TOKEN` | Auth Token from step 1 |
| `TWILIO_PHONE_NUMBER` | The number you bought in step 4, in **E.164** format (e.g. `+15125551234`) |
| `TWILIO_SKIP_SIGNATURE` | `false` in production. Set to `true` only for local testing where Twilio cannot reach you. |

These are picked up automatically the next time each function runs.

> The existing `RESEND_API_KEY`, `UNSUBSCRIBE_SECRET`, `ANTHROPIC_API_KEY`, and Stripe secrets stay in place — the email-driven weekly pulse still uses them.

---

## 7. Apply the database migration

If you haven't already, apply `supabase/migrations/sms_review_bot.sql` to your Supabase project:

```bash
# Local dev
supabase db push

# Or run it manually via Dashboard → SQL Editor
```

After the migration runs, regenerate types:

```bash
npm run gen:types
```

(The repo already ships hand-edited types matching the migration, so this is a no-op until you change schema.)

---

## 8. Deploy the Edge Functions

Deploy the four new functions and the modified `respond-to-review`:

```bash
supabase functions deploy send-review-sms
supabase functions deploy handle-sms-reply
supabase functions deploy send-otp-sms
supabase functions deploy verify-otp-sms
supabase functions deploy respond-to-review
```

The deprecated `send-negative-alert` and `approve-reply` can stay deployed — they're no longer called but having them online avoids broken behaviour for any in-flight email approval URLs from before the cutover.

---

## 9. End-to-end smoke test

1. **Phone verification flow** — go through `/onboarding` (step 3) or **Settings → Notifications**, enter a manager phone, accept consent (Privacy Policy and Terms links shown), and complete verification via inline OTP or manager link. You should receive a 6-digit code within 5 seconds when using inline OTP.
2. **Review alert flow** — manually insert a review in the SQL editor, or wait for `sync-reviews` to pull a real one in. Within ~10s you should receive an MMS:
   ```
   ⭐ New X-star review for <Business>
   "<text>" — <Reviewer>
   Reply:
   1 - Generate AI reply
   2 - Write your own
   3 - Skip
   ```
3. **Reply "1"** → you should receive the AI draft + a second 1/2/3 prompt.
4. **Reply "1" again** → confirmation MMS, and the reply appears under the review on Google Business Profile within seconds.
5. **Reply "STOP"** to your test number → check `sms_notification_contacts.is_active` flips to `false`. **Reply "START"** to re-enable.

If any step hangs, check **Supabase Dashboard → Edge Functions → Logs** for the relevant function and **Twilio Console → Monitor → Logs → Messaging** for delivery / inbound webhook status.

---

## TCPA compliance checklist

We're sending automated commercial SMS to US numbers. Before going live, confirm:

- [ ] **Explicit opt-in** is collected before any message — onboarding's consent checkbox + verified OTP satisfies this.
- [ ] **Opt-in record retention** — `sms_notification_contacts.consent_given_at` and `consent_text_version` are stored and immutable.
- [ ] **Disclosure language** — onboarding and Settings consent checkbox states message frequency, that data rates may apply, how to opt out (STOP), and links to https://www.rating.cards/privacy and https://www.rating.cards/terms; `/verify-sms/:token` shows the same disclosures for managers.
- [ ] **STOP / HELP** are honoured — Twilio handles `STOP` natively at the carrier level and our webhook also flips `is_active = false` so we stop trying to send.
- [ ] **A2P 10DLC campaign** is **Approved / Verified** (not Pending) before any production traffic.
- [ ] **No marketing without separate consent** — review notifications are transactional/relational; if you ever add promotional SMS, you'll need a new brand registration (Standard) with a separate campaign.
- [ ] **Volume within Sole Proprietor limits** — ~3,000 SMS/MMS segments/day to the US across all customers on the shared number; plan LLC + Standard brand before approaching this cap.

---

## Cost estimate

For a typical business getting **50 reviews/month** (shared infrastructure across all customers):

| Item | Cost |
|---|---|
| Phone number rental | $1.15 / month |
| A2P Sole Proprietor brand | ~$4 one-time |
| Campaign vetting | ~$15 one-time |
| A2P campaign fee | ~$2 / month |
| Outbound MMS (3 per review × 50) | 150 × $0.02 = $3.00 / month |

**~$6/month** operating cost for the shared Twilio stack, plus ~$19 one-time registration fees. Per-customer SMS cost scales with review volume; the number and campaign are shared.

---

## Troubleshooting

**Brand OTP never arrived or expired.**
The Sole Proprietor **brand** OTP (step 3b) is separate from our app's 6-digit onboarding OTP. Use a real US/CA mobile, not Twilio. Re-trigger from **Messaging → Regulatory Compliance → A2P 10DLC** if the 24-hour window passed.

**Console asks for EIN / Business type / ISV.**
You're on the **Business** or **ISV** path, not Sole Proprietor. Back out and use **Individual** primary profile + Sole Proprietor A2P wizard with **Direct Customer** (never ISV).

**Can't buy a phone number — compliance required.**
Finish step 2 (primary profile submitted) and step 3a–3b (brand + OTP) first, then retry. Attach the number in step 4 after the Messaging Service exists.

**Inbound replies aren't reaching the bot.**
Check Twilio's webhook logs (Console → Monitor → Logs → Messaging → click an inbound message → "Request Inspector"). 403 means our signature check rejected it — verify `TWILIO_AUTH_TOKEN` is exactly the value Twilio shows. 5xx means our function errored — check Supabase Edge Function logs.

**Messages send but never arrive.**
Look up the message in Twilio Logs. If status is `failed` with error `30007` ("Carrier violation"), your A2P 10DLC campaign isn't approved yet or the message content triggered a content filter (try shortening, removing all-caps, removing risky keywords).

**Verification codes always say "expired".**
The OTP has a 5-minute window. If your test SMS takes longer than that to arrive (common during 10DLC pre-approval throttling), request a new one.

**`Twilio is not configured` errors in Edge Function logs.**
The three `TWILIO_*` secrets are missing or misnamed in Supabase. Re-check **Project Settings → Edge Functions → Secrets**.

**Hitting daily send limits.**
Sole Proprietor caps at ~1,000 segments/day to T-Mobile (~3,000/day US total). Register an LLC, obtain an EIN, and [transition to a Standard brand](https://www.twilio.com/docs/messaging/compliance/a2p-10dlc/transition-sole-proprietor-to-standard-brand) before scaling further.
