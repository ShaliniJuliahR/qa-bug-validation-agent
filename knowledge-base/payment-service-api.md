# Feature: Payment Service API

## Overview

API tests for payment operations including Cybersource sign-fields, legal entity, saved account payment, tax in carts, merge carts, order placement (happy and error paths), PayPal init/save, user licenses, and PayPal vs card order flows.

## Test Flows

### API - Signed Field endpoint [P2] (from DVW-T47)

**Objective:** POST `/payment-methods/cards/sign-fields` returns signed fields for Flex/Hosted checkout; validates mandatory payload rules.

**Preconditions:**

- Authenticated caller; sample FieldsToBeSigned payload with profile_id, access_key, transaction_uuid, reference_number, amount 0, locale, signed_date_time, currency, signed_field_names, unsigned_field_names, billingCountry.

**Steps:**

1. POST valid JSON payload → Expected: 200; signedFields in response.
2. Omit or mismatch mandatory fields → Expected: error per validation.
3. US and UK profiles → Expected: 200 signed fields for each supported profile.
4. Amount must be 0 only → Expected: error for non-zero amount.
5. Locale must be en-US → Expected: error for other locales.
6. signed_field_names must match payload set and order → Expected: error if order or membership wrong.
7. signed_date_time must be current date/time → Expected: error for stale or wrong time.

---

### API - Get Legal Entity [P2] (from DVW-T54)

**Objective:** GET `payments/legal-entities` with account id + token returns currency, card profile, availableCardTypes, collection group, PayPal availability by country.

**Preconditions:**

- AO or SAO token; account with known country/currency; reference list of card types per region in Zephyr.

**Steps:**

1. GET with valid token + accountId → Expected: 200; currency, paymentMethods.card profileId/accessKey/availableCardTypes; collectionGroupName; PayPal where supported for country.
2. SAO same account → Expected: 200 same access as AO.
3. Product user → Expected: cannot access legal entity endpoint.
4. Multi-currency country → Expected: currency matches account setup currency.
5. Legal entity currency matches GetAccounts currency → Expected: aligned.
6. If GetAccounts omits currency → Expected: GetLegalEntity fails with 4xx per test.

---

### API - Get AccountPayment [P1] (from DVW-T57)

**Objective:** GET `/payments` returns saved payment token metadata for AO/SAO; product user blocked; 404 when none.

**Preconditions:**

- Account with saved card in CT customer custom object; AO and SAO users available.

**Steps:**

1. GET `/payments` with token + accountId of user with payment → Expected: 200; paymentType Card; data token, lastFourDigits, expiry, cardType.
2. CT GetCustomer for same customer → Expected: payment info stored on customer object.
3. SAO sees same payment as AO → Expected: identical payment response.
4. SAO updates payment in UI → Expected: AO sees updated payment.
5. Product user GET payment → Expected: payment not returned / forbidden per contract.
6. Account with no payment → Expected: 404 Payment method not found (detail references CT custom object 404 pattern in test).

---

### API - Verify new customers add payment method [P2] (from DVW-T63)

**Objective:** After Cybersource token ACCEPT, CDH and CT customer reflect billing address and payment custom fields; failures leave prior state.

**Preconditions:**

- Ability to drive Cybersource tokenization to ACCEPT vs non-ACCEPT; CDH GET accounts; CT GetCustomer.

**Steps:**

1. Token ACCEPT → CDH GET accounts → Expected: address reflects credit card billing address.
2. Token ACCEPT → CDH addresses billto/shipto flags for card address → Expected: both false as specified.
3. GetCustomer CT → Expected: payment model under customFields populated.
4. Validate payment object fields → Expected: transactionID, cardType, lastFourDigits, token, expiry, paymentType present.
5. GET legal-entities after add → Expected: profileId, accessKey, cardTypes in response.
6. Non-ACCEPT token → CDH GET → Expected: no credit card address in response.
7. Update payment then GetCustomer → Expected: payment model replaced, not duplicated.
8. Non-ACCEPT after add → GetCustomer → Expected: no payment under customFields.
9. Non-ACCEPT → GET legal-entities → Expected: no payment method parameters.
10. Update payment with non-ACCEPT Cybersource → GetCustomer → Expected: prior payment model unchanged.
11. Non-ACCEPT update → CDH → Expected: prior card address retained.
12. Address line2 populated then cleared on card update → CDH line2 → Expected: empty when cleared.

---

### SAGA -1 Place Order API [P2] (from DVW-T65)

**Objective:** POST `/orders` with userConsent and optional assignLicenseForLineItems; validates CT, Aria, CDH side effects and error cases.

**Preconditions:**

- Registered cart with non-zero tax; saved payment; AO/SAO token + account id.

**Steps:**

1. POST orders with valid body `{ userConsent: true, assignLicenseForLineItems: [lineItemGuids] }` → Expected: 201; orderId, status Processing.
2. First purchase → Expected: new Aria account; subsequent uses existing.
3. Aria → Expected: accountId and address captured.
4. Payment info in Aria → Expected: card type, expiry, token correct.
5. Second order after payment update → Expected: Aria shows updated card.
6. Each order → Expected: authorization and tax lines under same Aria account for repeat buyer.
7. Total and tax in Aria → Expected: match cart/order economics.
8. GET CT cart after order (billing vs card addresses) → Expected: cart ordered; billing shown; not raw card address when both exist.
9. GET CT cart for placed order → Expected: order created; isBillTo address as shipping; orderStatus Processing; lineItems assignment flags per payload.
10. CT merchant center subscription status → Expected: subscription created; custom attributes assign license per test.
11. POST order when cart already frozen → Expected: 422.
12. Cart currency vs legal entity mismatch → Expected: 4xx.
13. Product user role → Expected: 4xx.
14. AO and SAO separate carts → Expected: both orders succeed with respective tokens/accounts.
15. Invalid account or token → Expected: 4xx.
16. Invalid line item id in assignLicenseForLineItems → Expected: 400.
17. GET CT cart not yet frozen → Expected: active; no shipping address yet.
18. GET carts after order placed → Expected: 404 per registered cart lifecycle.
19. Further steps → Expected: see DVW-1563 in Zephyr.

---

### API - Tax Integration in carts [P2] (from DVW-T70)

**Objective:** Registered carts always carry tax for valid UK (and similar) addresses; Vertex parity; merge included.

**Preconditions:**

- UK account address on file; Vertex visibility; all five SKUs available.

**Steps:**

1. Create registered cart with UK address → Expected: tax populated in create response.
2. Update/add/remove line items, merge to user with/without cart, currency mismatch merge, quantity exceed merge, assign license recalc → Expected: tax always in response for valid flows.
3. Compare cart tax to Vertex calculation → Expected: match.
4. Tax never null when Vertex returns value → Expected: non-null tax fields.
5. All five product SKUs → Expected: tax computed per line.
6. Missing/invalid account address → Expected: error path when address required for tax.

---

### API Test: Merge Carts [P2] (from DVW-T72)

**Objective:** Merge anonymous into registered cart: quantity rules, currency mismatch flags, threshold warnings, errors, MXP line metadata.

**Preconditions:**

- Anonymous and registered carts; AO/SAO user; max qty per SKU (e.g. 40).

**Steps:**

1. General rule: each merge → Expected: totalNet, totalTax, totalGross recalculated; tax non-zero for registered; cartVersion matches CT.
2. New user no cart + anonymous with products → Expected: anonymous cart id becomes registered; totals correct.
3. Registered with 1 line + anonymous 1 product → Expected: quantities merged; totals correct.
4. Registered 2 lines + anonymous 1 product → Expected: merged quantities.
5. Registered 5 lines + anonymous 5 same order → Expected: merged.
6. Registered 5 lines + anonymous 5 different order → Expected: per-SKU quantities match expected sums in test data.
7. Registered 2 lines + anonymous 2 lines → Expected: merged; 5 total product types when applicable.
8. Registered 1 line + anonymous (4 new + 1 qty bump) → Expected: five product lines total with correct qty.
9–12. Empty registered cart + anonymous multi-product → Expected: all products appear; totals correct.
13–17. Threshold exceed scenarios (partial vs all SKUs over max) → Expected: mergeWarning THRESHOLD_EXCEED; over-max lines keep registered quantities; under-max merge.
18–21. Currency mismatch anonymous vs registered → Expected: mergeWarning CURRENCY_MISMATCH; result currency is registered; quantities merged.
22–23. Combined currency mismatch + threshold exceed → Expected: mergeWarning CURRENCY_MISMATCH_THRESHOLD_EXCEED.
24. Merge already-merged anonymous cart → Expected: PRODUCTS_1803 anonymous cart not found.
25. Invalid anonymous cart id → Expected: PRODUCTS_1803.
26. Invalid account or expired token → Expected: 401 fault.
27. Merge to TDX-only user account → Expected: PRODUCTS_1800 merge error.
28. Merge as product user → Expected: 401.
29. Pass registered cart id as anonymous → Expected: PRODUCTS_1802 invalid anonymous cart.
30. Merge own registered cart again → Expected: PRODUCTS_1802.
31. Expired anonymous cart (e.g. 7 days) → Expected: PRODUCTS_1803.
32. Reference updated merge response shape (threshold flags, cartSource MXP, proration fields) → Expected: entitlementId/assetId/lineItemType/oldSku null; cartSource MXP.
33. Confirm line metadata nulls and MXP source on responses → Expected: per spec.

---

### API Test: Create Order API Error Cases [P3] (from DVW-T74)

**Objective:** POST `/orders` negative paths: empty cart, no cart, tax zero, no payment, consent, media type, downstream failures.

**Preconditions:**

- Various user/cart/payment states; ability to simulate Aria/CT failures where noted.

**Steps:**

1. User with empty cart → Expected: 422 Cart is empty.
2. User with no cart → Expected: PAYMENTS_1000 Cart not found.
3. Cart tax zero → Expected: 422 Cart tax is empty.
4. No saved payment → Expected: 422 PAYMENTS_1003 Aria account creation failed pattern in test / payment missing.
5. userConsent false → Expected: PAYMENTS_1007 User Consent is required.
6. No body / empty body → Expected: 415 Unsupported Media Type.
7. Aria account creation failure simulation → Expected: 422 Client account group does not exist.
8. Aria authorization fail → Expected: error surfaced per test.
9. CT order creation fail → Expected: error message.
10. CT customer anomalies (duplicated CT number) → Expected: per test.
11. 3DS failure in Aria → Expected: per test.
12. UK user cart in USD mismatch → Expected: error.
13. Unrealistic large amount cart → Expected: Cybersource excessive amount; cart stays active.

---

### [API] First Order - Entitlement and Aria subscription [P1] (from DVW-T81)

**Objective:** First order SAGA: EMS entitlements, Aria account/plans, Cybersource auth, CT merchant center status, failure rollback behavior.

**Preconditions:**

- AO/SAO checkout path; EMS, Aria, Cybersource, CT access.

**Steps:**

1. AO/SAO first order → Expected: order id Processing in commerce flow.
2. EMS → Expected: entitlement per line item.
3. EMS quantities match order → Expected: verified via GetEntitlement by Account.
4. Entitlement status becomes active (note Aria VC may delay one day) → Expected: active when system allows.
5. Aria validations section → Expected: per subsection in Zephyr (billing group, plans, invoice).
6. After entitlement creation → Expected: Aria account + subscription per line; placeholder cancelled plan exists.
7. Aria account overview → Expected: address matches CDH; card last4/token/expiry match Cybersource; Aria source of truth after first order.
8. Edit address on checkout save → Expected: Aria account address and billing group statement sections update.
9. Overlong address save → Expected: Aria/CDH retain old address.
10. Account group collection maps to correct merchant id → Expected: correct MID.
11. Client defined fields → Expected: Tax_Registration_ISO_Code and Legal Entity set.
12. One billing group per Aria account → Expected: invariant holds.
13. Plan details → Expected: products with correct bill-through / anniversary dates.
14. Invoice/statement line amounts and tax → Expected: correct per line.
15. Successful transaction → Expected: no auth reversal data on “new reversal” action; unsuccessful retains authorized amount for reversal UI.
16. Cybersource → Expected: authorized amount under correct MID for success/fail.
17. CT merchant center → Expected: after delay status becomes Subscription Created.
18. SAGA failure at entitlement/subscription/DX order → Expected: CT status Entitlement creation failed / Subscription Failed / DX Order Failed as appropriate.
19. Partial entitlement failure → Expected: no further steps; status entitlement failed; any created entitlements disabled.

---

### API - PayPal Init & Save APIs [P2] (from DVW-T82)

**Objective:** InitPaypal creates Aria linkage token; savePaypal persists PayPal; error cases for auth, body, token validation.

**Preconditions:**

- UK PayPal QA collection group (e.g. paypal_uk_qa); UI or API to complete PayPal linking.

**Steps:**

1. InitPaypal valid account + token → Expected: Aria account created; token string in response.
2. Aria before save → Expected: no payment method; no PayPal collection group selected.
3. GET payments before save → Expected: no payment method found.
4. UI: add PayPal and save → Expected: PayPal saved.
5. During save, verify `/savePaypal` called → Expected: request observed.
6. savePaypal response → Expected: paymentType PayPal; paypalID and paypalUserName populated.
7. GET payment methods after save → Expected: PayPal type and data returned.
8. Aria → Expected: collection group paypal_uk_qa; payment and billing groups show PayPal.
9. InitPaypal no/invalid token → Expected: 401 Missing Credentials fault.
10. InitPaypal invalid account id → Expected: 403.
11. savePaypal no/invalid token → Expected: 401.
12. savePaypal invalid account id → Expected: 403.
13. savePaypal empty body → Expected: 415 Unsupported Media Type.
14. savePaypal `{}` or empty token string → Expected: PAYMENTS_9998 Token is required.
15. savePaypal invalid token value → Expected: PAYMENTS_1801 PayPal Save failed.

---

### API - Get Assigned User Licenses [P3] (from DVW-T83)

**Objective:** GET user licenses returns SKU to license id map for account.

**Preconditions:**

- Authenticated payments base URL; valid account id + token.

**Steps:**

1. GET `/user/licenses` with valid account id + token → Expected: 200; licenses array with sku and ids[].
2. User with no licenses → Expected: 404 PAYMENTS_1701 Licenses not found.
3. Mismatched account id and token → Expected: 403.

---

### API - Paypal Replace & Save, Place Order [P2] (from DVW-T85)

**Objective:** Switch between PayPal and card on fresh and established accounts; verify CDH, Aria, CT after each change; place orders with PayPal.

**Preconditions:**

- Accounts: fresh without payment; with card after first order; card without order; PayPal sandbox.

**Steps:**

1. Fresh account: add PayPal with card address → Expected: CDH GET accounts + Aria account/payment/billing groups updated.
2. Replace PayPal with another PayPal → Expected: CDH/Aria updated to new PayPal.
3. Update only billing address on PayPal → Expected: CDH/Aria reflect address change.
4. Replace PayPal with credit card → Expected: CDH/Aria show card method.
5. Change to another card → Expected: token/details/address update.
6. Update card billing address only (same PAN) → Expected: CDH/Aria address update.
7–10. Account with card, no first order: switch PayPal / another PayPal / card → Expected: CDH/Aria consistent after each step.
11–15. Place order flows: fresh PayPal-only first order; first order card then PayPal order → Expected: orders succeed; PayPal shown in Aria, CDH, CT custom fields.

---

## Key Validations

- Sign-fields and legal-entity responses enforce profile, locale, field order, and currency rules.
- Saved payment read path restricted to AO/SAO; CT customer custom object is system of record for card metadata.
- Orders require non-empty taxed cart, user consent true, valid line item ids for assignment, and coherent currency vs legal entity.
- Merge cart combines quantities with explicit warnings for threshold and currency mismatch; anonymous lifecycle and role gates enforced.
- PayPal init/save and license GET endpoints return consistent 401/403/404/415 error shapes per tests.

## Common Preconditions

- Payments service base URL (e.g. Trimble cloud business-systems payments 1.0) matching environment.
- `x-account-id` (or header contract) plus bearer token for authenticated payment calls.
- Cybersource, Aria, CT, CDH, and EMS access for SAGA-level tests; test cards and PayPal sandbox accounts approved for tier.
