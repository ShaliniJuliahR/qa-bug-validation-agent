# Feature: Product-Specific Flows (ProjectSight, Connect, SketchUp)

## Overview

Product-specific commerce flows for ProjectSight, Trimble Connect, and SketchUp AI including provisioning, purchase flows for US/non-US regions, renewals, upgrades, and license management.

## Test Flows

### DVR user with AO / SAO / PU role provisioning [P1] (from DVW-T80)

**Objective:** DVR users with Account Owner, Secondary Account Owner, or Product User roles: anonymous cart, sign-in, checkout routing, and currency behaviour.

**Preconditions:**

- DVR test users per role; MXP PDP and cart flyout.

**Steps:**

1. Not signed-in AO/SAO section headers → Expected: Use as scenario grouping in Zephyr.
2. PDP: add to cart; cart flyout → Sign in to checkout → Expected: Redirect to TID login.
3. DVR with AO/SAO after login → Expected: Treated as anonymous; anonymous cart kept.
4. If DVR account currency GBP but browsing as anonymous → Expected: Cart currency follows current region, not GBP.
5. Not signed-in Product User section → Expected: Scenario grouping.
6. PU: proceed to checkout → Expected: Account creation with PU/LI popup.
7. DVR AO/SAO from PDP flyout sign-in → Expected: Return to PDP after login.
8. Multi-role user: switch to AO/SAO from account switcher → Expected: Correct navigation and cart behaviour; auto-fill on account creation when applicable.

---

### ProjectSight – commerce flows UI & API merge scenarios [P2] (from DVW-T109)

**Objective:** Anonymous + signed-in merge paths for ProjectSight with Connect/SketchUp across US/Canada vs ROW: geo restriction, currency, quantity threshold, license length mismatches.

**Preconditions:**

- Carts with ProjectSight, Connect, SketchUp in varied quantities/currencies per Zephyr matrix.

**Steps (representative – full matrix in DVW-T109):**

1. US/Canada anonymous adds ProjectSight; signs into US/CA account without cart → Expected: Checkout shows ProjectSight.
2. US/CA anonymous with ProjectSight; signs into US/CA with Connect+SketchUp cart → Expected: Checkout shows all three.
3. ROW anonymous adds Connect+SketchUp; signs into US/CA account → Expected: Checkout with existing Connect+SketchUp.
4. ROW anonymous adds Connect+SketchUp; signs into US/CA with ProjectSight cart → Expected: Checkout includes all products.
5. US/CA anonymous ProjectSight; sign into ROW → Expected: Geo-restricted empty cart page.
6. US/CA anonymous with ProjectSight+Connect+SketchUp (high qty); sign in → Expected: Max quantity popup; then carts page after continue.
7. Currency / license-length / product-removal combinations → Expected: Popups and landing page (checkout vs carts) per each Zephyr row (EUR/USD, monthly vs yearly, etc.).

---

### Validation of freemium to paid upgrade (ProjectSight) [P1] (from DVW-T119)

**Objective:** Freemium ProjectSight customer: upgrade path differs for geo inside vs outside US/Canada.

**Preconditions:**

- Freemium ProjectSight account; geo inside/outside US+Canada.

**Steps:**

1. Sign in as freemium ProjectSight customer → Expected: Identified as freemium.
2. Click "Upgrade to Paid Version" → Expected: Redirect to ProjectSight PDP.
3. Outside US/Canada: PDP behaviour → Expected: Informational popup; flyout cart empty/minimized; no Add to Cart / qty / standard pricing; pricing variant without prices.
4. Inside US/Canada → Expected: Add to Cart, qty, pricing visible and work.
5. Add to cart inside US/CA → Expected: Quantity matches freemium project collaborator count.

---

### Product User / License Admin purchase flow [P1] (from DVW-T124)

**Objective:** PU/LA checkout from PDP flyout; account creation merge; AO+PU combined behaviour.

**Preconditions:**

- Users: PU/LA only; PU/LA + AO; PU/LA + AO + SAO.

**Steps:**

1. Sign-in checkout from PDP flyout; sign in as PU/LA → Expected: Section markers in Zephyr.
2. After sign-in on PDP → Proceed to checkout → Expected: Account creation popup; after creation, cart merged.
3. Continue shopping → Expected: New PU/LA account id in URL; latest PU/LA cart on PDP without errors.
4. PU/LA + AO → Proceed to checkout → Expected: No account switcher; default AO for product; order completes; verify in Commerce + AXP.
5. PU/LA + AO + SAO → Expected: Switcher allows AO or SAO; PU/LA disabled in switcher; orders succeed in either purchasable account.

---

### E2E payment update PS (ProjectSight) [P2] (from DVW-T126)

**Objective:** End-to-end payment method changes (card, PayPal), add seats, dunning, renewals for ProjectSight flows; CDH, Aria, Salesforce, CyberSource/PayPal alignment.

**Preconditions:**

- Try flows as AO and SAO alternately (per Zephyr note).
- New user with products to checkout.

**Steps (summary – DVW-T126 contains many sub-flows in Zephyr):**

1. First order: no PM → add card → place order → verify CDH/Aria/SF/CyberSource (0$ auth on first card order).
2. First order: PayPal path → verify PayPal transaction and downstream systems.
3. Add seats: switch PM card↔PayPal; verify each downstream system after change.
4. Remove PM then re-add (card or PayPal); place orders; verify systems.
5. Dunning scenarios: failing PayPal billing agreement, expired card via Aria API, renewal into dunning, then update PM in AXP to recover.
6. Card → dunning (no PM) → renewal → update to card or PayPal → verify.
7. PayPal → remove PM → renewal dunning → restore PayPal or card.
8. Final "verify in downstream systems" steps after each major branch → Expected: CDH, Aria, Salesforce, CyberSource/PayPal consistent.

---

### Remove upgrade & downgrade for Connect Business / Premium [P3] (from DVW-T127)

**Objective:** Trimble Connect Business and Business Premium (annual/monthly) must not show upgrade/downgrade in AXP manage subscription or purchase flow.

**Preconditions:**

- Test user; Connect Business or Premium subscription.

**Steps:**

1. Create account; place order for Connect Business or Premium (annual or monthly) → Expected: Order success.
2. AXP Admin → Manage Purchase → Subscription & License → Expected: Page opens.
3. Click subscription name → PDP → Expected: PDP displays.
4. Manage subscription → Expected: Small manage subscription screen.
5. On manage subscription page → Expected: No Upgrade or Downgrade options (see Zephyr screenshot).
6. For placed order → Expected: No upgrade/downgrade in post-purchase UI.
7. Purchase flow → Expected: No upgrade/downgrade offers.

---

### Connect & ProjectSight purchase – non US & Canada [P1] (from DVW-T129)

**Objective:** Anonymous cart with Connect + ProjectSight; sign-in with non-US/CA address removes ProjectSight and continues with Connect.

**Preconditions:**

- Anonymous cart with both products; ROW account creation credentials in Zephyr.

**Steps:**

1. Add Connect and ProjectSight as anonymous → Expected: Both in cart.
2. Verify cart shows both → Expected: Both line items visible.
3. Sign in to checkout → Expected: Prompt to sign in or create account.
4. Create ROW account → Expected: Account and address validated.
5. After login → Expected: Cart updated popup; ProjectSight removed from cart/checkout.
6. Popup shows removed product name → Expected: Correct product named.
7. Continue → Expected: Checkout with Connect only.
8. Complete purchase → Expected: Order confirmation.

---

### Validate renewal – Trimble Connect SKUs [P1] (from DVW-T130)

**Objective:** Connect Business & Premium monthly/annual renewals with aligned anniversary dates; payment validation in Aria and Salesforce.

**Preconditions:**

- New account; Connect SKUs; payment methods; renewal document link in Zephyr.

**Steps:**

1. Confirm Connect Business and Premium SKUs available for scenarios → Expected: SKUs ready.
2. Complete account setup with payment → Expected: Account and PM saved.
3. First order: annual; record anniversary → Expected: Annual order and anniversary captured.
4. Second order: monthly with same anniversary as first → Expected: Anniversary alignment.
5. Trigger renewals at anniversary → Expected: Both renew.
6. Validate SKUs post-renewal → Expected: Match expected combination.
7. Validate renewal payment in Aria & Salesforce → Expected: Amount, method, date match.
8. Repeat for monthly/annual mix combinations → Expected: All renew correctly.
9. Cross-check detailed scenarios → Expected: Match verified renewal document.

---

### ProjectSight & Connect purchase – US & Canada [P1] (from DVW-T132)

**Objective:** Anonymous cart with both products; US/CA account; full checkout without ProjectSight removal popup.

**Preconditions:**

- US/CA test account credentials in Zephyr.

**Steps:**

1. Add Connect + ProjectSight to cart → Expected: Both added.
2. Verify cart → Expected: Both products with details.
3. Sign in to checkout → Expected: Sign-in/create prompt.
4. Create US/CA account → Expected: Address validated.
5. Account setup → Expected: Details saved.
6. Payment → Expected: Card or PayPal accepted.
7. Order review → Expected: User reaches review to place order.
8. Place order → Expected: Confirmation / thank you page.
9. No "removed product" popup for US/CA → Expected: Popup does not appear.

---

### EP customer purchases PS [P2] (from DVW-T137)

**Objective:** EP-provisioned customers buying in DVW: account types, address validation, currency mapping, CDH/CT reuse, tax exempt parity, registered cart.

**Preconditions:**

- EP accounts: US order, invalid/valid addresses (US, India, etc.), Individual vs Org, VAT variants.

**Steps (condensed – 22 steps in Zephyr):**

1. EP user who ordered in US logs in → Expected: Anonymous cart OK; provisioning API not called prematurely.
2. EP Individual without VAT → Expected: Individual in DVW.
3. Proceed to checkout as EP AO/SAO → Expected: DVW account creation page.
4. Invalid city/state/zip/India invalid cases → Expected: Popups; editable fields per rules; country locked where specified.
5. Valid US / valid India address → Expected: Fields disabled or editable per rules; Address Doctor behaviour when correcting invalid EP address.
6. State mismatch (EP vs SFDC) → Expected: State/city/zip edit rules per country.
7. Currency step → Expected: Currency from mapping API; Canada/Australia display CAD/AUD though EP may show USD; TEBV per mapping.
8. Complete setup → Expected: Reuse CDH; new CT customer; DVW in source systems.
9. Place order → Expected: Proration for EP customer order.
10. EP PU/LA → Expected: Account creation; new CDH/CT as applicable.
11. US tax exempt in EP → Expected: Tax exempt in DVW.
12. EP AO/SAO on PDP → Expected: Registered cart created.

---

### API – Viewpoint license assignment and revoke [P?] (from DVW-T141)

**Objective:** Viewpoint: license assignment and revoke for managed vs usual TID users (API).

**Preconditions:**

- As defined when test script is completed in Zephyr.

**Steps:**

1. (Zephyr currently has placeholder step 0 with empty description – populate when script is finalized.)

---

### SketchUp AI (Bucky) – commerce checkout [P1] (from DVW-T166)

**Objective:** Bucky (SketchUp AI) monthly SKU via deep link / SketchUp app; cart merge with MXP/AXP; roles; tax/proration; renewals/dunning; errors.

**Preconditions:**

- Checkout URLs with sku=BN-SB-SKP-AI-EC-M and account; SketchUp frontend QA URL in Zephyr.

**Steps:**

1. New sale: complete SketchUp app steps to AI credits → checkout URL with quantity → Expected: Bucky monthly in checkout; order can complete.
2. Existing MXP monthly cart + append Bucky URL → Expected: Both monthly SKUs visible; purchase completes.
3. Annual SKP/Connect/PS cart + append Bucky monthly → Expected: Annual removed; Bucky added; popup explains conflict.
4. EP / DVR route section headers → Expected: Follow Zephyr subsections.
5. AO/SAO: account setup retrieve popup → Expected: Popup shown; purchase completes with Bucky URL append.
6. PU/LI purchase via Bucky URL → Expected: Success; emails to AO and PU/LI.
7. Non-AO/SAO role → Expected: Error restricting checkout.
8. AXP + MXP cart refresh → Expected: One cart wins per scenario (MXP deleted when AXP created and vice versa).
9. Checkout quantity change + refresh → Expected: UI updates then refresh resets to URL quantity.
10. Merge MXP cart with Bucky → Expected: Single MXP cart with all SKUs.
11. AXP vs Bucky priority scenarios → Expected: Per scenario A/B in Zephyr.
12. Tax & proration across legal entities → Expected: Correct tax lines and proration.
13. E2E: full funnel dataLayer/events consistency → Expected: Per Zephyr end section.
14. Email link to checkout → Expected: Redirect; licenses in Admin Console; "Action Required: User Request for SketchUp AI Subscription" email.
15. Orders on holds: return/cancel/refund/release → Expected: Flows complete per Zephyr.
16. Renewals & dunning → Expected: Continuation/cancellation behaviour correct.
17. Expired/invalid checkout link → Expected: Appropriate error page/message.

## Key Validations

- ProjectSight geo rules: US/CA vs ROW, merge popups, removal of ProjectSight for non-US/CA accounts.
- Connect renewal alignment, SKU swap mappings on legacy migration (see also promo-pricing swap doc).
- DVR/PU/LA/AO/SAO account selection and cart ownership.
- Bucky: URL-driven checkout, MXP/AXP cart precedence, annual vs monthly conflicts.

## Common Preconditions

- QA Netlify / shop URLs, test TID accounts, and AXP Admin access as listed in Zephyr.
- VPN or geo settings when cases specify region.
