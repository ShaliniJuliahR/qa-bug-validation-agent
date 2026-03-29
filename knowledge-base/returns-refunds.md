# Feature: Returns, Refunds & Chargebacks

## Overview

Return process validation including negative invoice generation in Salesforce, chargeback returns, cancel order API, partial returns, upgrade returns, and blocking returns of migrated orders.

## Test Flows

### Validation of Return Process and Negative Invoice Generation in Salesforce [P1] (from DVW-T121)

**Objective:** Support portal returns; latest-order-only rule; staged order errors; multi-line returns; Aria/SF/PayPal/Cybersource outcomes; upgrade return pricing; 120-day rule.

**Preconditions:**

- DVW account; support portal QA URL; SF and payment sandbox access.

**Steps:**

1. Create DVW account; place order. → Expected: Order succeeds.
2. Support initiates return in portal. → Expected: API success; return started.
3. Returns only for latest order on plan instance. → Expected: Older orders rejected.
4. Attempt return on staged downgrade/cancel order. → Expected: API error for staged orders.
5. /returns with multiple line items in one request. → Expected: Multi-line return processed.
6. Upgrade return (e.g., GO→PRO): revert to GO; refund upgrade delta only. → Expected: SKU reversion + partial refund amount.
7. Return renewed order. → Expected: Return succeeds for renewal case.
8. Return >120 days after placement. → Expected: API error time window.
9. CC paid order: Cybersource refund success status and amount. → Expected: Credit success in Cybersource.
10. PayPal paid order: sandbox shows Refunded for full amount. → Expected: PayPal refund matches payment.
11. SF: negative order line with correct SKU/qty/amount. → Expected: Negative line created.
12. SF: negative invoice matching original payment. → Expected: Negative invoice amount matches paid amount.
13. SF asset for SKU Cancelled (non-renewal). → Expected: Asset cancelled.
14. Renewed order return: asset status Expired in SF. → Expected: Expired not cancelled for renewal path.
15. Remove returned lines from AXP (not for renewed path per note). → Expected: AXP cleaned where applicable.
16. Refund credits original payment instrument even if user later changed default PM. → Expected: Refund to original card despite PM change.

---

### Validation of Chargeback Return Process /returns API [P1] (from DVW-T125)

**Objective:** returnType CHARGEBACK; skipIssueRefund; no PayPal/Cybersource refund rows; SF negatives; latest-order rule; conflict with standard return after chargeback.

**Preconditions:**

- Precondition Aria steps: unapply payment, void invoice per Zephyr before chargeback scenario.

**Steps:**

1. (Precondition) Aria unapply payment + void invoice sequence for test order. → Expected: Precondition satisfied.
2. /returns accepts returnType CHARGEBACK (and default RETURN). → Expected: API accepts parameter.
3. CHARGEBACK with skipIssueRefund=true: Aria skips issuing refund. → Expected: No Aria-issued refund for that call.
4. Upgrade return: revert to GO; refund upgrade portion only. → Expected: Same as standard upgrade return math.
5. CC: no Cybersource refund transaction for chargeback return. → Expected: No refund txn in CS portal.
6. PayPal: no refunded txn for chargeback return. → Expected: Sandbox shows no refund for chargeback path.
7. SF negative order line. → Expected: Negative line created.
8. SF negative invoice equals original payment. → Expected: Invoice amount matches original pay.
9. SF asset Cancelled. → Expected: Asset cancelled for full return path.
10. AXP lines removed for returned items. → Expected: AXP updated.
11. Standard RETURN after CHARGEBACK on same line items. → Expected: Error SUPPORT_SERVICE_1044 already returned.
12. Chargeback only on latest order for plan instance. → Expected: Else error.
13. Staged order return attempt. → Expected: Error as in T121.
14. Renewed order return. → Expected: Allowed; success path.
15. Renewed order asset Expired in SF after return. → Expected: Expired status.
16. Multi-line /returns body. → Expected: Processed in one request.
17. >120 days return blocked. → Expected: API error.

---

### Validation of /Cancel Order API for Order Status [P2] (from DVW-T128)

**Objective:** POST cancel-order only for HOLD orders; errors for D-country hold, wrong state, bad auth, bad ids.

**Preconditions:**

- Support API access; orders in HOLD, D-country hold, cancelled, etc.

**Steps:**

1. POST cancel for HOLD order (/orders/{id}/cancel-order). → Expected: Order cancelled; status updated.
2. Verify status in Commerce Tool → Cancelled. → Expected: CT reflects cancel.
3. POST cancel for D-country Hold order. → Expected: Error cannot cancel due to D-country hold.
4. POST cancel for non-HOLD order. → Expected: Error not in HOLD state.
5. POST cancel already cancelled. → Expected: Error already canceled.
6. POST without auth. → Expected: 401 Unauthorized.
7. Missing/bad parameters. → Expected: 400 Bad Request.
8. Malformed order id format. → Expected: 400 Bad Request.
9. Invalid/non-existent order id. → Expected: Error invalid order id.

---

### Partial returns test cases [P1] (from DVW-T146)

**Objective:** Single- and multi-line partial returns via support portal; Aria entitlement splits; Datadog returnIdentifier PR on SF negative invoice API; multi tax-line addresses.

**Preconditions:**

- PDP URLs for Connect Pro/Innovate (SIT netlify) per Zephyr; support portal SIT.

**Steps:**

1. Single line: buy 5 qty Connect Pro annual; place order. → Expected: Order success.
2. Support portal: search Account ID + Order Number; validate details. → Expected: Order found.
3. Return 3 qty; initiate return. → Expected: Success message.
4. Validate return (Aria new EID for Pro; qty update; SF negative invoice; notification amounts; curl reversible inv amount). → Expected: Full validation bundle per Zephyr step 3.
5. Multi line partial: 10 Pro + 5 Innovate; return 5 Pro + 3 Innovate. → Expected: Success; Aria/SF per step 6 validation.
6. Validate Aria after step 5 (new EID Pro; Innovate expired/canceled as specified). → Expected: Per step 9 narrative.
7. Multi line full return: 10 Pro + 5 Innovate; return 5 Pro + 5 Innovate. → Expected: Success.
8. Multi line single partial line: return 5 Pro only from combined order. → Expected: Success.
9. Validate Aria (step 6 pattern). → Expected: Entitlements per step 6.
10. Multi line additional partial: repeat order; return 5 Pro + 3 Innovate again. → Expected: Success.
11. Return 5 Pro + 2 Innovate next. → Expected: Success.
12. Validate Aria — both plans cancelled after final returns. → Expected: Per step 16.
13–22. Multi tax line verification orders with US addresses (tax line 1–3); return 3 qty each; Datadog PR + returnedQuantity/returnPrice/taxLine checks. → Expected: Per DVW-T146 indices 17–22.

---

### Upgrade Return [P1] (from DVW-T149)

**Objective:** Return upgrade within 14 days; promo; multi-tax US/Canada; partial return blocked on upgrade; chained returns; after 14 days blocked.

**Preconditions:**

- SKP-GO / SKP-PRO / SKP-STUDIO paths; support portal.

**Steps:**

1. GO → PRO upgrade; return PRO. → Expected: Mail pricing correct; downgrade in Aria; AXP active downgraded; SF negatives; CS/PayPal refund verified — all LE.
2. GO → STUDIO with promo; return STUDIO. → Expected: Refund matches discounted paid amount; same system checks.
3. GO monthly → PRO; after renewal return PRO. → Expected: Plan expired Aria/AXP; SF negatives; refund verified.
4. US addresses tax line scenarios for upgrade returns. → Expected: Same validation bundle across tax lines.
5. Canada addresses tax line scenarios. → Expected: Same validation bundle.
6. GO annual → PRO → STUDIO; return STUDIO then PRO then GO; try return GO again. → Expected: Error return already processed for line item.
7. PRO qty 5 → STUDIO; try partial return 3 STUDIO. → Expected: Portal error partial return not allowed for upgrade order type.
8. PRO qty 5 → STUDIO; return STUDIO then partial returns 1,1,3 on PRO. → Expected: Downgrade to PRO then partial sequence expires plan; SF/CS/PayPal ok.
9. Support portal shows order type Upgrade for upgrade orders. → Expected: Type displayed.
10. Upgrade older than 14 days: return blocked. → Expected: Cannot return via portal.

---

### DM-Charge Back [P1] (from DVW-T162)

**Objective:** Chargeback via support/runbook on migrated EP accounts — New Sale, Add Seats, Upgrade (with manual SF payload for some paths); promos; custom rate; price change scheduled.

**Preconditions:**

- Migrated EP account runbook IDs; Remove Seat / Downgrade Seat / Cancel Seat options.

**Steps:**

1. New Sale chargeback (Cancel Seat). → Expected: Entitlement inactive (EMS); licenses revoked (AXP); Aria plan Cancelled; service credits prorated/cancelled; no further Aria invoices for CB; SF negative order = new sale amount; negative order product; assets Cancelled.
2. Add Seats chargeback (Remove Seat). → Expected: Add-seat entitlement inactive; new entitlement still ties to active new sale (EMS); only add-seat licenses revoked; Aria plan stays ACTIVE; SF negative order = add-seat amount; order product negative qty; assets qty back to new sale qty; no extra Aria invoices.
3. Upgrade chargeback (Downgrade Seat + downgrade SKU; manual SF when skip). → Expected: Upgrade entitlement revoked; entitlement points to downgrade SKU; upgrade licenses revoked; downgraded licenses active; Aria plan shows downgraded SKU; service credits for upgrade cancelled; SF negative order = upgrade amount; upgrade product negative; upgraded asset cancelled; asset restored to downgrade SKU.
4. New Sale + Promo chargeback. → Expected: Same as (1) plus promo retained on chargeback order; negative price equals discounted paid amount.
5. Add Seats where New Sale had Promo + custom rate from runbook. → Expected: Same as (2) plus promo on chargeback line; negative matches discounted paid; custom rate applied in Aria; follow-on transactions respect custom rate.
6. New Sale with Price Change scheduled chargeback. → Expected: Like (1); negative price equals customer paid not future scheduled price.
7. Add Seats with Price Change scheduled chargeback. → Expected: Like (2); negative equals paid amount not new scheduled price; custom rate flags in Aria per code path.
8. Upgrade with Price Change scheduled chargeback. → Expected: Like (3); custom rate calculated post chargeback in Aria per Zephyr.

---

### Block Return of Migrated Orders [P2] (from DVW-T163)

**Objective:** Support portal and API block returns for migrated orders with SUPPORT_SERVICE_1084.

**Preconditions:**

- Example migrated Account ID / Order ID from Zephyr.

**Steps:**

1. (Intro) Applies to migrated data and migrated-placed orders; D and non-D countries. → Expected: Scope statement.
2. (Section) Frontend support portal. → Expected: Header.
3. Support portal → Return tab. → Expected: Return page with order search.
4. Open migrated order details. → Expected: Banner "This order was moved from a legacy system..."; no actions on return/refund/cancel/hold/release/details.
5. (Section) Backend return API. → Expected: Header.
6. POST return API for migrated order with support token. → Expected: 400 SUPPORT_SERVICE_1084 "Returns are not supported for migrated orders".
7. Same for migrated first order, upgrade, renewal, add-on — frontend and backend. → Expected: Consistent blocking.

---

### Cancel staged orders while performing return. [P1] (from DVW-T168)

**Objective:** Returning line items clears staged cancel/downgrade in Aria/AXP; popup shows staged action; partial returns with staged cancel; upgrade return with staged actions.

**Preconditions:**

- Staged cancel/downgrade before return per scenario.

**Steps:**

1. (Precondition) Stage action scheduled before return. → Expected: Documented.
2. SKP GO qty 5 + SKP PRO qty 3; stage cancel GO + downgrade PRO; return both lines. → Expected: Popup lists staged actions; confirm → return success; staged cleared (API + Aria); AXP qty/removal; SF negative order+invoice; email; Aria plan; EMS expired.
3. SKP GO qty 1 + SKP PRO qty 3; downgrade PRO only; return both. → Expected: Popup only when selecting line with downgrade; otherwise none; same post-success validations.
4. SKP STUDIO qty 3 + SKP GO qty 1; cancel STUDIO; return both. → Expected: Popup for cancel staged line only when that line selected.
5. SKP PRO qty 5; renew; after renewal stage cancel PRO; return renewal line. → Expected: Popup cancel staged; success; plan removed AXP; SF negatives; EMS expired.
6. SKP GO qty 1; renew; after renewal stage downgrade GO; return renewal line. → Expected: Popup downgrade staged; success; plan removed AXP; negatives; EMS expired.
7. SKP PRO qty 3; stage cancel; partial return 1 qty. → Expected: Popup; success for 1 qty; staged cleared; AXP qty reduced; SF negatives for returned qty only; EMS not expired if qty remains.
8. SKP PRO qty 2 + SKP STUDIO qty 3; renew; cancel both; partial return 2 STUDIO. → Expected: Popup rules per line selection; staged cleared only for affected line; other line staged remains; partial SF negatives; EMS partial alive.
9. SKP PRO qty 5 → upgrade STUDIO; stage cancel or downgrade; full return upgrade (no partial on upgrade). → Expected: Popup; success; AXP reverses to downgraded plan; SF negatives; email; Aria; new EMS for downgraded SKU.

---

## Key Validations

- Negative Salesforce orders/invoices match refunded or chargebacked commercial amounts and SKUs.
- Aria plan, entitlement, and license states stay consistent with return type (full, partial, upgrade, renewal, chargeback).
- API guardrails: latest order only, staged orders blocked (except DVW-T168 explicit flows), migrated orders blocked, cancel-order state machine.

## Common Preconditions

- Support portal credentials and x-account-id headers for return/cancel APIs.
- Runbook access for migrated chargeback field IDs (DVW-T162).
