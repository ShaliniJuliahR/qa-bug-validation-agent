# Feature: Product & Cart Service API

## Overview

API tests for product pricing (GetPrice), cart operations (create anonymous/registered, update, assign, merge), assign-license flag, and cart proration (UI + API cross-checks).

## Test Flows

### Verify GetPrice API [P1] (from DVW-T6)

**Objective:** Verify GetPrice with valid SKU and currency (USD/GBP/EUR) and error cases for missing/invalid params.

**Preconditions:**

- Unauthorized product price endpoint access pattern per environment; valid SKUs known for tier.

**Steps:**

1. GET price for valid SKU without currency/country params → Expected: 400 validation on Currency.
2. Invalid SKU → Expected: 400 PRODUCTS_1602 Invalid SKU.
3. Valid SKU + valid currency (USD, GBP, EUR) → Expected: amount + currencyCode in body.
4. Cross-check price in Commerce Tools for SKU → Expected: matches CT.
5. Valid SKU + invalid currency (e.g. RS) → Expected: 400 invalid currency value.

---

### API - Get Carts - Registered User [P1] (from DVW-T29)

**Objective:** Validate GET carts for registered user: line items, totals, version, auth, and Contentful image URLs.

**Preconditions:**

- `base_url_products`/carts (authorized); `x-account-id` + bearer token; user may be in CT or not.

**Steps:**

1. GET carts with valid token + CDH id; cart with products → Expected: 200; lineItems with imageURL, name, sku, quantity, price, totalPrice, assignLicenseToMe; totalNet, totalTax, totalGross.
2. Verify each product image URL vs Contentful → Expected: match.
3. User with empty cart → Expected: 200; empty lineItems.
4. Mismatched token vs account id → Expected: 403 Forbidden.
5. Invalid/expired token → Expected: invalid JWT / auth error.
6. Missing/invalid `x-account-id` → Expected: 403.
7. User not in Commerce Tools / no cart → Expected: 404 customer not in CT.
8. Cart version in response → Expected: matches CT get cart version.

---

### update_cart_by_id_anonymous Error Handling [P3] (from DVW-T37)

**Objective:** Negative tests for anonymous (and parallel registered) cart PATCH: auth, updateAction, SKU, quantity, line item id.

**Preconditions:**

- Anonymous or registered cart update endpoint; valid cart id where needed.

**Steps:**

1. Invalid/expired token on add/update/remove → Expected: 401 Authentication Failure fault.
2. Body missing `updateAction` → Expected: validation error array (e.g. PRODUCTS_9999).
3. Invalid `updateAction` → Expected: 400 JSON error.
4. Invalid SKU → Expected: PRODUCTS_1602 Invalid SKU.
5. Empty SKU → Expected: PRODUCTS_9999 validation error.
6. Missing `productSku` → Expected: PRODUCTS_9999.
7. Missing `quantity` → Expected: PRODUCTS_1005 (quantity must be greater than 0).
8. Negative quantity → Expected: PRODUCTS_1005.
9. Quantity above max threshold → Expected: PRODUCTS_1506 threshold exceeds.
10. Invalid line item id → Expected: PRODUCTS_1603 invalid line item.
11. Empty/missing lineitemId on update/remove → Expected: PRODUCTS_1006 line item id null/empty.

---

### Update Cart - Add / Update / Remove Line Item - Anonymous & Registered [P1] (from DVW-T40)

**Objective:** Happy and error paths for PATCH cart: add/update/remove line items, AXP cart restrictions, annual/monthly mix, cart version.

**Preconditions:**

- For registered flows: valid token + account id; anonymous cart id for guest flows.

**Steps:**

0. (Registered) Prerequisite: valid token and account id.
1. ADD_CART_LINEITEM valid sku/qty (anonymous) → Expected: 200; totalNet; anonymous totalTax 0; registered path shows tax; totalGross = net + tax.
2. ADD with qty exceeding max → Expected: 422 Product Quantity Threshold Exceeded.
3. UPDATE_CART_LINEITEM valid lineItemId/sku/qty → Expected: 200; totals as per anon vs registered tax rules.
4. UPDATE with sku mismatch vs cart line → Expected: line not updated per sku validation rules.
5. REMOVE_CART_LINEITEM valid id/sku → Expected: 200; line removed; totals updated.
6–8. AXP cart with ADD_SEATS: add/update/remove line item → Expected: 400 PRODUCTS_1606 / 1605 as specified per action.
9. UPDATE_LINEITEM invalid lineitemId on AXP → Expected: PRODUCTS_1605 quantity cannot be upgraded.
10–11. AXP FULL_UPGRADE repeat add/remove → Expected: PRODUCTS_1606.
12–13. Same AXP patterns → Expected: errors per step list in Zephyr.
14. Any update returns `cartVersion` → Expected: matches CT getCarts after each action.
15–16. Mix annual and monthly via add / update line item → Expected: PRODUCTS_1604 Cart should not have both annual and monthly plans.

---

### Create Anonymous Cart - Happy Path & Errors [P1] (from DVW-T41)

**Objective:** POST anonymous cart creation: currency, line items, thresholds, duplicate SKU, validation errors.

**Preconditions:**

- Endpoint `/carts/anonymous` (or environment equivalent).

**Steps:**

1. Valid currency + one line item (sku + qty under max) → Expected: 200; id, lineItems, totalNet, totalTax 0, totalGross.
2. Valid currency + multiple distinct line items → Expected: 200; combined totals; totalLineItemQuantity sum.
3. Invalid currency (rs, uk) → Expected: 400 body/currency conversion errors.
4. Invalid SKU → Expected: PRODUCTS_1502 error creating cart.
5. Quantity over max threshold (e.g. 40) → Expected: PRODUCTS_1506 threshold exceeds.
6. Empty lineItems array → Expected: PRODUCTS_9999 validation error.
7. Missing SKU or quantity in line item → Expected: 400 detail/errors per CT validation.
8. Add same SKU twice in one create → Expected: PRODUCTS_1008 Duplicate SKU.
9. Quantity 0, negative, or special chars → Expected: PRODUCTS_1005.
10. Currency sent as numeric enum string (1/2/3) → Expected: behavior per test (map to USD/EUR/GBP or reject — follow environment).

---

### Assign anonymous cart to user [P2] (from DVW-T42)

**Objective:** Merge anonymous cart into registered user cart; error handling for wrong cart type, ids, roles.

**Preconditions:**

- Ability to create anonymous cart; registered user AO/SAO token and account id.

**Steps:**

1. Create anonymous cart with valid product → Expected: 200; capture cart id.
2. GET cart by anonymous id → Expected: cart matches creation.
3. Set headers: account id + user token for target user.
4. POST assign with `anonymousCartId` → Expected: registered cart replaced or linked; anonymous cart becomes registered cart id in response flow.
5. GET `/carts` with account id → Expected: shows merged anonymous cart content.
6. GET `/carts/{oldAnonymousId}` → Expected: error cannot access registered user’s cart with anonymous id pattern.
7. Pass registered cart id as anonymous id → Expected: cannot access registered cart error.
8. Invalid anonymous cart id format → Expected: 400 validation errors on body/GUID.
9. Invalid account id or token → Expected: 403/401.
10. Assign to product user or company admin → Expected: 401 unauthorized for assign.
11. Assign empty anonymous cart → Expected: 200 OK empty cart assigned.
12. User with empty registered cart + anonymous cart with items → Expected: registered cart replaced with products.

---

### API - Create Cart - Registered user [P1] (from DVW-T55)

**Objective:** POST create registered cart for AO/SAO; forbid product user / company admin; auth mismatch errors.

**Preconditions:**

- Token + `x-account-id`; same error matrix as anonymous create (DVW-622) for additional cases.

**Steps:**

1. Create cart with AO or SAO token + matching account id + valid body → Expected: 200; lineItems, totalNet, totalTax, totalGross, assignLicenseToMe on lines.
2. Product user token + account id → Expected: 401 cart not created.
3. Company admin → Expected: 401 cart not created.
4. Confirm only AO/SAO can create registered cart → Expected: role gate holds.
5. Token/account id mismatch → Expected: 403.
6. Invalid/expired token → Expected: token expired / auth error.
7. Other cases per DVW-622 / anonymous parity → Expected: per linked issue.

---

### API - Update cart - Assign license to me [P2] (from DVW-T73)

**Objective:** PATCH `SET_LINE_ITEM_ASSIGN_LICENSE` toggles `assignLicenseToMe` and validates errors.

**Preconditions:**

- Registered cart with line items; frozen cart scenario available.

**Steps:**

1. PATCH with valid lineitemId, assignLicenseToMe true → Expected: 200; flag true on line; tax on updated line; other lines default true.
2. PATCH same with assignLicenseToMe false → Expected: 200; updated line false; others remain default true; tax present.
3. Invalid lineitemId → Expected: 400.
4. Frozen cart line item → Expected: 204 no content per test.
5. Cycle all five SKUs setting false one by one → Expected: each 200; tax as expected.
6. Invalid action type → Expected: JSON error.
7. Empty line item id → Expected: PRODUCTS_1006.
8. Empty assignLicenseToMe → Expected: JSON error.
9. Invalid token or invalid account id → Expected: 401 fault.
10. SET_LINE_ITEM_ASSIGN_LICENSE when user has no cart → Expected: PRODUCTS_1601 error updating cart.
11. Default across create/get/update/add/remove/merge/anonymous flows → Expected: assignLicenseToMe true by default on line items.

---

### API set assign license to true as default [P2] (from DVW-T76)

**Objective:** Confirm create cart, get cart, add/update line item responses include `assignLicenseToMe: true` per line by default (AO/SAO).

**Preconditions:**

- Registered user with cart spanning SKP-PRO, SKP-GO, monthly variants, SKP-STDO.

**Steps:**

1. Create cart (AO/SAO) → Expected: each line assignLicenseToMe true.
2. Get cart → Expected: true on all lines; totals include tax for registered.
3. ADD_CART_LINEITEM → Expected: new/updated lines show true.
4. UPDATE_CART_LINEITEM → Expected: true unless explicitly changed by assign-license API.

---

### Cart Proration [P2] (from DVW-T77)

**Objective:** End-to-end proration: first order, same-day adds, subsequent yearly/monthly purchases, Aria alignment, UI order summary (cart/checkout/review), anniversary and expiry edge cases.

**Preconditions:**

- New AO/SAO accounts and accounts with existing annual/monthly plans; Aria and CT visibility; optional Cybersource/PayPal sandbox for amount checks.

**Steps:**

1. Setup: new AO/SAO without plans; place first yearly order → Expected: no proration, no strike-through price.
2. First order on new account → Expected: no proration deduction or end date.
3. Same-day cart add by AO/SAO → Expected: proration deduction 0; proration end date present.
4. Subsequent days yearly adds → Expected: non-zero proration; end date aligns with first yearly plan in Aria.
5. Aria UpdateAcctWithPlans subsequent order API → Expected: invoice_total_amount matches cart total gross; tax diff matches total tax; proration fields match line totalPriceAfterProrationDeduction; rate*qty vs line_amount for deducted value.
6. Account with annual SKP-PRO/GO adding first monthly → Expected: proration 0; no proration end date for that monthly add per test.
7. After first monthly order, further monthly items → Expected: proration with end date aligned to first monthly instance.
8. Account with both annual and monthly → Expected: yearly prorates to first annual bill-through; monthly to first monthly bill-through.
9. CT UI total gross vs order API centAmount → Expected: match.
10. Repeat purchase UI → Expected: green prorated adjustment, strike-through unit price, tooltip text about estimated discount vs receipt.
11. Order summary on cart, checkout, review → Expected: proration line + renewal date shown.
12. Subsequent purchases → Expected: proration end dates align per SKU.
13. Anniversary date → Expected: proration deduction 0; proration end date aligns with renewed annual bill-through (and monthly variant).
14. After subscription expiry / no renewal → Expected: no prorated adjustment if purchasing after expiry.
15. Upgrade / add seats → Expected: Aria API fields match UI tax, gross, prorated line amounts.
16. Aria, Cybersource/PayPal, getCarts, commerce UI → Expected: same pricing details across systems.

---

## Key Validations

- GetPrice requires valid SKU + allowed currency; CT price parity.
- Anonymous cart: totalTax 0; registered cart: tax computed; cart version tracks CT.
- Cart updates reject invalid actions/SKUs/quantities/line ids; annual+monthly mix blocked (PRODUCTS_1604).
- Merge assign and registered create enforce AO/SAO; product user blocked from assign/create.
- Proration and tax totals align with Aria and downstream payment capture where tests specify.

## Common Preconditions

- Product service base URL, valid SKUs (SKP-GO, SKP-PRO, monthly, SKP-STDO), and max quantity (e.g. 40) for tier.
- Registered flows: bearer token + `x-account-id` from GetAccounts.
- Contentful image base URLs for optional image URL checks.
- For proration: Aria test env access and accounts with controlled subscription state.
