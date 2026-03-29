# Feature: Order Processing & Salesforce

## Overview

End-to-end order processing including Salesforce integration for first orders, renewals, upgrades, downgrades, add seats, cancellation, revenue sync, and EEUC processes.

## Test Flows

### API - Upgrade Asset Order Processing [P1] (from DVW-T88)

**Objective:** SKP-GO first order upgraded to SKP-PRO via AXP with Aria, GetOrders, and Salesforce asset/order product validation.

**Preconditions:**

- New TID account; first order SKP-GO with assign license true.

**Steps:**

1. (Section) UI steps for upgrade asset processing. → Expected: (Header.)
2. Precondition: create account; first order SKP-GO; assign license true. → Expected: First order created with SKP-GO.
3. AXP Purchases: SKP-GO visible. → Expected: Under Purchases/Subscriptions tab.
4. Manage subscriptions → Change subscription → pick SKP-PRO. → Expected: SKP-PRO selected.
5. Continue to checkout → Order Review; assign-license checkbox hidden for upgrade. → Expected: Order review populated; no assign-to-me checkbox.
6. Place Order; confirm confirmation UI. → Expected: Order details on confirmation; view orders.
7. (Section) Aria verification. → Expected: (Header.)
8. Verify SKU upgraded in plan. → Expected: SKU updated to upgraded product in plan.
9. Quantity unchanged. → Expected: Quantity remains same.
10. Verify invoices/payments vs Aria UpdateAcctWithPlans and UI (tax, proration, total). → Expected: Payments/invoices match UI.
11. GetOrders for account. → Expected: Line item type FULL_UPGRADE; Aria status Subscription_Created; dxOrderStatus Activated; entitlement fields populated.
12. assignLicenseToMe false; licenseId null. → Expected: As specified for upgrade flow.
13. Salesforce asset, order, order products. → Expected: New EMS; same SF asset id maintained; new order with upgrade type; updated EMS and rev id on products.
14. Asset history: previous and new EMS IDs; order products negative old SKU / positive new SKU. → Expected: History and two order product rows as specified.
15. Positive line: price ex tax and tax per line. → Expected: Line-level tax and price correct.
16. Order total and Vertex tax lines. → Expected: Totals and Vertex lines match UI.
17. AXP Purchases: SKP-PRO shown; subscriptions clubbed. → Expected: Subscription updated in AXP.

---

### Salesforce- Downgrade During Renewal [P1] (from DVW-T90)

**Objective:** First order SKP-STDO with assign license; downgrade path through renewal with Aria future plan change, Salesforce, and MXP blocking error.

**Preconditions:**

- DVW account; SKP-STDO first order with assign license checked.

**Steps:**

1. Create account; place first order SKP-STDO with assign license. → Expected: Order placed successfully.
2. Aria: account with plan instances. → Expected: Plan with entitlement and asset id.
3. Salesforce: account matches Get Accounts name. → Expected: Account created.
4. Related order: new sale; total, tax, order product. → Expected: Order captured correctly.
5. Order products: SKUs, asset id, PD per product. → Expected: As specified.
6. License id visible when assign license checked. → Expected: License id in assignment.
7. AXP: SKP-STDO under Purchases. → Expected: SKU visible.
8. Change subscription → SKP-PRO (downgrade scheduled). → Expected: SKP-PRO selected in change flow.
9. Aria Future Plan Change: Downgrade Scheduled in queue. → Expected: Plan change queued matches downgrade.
10. Note anniversary; advance virtual clock; pending invoice. → Expected: Pending invoice in Statements.
11. After pending invoice completes: Aria plan shows downgraded SKU; qty and anniversary correct. → Expected: Plan tab updated.
12. Future plan change moves to Completed; status Executed Successfully (Batch). → Expected: Queue completed with correct metadata.
13. (Section) Salesforce. → Expected: (Header.)
14. Salesforce: new order link; downgraded SKU; new EMS; license retained; service period updated. → Expected: SF matches downgrade outcome.
15. Datadog: successful downgrade logged. → Expected: Log evidence.
16. (Section) Without assign license. → Expected: (Header.)
17. Repeat flow without assign license on STDO downgrade. → Expected: License id empty on downgraded product asset in SF.
18. (Empty placeholder step in Zephyr.) → Expected: (N/A.)
19. Try same SKU purchase on MXP while downgrade pending. → Expected: Error PAYMENTS_1013 / SKU under processing prevents order.

---

### Failure Order Processing till Salesforce [P2] (from DVW-T91)

**Objective:** Entitlement failure, Aria subscription/invoice failure, SF failure, renewal failures, and reprocessing — booked orders, GetOrders statuses, DataDog, and SF payload fields.

**Preconditions:**

- Feature flags and monthly SKUs per Zephyr for forced failures.

**Steps:**

1. Place monthly order (e.g., SKP-GO-MONTH). → Expected: Order placed without UI toaster.
2. GetOrders after entitlement failure flag scenario. → Expected: Line type ENTITLEMENT_PROVISIONING_FAILED; Aria ENTITLEMENT_FAILED; dxOrderStatus BOOKED; entitlement fields populated.
3. DataDog: entitlement creation error logged. → Expected: Error message pattern in logs.
4. DB/saga: order ENTITLEMENT_FAILED. → Expected: Status updated.
5. Salesforce API: payload status booked; account and order booked with products. → Expected: SF booked state as defined.
6. Aria: no plan when entitlement fails before plan. → Expected: No plan created when scenario dictates.
7–22. (Aria subscription failure, Aria invoice failure, SF failure flags; renewal failure matrix; reprocess success/failure date rules; second failure on reprocess updates SF payload.) → Expected: Per Zephyr matrix — GetOrders statuses (SUBSCRIPTION_FAILED, BOOKED, etc.), SF failure reasons (ARIA_COLLECTION_FAILURE, ARIA_INVOICE_READ_FAILURE, DX_FAILURE), start/end date and failure reason rules on reprocess. See DVW-T91 for full indexed steps.

---

### Salesforce - First Order , Subsequent Order and Add Seats [P1] (from DVW-T92)

**Objective:** First order with two SKUs; Salesforce account/contact/currency; subsequent purchase; add seats from MXP/AXP with assign license.

**Preconditions:**

- DVW account creation capability.

**Steps:**

1. Create account; first order with 2 SKUs; assign license checked. → Expected: Order placed.
2. Aria: two plan instances with entitlements and asset IDs. → Expected: Two plans.
3. (Section) Payment capture. → Expected: (Header.)
4. Cybersource/PayPal amount matches Aria invoice. → Expected: Same amount captured.
5. (Section) Salesforce. → Expected: (Header.)
6. Account in SF matches name. → Expected: Account created.
7. Contact point address; Bill To / Ship To. → Expected: Correct addressing.
8. Currency, CDH, Oracle populated. → Expected: Fields populated.
9. Related order: new sale totals and tax. → Expected: Order header correct.
10. Order products: both SKUs, asset IDs, PD with entitlement dates. → Expected: PD matches Aria/EMS.
11. Vertex tax line items per order product. → Expected: Tax lines present.
12. (Section) Subsequent purchase. → Expected: (Header.)
13. Place order with different SKU on same account. → Expected: Order placed; asset created.
14. Repeat SF validations "as above". → Expected: Consistent SF structure.
15. GetOrder: DX new sale vs subsequent order type. → Expected: Subsequent order type in GetOrder.
16. (Section) Add seats. → Expected: (Header.)
17. Add seats from MXP and AXP. → Expected: Order succeeds.
18. DX order type Amendment; same asset id; new entitlement id; updated qty. → Expected: Amendment semantics.
19. Assign license on add seats: order in SF with license. → Expected: License assignment path works.
20. (Separator step in Zephyr.) → Expected: (N/A.)
21. Add seats after upgrade/downgrade still works. → Expected: Flow succeeds.
22. Verify as AO and SAO. → Expected: Actions work for both roles.

---

### Order Processing - Cancel Subscription [P1] (from DVW-T93)

**Objective:** AXP cancel subscription through Aria pending cancellation, Salesforce flags, GetOrders, add seats while pending, renewal batch, and related emails.

**Preconditions:**

- Active DVW subscriptions (e.g., SKP-GO, SKP-PRO, SKP-STDO) per scenario.

**Steps:**

1. (Section) Cancel flow from AXP. → Expected: (Header.)
2. AO/SAO: select active subscription; Manage → Cancel subscription. → Expected: Modal with discontinue-at-end-of-cycle text.
3. Cancel subscription modal content (reason box, warning text, buttons). → Expected: Modal matches spec.
4. Enter reason; confirm cancel. → Expected: Confirmation modal with renewal date and reactivate note.
5. (Section) Aria and Salesforce. → Expected: (Header.)
6. Cancel one SKU (e.g., SKP-GO); verify Aria. → Expected: That plan pending_cancellation; others active; future plan change queued.
7. SF asset Auto_renewal_opted_in false for pending cancellation asset. → Expected: Field false.
8. GetOrders for cancelled subscription user. → Expected: cancel_seats and Aria status subscription_created; dx_order_status activation successful (per case).
9. While pending_cancellation: add seats or upgrade from AXP. → Expected: Still allowed per case.
10. Add_Seats from MXP to pending_cancellation SKU. → Expected: Add seats succeeds; Aria qty and SF updated.
11. Advance Aria VC to anniversary; wait for batch. → Expected: Plan moves to cancelled / non-provisioned; future change completed.
12. Move asset end date to past in SF; check AXP. → Expected: SKU expired in AXP subscription page.
13. Cancel all three SKUs; repeat steps 6–13. → Expected: Same validations for each SKU set.
14. Mixed: purchase three; cancel one; downgrade one; advance VC; batch. → Expected: Cancelled plan non-provisioned; downgraded SKU provisioned; other renews plain; SF/ARIA updated.
15. (Section) Subscription emails. → Expected: (Header.)
16. After cancel: auto-renewal turned off email received. → Expected: Email received.
17. After VC to anniversary and batch: subscription expired email "Heads up: Your subscription has expired". → Expected: Email received.

---

### Order Processing - Renewal at end of term [P2] (from DVW-T96)

**Objective:** Yearly renewal batch for SKP-GO, SKP-PRO, SKP-STDO; EMS old/new; GetOrders renewal line; SF asset/order; mixed cancel/downgrade/add seat/upgrade at renewal.

**Preconditions:**

- Orders placed for three SKUs; provisioned in Aria and SF.

**Steps:**

1. (Precondition header.) → Expected: (Documented.)
2. Aria: note anniversary; advance VC; wait for batch. → Expected: Anniversary year+1; EMS updated; same asset id on plan; new invoice.
3. EMS GET old and new entitlement ids. → Expected: New active; old expired.
4. GetOrders: line type Renewal; Subscription_Created; Activated; order number not in commerce tools. → Expected: Response per renewal rules.
5. SF asset, order, products: renewal order type; new EMS and rev. → Expected: SF matches renewal.
6. SF asset actions: renewals listed; action type renewal-new sales. → Expected: Actions recorded.
7. (Precondition) Mixed changes: add seat/upgrade one SKU; cancel another; downgrade third. → Expected: Setup complete in Aria/SF.
8. Advance VC to anniversary. → Expected: Cancel executes; downgrade executes; plain renew on modified SKU; validate Aria and SF; steps 2–5 hold for plain renew portion.

---

### Salesforce Invoice - Add Seats [P1] (from DVW-T97)

**Objective:** Add seats invoice in SF: Quick Links invoice, external system info, revenue management, Vertex mapping, currency/VAT/sales channel.

**Preconditions:**

- Successful add seats from MXP/AXP.

**Steps:**

1. (Section) Add seats invoice. → Expected: (Header.)
2. Place add seats from MXP/AXP. → Expected: Order succeeds.
3. DX Amendment; same asset id; new entitlement id; qty update. → Expected: Amendment semantics verified.
4. Assign license during add seats: order in SF. → Expected: Order visible with assignment.
5. SF: place account id; retrieve user. → Expected: Account located.
6. Quick Links: invoice exists with number, dates, totals, status, payment status, payments, credits, balance. → Expected: Invoice header complete.
7. External system info: external invoice date; push mule completed. → Expected: Fields updated.
8. Full invoice with invoice line and vertex line; mapped to DX fields. → Expected: Lines and mapping correct.
9. Invoice history push mule In Progress for revenue management link on line. → Expected: RM link generation path.
10. Currency, VAT, qty, sales channel on invoices. → Expected: All accurate across invoices.

---

### Get Order API (Upgrade, Downgrade , Renewal, cancellation , Add Seats, New Purchase  And  Sub sequent) [P2] (from DVW-T108)

**Objective:** Comprehensive GetOrders validation for new purchase, subsequent, add seats, upgrade paths, Aria alignment, license APIs, renewal snippet, and failure modes (SF down, OPF down, entitlement vs subscription failures).

**Preconditions:**

- Account with ability to place SKP-PRO and other SKUs per scenario.

**Steps:**

1. (Section) New purchase example SKP-PRO. → Expected: (Header.)
2. Purchase new SKU not owned before; complete order. → Expected: Order succeeds.
3. GetOrders: NEW_PURCHASE lines; SUBSCRIPTION_CREATED; assignLicenseToMe per product; currentEntitlementId null; newEntitlementId set; sku, qty, subscription type, dates. → Expected: Matches spec (see Zephyr sample JSON).
4. Order SUBSCRIPTION_CREATED; dxOrderStatus ACTIVATED; Aria account and payment method. → Expected: Header fields correct.
5. Aria: overview, groups, payment, plans updated; entitlement id matches GetOrders. → Expected: Consistency across Aria and API.
6. CT: ctOrderLineItemId correct. → Expected: CT linkage displayed.
7. GetUserLicenses / EMS GetLicenseIDByEntitlementId for SKP-PRO. → Expected: License id matches EMS for assign=true case; SKP-PRO example may be null per case note.
8. (Section) Subsequent: SKP-GO (assign false) and SKP-STDO (assign true). → Expected: (Header.)
9. Place subsequent order with both. → Expected: Order succeeds.
10. assignLicenseToMe and licenseId rules per product; SKP-PRO rule in Zephyr. → Expected: Fields per matrix.
11. GetUserLicenses for SKP-STDO. → Expected: Matches EMS.
12. Order header SUBSCRIPTION_CREATED; ACTIVATED; Aria fields. → Expected: As new purchase header checks.
13. Aria plans updated. → Expected: Mirrors API.
14. Plan instance entitlement id matches GetOrders. → Expected: Same id.
15. CT order SUBSCRIPTION_CREATED. → Expected: CT status.
16. (Section) Add seats previously purchased SKU e.g. SKP-STDO. → Expected: (Header.)
17. Place add seats order. → Expected: Order succeeds.
18. GetOrders: ADD_SEATS; SUBSCRIPTION_CREATED; currentEntitlementId old; newEntitlementId new; other fields. → Expected: Add seats shape.
19. Order header SUBSCRIPTION_CREATED; ACTIVATED. → Expected: Header ok.
20. Aria updated. → Expected: Plans reflect add seats.
21. Plan instance updated entitlement id; units updated. → Expected: Matches GetOrders current/new ids.
22. GetUserLicenses: STDO license same as before add until revocation rules apply. → Expected: Per step description.
23. EMS GET with currentEntitlementId. → Expected: Status Expired; child id = newEntitlementId.
24. EMS GET with newEntitlementId. → Expected: Active; parent = currentEntitlementId.
25. Uniqueness / SKUs under processing (not testable per Zephyr). → Expected: Documented limitation.
26. CT SUBSCRIPTION_CREATED. → Expected: CT shows order.
27. dxOrderStatus Activated. → Expected: Field Activated.
28. AXP purchases show licenses. → Expected: Licenses listed.
29. (Section) Error cases. → Expected: (Header.)
30. SF down: dxOrderStatus Not_Created when SF unavailable; else success path. → Expected: Per environment simulation.
31. Order Processing Function down: no entitlements/subscriptions; auth reversal; no capture. → Expected: No card/PayPal capture.
32. GetOrders: ENTITLEMENT_CREATION_FAILED; Booking_Failed; line types NEW_PURCHASE or ADD_SEATS as specified. → Expected: Failure shape.
33. Entitlement ok but subscription creation fails. → Expected: Line SUBSCRIPTION_CREATION_FAILED; order Subscription_Failed; dxOrderStatus Booked; entitlement id present.
34. (Section) Plain renewal snippet. → Expected: (Header.)
35. Aria plan instance navigation. → Expected: (Context.)
36. VC to anniversary; pending invoice. → Expected: Pending invoice triggered.
37. GetOrder: Renewal line; ctOrderLineItemId null; Source Aria. → Expected: Renewal type in response.
38. (Additional Aria/CT steps in Zephyr.) → Expected: Complete renewal validation per DVW-T108.

---

### View Order Status API [P2] (from DVW-T111)

**Objective:** Get order status API returns GetOrders fields plus email, currency, orderStatus; error SUPPORT_SERVICE_1030 for non-UUID orders.

**Preconditions:**

- Support URL access token; valid order id.

**Steps:**

1. Call get order status with Account ID header, order id in path, support token. → Expected: Email, currency, order status returned with GetOrders-equivalent fields.
2. Order id without UUID. → Expected: SUPPORT_SERVICE_1030 error code.
3. Other cases per linked Google Sheet in Zephyr. → Expected: Per sheet.

---

### Salesforce Invoice - renewals [P1] (from DVW-T117)

**Objective:** After renewal batch: GetOrders dxInvoiceSent; SF invoice orderType Renewal; payment status; external info; invoice lines; Vertex; revenue arrangements.

**Preconditions:**

- Three SKUs ordered; one downgraded; one cancelled; VC advanced for renewal.

**Steps:**

1. (Pre-requisite section.) → Expected: (Documented.)
2. After renewal batch: GetOrders renewal line dxInvoiceSent Invoice_Paid_Sent. → Expected: Field populated.
3. SF account: latest renewal invoice. → Expected: Invoice present.
4. Invoice orderType Renewal. → Expected: Field Renewal.
5. Subtotal, tax, total match Aria invoice. → Expected: Amounts match.
6. SF inspector TNV_Aria_Payment_Status__c Paid. → Expected: Paid.
7. External system information: no errors. → Expected: Clean external info.
8. invoiceGeneratedDate populated and formatted. → Expected: Correct date on invoice.
9. Data export query (invoice doc in Zephyr). → Expected: Query fields on invoice.
10. Data export query (invoice lines doc). → Expected: Line fields present.
11. Vertex tax lines on invoice lines. → Expected: Tax lines generated and linked.
12. External invoice status and push mule complete. → Expected: Status complete.
13. Revenue arrangements on lines after external complete. → Expected: Arrangements generated.
14. RevPro/RevHub success on revenue arrangements. → Expected: No errors; success status.
15. Two invoice lines: plain renewal SKU and downgraded SKU; no cancelled SKU line. → Expected: Line composition per scenario.

---

### Revenue sync on hold for renewal orders [P1] (from DVW-T145)

**Objective:** Salesforce, GetOrders, support portal, and negative invoice behaviors when renewal orders hit Dunning, D-country hold, Compliance hold, release sequences, returns, tax lines, and partial return rules.

**Preconditions:**

- Renewal orders with holds and dunning per scenario matrix in Zephyr.

**Steps:**

*(Ordered by Zephyr `index` 0–28. Expected results are the `<ol>` items in DVW-T145 — not repeated in full here.)*

1. Index 0 — Holds: compliance-only vs D-country-only vs both; SF Activated vs Booked. → Expected: See DVW-T145 expected list for index 0.
2. Index 1 — Compliance released; D-country still on hold. → Expected: SF Booked; GetOrders EEUC Hold / Compliance Released; invoice Booked; AXP processing.
3. Index 2 — D-country released; compliance still on hold. → Expected: SF Booked; GetOrders EEUC Released / Compliance Hold; invoice Booked; AXP processing.
4. Index 3 — Compliance then D-country released in sequence. → Expected: SF Activated Paid; GetOrders Released; DX Activated; AXP complete.
5. Index 4 — Return before hold release (customer request). → Expected: SF Activated; negative invoice; GetOrders EEUC/Compliance Failed; AXP canceled.
6. Index 5 — Support return; compliance failed; D release pending. → Expected: Booked→Activated on support cancel; negative invoice; AXP canceled.
7. Index 6 — Support return; D-country failed; compliance release pending. → Expected: Mirror of index 5 for opposite failure side.
8. Index 7 — Renewal on dunning and hold (compliance + D-country). → Expected: SF Booked; GetOrders EEUC/Compliance Hold; invoice Booked; AXP processing.
9. Index 8 — Dunning revoked; still on compliance and D-country hold. → Expected: SF Activated; negative Dunning invoice; AXP canceled.
10. Index 9 — Dunning revoked; compliance on hold; D-country released. → Expected: SF Activated; negative Dunning invoice; AXP canceled.
11. Index 10 — Dunning revoked; D-country on hold; compliance released. → Expected: SF Activated; negative Dunning invoice; AXP canceled.
12. Index 11 — In dunning; both compliance and D-country released. → Expected: SF Activated Unpaid; negative Dunning invoice; AXP canceled.
13. Index 12 — In dunning stage with both compliance and D-country released (invoice/payment state per Zephyr). → Expected: See DVW-T145 ol for index 12.
14. Index 13 — D-country or compliance fails during dunning stages. → Expected: SF Activated Paid; negative Dunning invoice; AXP canceled; dunning stages advance/cancel per ol.
15. Index 14 — Customer updates PM to valid after dunning; then D-country or compliance fails. → Expected: SF Activated Paid; negative invoice; AXP canceled (per ol).
16. Index 15 — Customer updates PM to valid after dunning; D-country and compliance holds released subsequently. → Expected: SF Activated Paid; AXP complete (per ol).
17. Index 16 — Compliance and D-country releases happen; then customer updates PM to valid. → Expected: SF Activated Paid; AXP complete (per ol).
18. Index 17 — (Section) Multiple line item return. → Expected: Section header only.
19. Index 18 — Return multiple line items; single negative invoice; Revpro success. → Expected: Clubbed negative invoice; no processing errors.
20. Index 19 — (Section) Support portal changes. → Expected: Section header only.
21. Index 20 — Support portal: renewal on hold. → Expected: Processing; EEUC On-Hold if D hold; Compliance On-Hold if compliance hold.
22. Index 21 — Support portal: renewal on dunning. → Expected: Order status Dunning.
23. Index 22 — Support portal: renewal on dunning + holds. → Expected: Dunning with EEUC/Compliance hold labels per matrix.
24. Index 23 — Return button disabled while order processing. → Expected: Direct return of hold orders blocked; cancel first to move to processed DB.
25. Index 24 — Cancel order in portal; then return succeeds. → Expected: Return succeeds after cancel.
26. Index 25 — Portal displays order / D-country / compliance status by backend state. → Expected: Labels match Processing/Dunning/On-Hold/Failed combinations.
27. Index 26 — (Section) Tax line items in SF order API. → Expected: Section header only.
28. Index 27 — Tax lines on SF order API for renewal hold (tax countries, add-seat hold, booked vs activation). → Expected: Tax line rules per bullet list in Zephyr.
29. Index 28 — Partial return on canceled renewal-hold order (D-country/compliance cancel path). → Expected: Partial returns not allowed.

---

### EEUC Process for D Group Orders [P1] (from DVW-T157)

**Objective:** EEUC form for D-country checkout; Post Status ssucFormRequired; terms calls; acceptance; GetOrder and account container; renewals; holds; notifications; localization; D-country list cache.

**Preconditions:**

- D-country list and test SKU (e.g., SketchUp Go) per Zephyr.

**Steps:**

1. PDP → checkout with D-country on account setup → order review → EEUC form. → Expected: Form UI (12 conditions, Yes/No, signature block); Post Status ssucFormRequired true; Get Terms includes text; after POST terms acceptance date and endUserEntity true.
2. Expire EEUC via terms before Place Order; Place Order. → Expected: Error "Order can't be initiated for D-Country account"; redirect to EEUC.
3. First order after EEUC acceptance. → Expected: SF Signed EEUC Yes; Valid Up To acceptance + 1 year − 1 day.
4. Subsequent purchase with active EEUC. → Expected: No EEUC for one year; account container dates valid.
5. Expired EEUC DB backdate; subsequent purchase; accept form. → Expected: EEUC shown; new expiry in SF after accept.
6. Renewals after D-country order. → Expected: Same validations as step 1 plus GetOrder/account/Aria/Cybersource/PayPal consistency.
7. EEUC URL tampering on form page. → Expected: Same post-acceptance validations as step 1 subset.
8. Compliance hold: Azerbaijan order; release via hold API. → Expected: Order holds until release; then processes without D-country hold; GetOrders status check.
9. Multiple SKU order with D-country address. → Expected: Same validation bundle as step 1.
10. Localized EEUC; block terms URLs to induce errors per Zephyr. → Expected: Translated strings and locale; error handling per blocked endpoints.
11. Hold release flow: renewals, expiration move, renewal on hold, support release, subsequent purchase with EEUC. → Expected: BOOKED and no SF invoice on HOLD; notifications; SF expiration update; after release invoice generates; subsequent works after EEUC accept.
12. Post-renewal notifications with EEUC link (days 3, 9, 12). → Expected: Links open AXP EEUC; SF date checks.
13. Pre-expiry notifications (90/60/30/15) with EEUC link. → Expected: SF dates updated after accept.
14. D-country list from SF via API; cache clear; re-fetch. → Expected: List returned and cache behavior verified (Datadog optional).
15. Epic scenarios reference link. → Expected: (Pointer to spreadsheet.)

---

### EEUC Renewals [P1] (from DVW-T167)

**Objective:** Auto-renewal hold when D-country and EEUC expired; emails days 0/3/9/12; 14-day grace auto-return; support manual release.

**Preconditions:**

- Migrated or direct account; D-country in account or billing; EEUCExpirationDate past at renewal.

**Steps:**

1. Renewal with expired EEUC (D-country only). → Expected: Renewal hold email + account statement; SF Booked; EUC Accept in hold UI → TID login → EEUC form; accept releases holds; SF Booked→Activated.
2. D-country + Compliance + expired EEUC. → Expected: Renewal hold + compliance + account statement emails; SF Booked; EUC Accept; compliance still blocks until released; orders container updates when compliance releases.
3. Customer accepts via Day 3 email link. → Expected: Day 3 email content; accept stops days 9/12; SF Activated; orders not on hold for one year across order types.
4. Customer accepts via Day 9 link (skipped 3). → Expected: Day 9 email rules; SF Booked until accept; Day 12 not sent after accept.
5. Customer accepts via Day 12 link. → Expected: Day 12 only if prior reminders ignored; SF Activated; no further reminders.
6. No acceptance within 14 calendar days from invoice date (Aria). → Expected: Renewal canceled and returned; entitlements revoked; Return/Access Termination email; SF return job creates return order; AXP canceled; further refund/return error; new orders allowed with EEUC acceptance.
7. Support manual release for expired EEUC hold orders. → Expected: Dates update in accounts container when released with valid dates; retain dates when released without new EEUC expiry.

---

## Key Validations

- GetOrders line types and statuses match Aria plan state and DX order status for purchase, upgrade, downgrade, renewal, cancel seats, and failure flags.
- Salesforce orders, assets, invoices, and negative lines reflect the commercial outcome (including holds, dunning, and returns).
- EEUC and D-country gates align across commerce, notifications, SF, and GetOrders.

## Common Preconditions

- Aria virtual clock access for renewal/downgrade batch testing; SF inspector or data export queries where referenced.
- Feature flags for forced failure paths in DVW-T91 and similar cases.
