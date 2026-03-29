# Feature: Checkout Page

## Overview

Checkout page UI validation including payment accordion, credit card forms, edit payment, cart updates, sign-in redirection, Cybersource iframe, and company name display.

## Test Flows

### Credit Card Error Handling Scenarios [P2] (from DVW-T50)

**Objective:** Card-type validation, field errors, 3DS OTP, AVS/toaster errors, frictionless failure matrix, URL tamper recovery.

**Preconditions:**

- New customer path to checkout; test PANs per Google Sheet in Zephyr; Not_Automated emphasis on manual runs.

**Steps:**

1. Open checkout from cart or signed-in PDP → Expected: Checkout displayed.
2. Expand "Add New Payment Option" → Expected: Section expands.
3. Scenario header: handling card errors → Expected: (section marker).
4. Choose Credit or Debit Card → Expected: Cybersource/card form shown.
5. Invalid card number → Expected: Inline "Enter a valid card number"; Continue to Order Review disabled.
6. Alphanumeric PAN → Expected: Same invalid number error.
7. Wrong length PAN → Expected: Length error message.
8. Wrong card type vs selected brand → Expected: Card type mismatch error.
9. Bad expiry → Expected: Invalid expiry message.
10. Empty PAN/CVV/expiry/type → Expected: Required-field errors on save.
11. 3DS card → Expected: OTP dialog appears after save attempt.
12. Wrong OTP → Expected: Invalid OTP error.
13. OTP timeout → Expected: Timeout error message.
14. AVS bad billing address → Expected: Toaster: problem processing card, retry.
15. Sheet cases (frictionless/step-up failures) → Expected: Matching errors; no payment token (per sheet).
16. Fix all fields → Expected: Errors clear; Continue to Order Review enables.
17. URL tamper scenario precondition → Expected: Documented mid-checkout bad URL.
18. Break checkout URL → Expected: Checkout error page with copy about unavailable page and Back to Cart.
19. Click Back to Cart → Expected: Returns to cart.
20. View collapsed saved payment → Expected: Collapsed payment summary visible.
21. Scenario: URL change grouping → Expected: (section marker).

---

### Edit Payment Details, Replace Saved Payment - Overlay [P2] (from DVW-T51)

**Objective:** Replacing saved card: overlay messaging, Continue vs Cancel/X, Cybersource form, default payment update, order review edit entry.

**Preconditions:**

- User with saved payment on checkout.

**Steps:**

1. Open checkout from cart or PDP → Expected: Checkout page.
2. Account address default → Expected: Billing/company from account creation selected.
3. Saved payment visible → Expected: Prior card shown in payment section.
4. Expand "Add a New Payment Option" → Expected: Accordion opens.
5. Select Credit Card → Expected: Payment type chosen.
6. Cybersource form + overlay → Expected: Replacement warning overlay appears with long override message.
7. Rest of page greyed → Expected: Continue to Order Review disabled until overlay resolved.
8. Cancel or X on overlay → Expected: Overlay closes; accordion stays expanded; no payment change.
9. Cybersource form closes on cancel path when specified → Expected: Per Zephyr dual check.
10. Continue to Order Review after cancel path → Expected: Button enabled when valid.
11. Re-open add new → Expected: Overlay appears again.
12. Click Continue on overlay → Expected: Overlay dismisses; full Cybersource form usable.
13. Submit new card → Expected: Old method replaced; becomes default for Aria-managed subs.
14. Continue to Order Review → Expected: Enabled after successful save.
15. Order review edit section → Expected: (section marker).
16. DVW/TDX first-order users → Expected: Edit on order review navigates to checkout with payment accordion expanded.

---

### Validation of Checkout (No Saved Payment) & Payment Accordion [P1] (from DVW-T52)

**Objective:** First-time payer: order summary, cart container, address, legal-entity-driven methods, Cybersource happy path, token, collapse, toaster, re-login persistence.

**Preconditions:**

- New user without saved payment.

**Steps:**

1. Precondition logged → Expected: User has no saved cards.
2. Proceed to checkout → Expected: Checkout page.
3. Trimble logo → Expected: Goes to www.trimble.com; back returns to checkout.
4. Order summary → Expected: Subtotal, promotions, estimated tax, estimated total visible.
5. Checkout cart preview → Expected: Product name, price, qty.
6. Show Details → Expected: Expand lists cart lines; Continue Shopping goes to recent PDP.
7. Account address default + Edit link → Expected: Default address; Edit opens form; country not editable when specified.
8. Use address for new card checkbox → Expected: Checking autopopulates billing iframe when expanded.
9. Uncheck while iframe open → Expected: Close/reopen iframe clears billing fields until rechecked behavior per spec.
10. Re-check behavior → Expected: Address repopulates after manual iframe toggle sequence.
11. No saved payment message → Expected: Prompt to add new payment under section.
12. Order review accordion → Expected: Disabled until payment satisfied.
13. Add New Payment Option → Expected: Dropdown/accordion available.
14. Payment methods list → Expected: Driven by Legal Entity / country API.
15. Pick Credit Card → Expected: Form + billing prefill when checkbox used.
16. Enter valid Visa (repeat Mastercard/Amex/Discover per note) → Expected: Validates vs billing address without false errors.
17. Submit → Expected: Accordion collapses to saved-card summary; no blank page.
18. Toaster → Expected: "Your Credit or Debit Card has been saved."
19. Cybersource token → Expected: Token created in network/Cybersource.
20. Success response → Expected: Payment section collapsed edit view with last4/expiry/brand.
21. Order review accordion → Expected: Enabled after payment saved.
22. Logout/login → Expected: Saved method still default on return to checkout.
23. UI copy → Expected: Strings match verbiage spreadsheet.
24. Figma parity → Expected: Layout matches design.

---

### Checkout Page - Cart Update, and Empty Cart [P2] (from DVW-T53)

**Objective:** Inline cart on checkout: show/hide details, qty/remove, assign-license flag API, max qty error, empty cart messaging, Continue shopping.

**Preconditions:**

- Registered checkout after PDP add + sign-in to checkout.

**Steps:**

1. Reach checkout with items → Expected: Cart container lists products.
2. Show Details → Expected: Expands to line details; toggles Hide Details label.
3. Remove control and +/- → Expected: Visible per line.
4. Decrease qty → Expected: Totals and tax in order summary update.
5. Rapid qty changes → Expected: Debounced API; no errors when spamming.
6. At qty 1 → Expected: Trash + "+" shown.
7. Enter qty > 40 (per env max) → Expected: Max quantity error message.
8. Remove last item → Expected: "Your cart is currently empty" in container.
9. Order summary → Expected: Values go to zero.
10. Assign license rows → Expected: Checkbox + tooltip text per spec.
11. Tooltip click → Expected: Copy about license assignment.
12. Assign checked updates API → Expected: Network update uses true for flag.
13. Assign unchecked → Expected: Network uses false.
14. Change assign; hit /carts and order review → Expected: Checkbox state consistent everywhere.
15. Continue Shopping in container → Expected: Navigates to PDP to add more.

---

### Validation of Credit Card Details Functionality (Cybersource iframe) [P2] (from DVW-T58)

**Objective:** Iframe UX: no scrollbar, field validation, billing autofill checkbox, Address Doctor, character counters, Save/Cancel, Edit flow, responsive/Figma.

**Preconditions:**

- Browser-stored cards optional; user on billing section; Partially_Automated.

**Steps:**

1. Precondition: saved browser card optional; on checkout billing → Expected: Credit card option listed.
2. Iframe scrollbar → Expected: No scrollbar inside Cybersource iframe.
3. Address line 1/2 max length → Expected: Error if >40 chars.
4. First/last name max → Expected: Error if >39 chars; valid up to 39 accepted.
5. Address lines valid length → Expected: Up to 40 chars accepted.
6. Navigate to checkout if not already → Expected: Checkout displayed.
7. Default account address → Expected: Selected in account section.
8. Open Add new payment → Expected: Section reachable.
9. Select credit/debit → Expected: Billing form stage shown.
10. "Use account address for billing" checkbox + tooltip → Expected: Present; hover explains autofill behavior.
11. Check autofill → Expected: Billing fields prefilled from account/CDH match.
12. Clear All → Expected: Clears fields and errors.
13. Required helper text under Clear All → Expected: "* All fields required unless optional" shown.
14. Validate First/Last name fields → Expected: Placeholders clear on type; required enforcement.
15. Country dropdown → Expected: Required; Save disabled until chosen.
16. Street, optional building, city, zip rules → Expected: Special-char errors per messages; valid clears errors.
17. State dropdown → Expected: Placeholder behavior; Save rules per Zephyr.
18. Phone mask/validation → Expected: Invalid chars blocked; Save disabled if invalid.
19. Email validation → Expected: Invalid email error message.
20. Save billing step → Expected: Save button available; valid data proceeds to card step.
21. After Save → Expected: Shows entered billing + Edit + Cybersource step 2.
22. Edit billing then submit card → Expected: Updates propagate; card submit completes flow.
23. Cancel → Expected: Collapses credit section.
24. Browser autofill on billing fields → Expected: Suggestions fill fields.
25. Character counters on named fields → Expected: x/max format per spec table in Zephyr.
26. Address Doctor: partial address → Expected: Suggestions list.
27. Select suggestion → Expected: Fields populate.
28. Incorrect address path → Expected: Suggestion or confirm incorrect flows per steps 37–39.
29. Save without edits after prior save → Expected: No redundant Doctor popup.
30. Responsive + Figma compare → Expected: Breakpoints usable; spacing matches design.
31. CDH / further payment tests → Expected: Cross-reference DVW-1108 and DVW-2270 per Zephyr notes.

---

### Sign-in Checkout Redirection [P2] (from DVW-T61)

**Objective:** From anonymous PDP/cart slide-out or registered paths, reach checkout with correct account id, merges, and Figma-driven new/returning user journeys.

**Preconditions:**

- Logged-out PDP start; valid/invalid account query params; Figma links for new vs returning user matrices.

**Steps:**

1. Log out; PDP; add to cart → Expected: Slide-out opens.
2. Scenario: sign-in from anonymous PDP → Expected: (section).
3. Click Sign in to checkout → Expected: TID login.
4. After login → Expected: Checkout with default account id in URL; merged carts in container when applicable.
5. Verify merged products → Expected: Anonymous lines combined with registered cart.
6. Scenario: registered PDP proceed → Expected: (section).
7. Continue shopping / new tab PDP still signed in → Expected: PDP loads; slide-out has Proceed to checkout.
8. Add products; Proceed from slide-out → Expected: Checkout opens.
9. Direct `checkout?account={id}` → Expected: Checkout for that account when authorized.
10. URL shows account id on checkout → Expected: Query matches active account.
11. Invalid account id → Expected: Dead end + Continue shopping.
12. Anonymous cart page sign-in scenario → Expected: Sign in to checkout → TID → merged checkout as specified.
13. Account id in URL after cart page sign-in → Expected: Registered id present.
14. Continue shopping back to PDP; add more; View cart → Expected: Registered cart page; Proceed to checkout visible.
15. Click Proceed → Expected: Checkout displayed.
16. Figma scenarios 24–32 (new/returning user variants) → Expected: Follow linked Figma nodes for exact steps and assertions.

---

### Continue Shopping Button Functionality [P3] (from DVW-T62)

**Objective:** Continue Shopping from billing/cart summary returns to recent PDP; empty cart; order review; account id in URL; dead-end pages use Continue Shopping to MXP.

**Preconditions:**

- Checkout session with items; optional order review step.

**Steps:**

1. PDP → sign-in to checkout → Expected: Checkout (sample URL in Zephyr).
2. Trimble logo path to create account entry (per step) → Expected: Navigation chain works.
3. After TID create, merge → Expected: Redirect to cart then checkout via proceed.
4. Expand Show Details; Continue Shopping → Expected: Lands on last visited PDP.
5. Collapsed cart Continue Shopping → Expected: Same PDP return.
6. Navigate back to checkout → Expected: Checkout shown.
7. Repeat collapsed vs expanded Continue Shopping → Expected: Consistent PDP return.
8. Order review path: Continue Shopping with consent checked/unchecked → Expected: CTA visible; navigates to PDP; account id in URL for registered flows.
9. Empty cart via Show Details removes → Expected: Empty message; Continue Shopping from empty state → PDP.
10. Generic dead-end pages → Expected: Continue Shopping replaces Back to Cart where specified; routes to MXP to resume shopping.

---

### Edit Account Address in Checkout Page [P2] (from DVW-T71)

**Objective:** Account address display/edit: mandatory rules, special chars, typeahead, Address Doctor, DVR read-only, TDX/EBS no-edit rules, PATCH verification, order review truncation.

**Preconditions:**

- Checkout access; UK/US profile users for state validation; Not_Automated for full pass.

**Steps:**

1. Open checkout → Expected: Account address with Edit when allowed.
2. Validate display gaps → Expected: No extra gaps when optional lines missing.
3. DVR provisioned user → Expected: No Edit; read-only address.
4. Section: edit from order review → Expected: (marker).
5. DVW source users → Expected: Edit on checkout and order review; OR Edit opens checkout address form.
6. TDX/EBS users → Expected: No Edit CTA on checkout/order review.
7. Country field → Expected: Not editable in edit form.
8. Street/building/city/state/zip defaults → Expected: Prefilled; mandatory/optional per country rules.
9. Clear required fields → Expected: Placeholders; Save disabled when invalid.
10. Special characters in street/building/city/zip → Expected: Matching error strings.
11. Email read-only; phone disabled → Expected: Values from account creation.
12. UK user picks non-UK suggestion → Expected: State/province validation error until fixed.
13. State optional countries → Expected: Field becomes optional with visual hint.
14. State required countries → Expected: Stays mandatory.
15. Valid save → Expected: Address updates in UI; PATCH payload includes required fields; CDH/GetAccounts isBillTo true.
16. Typeahead: 10 suggestions max; first five visible + scroll → Expected: Behavior per spec.
17. Address Doctor flows (with/without typeahead, partial edits) → Expected: Popups or skip verification per step group 31–33 and 40–44.
18. Wait 1h then Save/Cancel → Expected: Save updates; Cancel shows latest without full reload.
19. Browser autofill on street → Expected: Fills all fields from picker.
20. Order review collapsed payment view → Expected: Long addresses truncate cleanly.

---

### Cybersource IFrame & Session Timeout [P3] (from DVW-T89)

**Objective:** Saved-card autofill in iframe, session timeout collapse + reload, validation errors, Figma.

**Preconditions:**

- User already saved a card (see DVW-1136 reference); checkout payment accordion.

**Steps:**

1. Precondition: expand add payment; save card per linked test → Expected: Baseline saved payment exists.
2. Iframe fields → Expected: Card type, number, expiry, CVV, Submit; mandatory labels.
3. Legal entity drives card types → Expected: Options match country/locale mapping.
4. Hover/focus card field → Expected: Browser saved cards listed if any.
5. Pick saved card → Expected: Fills type/number/expiry.
6. Fields remain editable → Expected: User can adjust before submit.
7. Submit → Expected: Saves; shows last4 + expiry + logo.
8. Session timeout section → Expected: (marker).
9. Stay idle ~15 min in iframe → Expected: Timeout response; accordion auto-collapses.
10. Toaster → Expected: Session timeout indicated.
11. Re-expand accordion → Expected: Iframe reloads fresh.
12. Submit with missing mandatory → Expected: Per-field required errors.
13. Default payment → Expected: New saved option becomes default.
14. Figma → Expected: UI matches mockup.

---

### Display company name on checkout and order-review page [P2] (from DVW-T139)

**Objective:** Organization accounts show company name under address on checkout and order review; individual accounts do not.

**Preconditions:**

- Ability to create org vs individual account during setup.

**Steps:**

1. PDP add products → Expected: Items in cart.
2. Sign in to checkout → Expected: TID sign-in.
3. Account setup: Yes to business/org + company details → Expected: Form accepts org data.
4. Complete account setup → Expected: Logged in as org user.
5. On checkout → Expected: Company name visible under account address.
6. Enter shipping/billing → Expected: No blocking errors.
7. Continue to Order Review → Expected: Order review opens.
8. On order review → Expected: Company name still under address.
9. Visual check → Expected: Company name legible; no overlap.
10. Create individual via flyout Create Account → Expected: Account creation form.
11. After creation → Expected: Return to PDP.
12. Add to cart; Proceed to checkout → Expected: Account setup for individual path if needed.
13. Profile: No to business purchase → Expected: Personal details only.
14. Complete setup → Expected: Checkout for individual.
15. Checkout address → Expected: No company name line.
16. Enter shipping/billing → Expected: Completes without company field.
17. Continue to Order Review → Expected: Order review route.
18. Order review address → Expected: No company name; personal lines only.

---

## Key Validations

- Payment accordion gating: order review stays locked until valid billing + tokenized card (or allowed method) is saved.
- `Continue to Order Review` only enables when card validation passes; overlays block background interaction.
- Inline checkout cart changes reprice tax/totals and respect assign-license API flags.
- Company name visibility matches account type (org vs individual) on both checkout and order review.

## Common Preconditions

- Registered user through account setup with billing country matching payment method availability.
- Test cards and 3DS simulator data from linked spreadsheets in Zephyr.
- Cybersource and Address Doctor available in target environment; session timeout tests need 15+ minute window.
