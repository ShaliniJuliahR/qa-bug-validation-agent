# Feature: Order Review & Confirmation

## Overview

Order review page validation including hover information, cart updates, empty cart handling, upgrade subscription flow, add licenses flow, and order submission/generation.

## Test Flows

### Order Review Page - Hover Information, Cart Update, and Empty Cart [P2] (from DVW-T48)

**Objective:** Verification of hovering over the Estimated Tax icon, updating the cart, and checking placeholder messages when the cart is empty.

**Preconditions:**

- At least one product in cart; user reaches Order Review with products loaded.

**Steps:**

1. Navigate to Order Review Page with at least one product. → Expected: Order Review Page is displayed.
2. Verify account address and billing address are displayed in collapsed view with edit button. → Expected: Account address and billing address are displayed in collapsed view with edit button.
3. Validate Building, Floor, Suite; City, State/Province; Zip/Postal Code; Country/Territory; gaps should not show when optional fields are empty. → Expected: Mandatory and optional address fields display without visible gaps when non-mandatory fields are missing.
4. Confirm lengthy address characters are truncated and displayed. → Expected: Lengthy address characters are truncated and displayed.
5. Ensure payment method shows user's card details (last four digits, expiry). → Expected: Displayed First Name and Last Name match user information (card section).
6. Verify the logo of the respective card is displayed in payment method. → Expected: Respective card logo is displayed.
7. Verify Edit is displayed and clickable; redirects to checkout. → Expected: Edit works and redirects to checkout to update payment.
8. In Checkout, click Edit to update payment method. → Expected: Updated payment method by clicking Edit.
9. Click Continue to Order review; verify updated payment in collapsed view. → Expected: Updated payment method displayed correctly.
10. Hover over Estimated Tax info icon. → Expected: Respective information is displayed.
11. Verify Assign License Container per line item: radio options "Assign a license to me" (default) and "Do not assign a license to me". → Expected: Assign License Container as specified; default is assign to me.
12. Click info tooltip on assign license. → Expected: Tooltip: "By selecting this, you will have a access to use this subscription."
13. Verify cart update API shows true when "Assign a license to Me" is selected (network). → Expected: Backend reflects true when adding/removing with assign to me.
14. Verify cart update API shows false when "Do not assign a license to me" is selected (network). → Expected: Backend reflects false for do-not-assign selection.
15. Go to /carts and back through checkout/order review; verify assign license persistence rules. → Expected: Assign license change reflected per navigation rules in case.
16. Rapidly change quantities; verify debounced API (pause before update). → Expected: Fewer network calls than interactions; no error.
17. Click '+' to increase quantity. → Expected: Quantity is increased.
18. Click '-' to decrease quantity. → Expected: Quantity is decreased.
19. Click '-' until quantity 1. → Expected: Trash can appears left of quantity.
20. Enter quantity greater than 40. → Expected: Error: quantity exceeds maximum; contact sales message.
21. Verify Remove CTA and click Remove. → Expected: Remove CTA displayed and clicked.
22. If removing last product, confirm popup "Are you sure you want to remove this item from your cart". → Expected: Popup appears.
23. Verify "Yes, Delete It" and "No, Keep it" in popup. → Expected: Both buttons displayed and clickable.
24. Click "Yes, Delete It" and remove all products. → Expected: Message "Your cart is currently empty" is displayed.
25. Click "No, Keep it". → Expected: All products remain in cart.
26. Update cart; verify order summary recalculates. → Expected: Prices and taxes recalculated for every cart update.
27. Verify UI vs Figma across Mobile, Tablet, Laptop. → Expected: UI matches Figma including responsive.
28. Update quantity or add products; verify subtotal, estimated tax, estimated total. → Expected: Order Summary fields update accordingly.

---

### Submit Order Page- Generate Order [P2] (from DVW-T64)

**Objective:** Successful order placement, Aria account creation, payment authorization, redirect to Order Confirmation; UI vs Figma; failure and deep-link behaviors.

**Preconditions:**

- Completed checkout through Order Review with valid payment where applicable.

**Steps:**

1. From Order Review, verify payment and billing, accept Disclosure & Consent, click Place Order. → Expected: Navigate to Submit Order with Processing pop-up.
2. Verify Processing pop-up text ("Processing..", "Please Wait.", "We are placing your order") and greyed background. → Expected: Pop-up and non-editable greyed screen.
3. Verify transition to success: Cybersource full authorization; order summary. → Expected: Processing changes to confirmation; tax and totals correct on submit page.
4. Verify Trimble logo on submit page; click goes to www.trimble.com. → Expected: Logo and navigation as specified.
5. Click browser back. → Expected: User returns to empty cart page.
6. Verify order number matches Place Order API response. → Expected: Order number matches backend.
7. On failures during Aria account creation, card authorization, entitlement, or subscription: verify error toaster on Order Review. → Expected: Error toaster on order review for listed failures.
8. Place order without payment method. → Expected: Toaster about insufficient funds / update payment method.
9. Cart vs account currency mismatch with entitlement failure but order generated case. → Expected: Error toaster behavior per case definition.
10. Annual and monthly in cart with entitlement failure but order shown. → Expected: Order number generated and details displayed per case.
11. Tax 0 / Aria not setup scenario. → Expected: Specific support toaster on order review.
12. Large gross / card authorization failure. → Expected: Error toaster on order review.
13. Legal entity failure (no currency). → Expected: Error toaster on order review.
14. Verify confirmation page: order number, line items, totals, thank-you message with email. → Expected: All confirmation elements displayed.
15. Deep-link matrix: from /orderConfirmation navigate to /checkout, /carts, /account, /orderReview, base URL, and same-page reload — verify redirect targets and empty-cart rules per DVW-T64 indices 15–27. → Expected: Each URL pair behaves as specified in Zephyr (mostly redirect to /carts or /checkout with empty cart).
16. Verify confirmation message per Figma. → Expected: Matches design.
17. Responsive Figma check. → Expected: Mobile, tablet, laptop.
18. Verify confirmation text vs verbiage spreadsheet linked in Zephyr. → Expected: Copy matches document.

---

### Order Review Page Validation [P2] (from DVW-T67)

**Objective:** Field and layout validations on Order Review (billing, cart, disclosure, assign license, CTAs).

**Preconditions:**

- User on Order Review with at least one subscription in cart; credit card selected in checkout.

**Steps:**

1. (Pre-condition step) User on Order Review with subscription in cart and card payment. → Expected: Precondition documented in Zephyr.
2. Click Continue to Order Review from checkout. → Expected: Order Review displayed with required elements.
3. Verify Trimble logo; click navigates correctly. → Expected: www.trimble.com page.
4. Back button. → Expected: Returns to Order Review context per flow.
5. Collapsed billing with address and payment. → Expected: Collapsed billing shows payment and billing address.
6. Address shows user first/last name with billing address. → Expected: As specified.
7. Card logos (Visa/Mastercard/Amex) in collapsed payment. → Expected: Correct logos.
8. Loader until payment method loads in collapsed view with Edit. → Expected: Loader until loaded.
9. After hard refresh, logout/login, loader until payment visible. → Expected: Loader behavior confirmed.
10. Shopping cart container below billing per Figma. → Expected: Cart aligned per mockup.
11. Order summary beside collapsed billing. → Expected: Summary placement correct.
12. Cart details include tax in summary. → Expected: Tax details in cart/summary.
13. Assign license options with radios. → Expected: Options displayed for convenience.
14. Assign-license tooltip matches updated Figma text. → Expected: Tooltip text as per design.
15. Disclosure & Consent with Supplemental Terms and Offering Terms links; Place Order disabled until acceptance. → Expected: Disclosure and disabled Place Order until accept.
16. Accept disclosure; Place Order enables. → Expected: Place Order enabled after acceptance.
17. Open Supplemental Terms link in new tab. → Expected: Correct legal URL in new tab.
18. Open Offering Terms link in new tab. → Expected: Correct legal URL in new tab.
19. Edit in billing redirects to Checkout. → Expected: Redirect to checkout.
20. Continue Shopping CTA to PDP. → Expected: CTA visible and routes to PDP.
21. Responsive Figma validation. → Expected: All breakpoints match.

---

### Cart Gross total Response Frontend Integration [P2] (from DVW-T75)

**Objective:** Tax and gross totals update across PDP/cart/checkout/order review/submit; backend cart tax recalculation; GetCarts source behavior.

**Preconditions:**

- Registered user; TID login; product in cart.

**Steps:**

1. PDP slideout sign-in; login with TID. → Expected: Navigated to PDP.
2. Add to registered cart; View Cart. → Expected: Registered cart page.
3. On carts page, increase quantity; verify order summary (price, estimated tax, total). → Expected: Summary values correct.
4. Verify cart total gross tax recalculated at backend API. → Expected: Backend tax recalculated.
5. Proceed to Checkout; increase quantity in checkout cart; verify summary. → Expected: Summary correct on checkout.
6. Verify backend tax recalculation again. → Expected: Backend matches.
7. Continue to Order Review; increase quantity; verify summary. → Expected: Order review summary correct.
8. Verify backend tax recalculation. → Expected: Backend correct.
9. Agree terms; Place Order; on Submit Order page verify summary. → Expected: Submit page summary correct.
10. Verify backend tax after submit step. → Expected: Backend consistent.
11. GetCarts with source mxp when cart from MXP. → Expected: Cart retrieved and displayed.
12. Retrieve AXP-sourced cart then delete; response not found. → Expected: Not found after delete per scenario.

---

### Upgrade Subscription - Order Review Flow for AXP [P1] (from DVW-T87)

**Objective:** AXP-driven upgrade subscription through commerce order review with proration, EMS/Aria/Salesforce checks, and edge cases (license admin, stale tab).

**Preconditions:**

- Post-commerce purchase: Salesforce assets with EMS IDs; subscriptions visible in AXP (per Zephyr).

**Steps:**

1. (Section) Upgrade subscription from AXP. → Expected: Narrative header in Zephyr.
2. AO/SAO: account with DVW source and commerce purchase; Manage subscription → Change subscription. → Expected: Comparison screen; current subscription selected; tier SKUs with quantity, price, software.
3. Select valid upgrade SKU; verify continue-to-checkout message with prorated charge text and CTA. → Expected: Message and Continue to checkout as specified.
4. Continue to checkout. → Expected: Commerce order review.
5. (Section) Subscription upgrade validations in Commerce. → Expected: Header.
6. Try to change quantity of upgradable SKU on order review. → Expected: Quantity disabled; cannot alter count.
7. Verify subtotal, prorated adjustment, estimated tax, total vs Aria UpdateAcctWithPlans and Vertex. → Expected: Amounts match Aria/Vertex.
8. Place Order with upgraded SKU. → Expected: Success and Thank You page.
9. Verify checkout/cart with AXP upgrade cart (same as prior cases). → Expected: Same as cases 6–7.
10. (Section) Order details via GetOrder. → Expected: Header.
11. Hit GetOrder for account; validate Subsequent purchase shape per spreadsheet. → Expected: 200 OK; body matches doc.
12. (Section) ARIA plan instance. → Expected: Header.
13. Find plan instance by client account ID; Plans. → Expected: Same plan instance ID retained; plan name = upgradable SKU.
14. Verify EMS ID under plan instance fields; EMS GetEntitlement matches dates and license counts. → Expected: EMS aligns with GetOrder EMS display.
15. (Section) Salesforce. → Expected: Header.
16. Verify EMS ID, taxes, quantity, license records in Salesforce. → Expected: Updated EMS; licenses from old asset on new asset; taxes/qty per upgraded SKU.
17. Verify upgraded subscription asset id in Salesforce. → Expected: New asset id for upgraded subscription.
18. (Section) MXP cart flow. → Expected: Header.
19. With existing AXP cart, add MXP product to same account. → Expected: AXP cart replaced; only MXP product shows.
20. Open order review with AXP cart in one tab; in another tab add MXP product and open order review; on first tab Place Order. → Expected: Popup to refresh browser to sync cart.
21. Login as license admin to AXP. → Expected: Cannot upgrade subscription from AXP.

---

### Add Licenses to a Subscription - Order Review flow for AXP [P1] (from DVW-T99)

**Objective:** Add seats from AXP through commerce order review with proration, GetOrder, Aria, EMS, Salesforce, and MXP cart conflict behavior.

**Preconditions:**

- Commerce purchase completed; Salesforce assets with EMS; subscriptions in AXP.

**Steps:**

1. (Section) Add seats from AXP. → Expected: Header.
2. AO/SAO: DVW account with commerce purchase; Manage subscription → Change license count. → Expected: Popup to modify quantity.
3. Update license count; Continue to checkout. → Expected: Commerce order review.
4. (Section) Add seats validations in Commerce. → Expected: Header.
5. On order review, change quantity of SKU; verify order summary updates. → Expected: Quantity enabled; summary updates with quantity changes.
6. Verify subtotal, prorated adjustment, tax, total vs Aria and Vertex. → Expected: Matches APIs.
7. Place Order. → Expected: Thank You page.
8. Verify checkout/cart behavior vs add-seats cart (note version note in Zephyr on upgrade vs add seats). → Expected: Per T6–T7 with noted differences.
9. (Section) GetOrder. → Expected: Header.
10. GetOrder for user; validate Subsequent purchase per spreadsheet. → Expected: 200 and valid body.
11. (Section) ARIA plan instance. → Expected: Header.
12. Plan instance retained; quantity updated per selected count (note: promo may create separate plans). → Expected: Plan id retained; qty updated.
13. EMS ID under plan instance; EMS GetEntitlement; note new EID for subsequent same SKU. → Expected: EMS matches GetOrder; correct QTY and dates.
14. (Section) Salesforce. → Expected: Header.
15. EMS, taxes, quantity, licenses in Salesforce. → Expected: Updated; note two assets possible per promo changes.
16. Asset id for add seats: should not change to new asset for license count update only. → Expected: Asset id behavior per case.
17. MXP cart while AXP cart exists. → Expected: AXP cart cleared; MXP product only.
18. Stale-tab Place Order after MXP add (same pattern as upgrade). → Expected: Refresh/sync popup per Zephyr.
19. (Section) Negative. → Expected: Header.
20. License admin login. → Expected: Cannot add licenses from AXP.

---

### Order_Review Page Error Scenarios [P3] (from DVW-T113)

**Objective:** Error handling for missing payment, slow network, cart/address failures, place-order failures, support CTA, stale multi-tab cart.

**Preconditions:**

- Scenarios use checkout without payment, throttled network, or invalid token as described per step.

**Steps:**

1. (Section) Failure to save / missing payment on Order Review. → Expected: Header.
2. Checkout with no payment method saved. → Expected: Navigated to checkout; no payment on account.
3. Continue to Order Review disabled when no payment. → Expected: Button disabled and not clickable.
4. Access /order_review without payment (deep link scenario). → Expected: Redirect to checkout to add payment (per case).
5. Attempt continue without payment option. → Expected: Toast: enter payment before order review (per Figma).
6. Throttle network; add card; Continue to Order Review. → Expected: Toast to refresh page if payment cannot be loaded on order review.
7. (Section) Failure to retrieve account address. → Expected: Header.
8. Simulate address fetch failure (network or invalid token). → Expected: Full-page error "We're Sorry, we encountered an error."
9. (Section) Failure to fetch carts. → Expected: Header.
10. Simulate cart fetch failure. → Expected: Full-page "We're Sorry, we encountered an issue with your cart."
11. Change quantity on order review with save-cart failure (slow network). → Expected: Toast: unable to update shopping cart, refresh.
12. Place order on slow network. → Expected: Toast processing error with customer support CTA.
13. Click Customer Support CTA. → Expected: Redirect to help center URL in Zephyr.
14. Duplicate order review tabs; change qty in one; place order from stale tab. → Expected: Popup to refresh for latest cart; after refresh, order can complete.

---

## Key Validations

- Order summary, tax tooltips, and cart mutations stay consistent with backend cart and payment APIs.
- Disclosure acceptance gates Place Order; legal links open correctly.
- Upgrade/add-seats flows enforce quantity rules and sync across AXP, commerce, Aria, EMS, and Salesforce.
- Confirmation and error paths match Figma copy and toaster behavior.

## Common Preconditions

- Valid Trimble Identity, shop and AXP URLs for target environment, and test SKUs (e.g., SKP-GO, SKP-PRO) as required by the case.
- Access to browser devtools (network) when validating assign-license and cart APIs.
