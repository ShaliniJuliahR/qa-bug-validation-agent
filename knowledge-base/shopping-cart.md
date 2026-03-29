# Feature: Shopping Cart

## Overview

Shopping cart UI validation for guest and registered users including cart page display, quantity adjustments, cart merging, tax calculation display, and error scenarios.

## Test Flows

### API - Get Cart By ID (Anonymous cart) [P1] (from DVW-T13)

**Objective:** Exercise get-cart-by-ID behavior for anonymous carts (valid ID, registered ID misuse, invalid ID, expired cart) and validate response shape.

**Preconditions:**

- API access to products/carts service; known anonymous cart ID; sample registered cart ID and invalid/expired IDs for negative cases.

**Steps:**

1. Call GET /carts/{cart-id} with a valid anonymous cart ID → Expected: Cart API returns line items and totals; products match cart.
2. Verify HTTP 200 and payload → Expected: lineItems include imageURL, id, sku, quantity, price, totalPrice, name; totalNet, totalLineItemQuantity, totalTax (0 where applicable), totalGross consistent with line items.
3. Compare each product imageURL to Contentful → Expected: Image URLs align with Contentful source.
4. Call GET with a registered cart ID in the anonymous flow → Expected: Error body errorCode PRODUCTS_1300, errorDescription "Cart not found".
5. Call GET with invalid/malformed cart id → Expected: 400 validation error on id field.
6. Call GET with expired (~7 day) anonymous cart id → Expected: PRODUCTS_1300 Cart not found response.

---

### Guest Cart Assignment to Registered User (No Previous Carts) [P3] (from DVW-T21)

**Objective:** Guest cart is assigned to a registered user who has no existing cart when signing in to checkout.

**Preconditions:**

- Guest session with at least one product in cart.

**Steps:**

1. Start as guest with product(s) in cart → Expected: Cart populated as guest.
2. Click "Sign in to checkout" → Expected: Navigated to TID login.
3. Complete sign-in as user with no prior cart → Expected: Guest cart assigned to logged-in user.

---

### Adjusting Product Quantity in Shopping Cart (Registered) [P2] (from DVW-T22)

**Objective:** Quantity changes on cart page update mini cart, headers, subtotals, tax/totals, removal, max-qty errors, empty state, and debounced updates.

**Preconditions:**

- Registered customer with items on full shopping cart page.

**Steps:**

1. Log in and open shopping cart → Expected: Shopping cart page displayed.
2. Confirm order summary fields → Expected: Estimated subtotal, estimated tax, estimated total show expected values.
3. Increase quantity with "+" for one product → Expected: Mini cart and "Your Cart" sub-header reflect new quantity.
4. Confirm sub-header count → Expected: Item quantity in sub-header updated.
5. Verify cart and order summary subtotals → Expected: Cart subtotal, order summary subtotal, estimated total update.
6. Decrease quantity with "-" on another line → Expected: Mini cart and sub-header update.
7. Re-verify subtotals → Expected: All summary totals stay consistent.
8. Rapidly change quantities → Expected: Updates batch after pause (fewer requests than clicks); no error spam.
9. Remove a line with remove control → Expected: Subtotals and estimated total update after removal.
10. Open promo area → Expected: "Do you have a promo code?" shown but not functional.
11. Try quantity 0 or backspace → Expected: Error: quantity cannot be below 1.
12. At quantity 1, confirm trash icon → Expected: Minus replaced by trash; removal works.
13. Set quantity above max (e.g. 41) → Expected: Inline error about max quantity; field border red; "+" disabled when over max.
14. Confirm Contact Sales link → Expected: Contact Sales is a working hyperlink.
15. Remove all products → Expected: Empty cart message per copy; cart reloads to empty state.
16. On empty cart → Expected: "Continue Shopping" visible and enabled.
17. Compare layout to design → Expected: Fonts, colors, alignment match Figma.

---

### Guest Cart Assignment (Existing Registered Cart) [P3] (from DVW-T26)

**Objective:** Guest cart replaces/assigns when the user already has a saved cart.

**Preconditions:**

- Guest with cart; registered user with existing cart.

**Steps:**

1. Guest with product(s) in cart → Expected: Guest cart ready.
2. Click "Sign in to checkout" → Expected: TID login.
3. Sign in as user with existing cart; return to cart → Expected: Guest cart assigned; replaces prior registered cart per spec.

---

### Validation of Shopping Cart Page - Guest/Registered [P2] (from DVW-T30)

**Objective:** Full cart page for guest (and extended registered/merge flows): layout, line items, order summary, CTAs, assign-license, responsive behavior, and verbiage.

**Preconditions:**

- Guest or registered access; multiple products for scroll/aggregate cases; Figma/reference spreadsheet for copy.

**Steps:**

1. As guest, open cart from PDP "View cart" → Expected: Shopping cart page loads.
2. Verify structure → Expected: "Your Cart" header; cart and order summary side by side; styling matches Figma.
3. Verify line list → Expected: Products show with images; scrollbar when many items.
4. Verify per-line counts in header region → Expected: Total items count matches sum of line quantities.
5. Verify subtotal → Expected: Subtotal matches priced quantities (USD formatting).
6. Verify +/- on quantities → Expected: Controls available per line.
7. Verify per-license price → Expected: Price per license in $0,000 USD style.
8. Remove items until empty → Expected: Empty message matches updated verbiage doc.
9. Order summary → Expected: Subtotal, promotions, estimated tax, estimated total in USD.
10. Promo dropdown → Expected: "Do you have promo code?" shown.
11. Hover info on estimated tax → Expected: Tooltip explains tax estimate.
12. Order summary CTAs (guest) → Expected: "Sign in to checkout" and "Continue shopping" visible.
13. Resize / mobile/tablet → Expected: Layout usable; no broken overlap.
14. Sign in to checkout as user with cart → Expected: After login, checkout with merged carts as designed.
15. Continue shopping → Expected: Return to PDP; as registered, "View cart" opens registered cart with scroll if needed.
16. Cross-check steps 2–11 → Expected: All prior UI checks still pass where applicable.
17. Assign license checkbox per line → Expected: Default selected; toggling updates selection.
18. Toggle assign off and update cart → Expected: Network/update API shows false for assign flag when unchecked.
19. Proceed to checkout and order review → Expected: Assign-license state reflected on checkout and order review.
20. Verify "Proceed to checkout" / "Continue shopping" → Expected: Proceed navigates to checkout.
21. New user path from guest cart sign-in → Expected: Account setup after TID if new user without account.
22. Verify copy vs documentation → Expected: Strings match linked verbiage spreadsheet.

---

### Shopping Cart CTA Navigation (referrerUrl / Continue shopping) [P2] (from DVW-T43)

**Objective:** Cart page and slide-out show correct CTAs; `referrerUrl` returns user to prior PDP; Trimble logo goes to trimble.com.

**Preconditions:**

- Anonymous, registered, empty, and account-specific cart URLs as applicable.

**Steps:**

1. Open cart page or slide-out after add-to-cart → Expected: Context shows Continue shopping where applicable.
2. Click Trimble logo → Expected: Navigate to www.trimble.com.
3. On anonymous cart URL, confirm `referrerUrl` encodes prior PDP path; click Continue shopping → Expected: Return to that PDP.
4. Repeat referrer check on registered cart, empty registered cart, empty anonymous cart, new user empty cart, account-linked `?account=` cart → Expected: Each restores correct relative PDP via Continue shopping.
5. In slide-out after add → Expected: Guest sees Sign in to checkout; registered sees Proceed to checkout (per steps in Zephyr matrix).

---

### Integration Of PDP with Carts Page - Anonymous Flow [P2] (from DVW-T44)

**Objective:** From PDP, add to cart opens flyout; sign-in to checkout assigns anonymous cart across user types; invalid doSignIn URL shows unauthorized.

**Preconditions:**

- Anonymous PDP session; TID test users (with/without CDH, with/without cart).

**Steps:**

1. Anonymous user on PDP → Expected: PDP displayed.
2. Set quantity/plan and add to cart → Expected: Quantity and price updated.
3. Add to cart → Expected: Flyout shows items, prices, images; Sign in to checkout and View cart.
4. Click Sign in to checkout → Expected: TID login; URL pattern uses anonymous cart doSignIn.
5. Login as user with existing cart → Expected: Anonymous cart assigned; cart shown per merge rules.
6. New user without CDH → Expected: Account creation; then anonymous cart assigned.
7. User with CDH, no cart → Expected: Anonymous cart assigned and shown.
8. User with CDH and cart → Expected: Anonymous cart assigned and shown per rules.
9. Hit doSignIn with registered cart id substituted → Expected: Unauthorized error page.

---

### Merge Shopping Carts on Sign-in to Checkout [P2] (from DVW-T49)

**Objective:** Merging guest cart into registered cart on sign-in: happy paths, quantity caps, currency mismatch, subscription mixing, and error handling. (Zephyr contains a large scenario matrix; below are consolidated checkpoints.)

**Preconditions:**

- Guest cart from PDP slide-out; registered users with empty, partial, max-qty, GBP vs USD, and monthly/yearly mixes per scenario doc.

**Steps:**

1. Global check (all positive merges) → Expected: After merge, tax recomputed; subtotal, estimated tax, and total correct on cart, checkout, order review, and submit order.
2. Guest with items; registered with no cart → Expected: After sign-in, registered cart shows guest lines; redirect to checkout when flow specifies.
3. Guest + registered both with items → Expected: Carts merge (quantities combine for same SKU where allowed).
4. Same SKU in both carts → Expected: Single line with combined quantity without duplicate rows.
5. Different SKUs → Expected: All distinct products appear on merged cart.
6. Sign-in from slide-out → Expected: TID then merge outcome per scenario (checkout redirect when specified).
7. Max quantity exceeded on merge (single SKU) → Expected: Cart Quantity Update popup; quantities not increased past max; Continue/X behavior per step (checkout vs cart).
8. Multiple SKUs over max → Expected: Popup lists combined-merge issue; user sent to cart to fix.
9. Currency mismatch (e.g. guest USD, registered GBP) → Expected: Currency popup; registered currency wins; screen greyed until dismissed; optional dual currency+threshold popup text.
10. Threshold + currency conflict → Expected: API may return mergeWarning CURRENCY_MISMATCH_AND_THRESHOLD_EXCEED; UI matches spec.
11. Merge already-consumed anonymous cart (two browsers) → Expected: 404 / error page or merge-failure UX per spec.
12. Merge wrong user's registered cart → Expected: 400 invalid merge / error page.
13. Expired anonymous cart → Expected: 404 not found.
14. Monthly + yearly in guest cart to empty/new user → Expected: Attention popup; monthly/yearly cannot combine; Continue → empty cart page.
15. Monthly/yearly conflicts with existing subscriber → Expected: Popup; Continue returns to cart preserving prior subscription term.
16. Follow Zephyr indices 36–55 for remaining edge repeats → Expected: Same classes of popups, navigation to cart/checkout, and cart-not-updated behaviors as documented in DVW-T49.

---

### Tax Calculation and Display in Cart, Checkout, Order Review [P2] (from DVW-T60)

**Objective:** Tax displays correctly, excludes from subtotal where required, recalculates on cart/qty/address changes; credit card billing address does not retax; Vertex spot-check; merge recalculates tax.

**Preconditions:**

- Logged-in user in checkout/cart; ability to change billing address (US/UK states); optional Vertex comparison.

**Steps:**

1. Preconditions: user in checkout flow → Expected: Ready state documented.
2. On cart, change quantities/lines → Expected: Tax recalculates per update.
3. Confirm subtotal vs total → Expected: Subtotal excludes tax; estimated total includes subtotal + tax.
4. On checkout, view order summary → Expected: Tax shown for default billing + cart.
5. Change quantities on checkout/order review/submit → Expected: Tax updates with qty.
6. Empty cart → Expected: Tax goes to 0.
7. Change checkout billing address (US/UK provinces) → Expected: Tax recalculates for jurisdiction.
8. Edit credit card billing address only → Expected: Tax not recalculated from that change alone.
9. Order review summary → Expected: Estimated total includes tax.
10. Change qty on order review cart section → Expected: Tax updates through submit step.
11. Compare Vertex rate to UI → Expected: Estimated tax matches Vertex percentage for address.
12. After anonymous+registered merge via sign-in → Expected: Tax recomputed and displayed post-merge.
13. Induce tax error address (QA doc cases) → Expected: Error UI matches Figma tax-error template.

---

### Post-Purchase Shopping Cart Functionality [P1] (from DVW-T100)

**Objective:** AXP/MXP flows after first purchase: order review restrictions, add seats, upgrades, cart source MXP vs AXP, assign-license visibility.

**Preconditions:**

- User A completed first order (SKP-GO); access to billing, subscriptions, AXP purchase tab; multiple scenarios in Zephyr.

**Steps:**

1. After first order, use View Orders → Expected: Navigate to /billing; subscriptions show SKP-GO quantity.
2. Open subscription → Expected: Manage subscription available.
3. Add seats from AXP → Expected: Flow reaches commerce order review.
4. On order review → Expected: Address, payment, SKU, tax, proration, quantity visible.
5. Continue to checkout from subscription quantity update → Expected: Lands on order review.
6. User with existing entitlement → Expected: Assign license to me hidden where not applicable.
7. Quantity editable; remove/trash disabled where specified → Expected: Matches AXP upgrade rules.
8. Cancel and go back → Expected: Returns to SKP-GO subscription page.
9. Order review: no Continue shopping → Expected: Only cancel/go back in cart summary where specified.
10. Edit address/payment from order review → Expected: Checkout opens; cart summary shows cancel/go back not continue shopping.
11. Complete or cancel order → Expected: Confirmation with View Orders when placed.
12. Upgrade path SKP-GO → SKP-PRO / SKP-STDO → Expected: Assign license rules per prior purchases (see Zephyr 14–31).
13. AXP cart non-persistence → Expected: New tab PDP while in AXP flow shows empty flyout; getCart 404; starting MXP purchase replaces AXP cart (cart source MXP in create response).
14. Repeat Zephyr branches for second order, add seats, upgrade combinations → Expected: Checkbox visibility and navigation match entitlement state.

---

### Shopping Cart Page & Checkout page: Error Scenarios [P3] (from DVW-T107)

**Objective:** Failure UX for cart fetch, checkout fetch, updates, tax, merge, address services, and payment iframe/PayPal.

**Preconditions:**

- Ability to simulate bad network, invalid tokens, wrong-env cart ids, concurrent merge, etc.

**Steps:**

1. Anonymous cart fetch failure / invalid short id → Expected: Error page.
2. Registered cart id on anonymous endpoint → Expected: 404 / empty cart page per PRODUCTS_1300.
3. Wrong-environment cart id → Expected: 404 + error code 1302 + error page.
4. Registered /carts fetch 4xx/5xx → Expected: Error page (e.g. cross-env user).
5. Network issue on /carts → Expected: Dead end + Continue shopping.
6. Checkout cart fetch failure → Expected: Error page.
7. Network on checkout/order review → Expected: Dead end + Back to Cart.
8. Cart update race (rapid qty/remove) → Expected: Error toaster, not silent failure.
9. Max qty +1 in UI → Expected: Inline field error only, no toaster.
10. Registered cart tax fetch fail → Expected: Error toaster.
11. Checkout tax calculation fail → Expected: Aria tax error page.
12. Merge invalid/re-used anonymous cart → Expected: Merge popup; continue → cart.
13. Address save failure on checkout → Expected: Error toaster.
14. Address typeahead failure → Expected: Error toaster.
15. Address Doctor down → Expected: No Try Again; user can proceed with typed address + service unavailable notice.
16. No address match → Expected: Copy explains no match + recommended actions.
17. Cybersource iframe sign field failure → Expected: Error toaster.
18. Saved payment method load failure → Expected: Error toaster.
19. Legal entity fetch failure → Expected: Error toaster.
20. PayPal init failure → Expected: Error toaster.

---

## Key Validations

- Guest vs registered cart URLs, merge behavior, and redirects (checkout vs cart) stay consistent with account id query params.
- Quantity and removal update mini cart, headers, and all summary totals; debouncing prevents request storms.
- Tax: subtotal vs total semantics, recalculation triggers, and merge/currency mismatch popups match API warnings.
- Error paths surface the correct page (dead end, empty cart, toaster, inline) per endpoint.

## Common Preconditions

- Target e-commerce environment (e.g. QA) with TID test users, optional VPN/geo for locale-specific cases.
- Products with known max quantities and media in Contentful for image checks.
- For API case DVW-T13: valid tool access and non-production cart IDs only.
