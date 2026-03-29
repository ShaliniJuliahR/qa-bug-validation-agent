# Feature: Compliance & Tax Exemption

## Overview

Compliance hold processes, tax exemption certificate holds, D-country release, renewal holds, price change mid-term, and manage tax exempt flows.

## Test Flows

### Compliance test cases & compliance hold notification email for all order types [P1] (from DVW-T102)

**Objective:** For compliance, account name, account address, CC bill-to / PayPal address, and related data are sent to Precision; holds, releases, ECCN, and renewal/compliance behaviour are validated.

**Preconditions:**

- Test accounts and payment methods per scenario (embargo, failing names, secondary roles, renewal setups).
- Access to Precision, CT merchant center, getOrder API, Datadog as applicable.

**Steps:**

1. Verify every order is precision checked and passed to the order processing queue → Expected: Only on Precision success, order reaches processing queue.
2. Validate renewal order is created then compliance check runs → Expected: Post-order compliance check completes.
3. Verify account name, address, card/PayPal address, first/last name + legal entity go to Precision → Expected: Data passed for verification.
4. If order goes on HOLD, verify manual release from Precision before processing → Expected: Only after manual release, order moves to Processing.
5. After release, verify subsequent purchases may go on HOLD → Expected: Orders after release can go on HOLD (note: not always required).
6. D-country hold release: verify addresses precision-checked then processed → Expected: Released D-country addresses are precision checked.
7. Verify HOLD visible in CT and getOrder at order level → Expected: Status HOLD in CT and APIs.
8. When order on hold in CT, verify cart not active → Expected: Cart not in active state in CT APIs.
9. UI on HOLD matches Figma scenario A → Expected: Correct hold messaging per design.
10. If compliance fails for non-PASS/FAIL reasons (e.g. address length), verify error → Expected: errorCode PAYMENTS_2026, errorDescription "Compliance operation could not be performed".
11. Order on HOLD during renewal: verify no duplicate renewal order; processed orders renew → Expected: HOLD orders unchanged; processed orders renew.
12. Log in as user with failing TID name (e.g. "Osama Bin Laden"); place order → Expected: Compliance HOLD + email to user.
13. Log in with credit card address in embargoed country; place order → Expected: Compliance HOLD + email.
14. Normal user, secondary role to account with failing name; place order → Expected: No HOLD if logged-in TID name has no errors (per scenario).
15. Secondary owner card in embargo; normal user checkout → Expected: HOLD due to secondary card address + email to logged-in user.
16. Failing TID + secondary with normal details → Expected: HOLD on TID + email.
17. Failing TID + secondary embargo card → Expected: HOLD on card + email.
18. Failing TID + secondary failing name → Expected: HOLD on TID + email.
19. Verify SU SKU ECCN numbers/groups per matrix → Expected: ECCN and ECCNGroup correct per shipping country rules.
20. Renewal with compliance-failing payment name: verify email lists failed SKUs → Expected: Email lists SKUs that failed renewal (single/multi line).
21. Verify GET orders API compliance status after renewal → Expected: Line type Renewal; Aria Subscription_Created; dxOrderActivated; complianceHoldStatus HOLD when applicable.
22. D-country + compliance holds: verify customer email for single/multi line → Expected: Email sent for both hold types.
23. After renewal on HOLD, verify user cannot add seats or upgrade → Expected: Blocked; existing line items still accessible.

---

### D-country release [P1] (from DVW-T115)

**Objective:** D-country hold release via support API and support portal (QA), EEUC, renewals, and error handling.

**Preconditions:**

- D-country and embargo lists (reference spreadsheet in Zephyr).
- QA support portal: https://qa.shop-support.trimble.com
- d-country-release API URL with function key (see Zephyr objective).

**Steps:**

1. Reference D-country/embargo lists → Expected: Use current list for test data.
2. Support UI: place D-country hold order; search by Order ID and Account ID → Expected: Order/account/price/tax; expiration selector null by default; release without date.
3. After release, verify getOrders and account container; expiration null → Expected: Released status; expiration null in accounts container.
4. Release with past expiration → Expected: Error; past date not accepted.
5. Release with future expiration → Expected: Order released with expiration in account container.
6. Subsequent order/renewal within expiration → Expected: No D-country hold.
7. Subsequent purchase after EEUC acceptance → Expected: EEUC form not shown for 1 year; validate account container dates.
8. Renewal within EEUC period vs after expiry → Expected: Renewal succeeds; D-country hold email to support + user per template when applicable.
9. D-country + embargo order (e.g. Ukraine); call d-country-release API with order id + account header → Expected: 200; order id and status Hold in response; order failed compliance; not in processing queue; Datadog logs for release and compliance.
10. Renewals after D-country: verify getOrder, account, Aria, SF, payment systems → Expected: Consistent with form acceptance.
11. EEUC URL tampering (wrong account in URL) → Expected: Validation per security rules.
12. Order after expiration date (DB-adjusted dates) → Expected: EEUC shown again.
13. Order not on D-country hold; call release API → Expected: 422 SUPPORT_SERVICE_1002 "Status is not D-Country Hold".

---

### Renewal – D-country & compliance hold – account on hold [P1] (from DVW-T140)

**Objective:** Subsequent non-renewal orders respect account hold state after compliance or D-country outcomes.

**Preconditions:**

- Accounts in each hold/release/cancel state as labeled in Zephyr.

**Steps:**

1. Non-renewal – compliance hold; place subsequent order → Expected: Error on place order; order not placed.
2. Non-renewal – compliance released; place subsequent order → Expected: Hold cleared; further orders allowed.
3. Non-renewal – compliance cancelled; place subsequent order → Expected: Hold cleared; further orders allowed.
4. Non-renewal – D-country hold; place subsequent order → Expected: Error; order not placed.
5. Non-renewal – D-country released; place subsequent order → Expected: Hold cleared; further orders allowed.
6. Non-renewal – D-country cancelled; place subsequent order → Expected: Hold cleared; further orders allowed.
7. Non-renewal – no holds; place subsequent order → Expected: Order places successfully.

---

### Tax exemption certificate hold – order hold, Salesforce case, notifications, release [P1] (from DVW-T151)

**Objective:** US tax exemption path: tax hold, Salesforce case, reminders, certificate submission or 168h auto-release, email localization rules.

**Preconditions:**

- US account with business + tax certificate option.
- Customer language scenarios per Zephyr (supported vs unsupported for reminders).

**Steps:**

1. On account creation, opt US business + tax certificate → Expected: UI shows tax exemption certificate messaging.
2. Profile shows "State Tax Exemption Certificate: Awaiting Submission" → Expected: Correct profile status.
3. Place order with US tax exemption → Expected: Order succeeds; goes to tax exempt hold for validation.
4. Verify Salesforce case in GHS queue and logs → Expected: Case created; log evidence.
5. While on tax hold, user cannot place another order on same account → Expected: Blocked until release.
6. Initial emails: Commerce + Salesforce on-hold tax certificate → Expected: Both received with correct subjects/body.
7. Reminders at 24h, 72h, 96h if no certificate → Expected: Reminder emails per schedule.
8. Final reminder at 144h → Expected: Final reminder email sent.
9. After certificate submitted and order released → Expected: No further 24/72/96/144h reminders.
10. On-hold + reminder email translation → Expected: Unsupported langs (zh-TW, zh-CN, it-IT, ru-RU) not translated; others translated per locale.
11. Reminder/final reminder translation → Expected: Same unsupported vs supported rule as step 10.
12. Submit valid certificate within 7 days → Expected: Vertex/support validates; order released with 0 tax.
13. After approval → Expected: Account statement + order confirmation emails.
14. If no certificate by 168h → Expected: Auto-release with tax charged.
15. Verify auto case closure after auto-release → Expected: SF case closed.
16. GET /orders tax hold status with account + JWT → Expected: HOLD / RELEASED / FAILED per outcome.

---

### Manage tax exempt [P2] (from DVW-T154)

**Objective:** Support portal actions for tax-hold orders: release exempt, proceed with tax, cancel; popups and downstream systems.

**Preconditions:**

- Order on tax hold (DVW-T154).
- QA support portal: https://qa.shop-support.trimble.com

**Steps:**

1. Open Releasing Tax Exempted Holds; query account on tax hold → Expected: Account layout per spec; release vs proceed options; Tax Status shown; none selected by default.
2. Release as tax exempt → Expected: ON-Hold → Processed; Aria tax exemption flags State/Province + Federal; SF processed; AXP Completed; history/APIs clean; compliance + D-country after release; further orders without hold (test env tax may still calculate).
3. Proceed with tax (not exempt) → Expected: ON-Hold → Processed; Aria No TAX Exception; SF with tax; similar downstream checks; further orders with tax.
4. Cancel order → Expected: ON-Hold → Cancelled; Aria No TAX Exception; not processed to SF; AXP Cancelled; no compliance/D-country for cancelled; further orders with tax.
5. "Why am I verifying tax status" popup → Expected: Clear outcome messaging; proceed after confirm.
6. Popup layout → Expected: Messages do not overlap.
7. While on tax hold → Expected: No further orders until release/proceed/cancel completes.
8. Tax hold email → Expected: Customer notified per template.

---

### Price change mid-term [P1] (from DVW-T158)

**Objective:** Renewals, refunds, add seats, upgrades, downgrades, and promos after catalog price changes use correct old vs new rates.

**Preconditions:**

- Example prices in Zephyr (e.g. old $100 → new $120).
- Ability to trigger renewal, return, add seats, upgrade, downgrade, dunning revoke.

**Steps:**

1. Scenario A header: renewal after price update → Expected: (see following steps).
2. Buy at old price; update price; trigger renewal → Expected: Renewal at new unit price.
3. Return renewed product → Expected: Refund uses new price schedule.
4. PRO old / GO new; downgrade; trigger → Expected: Downgraded GO billed at new GO price.
5. Return downgraded SKU → Expected: Refund uses new GO price.
6. Scenario B: add seats after price update → Expected: (see steps).
7. Buy old price; update price → Expected: Price update applied.
8. Same-day add seats (+2) → Expected: No proration UI; new seats at new price.
9. Partial return FO → Expected: Refund at old rate.
10. Partial return add-seats line → Expected: Refund at new rate.
11. After delay, add more seats → Expected: Proration at new price.
12. Scenario C: upgrade after price update → Expected: (see steps).
13. GO old/new; same-day upgrade GO→PRO → Expected: Negative line for GO at old price.
14. GO with 20% promo; same-day upgrade → Expected: Negative GO at old promo price.
15. GO promo old; update GO & PRO; later upgrade → Expected: PRO at new price with promo; negative GO at old promo price.
16. Return upgraded PRO → Expected: Refund at new PRO rate with promo logic.
17. Partial return GO (FO) → Expected: Refund at old GO with promo.
18. New account: GO old; update; upgrade with promo; dunning revoke → Expected: Negative and invoice lines match old promo / new promo rules.
19. Verify Aria invoice, Salesforce order, emails for all scenarios → Expected: Rates and promos consistent everywhere.

## Key Validations

- Precision/compliance and D-country holds, releases, and API-visible status (getOrder, accounts container, CT).
- Tax exempt hold: Salesforce case lifecycle, reminder cadence, 168h auto-release, `/orders` tax hold status.
- Price-change scenarios: explicit old vs new rate rules for renewal, refund, add seats, upgrade, downgrade.

## Common Preconditions

- QA environments, test payment methods, support/API endpoints and keys as documented in Zephyr per case.
- Access to Aria, Salesforce, AXP, email inboxes, and Datadog where steps require downstream verification.
