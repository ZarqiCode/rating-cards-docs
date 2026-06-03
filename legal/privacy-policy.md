---
product: rating.cards
layer: legal
domains: [sms, billing, gbp, email, shop]
auto_sync: false
last_verified: 2026-06-03
---

# Privacy Policy

**Effective date:** June 3, 2026

**Last updated:** June 3, 2026

**Published version:** [https://www.rating.cards/privacy](https://www.rating.cards/privacy)

This Privacy Policy describes how rating.cards ("**we**," "**us**," or "**our**") collects, uses, and shares information when you visit our website at [https://www.rating.cards](https://www.rating.cards), use our web application at [https://app.rating.cards](https://app.rating.cards), purchase hardware at [https://shop.rating.cards](https://shop.rating.cards), interact with our SMS programs, or tap or scan one of our physical review devices at a participating venue.

By using our services, you agree to the practices described in this Privacy Policy. If you do not agree, please do not use our services.

For the rules governing your use of our services, please also read our [Terms of Service](./terms-of-service.md).

---

## 1. Who we are

rating.cards is operated as a sole proprietorship doing business as **rating.cards**, located at:

**1890 Valencia Ave**  
**Carlsbad, CA 92008**  
**United States**

**Contact:** [hello@rating.cards](mailto:hello@rating.cards)

We provide physical NFC and QR review products and an Autopilot software subscription that helps hospitality businesses collect, monitor, and respond to Google Business Profile reviews.

---

## 2. Scope of this policy

This Privacy Policy applies to:

- Visitors to **rating.cards** (our marketing and landing site)
- Account owners who sign up for and use **app.rating.cards**
- Purchasers at **shop.rating.cards** (hardware shop, guest checkout — see Section 3.8)
- Store managers who receive SMS messages from rating.cards (even if they never create an account)
- End customers who tap or scan a rating.cards device at a venue (limited data only — see Section 3.5)

This policy does **not** apply to third-party websites or services you may access through our products, including **Google** (Google Business Profile, Google Reviews) and **Stripe** (payment processing). Those services have their own privacy policies.

---

## 3. Information we collect

### 3.1 Account owner information

When you create an account or subscribe to Autopilot, we collect:

- Name and email address
- Business or organisation name
- Account credentials (managed through Supabase Auth)
- Stripe customer ID and subscription status (payment card details are collected and stored by Stripe, not by us)
- Google OAuth tokens and connected Google account identifiers (when you connect Google Business Profile)
- Brand voice preferences and settings you configure in the app

### 3.2 Manager phone numbers and SMS consent records

When an account owner adds a store manager's phone number for SMS review notifications, we collect:

- The manager's US mobile phone number
- A record that the account owner confirmed consent on the manager's behalf, including:
  - The timestamp of consent (`consent_given_at`)
  - The version of the consent text shown (`consent_text_version`, currently **`sms_v2`**)
- OTP verification status (whether the manager completed phone verification)
- SMS opt-out status (if the manager replies STOP)

The consent text shown to account owners (version `sms_v2`) is:

> *"By providing this number, you confirm the recipient agrees to receive review notification SMS/MMS from rating.cards. Message frequency varies; message and data rates may apply. Reply STOP to opt out."*

Links to our [Privacy Policy](https://www.rating.cards/privacy) and [Terms of Service](https://www.rating.cards/terms) appear alongside this checkbox in the app. Records with `consent_text_version` **`sms_v1`** remain valid for opt-ins collected before this update.

### 3.3 Google Business Profile data

When you connect your Google account, we access and store data from your authorised Google Business Profile listings, including:

- Business listing name, address, and identifiers
- Customer reviews (text, star rating, reviewer name, date)
- Your existing replies to reviews
- OAuth access and refresh tokens (stored securely to maintain the connection)

We use this data solely to sync reviews, draft replies, and post approved replies on your behalf.

### 3.4 SMS message content

For verified manager phone lines, we process:

- Outbound SMS/MMS messages we send (verification codes, review alerts, AI draft previews, confirmation messages)
- Inbound SMS replies from managers (approve, skip, custom reply text, STOP, HELP, and other conversational inputs)

We retain SMS message content to operate the review-approval workflow and for compliance purposes.

### 3.5 End-customer tap data (venue guests)

When a customer at a participating venue taps or scans a rating.cards physical device (card, sticker, stand, or poster), our redirect service logs:

- Device identifier (public ID)
- Timestamp of the tap
- User agent string (browser/device type)
- Approximate country (derived from IP address)

We do **not** set cookies on the guest's device, do **not** collect the guest's name, phone number, or email, and do **not** require the guest to create an account. The guest is redirected directly to the venue's Google Reviews page.

The venue (our customer) is responsible for any guest-facing privacy disclosure required by applicable law in their jurisdiction.

### 3.6 Email communications

If you are an account owner or have an email address on file for notifications, we collect:

- Email address
- Email open/click data (if applicable, via our email provider)
- Unsubscribe preferences

### 3.7 Cookies and similar technologies

We use **essential cookies and local storage only** — no analytics, advertising, or marketing pixels.

| Cookie / technology | Purpose | Provider |
|---------------------|---------|----------|
| Authentication session | Keep you logged in to app.rating.cards | Supabase |
| Stripe checkout session | Process subscription and hardware payments | Stripe |
| Cart persistence (`localStorage`) | Save shop cart contents between visits on shop.rating.cards | rating.cards (browser storage) |

When you complete hardware checkout, Stripe's hosted checkout page may set cookies necessary to process your one-time payment. We do not set analytics or advertising cookies on shop.rating.cards.

We do not use Google Analytics, Meta Pixel, or similar tracking tools on our websites.

### 3.8 Hardware shop purchasers (guest checkout)

When you purchase hardware at **shop.rating.cards** without creating an account, we collect:

- **Business search data** — when you select your business on a product page, we receive data from the Google Places API, including `placeId`, business name, formatted address, city, country, latitude, longitude, and your Google review URL. You submit this with your cart so we can pre-configure each device before shipping.
- **Cart data** — product selections, quantities, and per-line business details stored in your browser's **`localStorage`** until you complete checkout or clear your cart.
- **Checkout and order data** — at Stripe Checkout, you provide your email address, US shipping address, and payment method (payment card details are collected and stored by Stripe, not by us). We receive order metadata (including your business configuration per line item) via the Stripe Checkout Session and in internal fulfillment communications to our team.

We use this information to process your hardware order, configure devices with your review link, ship your order, and respond to support or warranty requests. Shop v1 does **not** create a rating.cards account automatically.

---

## 4. How we use information

We use the information we collect to:

- Provide, operate, and maintain the rating.cards service
- Send transactional SMS messages to verified store managers (verification codes, review alerts, approval conversations)
- Send transactional and informational emails to account owners (weekly pulse summaries, negative review alerts, account notifications)
- Sync reviews from Google Business Profile and post approved replies
- Generate AI-drafted review replies based on your brand voice settings
- Process subscription payments through Stripe
- Process hardware purchases through Stripe Checkout at shop.rating.cards
- Provision, activate, and track physical review devices
- Analyse aggregate tap data for dashboard analytics (tap counts per device)
- Detect, prevent, and address fraud, abuse, and security issues
- Comply with legal obligations and enforce our [Terms of Service](./terms-of-service.md)
- Respond to your support requests and privacy rights requests

We do **not** use personal information for cross-context behavioural advertising. We do **not** sell personal information.

---

## 5. SMS Communications & Consent

This section describes our SMS program in detail. It is designed to comply with the Telephone Consumer Protection Act (TCPA), CTIA messaging guidelines, and Twilio A2P 10DLC requirements.

### 5.1 Program name

**rating.cards Review Notifications**

### 5.2 Who receives messages

SMS/MMS messages are sent to **store managers** at hospitality venues whose phone numbers are added by the venue's account owner (CEO) through the rating.cards dashboard or during onboarding. Managers do not need to create a rating.cards account to receive messages.

Messages are sent to **US mobile numbers only**.

Supported carriers include AT&T, Verizon, T-Mobile, and other US wireless carriers. Carriers are not liable for delayed or undelivered messages.

### 5.3 Types of messages

| Message type | When sent | Example |
|--------------|-----------|---------|
| **Verification OTP** | When a manager phone number is first added | "Your rating.cards verification code is: 123456. Reply STOP to opt out." |
| **Review alert** | When a new Google review is received for a verified storefront | "⭐ New 5-star review for [Business Name]..." |
| **AI draft preview** | When a manager requests an AI-drafted reply via SMS | "Here's your AI-drafted reply: ..." |
| **Approval confirmation** | After a reply is posted or skipped | "✅ Reply posted to Google for [Reviewer]'s 5-star review." |
| **Queue prompt** | When additional reviews are waiting for response | "You have 2 more reviews waiting..." |
| **HELP response** | When a manager replies HELP | Contact information and instructions (email hello@rating.cards) |
| **Opt-out confirmation** | When a manager replies STOP | Confirmation that the number has been unsubscribed |

These are **transactional and service-related messages**. We do not send marketing or promotional SMS messages through this program.

### 5.4 How consent is obtained

Consent for SMS messages is obtained through a **two-step process**:

1. **Account owner representation:** Before adding a manager's phone number, the account owner must check a consent checkbox in the app confirming: *"By providing this number, you confirm the recipient agrees to receive review notification SMS/MMS from rating.cards. Message frequency varies; message and data rates may apply. Reply STOP to opt out."* We record the timestamp and consent text version (`sms_v2`) in our database. Privacy Policy and Terms of Service links are shown alongside the checkbox.

2. **Manager verification:** The manager receives an SMS with a verification link and must complete a one-time OTP (one-time password) confirmation to activate their line. This confirms the phone number is valid and reachable.

**Important:** By submitting a manager's phone number, the account owner **warrants** that they have obtained that manager's **prior express written consent** to receive SMS/MMS messages from rating.cards for the purposes described above. The account owner is responsible for maintaining records of that consent. See our [Terms of Service](./terms-of-service.md) for indemnification obligations.

### 5.5 Message frequency

Message frequency **varies** based on review volume at each storefront. A typical manager line receives **0 to 5 messages per week**, but frequency may be higher during periods of high review activity or when multiple reviews are queued for approval.

Messages are **recurring** for as long as the manager's line remains verified and active.

### 5.6 Opt-out (STOP)

Managers can opt out of SMS messages at any time by replying **STOP** to any rating.cards message. After opting out:

- The phone number is added to our suppression list
- No further SMS messages will be sent to that number from any rating.cards program
- The suppression record is retained indefinitely to honour the opt-out request
- Autopilot review alerts and SMS approval for that storefront will be disabled until a new verified manager line is configured

To re-subscribe after opting out, the account owner must remove and re-add the manager's phone number in Settings, and the manager must complete verification again.

### 5.7 Help (HELP)

Managers can reply **HELP** to any rating.cards message for assistance, or contact us at [hello@rating.cards](mailto:hello@rating.cards).

### 5.8 Message and data rates

**Message and data rates may apply.** Your mobile carrier (not rating.cards) may charge you for receiving SMS/MMS messages depending on your phone plan. Contact your carrier for details about your messaging plan.

Carriers are not liable for delayed or undelivered messages.

---

## 6. Email communications

We send the following emails to account owners:

- **Weekly pulse** — a Monday-morning summary of review activity across your organisation
- **Negative review alerts** — notifications when a low-rated review is received
- **Account notifications** — onboarding confirmations, billing receipts (via Stripe), and service updates

You can unsubscribe from non-essential email notifications at any time by clicking the unsubscribe link in any email, or by contacting us at [hello@rating.cards](mailto:hello@rating.cards). Transactional emails related to your account, billing, or security cannot be opted out of while your account is active.

---

## 7. How we share information

We share personal information only with the following categories of service providers, solely to operate our service:

| Provider | Purpose | Data shared |
|----------|---------|-------------|
| **Supabase** | Database, authentication, edge functions | Account data, reviews, SMS records, device data |
| **Stripe** | Payment processing and subscription management; hardware checkout | Name, email, shipping address, payment method (stored by Stripe) |
| **Google Maps Platform (Places API)** | Business search autocomplete on shop.rating.cards | Business name, address, place ID, and related fields you select |
| **Twilio** | SMS/MMS delivery and receipt | Manager phone numbers, message content |
| **Resend** | Transactional email delivery | Account owner email, email content |
| **Google** | Google Business Profile API (OAuth) | Review data, reply content, listing metadata |
| **Anthropic** | AI draft generation | Review text, brand voice settings (no personal identifiers sent beyond review content) |

We require our service providers to process data only on our instructions and in accordance with applicable privacy laws.

We may also disclose information:

- To comply with law, regulation, legal process, or government request
- To enforce our [Terms of Service](./terms-of-service.md)
- To protect the rights, property, or safety of rating.cards, our users, or the public
- In connection with a merger, acquisition, or sale of assets (with notice to affected users)

We do **not** sell personal information. We do **not** share personal information for cross-context behavioural advertising.

---

## 8. Data retention

We retain personal information according to the following schedule:

| Data category | Retention period |
|---------------|-----------------|
| Account owner profile and settings | While your account is active, plus **90 days** after account closure |
| Manager phone numbers and consent records | While the line is active, plus **90 days** after account closure |
| Google review data and reply history | While your account is active, plus **90 days** after account closure |
| SMS message content | While your account is active, plus **90 days** after account closure |
| Email notification preferences | While your account is active; suppression records retained indefinitely |
| SMS and email opt-out / suppression lists | **Indefinitely** (to prevent re-contacting opted-out individuals) |
| End-customer tap logs (user agent, country) | Retained as aggregate analytics data; not tied to identifiable individuals |
| Hardware order and fulfillment records | Up to **7 years** after the transaction (for tax, audit, and warranty support; payment records stored by Stripe) |
| Billing and payment records | Up to **7 years** after the transaction (for tax and audit purposes; stored by Stripe) |

When your account is closed, we delete or anonymize personal data within 90 days, except where a longer retention period is required by law or needed for billing records and opt-out suppression.

You may request earlier deletion by contacting us at [hello@rating.cards](mailto:hello@rating.cards) (see Section 9).

---

## 9. Your California privacy rights (CCPA/CPRA)

If you are a California resident, you have the following rights under the California Consumer Privacy Act (CCPA) as amended by the California Privacy Rights Act (CPRA):

### 9.1 Right to know

You have the right to request that we disclose:

- The categories of personal information we have collected about you
- The categories of sources from which we collected it
- The business or commercial purpose for collecting it
- The categories of third parties with whom we share it
- The specific pieces of personal information we hold about you

### 9.2 Right to delete

You have the right to request deletion of personal information we hold about you, subject to certain exceptions (e.g., billing records we must retain for tax purposes, opt-out suppression lists).

### 9.3 Right to correct

You have the right to request correction of inaccurate personal information we hold about you.

### 9.4 Right to opt out of sale or sharing

We do **not** sell personal information and do **not** share personal information for cross-context behavioural advertising. We do not offer a "Do Not Sell or Share My Personal Information" link because no such activity occurs.

### 9.5 Right to limit use of sensitive personal information

We do **not** collect sensitive personal information as defined by the CPRA (such as precise geolocation, biometric data, or health information).

### 9.6 Right to non-discrimination

We will not discriminate against you for exercising any of your privacy rights.

### 9.7 How to submit a request

To exercise any of the above rights, email us at [hello@rating.cards](mailto:hello@rating.cards) with the subject line **"Privacy Request"** and include:

- Your full name and email address associated with your account
- The specific right you wish to exercise
- Enough information for us to verify your identity

We will respond to verified requests within **45 days**. If we need additional time (up to 90 days total), we will notify you in writing.

You may designate an authorised agent to submit a request on your behalf by providing written authorisation.

---

## 10. Security

We implement reasonable technical and organisational measures to protect personal information, including:

- Encryption in transit (TLS/HTTPS) for all web and API communications
- Encryption at rest for data stored in Supabase
- Scoped service-role credentials for backend operations
- Access controls limiting who can view production data

However, no method of transmission or storage is 100% secure. We cannot guarantee absolute security of your information.

If you believe your account has been compromised, contact us immediately at [hello@rating.cards](mailto:hello@rating.cards).

---

## 11. Children's privacy

Our services are not directed to individuals under the age of 13, and we do not knowingly collect personal information from children under 13. If we learn that we have collected personal information from a child under 13, we will delete it promptly. If you believe a child has provided us with personal information, contact us at [hello@rating.cards](mailto:hello@rating.cards).

---

## 12. International users

rating.cards is operated from the United States. If you access our services from outside the US, your information will be transferred to, stored, and processed in the United States. By using our services, you consent to this transfer.

Our services are currently available to US-based businesses only.

---

## 13. Changes to this policy

We may update this Privacy Policy from time to time. When we make material changes, we will:

- Post the updated policy on this page with a new effective date
- Notify account owners by email at least 30 days before the changes take effect

Your continued use of our services after the effective date of an updated policy constitutes acceptance of the changes. If you do not agree, you should stop using our services and contact us to close your account.

---

## 14. Contact us

If you have questions about this Privacy Policy or wish to exercise your privacy rights, contact us:

**Email:** [hello@rating.cards](mailto:hello@rating.cards)  
**Mail:** rating.cards, 1890 Valencia Ave, Carlsbad, CA 92008, USA

---

## Changelog

- 2026-06-03: shop.rating.cards scope; hardware guest checkout data (Places, localStorage, Stripe); Google Maps Platform provider; retention for order fulfillment
- 2026-05-23: sms_v2 consent text (frequency + STOP disclosures); published URL; legal links in app opt-in surfaces
- 2026-05-23: Initial Privacy Policy draft covering SMS consent (sms_v1), CCPA/CPRA rights, data retention, and service provider disclosures
