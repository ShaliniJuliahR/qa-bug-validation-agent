# Feature: Miscellaneous & Error Handling

## Overview

Error page handling, commerce flow sanity, account switching, L2 support retry, DigiCert SSL, cookie banner, trials provisioning, analytics events, localization, DM sanity, and other cross-cutting concerns.

## Test Flows

### Verify error page handling for invalid URLs and account IDs [P2] (from DVW-T84)

**Objective:** Cart, checkout, order review, and order confirmation show standard error copy for bad URL or account id.

**Preconditions:**

- Registered user with product in cart through cart page.

**Steps:**

1. Precondition: user on cart from PDP → Expected: Baseline valid session.
2. On cart, use invalid URL or account id → Expected: "Oops, sorry. That page can't be found" or "We've hit a dead end."
3. Back from cart → Expected: Valid cart page.
4. Proceed to checkout → Expected: Checkout page.
5. On checkout, invalid URL/account id → Expected: Same error copy.
6. Back from checkout → Expected: Checkout restored valid.
7. Continue to order review → Expected: Order review page.
8. Back from review → Expected: Review state valid.
9. Place order to confirmation → Expected: Confirmation page.
10. On confirmation, invalid URL/account id → Expected: Same error copy.

---

### Commerce flow (billing field limits) [P3] (from DVW-T86)

**Objective:** Billing form max lengths and validation toasters on checkout.

**Preconditions:**

- TID login; checkout with card payment.

**Steps:**

1. Login → checkout → open card billing form → Expected: New billing form visible.
2. "Use account address" checkbox → Expected: Visible and toggles autofill from account address.
3. Field limits → Expected: First/last/city ≤50 chars; building/floor ≤254; zip ≤25; street/state unrestricted per spec.
4. Validation errors → Expected: Toasters for required/invalid fields listed in Zephyr.
5. Cancel/Save → Expected: Both work.
6. After save → Expected: Summary with EDIT.
7. EDIT → Expected: Return to form.

---

### Account switching experience [P2] (from DVW-T95)

**Objective:** Flyout account switcher, default account, multi-account cart isolation on MXP.

**Preconditions:**

- PDP URL in Zephyr; users with single vs multiple accounts and roles.

**Steps:**

1. Navigate to PDP → Expected: Page loads.
2. Single account → Expected: No switcher in flyout.
3. New user sign-in with no linked accounts → Expected: Baseline.
4. Flyout Trimble menu → Expected: Expandable; shows name; My Profile, My Products, Sign out.
5. Account id visible in commerce UI → Expected: Matches selection.
6. Multi-account AO → Expected: Accounts listed under user in flyout on MXP.
7. Role setup (AO/SAO/PU/company admin) → Expected: All roles visible for commerce user.
8. Switch account → Expected: MXP shows that account id; cart changes isolated; order uses selected account.
9. Set default account → Expected: getAccounts on cart/checkout returns default, not lowest id.
10. Default not copied across separate email logins → Expected: Default is per-account setting.
11. "Set as default" available for each eligible account → Expected: UI offers default on each.
12. After re-login → Expected: Previous default still flagged.

---

### L2 support retry API for failed orders [P2] (from DVW-T98)

**Objective:** OrderRetryProcessingFn behaviour for failed vs invalid orders; Datadog; no duplicate downstream records.

**Preconditions:**

- QA function URL and code key in Zephyr; order ids in various statuses.

**Steps:**

1. POST retry with failed order id → Expected: 200 body true; order re-queued; Datadog retry + processing logs; no D-country/compliance re-run on retry; success → getOrders Activated / SUBSCRIPTION_CREATED.
2. After successful retry → Expected: No duplicate plans (Aria), assets/orders/invoices (SF), EMS records.
3. Retry with already-processed order id → Expected: 400 PAYMENTS_2031; not re-queued.
4. Retry while status still processing → Expected: 400 PAYMENTS_2031.
5. Retry compliance HOLD order → Expected: 400 PAYMENTS_2030; not re-queued.
6. Retry DCOUNTRYHOLD order id → Expected: 400 PAYMENTS_2030; not re-queued.
7. Retry same failed retry again → Expected: Processing can restart per Zephyr.

---

### Online store security (DigiCert / SSL) [P3] (from DVW-T114)

**Objective:** HTTPS, DigiCert badge, padlock, favicon across commerce pages.

**Preconditions:**

- Commerce base URL.

**Steps:**

1. Open store with HTTPS → Expected: URL uses https.
2. Force http URL → Expected: Site redirects to HTTPS.
3. Random HTTP redirect test → Expected: No DigiCert icon / cert where HTTP stuck (per scenario).
4. DigiCert on account, cart, checkout, review, confirmation → Expected: Badge present.
5. Click DigiCert → Expected: Certificate details.
6. Browser padlock → Expected: Shows cert on all pages.
7. Favicon → Expected: Matches latest Figma / account experience.
8. Footer DigiCert → Expected: On every page.

---

### OneTrust cookie banner [P3] (from DVW-T116)

**Objective:** Cookie consent persistence for anonymous vs signed-in; US vs other regions.

**Preconditions:**

- MXP add to cart → view cart as anonymous.

**Steps:**

1. Anonymous cart → Expected: OneTrust banner shown.
2. Reject all → sign-in checkout → Expected: Banner shows again on checkout if not accepted.
3. New window: accept on cart → sign in → Expected: Banner behaviour per Zephyr step 2–3 (re-display rules).
4. After accept (anonymous or registered) → Expected: Banner not shown again in flow.
5. US region → Expected: US-specific banner per OneTrust config.
6. Non-US → Expected: Regional banner per config.

---

### Commerce flow – threshold, merge carts [P3] (from DVW-T120)

**Objective:** Flyout/sign-in checkout merge for US/CA vs ROW with ProjectSight + Connect; quantity and currency popups.

**Preconditions:**

- Users: US, Canada, ROW; cart compositions in Zephyr.

**Steps:**

1. US user scenarios (headers 0–1) → Expected: Section grouping.
2. CA user sign-in from flyout → Expected: Merge popup; Continue → carts → checkout → order → three emails.
3. ROW user cannot add ProjectSight → Expected: "not available in your country or region".
4. ProjectSight-only vs Connect+PS anonymous carts; sign-in variants → Expected: Currency-only, threshold-only, or combined popups; then checkout or carts per scenario.
5. New TID/PU/LA only user merges → Expected: Merge success path with emails.

---

### Sanity test case [P?] (from DVW-T138)

**Objective:** Pre-release sanity across geo currency, checkout PMs, FO, add seats, upgrade, downgrade, cancel, renewal, dunning, D-country, compliance, returns, EP/DVR/PU flows, JP proration+promo.

**Preconditions:**

- VPNs for geo; VISA/Mastercard only for JP rounding scenario; support portal for returns.

**Steps (high level):**

1. Anonymous PDP add → Expected: Currency matches geo + currency API.
2. Multi-role sign-in checkout → Expected: Account selector; non-purchasable disabled.
3. Checkout PMs → Expected: Legal entities, card types, PayPal per LE.
4. PayPal init vs CC add → Expected: Aria account after PayPal init; not after CC-only save pre-order.
5. Switch CC↔PayPal → Expected: Aria PM updates.
6. Cart edits on checkout/review → Expected: Updates succeed.
7. Order review edit payment → Expected: Works.
8. FO → Expected: Notifications, Aria, SF verified.
9. AXP order history / subscription dates → Expected: Correct.
10. Add seats → Expected: Downstream OK.
11. Upgrade / downgrade → Expected: EMS, AXP, Aria, SF OK.
12. Cancel monthly → Expected: Cancel path OK.
13. Renewal (monthly new account) → Expected: Downstream OK.
14. Dunning monthly/yearly add seats → Expected: Stages OK; only add-seat qty revoked.
15. Expired card renewal → Expected: Dunning.
16. D-country orders (account vs payment address) → Expected: Accounts container + SF.
17. Release D-country → Expected: Downstream OK.
18. Compliance hold order → Expected: Notifications + container.
19. Returns from support → Expected: Aria negative invoice, AXP subscriptions.
20. DVR / EP / PU purchase flows → Expected: Per role sanity.
21. Japan JPY proration + promo + returns → Expected: Rounding rules, UI parity, AXP invoice stamp, SF revenue, support returns.

---

### Trials accounts DVW provisioning [P2] (from DVW-T142)

**Objective:** Trial users without DVW provision land on account creation; provisioned trials checkout; multi-trial account switcher.

**Preconditions:**

- Trial provisioning doc link in Zephyr.

**Steps:**

1. Trial user not DVW provisioned: PDP add → checkout → Expected: Account creation; only name/email prefilled.
2. Place order with provisioned trial → Expected: Order succeeds.
3. Complete DVW account creation with trial → Expected: getAccounts sourceSystem includes DVW and FTSERVICE.
4. Multi trial: pick provisioned account → checkout → Expected: Checkout for selected account.
5. Multi trial: pick unprovisioned → Expected: Account creation.
6. Unprovisioned DVR + unprovisioned trial → select trial → Expected: Account creation.
7. Multiple provisioned trials → pick one → checkout → Expected: Checkout for that account.

---

### Accounts container sanity [P1] (from DVW-T144)

**Objective:** Currency, VAT, tax, carts, orders, PayPal init, D-country EEUC, notifications, role transfer after accounts container changes.

**Preconditions:**

- API + UI access for account creation, merge, patch cart, legal entity, orders.

**Steps:**

1. Create account without VAT → Expected: VAT empty in container.
2. US organization → Expected: Currency stored.
3. UK org with VAT → Expected: VAT ID in container and AXP.
4. API validation on create → Expected: Currency, VAT, exemption flags.
5. Merge API → Expected: Currency correct.
6. Registered carts USD/EUR/GBP/AUD/CAD → Expected: Currency from container.
7. Patch registered cart → Expected: Line items and currency correct.
8. getLegalEntity → Expected: Currency from container.
9. PayPal init → Expected: Aria currency matches container after init.
10. PayPal order → Expected: Aria, SF request, SF UI match container.
11. Notifications (3 order types) → Expected: Currency from container.
12. Org with VAT in notification → Expected: VAT in details.
13. D-country hold → Expected: EEUC HOLD in container.
14. Release / cancel D-country → Expected: Dates/status APPROVED or FAILED.
15. View order / invoice APIs → Expected: Currency and tax from container.
16. Price estimate API → Expected: Uses container currency.
17. Add seats, upgrade, downgrade, cancel, renewal → Expected: Processing uses container currency.
18. Update VAT in AXP → Expected: Container updated; not CDH-only.
19. DVR/EP provisioning → Expected: Currency, VAT, exemption in container.
20. Role transfer → Expected: Currency/tax unaffected.

---

### MX events with UTM params (profile, cart, review, purchase) [P2] (from DVW-T150)

**Objective:** Server-side MX payloads in Datadog + MarketDev DB; UTM on hc_profileCreate, hc_updateCart, hc_reviewCart, hc_purchase; marketing consent.

**Preconditions:**

- Bearer token tools; Datadog search; MarketDev data lake access.

**Steps:**

1. Account setup submit → Expected: hc_profileCreate fields match spec (Event_Name, Trimble_Id_UUID, address, Marketing_Consent, etc.).
2. hc_updateCart: add line, add second line, remove, increase qty, decrease qty → Expected: Each event shape and Old/New quantity rules per Zephyr examples.
3. Order review navigation → Expected: hc_reviewCart payload matches cart.
4. Place order → Expected: hc_purchase includes paymentType etc.
5. Marketing consent checked/unchecked → Expected: true vs null in profile events + DB.
6. Launch PDP with UTM query string → Expected: UTM captured (note Netlify UAT caveat in Zephyr).
7. Login + flow → Expected: UTM on all server events.
8. Add/review/purchase with UTM → Expected: utm_* keys in payloads.
9. Network Application tab → Expected: Cookie header present with UTM-related requests.
10. Datadog "Sending Mx payload" → Expected: All events include UTM fields.
11. MarketDev lake → Expected: UTM stored for profile and cart events.

---

### DM sanity [P1] (from DVW-T160)

**Objective:** Data migration: AXP account settings, tax rate updates post-migration, upgrades/add seats/returns, swap SKU renewal, VAT synthetic invoices.

**Preconditions:**

- Migrated accounts per LE (GB, FI, CA, FR, DE, AU, etc.).

**Steps:**

1. GB migrated (example): accounts.trimble.com settings, billing, subscriptions → Expected: UI complete; correct payment options.
2. Post-migration upgrade → Expected: Tax at latest LE rate.
3. Checkout payments → Expected: Match LE mapping.
4. Return upgraded order → Expected: Return amount correct.
5. Price change / swap scenario headers → Expected: Follow Zephyr subsections.
6. Add seats on migrated subscription → Expected: Proration at new rate schedule.
7. Return add seats → Expected: Refund at new rate.
8. Upgrade with promo + dunning revoke → Expected: Negative old rate, positive new with promo.
9. Upgrade on migrated sub without promo timing case → Expected: Proration at new rate.
10. Swap table renewals → Expected: Each legacy SKU maps to target BN-* SKU on renewal across AXP/Aria/SF.
11. Pro 2yr migrated + downgrade + renewal → Expected: Post-renewal SKU per staged downgrade rule.
12. GB VAT synthetic invoice + upgrade → Expected: 0% tax synthetic; follow-on lines taxed at 20%.
13. EU VAT (non-NL) Italy example → Expected: Synthetic 0%; upgrade lines 0% on positive/negative per spec.
14. Aria CDF LE after migration → Expected: LE matches new mapping.

---

### Localize full commerce customer journey and notifications [P1] (from DVW-T161)

**Objective:** MXP through confirmation + emails in selected locales; API-triggered notifications; unsupported language toast.

**Preconditions:**

- Registered and anonymous sessions; notification API collection + identity token tool; locale list in Zephyr.

**Steps:**

1. Registered user: choose language on PDP/profile → complete order → Expected: MXP, account setup, cart, checkout, review, confirmation translated; emails match templates.
2. Anonymous: switch language on MXP → login → order → Expected: Same validation as 1.
3. Trigger notifications via collection with uuid + locale → Expected: Payloads complete; translations match templates for each language.
4. Profile language save + refresh PDP → Expected: UI in selected language from supported list.
5. Pick unsupported language → Expected: Toast "Sorry! Translation in your preferred language is not yet available."; UI falls back to English.
6. Reference translated strings / email folders in Zephyr → Expected: Used for expected copy checks.

---

### Account-correction portal [P3] (from DVW-T164)

**Objective:** `/validate-fields` access control; VAT-only and address updates; US ZIP suggestion rules; Canada/Brazil state mandatory; Digi card.

**Preconditions:**

- Test emails and account numbers in Zephyr.

**Steps:**

1. `/validate-fields` as AO without account → Expected: Generic error message.
2. Valid account query as AO → Expected: Address update page.
3. Invalid account → Expected: Error loading address tax profile.
4. SAO/PU account id → Expected: Access denied; AO only.
5. GB VAT-only update (twice) → Expected: VAT updates in UPD VAT FIELD.
6. IT VAT + address (two updates) → Expected: Latest values kept.
7. US/IN/BR address-only multiple updates → Expected: Success; last save retained.
8. US ZIP 9→5, 5→9, 9→random 5 → Expected: Suggestion popup rules per step.
9. Suggested address + line 2 only → Expected: Popup behaviour per spec.
10. Canada/Brazil without state → Expected: Address not shown; state mandatory.
11. Manual edit after DnB auto-fill path sections → Expected: Per related D&B case coordination.
12. Digi card verification step → Expected: Validation success.
13. Double address update → Expected: "Address to Be Saved" shows last saved.

---

### Add enriched datapoints to existing DataLayer variables [P2] (from DVW-T165)

**Objective:** GTM dataLayer: item_brand and item_category from CommerceTools on begin_checkout, update_cart, add_payment_info, remove_from_cart, review_cart, purchase.

**Preconditions:**

- Browser console dataLayer inspection; CommerceTools list document in Zephyr.

**Steps:**

1. Add SKU, complete account setup → begin_checkout → Expected: item_brand and item_category for SketchUp Go annual example.
2. Change qty on checkout → update_cart → Expected: Updated brand/category.
3. Enter payment → add_payment_info → Expected: Values for all cart SKUs.
4. Remove SKU → remove_from_cart → Expected: Values for removed item.
5. Order review → review_cart → Expected: All SKUs.
6. After purchase → purchase → Expected: Values match CommerceTools.
7. Full E2E pass → Expected: Consistency across all events vs CommerceTools mapping doc.

---

### UUID & AO not found (renewal when TID deleted) [P2] (from DVW-T171)

**Objective:** Renewal routing when AO/SAO Trimble IDs deleted; CannotProcessOrderQTriggerFn; Aria/SF/Admin Console end states.

**Preconditions:**

- Ability to delete TID from My Profiles; trigger renewals; optional failure simulation for retry step.

**Steps:**

1. AO+SAO both TID deleted after purchase + refund + active sub renewal → Expected: CANNOT_PROCESS; line SF_AUTO_RENEWAL_DISABLED_SUCCESS; Aria cancelled; SF auto_renewal false; EMS expiry behaviour per Zephyr.
2. AO deleted, SAO valid → Expected: Renewal succeeds with SAO UUID; notifications to SAO; assets active.
3. AO valid only → Expected: Normal renewal.
4. Multi-line: active + staged cancel + staged downgrade; both TIDs deleted → Expected: Per-line statuses CANCEL_SEATS, FULL_DOWNGRADE, etc.; plan instances cancelled.
5. Monthly + yearly same account; both TIDs deleted → Expected: Independent renewal handling per SKU timing.
6. Optional: simulate failure in cancel/discard/auto-renewal steps → Expected: Retry from support resumes (if simulable).
7. AO no UUID, SAO has UUID → Expected: Processing uses SAO; renewals group; emails to SAO.

---

### Payment error reason codes (localized) [P3] (from DVW-T172)

**Objective:** Known CyberSource decline reasons show specific, actionable, localized messages (not generic).

**Preconditions:**

- Decline test card (e.g. 4000000000000002); US/GB billing; fr-FR, de-DE, es-ES for translation step.

**Steps:**

1. Checkout with card that returns known reason (insufficient funds, bad CVV, etc.) → Expected: Mapped message, actionable text, support reference; no generic fallback for known codes.
2. Same with UI in French/German/Spanish → Expected: Message matches approved translation.

---

### D&B commerce integration [P?] (from DVW-T173)

**Objective:** Company search UI, DUNS capture, Address Doctor bypass/trigger, SF storage rules, API rate limit and quota per IP.

**Preconditions:**

- DVW account setup with company field; SF sandbox access for D_B_DUNs_Number__c.

**Steps:**

1. Company name before country → Expected: Error "Select a country/territory prior to entering your company name".
2. Country + company search → Expected: Params sent; suggestions show name + address.
3. Suggestion UI → Expected: Matches Figma (name + full address).
4. Select company → Expected: DUNS captured; address fields populated.
5. Submit auto-populated address without edits → Expected: Address Doctor not triggered.
6. Place order; DUNS from company select → Expected: D_B_DUNs_Number__c populated in SF.
7. Manual edit after DnB fill → Expected: Address Doctor runs; DUNS cleared from request path per spec.
8. Address Doctor-only path order → Expected: DUNS not stored when only AD triggers (per step 8 expected).
9. Rate limit: 20 requests/IP/minute → Expected: 20×200 OK; 21st→429; after 1 min→200.
10. IP isolation for rate limit → Expected: IP B unaffected when IP A throttled.
11. Quota: 100 success/IP/24h → Expected: 100th OK; 101st→403; after 24h reset.
12. Quota isolation → Expected: IP B OK when IP A over quota.

## Key Validations

- Invalid deep links / account ids → consistent 404-style UX on all commerce steps.
- Retry API only for eligible failed orders; error codes PAYMENTS_2030/2031 for bad states.
- Accounts container drives currency/VAT/tax across APIs, PayPal, notifications, orders.
- Analytics: MX server events and dataLayer enriched fields aligned with CommerceTools and UTM policy.

## Common Preconditions

- QA/SIT URLs and function endpoints as embedded in Zephyr objectives.
- Test cards, TID accounts, and SF/Aria/AXP read access where steps require verification.
