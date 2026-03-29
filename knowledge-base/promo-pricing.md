# Feature: Promo & Pricing

## Overview

Promo code validation at cart level and order processing, legal entity order verification, custom rate calculations for amendment orders, and SKU swap functionality.

## Test Flows

### Legal entity order and payment verification workflow [P2] (from DVW-T94)

**Objective:** End-to-end purchase per legal entity country: currency, legal-entities API, card types, collection groups, PayPal availability, Aria CDFs, Cybersource, Salesforce.

**Preconditions:**

- DVW account; sample addresses (e.g. Belgium) per Zephyr sheet.
- Reference spreadsheet for LE numbers and ISO codes in Zephyr.

**Steps:**

1. Create DVW account → Expected: Reach account setup.
2. Profile → Continue to account address → Expected: Address section open.
3. Pick legal entity country (e.g. Belgium) → Expected: State/province options load.
4. Enter valid address → Expected: Continue to currency enabled.
5. Open currency section → Expected: Currency section shown.
6. Preferred currency → Expected: Symbol and options match country; complete setup to cart.
7. Add product (e.g. SKU-Pro) → Expected: Add to cart for that LE country.
8. Checkout: capture legal-entities network call → Expected: availableCardTypes, collectionGroupName, PayPal per country rules.
9. SAO same account → Expected: Same legal-entity API access.
10. Add new payment method → Expected: Card vs PayPal UI per country.
11. Billing save → Expected: Card types match API; CyberSource details.
12. Submit payment → Expected: PM saved.
13. Place order → Expected: Success; notifications; Aria account.
14. Aria client-defined fields → Expected: Legal_Entity and Tax_Registration_ISO_Code correct.
15. Cybersource merchant search → Expected: Transaction matches amount, qty, merchant ID.
16. Aria account group tab → Expected: Collection group mapped per LE.
17. Salesforce order → Expected: SKU, qty, amount, invoice, tax correct.
18. PayPal-supported country flow → Expected: PayPal section; place order; Aria PayPal mapping; sandbox verification.

---

### Calculate custom rates – amendment order on top of promo [P?] (from DVW-T147)

**Objective:** Custom rates for amendment orders when an underlying promo exists (draft case in Zephyr).

**Preconditions:**

- Test script in Zephyr currently has empty step body – define when case is completed.

**Steps:**

1. (No detailed steps in Zephyr yet – use DVW-T148 related flows until T147 is authored.)

---

### Calculate custom rates for amendment orders [P2] (from DVW-T148)

**Objective:** After partial return or upgrade on promo-priced orders, Aria plan instance `isCustomRateSet` and proration/refund math.

**Preconditions:**

- First order e.g. SKP-Go with promo.

**Steps:**

1. First order with promo; check Aria plan instance fields → Expected: isCustomRateSet not present/false initially.
2. Partial return (e.g. 3 of 5 qty) → Expected: isCustomRateSet true; refund = total paid / total qty × returned qty.
3. Upgrade GO→PRO → Expected: Proration uses custom rate (subtotal − first-order promo per unit).
4. Upgrade PRO→STDO → Expected: Proration uses original unit price (not custom promo rate path per step).
5. First order with promo; trigger renewal → Expected: Capture at original SKU unit price; custom rates not applied to renewal charge.

---

### Promo code validation – cart level & order processing [P1] (from DVW-T152)

**Objective:** Cart promo apply/remove errors; order placement; getOrders/SF/Aria/emails; upgrades, add seats, holds, dunning, returns with promo.

**Preconditions:**

- Valid/invalid/expired/wrong-SKU promo codes in Zephyr (e.g. STUDIO20, INVALIDCODE, STUDENT15).

**Steps:**

1. PDP add Studio → checkout → account details → Expected: Reach checkout with product.
2. Apply STUDIO20 → Expected: Apply disabled; discount under Promotions.
3. INVALIDCODE → Expected: "Invalid promo code".
4. EXPIREDCODE → Expected: "Promo code has expired".
5. PRO10 on Go cart → Expected: "not valid for the items in your cart".
6. Remove promo → Expected: Discount cleared; Apply re-enabled.
7. Order summary after promo → Expected: Discount and total correct.
8. Non-eligible user STUDENT15 → Expected: "Not eligible for this promo".
9. Very long code → Expected: Accept or truncate per product rules.
10. Change SKU after promo → Expected: Old promo removed; re-apply valid promo for new SKU.
11. Place order with valid promo → Expected: getOrders, SF payload/order product, Aria invoice, emails show promo; unit price net of promo in emails.
12. FO+promo then add seats same SKU with promo → Expected: Promo applies again; math matches step 11 expectations.
13. FO+promo then different SKU with promo → Expected: Same validation pattern as 11.
14. Upgrade without FO promo then upgrade with promo → Expected: Negative line without promo; positive with promo per SF rules.
15. Upgrade with FO promo and upgrade promo → Expected: Negative line includes promo; positive includes promo.
16. Upgrade with delay + proration + promo → Expected: Per Zephyr steps 16–17.
17. EP orders with/without proration + promo → Expected: Emails show unit price − proration/unit − promo/unit where applicable.
18. DVR with promo → Expected: Same promo visibility rules as 11.
19. Compliance hold + promo → Expected: Promo hidden in getOrders until released.
20. D-country hold + promo → Expected: Promo visible in getOrders; unchanged after release.
21. Dunning paid with promo → Expected: CyberSource/PayPal collects promo-adjusted amount.
22. Dunning revoke with promo → Expected: Negative invoice/order product include promo amounts.
23. Returns (FO, add seat, upgrade, combined scenarios) → Expected: Refunds and negative invoices reflect promo.
24. FO promo + partial return + renewal → Expected: Renewal uses pre-promo catalog amount; partial return refunds promo portion.
25. Complex chained scenarios (steps 27–31 in Zephyr) → Expected: Promo math and fields match step 11 baseline.

---

### Swap SKU [P1] (from DVW-T156)

**Objective:** Legacy Connect/SketchUp SKUs map to new SKUs at renewal; restrictions before swap; emails; dunning; holds.

**Preconditions:**

- Migrated EP accounts per legacy SKU; SWAP scheduled; optional Get Staged Order / Aria future plan for pre-renewal checks.

**Steps:**

1. Prerequisites block → Expected: Migrated accounts; six SKUs scheduled; staged order / Aria future plan for pre-renewal validation.
2. SketchUp Pro 2yr → renew → Expected: Converts to BN-SB-SKP-PRO-EC-A; verify AXP, Aria, SF, qty, subtype SWAP + type Renewal, entitlements, getOrder, emails.
3. Studio 2yr → BN-SB-SKP-STD-EC-A → Expected: Same verification pattern.
4. Connect Business Annual → BN1-SB-TCTAR2-EC-A → Expected: Same pattern.
5. Connect Business Monthly → BN1-SB-TCTAR2-EC-M → Expected: Same pattern.
6. Connect Business Premium annual → BN1-SB-TCTR3-EC-A → Expected: Same pattern.
7. Shop SKU SKP-SHOP-YR-WEB-01 → Go subscription → Expected: Swap on renewal.
8. Before renewal: AXP operations → Expected: No upgrade/downgrade/add seats until renewal completes (per spec).
9. Email templates before/after renewal → Expected: SKU name, price, qty correct.
10. Dunning on renewal for SWAP → Expected: Plan revert behaviour when payment fixed.
11. Renewal on hold → Expected: Tax in SF when hold; success after release.
12. Downgrade + renewal swap example (Pro 2yr migrated) → Expected: Post-renewal SKU per downgrade+swap scenario in Zephyr.

## Key Validations

- Promo: cart errors, downstream persistence (getOrders, SF, Aria, email), holds/dunning/returns.
- Custom rates: `isCustomRateSet`, refund and upgrade proration after promo FO.
- LE: legal-entities API drives PM UI and Aria/SF/CyberSource/PayPal routing.
- Swap: renewal-time SKU mapping, SF subtype SWAP, cross-system consistency.

## Common Preconditions

- QA catalog prices and promo codes active in test env.
- Legacy migration completed for swap scenarios (DVW-T156 precondition).
