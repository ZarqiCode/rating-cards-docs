---
product: rating.cards
layer: legal
domains: [billing, sms, gbp]
auto_sync: false
last_verified: 2026-05-23
---

# Terms of Service

**Effective date:** May 23, 2026

**Last updated:** May 23, 2026

**Published version:** [https://www.rating.cards/terms](https://www.rating.cards/terms)

These Terms of Service ("**Terms**") govern your access to and use of the products and services offered by rating.cards ("**we**," "**us**," or "**our**"), including our website at [https://www.rating.cards](https://www.rating.cards), our web application at [https://app.rating.cards](https://app.rating.cards), our physical review devices, and our Autopilot software subscription (collectively, the "**Service**").

By accessing or using the Service, creating an account, purchasing hardware, or subscribing to Autopilot, you agree to be bound by these Terms and our [Privacy Policy](./privacy-policy.md). If you do not agree, do not use the Service.

Please read these Terms carefully, especially Sections 13 (Disclaimers), 14 (Limitation of Liability), 15 (Indemnification), and 17 (Dispute Resolution).

---

## 1. Who we are

The Service is operated by a sole proprietorship doing business as **rating.cards**, located at:

**1890 Valencia Ave**  
**Carlsbad, CA 92008**  
**United States**

**Contact:** [hello@rating.cards](mailto:hello@rating.cards)

---

## 2. Definitions

- **"Account Owner"** (or **"you"**) — the individual or business entity that creates a rating.cards account, subscribes to Autopilot, or purchases hardware.
- **"Manager"** — a store-level employee or authorised representative whose phone number is added by the Account Owner to receive SMS review notifications and approve replies. Managers do not create rating.cards accounts.
- **"End Customer"** — a guest or patron at a venue who taps or scans a rating.cards physical device and is redirected to leave a Google review.
- **"Hardware"** — physical review products sold by rating.cards, including NFC/QR cards, stickers, stands, and posters.
- **"Autopilot"** — the software subscription that provides AI-drafted review replies, SMS review notifications, and weekly email summaries.
- **"Storefront"** — a single Google Business Profile location imported into your rating.cards account.

---

## 3. Eligibility

To use the Service, you must:

- Be at least **18 years old**
- Operate a business located in the **United States**
- Have the authority to bind the business you represent to these Terms
- Provide accurate and complete registration information

By using the Service on behalf of a business, you represent that you have authority to act on that business's behalf.

---

## 4. Account and security

### 4.1 Account creation

Account Owners create accounts through Supabase Auth at app.rating.cards using an email address and password. You are responsible for maintaining the confidentiality of your credentials and for all activity that occurs under your account.

### 4.2 Account linking

If you subscribe to Autopilot before creating an account, your subscription and any pre-provisioned business profile will be linked automatically when you sign up using the same email address used at checkout.

### 4.3 Notification of unauthorised access

Notify us immediately at [hello@rating.cards](mailto:hello@rating.cards) if you suspect unauthorised access to your account.

---

## 5. Hardware purchases

### 5.1 Products and setup

Hardware is sold as a **one-time purchase** with no subscription requirement. Each device is configured and activated by the rating.cards team before delivery, so it arrives ready to use. Product types include cards, stickers, stands, and posters.

### 5.2 Title and risk

Title to Hardware passes to you upon delivery. Risk of loss passes to you upon delivery to the carrier.

### 5.3 Returns and refunds

Because Hardware is **personalised per business** (each device is configured with a unique identifier and destination URL), we do not accept returns for buyer's remorse or change of mind.

We will provide a **replacement or full refund** (at our option) if Hardware arrives **defective or damaged on arrival (DOA)**, provided you notify us within **30 days of delivery** at [hello@rating.cards](mailto:hello@rating.cards) with a description of the defect and, if requested, photos.

After 30 days, all Hardware sales are final except where required by applicable law.

### 5.4 Shipping

Shipping times and costs will be communicated at the time of purchase. We are not responsible for delays caused by the carrier.

---

## 6. Autopilot subscription

### 6.1 Subscription terms

Autopilot is a **monthly subscription** currently priced at **$20 per month, flat per account** (not per storefront). Pricing is subject to change as described in Section 6.4.

Autopilot includes:

- AI-drafted review replies (per verified storefront)
- SMS review notification and approval bot (per verified manager line)
- Weekly pulse email summary

Hardware is sold separately and is not included in the subscription.

### 6.2 Billing and renewal

Subscriptions are billed monthly through **Stripe** and **auto-renew** at the end of each billing period unless cancelled. By subscribing, you authorise us to charge your payment method on a recurring basis.

Payment receipts are sent by Stripe to the email address on your account.

### 6.3 Cancellation

You may cancel your Autopilot subscription at any time through the **Stripe customer portal** accessible from Settings → Billing in the app.

Upon cancellation:

- Your subscription remains active until the **end of the current billing period**
- You will not be charged for subsequent periods
- **No pro-rated refunds** are provided for partial months
- After the billing period ends, Autopilot features (AI drafts, SMS bot, weekly pulse) will be disabled; your account data will be retained according to our [Privacy Policy](./privacy-policy.md) retention schedule

### 6.4 Price changes

We may change subscription pricing with at least **30 days' advance notice** by email to your account address. Price changes take effect at the start of your next billing period after the notice period. If you do not agree to a price change, you may cancel before it takes effect.

---

## 7. SMS obligations

### 7.1 Manager consent warranty

When you add a Manager's phone number to your account, you **represent and warrant** that:

- You have obtained that Manager's **prior express written consent** to receive SMS/MMS messages from rating.cards for review notification and approval purposes, as described in our [Privacy Policy](./privacy-policy.md) (Section 5)
- The consent was obtained before you submitted the phone number
- You will maintain records of that consent

You must check the consent confirmation in the app before submitting any Manager phone number. The consent text (version `sms_v2`) is:

> *"By providing this number, you confirm the recipient agrees to receive review notification SMS/MMS from rating.cards. Message frequency varies; message and data rates may apply. Reply STOP to opt out."*

### 7.2 Prohibited SMS use

You may not:

- Add phone numbers without the recipient's prior express written consent
- Use the SMS program for marketing, promotional, or unrelated messages
- Add phone numbers belonging to individuals who are not authorised Managers at your storefront

### 7.3 Indemnification for SMS violations

You agree to **indemnify and hold harmless** rating.cards against any claim, fine, penalty, or liability arising from your submission of a phone number without proper consent, or any violation of the Telephone Consumer Protection Act (TCPA), state mini-TCPA laws, CTIA messaging guidelines, or applicable carrier requirements. See Section 14 for full indemnification terms.

### 7.4 Manager opt-out

Managers may opt out at any time by replying **STOP** to any rating.cards message. You are responsible for ensuring a replacement verified Manager line is configured if you wish to continue receiving SMS review alerts for a storefront.

---

## 8. Google Business Profile

### 8.1 Authorisation

By connecting your Google account through OAuth, you grant rating.cards permission to:

- Access your authorised Google Business Profile listings
- Read reviews posted to those listings
- Post replies to reviews on your behalf (only after Manager approval via SMS, or as otherwise configured)

You represent that you have authority to grant this access for each listing you connect.

### 8.2 Your responsibilities

You are solely responsible for ensuring that your use of the Service complies with:

- **Google's review policies**, including prohibitions on incentivised reviews, review gating, and fake reviews
- **FTC endorsement and testimonial guidelines**
- Any applicable state or local consumer protection laws

rating.cards does not guarantee that your use of the Service will comply with Google's policies. We are not responsible if Google restricts, suspends, or removes your Business Profile listing.

### 8.3 Reply posting

Replies are posted to Google only after a verified Manager approves them via SMS (or via the dashboard, if available). You remain responsible for the content of all replies posted through the Service.

---

## 9. AI-drafted replies

### 9.1 How AI drafts work

Autopilot uses artificial intelligence to generate suggested replies to Google reviews based on the review content and your configured brand voice settings. Drafts are presented to Managers for review and approval before posting.

### 9.2 Ownership

Once a reply is approved and posted, **you own the reply content**. rating.cards does not claim ownership of replies you approve and post through the Service.

### 9.3 No warranty on AI output

AI-generated drafts are provided **as suggestions only**. We make **no warranty** that AI drafts are accurate, appropriate, complete, free of errors, or compliant with Google's review policies or any applicable law. You and your Managers are responsible for reviewing and approving all replies before they are posted.

We are not liable for any consequence arising from a reply posted through the Service, including but not limited to Google policy violations, customer complaints, or reputational harm.

---

## 10. Acceptable use

You agree not to use the Service to:

1. Solicit, incentivise, or exchange reviews for discounts, free products, or other compensation (in violation of Google's review policies or FTC guidelines)
2. Filter, gate, or redirect negative reviews away from Google (review gating)
3. Post fake, misleading, or fraudulent reviews
4. Add phone numbers to the SMS program without the recipient's prior express written consent
5. Send marketing, promotional, or unsolicited messages through the SMS bot
6. Reverse engineer, decompile, or attempt to extract the source code of the Service
7. Scrape, crawl, or systematically extract data from the Service
8. Resell, sublicense, or redistribute access to the Service
9. Use the Service to harass, defame, threaten, or spam any person
10. Infringe any third party's intellectual property or other rights
11. Violate any applicable law or regulation

Violation of this Acceptable Use Policy may result in **immediate suspension or termination** of your account without refund, at our sole discretion.

---

## 11. End customers (venue guests)

When an End Customer taps or scans a rating.cards device at your venue, they are redirected to your Google Reviews page. Our redirect service logs the tap with non-identifying data (user agent and approximate country) for analytics purposes, as described in our [Privacy Policy](./privacy-policy.md) (Section 3.5).

**You (the venue / Account Owner) are responsible** for any privacy disclosure to your End Customers required by applicable law in your jurisdiction. rating.cards does not interact directly with End Customers and does not collect their personal information.

---

## 12. Service availability and changes

### 12.1 No uptime guarantee

The Service is provided on an **"AS IS"** and **"AS AVAILABLE"** basis. We do not guarantee any specific uptime, availability, or performance level. The Service depends on third-party providers (Supabase, Stripe, Twilio, Google, Resend) whose outages or changes may affect functionality.

### 12.2 Beta features

Some features may be labelled or treated as **beta**. Beta features may be incomplete, change without notice, or be removed. We make no commitment to maintain or support beta features.

### 12.3 Changes to the Service

We may modify, suspend, or discontinue any part of the Service at any time, with or without notice. We will use reasonable efforts to notify Account Owners of material changes that affect core functionality.

---

## 13. Disclaimers

TO THE MAXIMUM EXTENT PERMITTED BY APPLICABLE LAW, THE SERVICE (INCLUDING HARDWARE, AUTOPILOT, AI DRAFTS, SMS NOTIFICATIONS, AND ALL RELATED FEATURES) IS PROVIDED **"AS IS"** AND **"AS AVAILABLE"** WITHOUT WARRANTIES OF ANY KIND, WHETHER EXPRESS, IMPLIED, OR STATUTORY.

WE DISCLAIM ALL WARRANTIES, INCLUDING BUT NOT LIMITED TO:

- IMPLIED WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE, AND NON-INFRINGEMENT
- WARRANTIES THAT THE SERVICE WILL BE UNINTERRUPTED, ERROR-FREE, OR SECURE
- WARRANTIES THAT AUTOPILOT WILL RESULT IN ANY SPECIFIC NUMBER OF REVIEWS, STAR RATING IMPROVEMENT, OR BUSINESS OUTCOME
- WARRANTIES REGARDING THE ACCURACY, COMPLETENESS, OR APPROPRIATENESS OF AI-GENERATED DRAFTS

Some jurisdictions do not allow the exclusion of implied warranties, so some of the above exclusions may not apply to you.

---

## 14. Limitation of liability

TO THE MAXIMUM EXTENT PERMITTED BY APPLICABLE LAW:

1. **Cap on liability:** Our total cumulative liability to you for any and all claims arising out of or related to these Terms or the Service shall not exceed the **total fees you paid to rating.cards in the twelve (12) months immediately preceding the event giving rise to the claim**.

2. **Excluded damages:** IN NO EVENT SHALL WE BE LIABLE FOR ANY INDIRECT, INCIDENTAL, SPECIAL, CONSEQUENTIAL, OR PUNITIVE DAMAGES, INCLUDING BUT NOT LIMITED TO LOSS OF PROFITS, REVENUE, DATA, GOODWILL, OR BUSINESS OPPORTUNITY, EVEN IF WE HAVE BEEN ADVISED OF THE POSSIBILITY OF SUCH DAMAGES.

3. **Third-party services:** We are not liable for any act, omission, or failure of third-party service providers (including Google, Stripe, Twilio, Supabase, or Resend).

Some jurisdictions do not allow the limitation or exclusion of liability for incidental or consequential damages, so some of the above limitations may not apply to you.

---

## 15. Indemnification

You agree to **indemnify, defend, and hold harmless** rating.cards and its operators from and against any and all claims, damages, losses, liabilities, costs, and expenses (including reasonable attorneys' fees) arising out of or related to:

- Your use of the Service
- Your violation of these Terms or any applicable law
- Your submission of Manager phone numbers without proper consent
- Any TCPA, state mini-TCPA, CTIA, or carrier compliance claim related to phone numbers you submitted
- Your violation of Google's review policies, FTC guidelines, or other third-party platform rules
- Content you or your Managers submit, approve, or post through the Service (including AI-drafted replies you approve)
- Any claim by an End Customer, Manager, or third party related to your use of the Service

We reserve the right to assume the exclusive defence and control of any matter subject to indemnification by you, at your expense.

---

## 16. Termination

### 16.1 Termination by you

You may terminate your account at any time by:

- Cancelling your Autopilot subscription through the Stripe customer portal
- Contacting us at [hello@rating.cards](mailto:hello@rating.cards) to request account closure

### 16.2 Termination by us

We may suspend or terminate your account immediately, without refund, if:

- You violate these Terms or the Acceptable Use Policy
- You fail to pay subscription fees
- We are required to do so by law
- We reasonably believe your use poses a legal, security, or reputational risk to rating.cards or third parties

### 16.3 Effect of termination

Upon termination:

- Your access to Autopilot features will end (immediately if terminated by us for cause; at end of billing period if you cancel)
- Your data will be handled according to our [Privacy Policy](./privacy-policy.md) retention schedule
- Sections that by their nature should survive termination (including Sections 5.3, 7.3, 9.3, 13, 14, 15, and 17) will survive

---

## 17. Dispute resolution

**Please read this section carefully. It affects your legal rights.**

### 17.1 Informal resolution

Before initiating any formal dispute, you agree to contact us at [hello@rating.cards](mailto:hello@rating.cards) and attempt to resolve the dispute informally for at least **30 days**.

### 17.2 Binding arbitration

If informal resolution fails, any dispute, claim, or controversy arising out of or relating to these Terms or the Service shall be resolved by **binding arbitration** administered by **JAMS** under its Streamlined Arbitration Rules and Procedures, except as modified herein.

- **Location:** San Diego County, California
- **Language:** English
- **Arbitrator:** One arbitrator, mutually agreed upon or appointed by JAMS
- **Remedies:** The arbitrator may award any relief available in court, subject to the limitations in Section 14

### 17.3 Class action waiver

**YOU AND RATING.CARDS AGREE THAT EACH MAY BRING CLAIMS AGAINST THE OTHER ONLY IN YOUR OR ITS INDIVIDUAL CAPACITY, AND NOT AS A PLAINTIFF OR CLASS MEMBER IN ANY PURPORTED CLASS, COLLECTIVE, OR REPRESENTATIVE PROCEEDING.**

The arbitrator may not consolidate more than one person's claims and may not preside over any form of class or representative proceeding.

### 17.4 Small claims court carve-out

Notwithstanding the above, either party may bring an individual claim in **small claims court** in San Diego County, California, if the claim qualifies.

### 17.5 Governing law

These Terms are governed by the laws of the **State of California**, without regard to conflict-of-law principles. The Federal Arbitration Act governs the interpretation and enforcement of Section 17.2.

### 17.6 Opt-out

You may opt out of the arbitration and class action waiver provisions by sending written notice to [hello@rating.cards](mailto:hello@rating.cards) within **30 days** of first accepting these Terms. Your notice must include your name, email address, and a clear statement that you wish to opt out of arbitration.

---

## 18. Changes to these Terms

We may update these Terms from time to time. When we make material changes, we will:

- Post the updated Terms on this page with a new effective date
- Notify Account Owners by email at least **30 days** before the changes take effect

Your continued use of the Service after the effective date constitutes acceptance of the updated Terms. If you do not agree, you must stop using the Service and cancel your subscription before the changes take effect.

---

## 19. General provisions

- **Entire agreement:** These Terms and the [Privacy Policy](./privacy-policy.md) constitute the entire agreement between you and rating.cards regarding the Service.
- **Severability:** If any provision of these Terms is found unenforceable, the remaining provisions remain in full force and effect.
- **Assignment:** You may not assign or transfer these Terms without our prior written consent. We may assign these Terms in connection with a merger, acquisition, or sale of assets.
- **No waiver:** Our failure to enforce any provision of these Terms does not constitute a waiver of that provision.
- **Electronic communications:** By using the Service, you consent to receive communications from us electronically (email, SMS, and in-app notifications) and agree that such communications satisfy any legal requirement that communications be in writing.
- **Force majeure:** We are not liable for any failure or delay due to circumstances beyond our reasonable control, including natural disasters, war, pandemic, government action, or third-party service outages.

---

## 20. Contact us

If you have questions about these Terms, contact us:

**Email:** [hello@rating.cards](mailto:hello@rating.cards)  
**Mail:** rating.cards, 1890 Valencia Ave, Carlsbad, CA 92008, USA

---

## Changelog

- 2026-05-23: sms_v2 consent text (frequency + STOP disclosures); published URL
- 2026-05-23: Initial Terms of Service draft covering hardware, Autopilot subscription, SMS consent obligations, AI disclaimers, AUP, and JAMS arbitration in San Diego County
