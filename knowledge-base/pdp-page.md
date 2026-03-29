# Feature: Product Detail Page (PDP)

## Overview

PDP page validation including product media display, quantity field validation, cart slide-out menu, add to cart functionality, and navigation flows for guest and registered users.

## Test Flows

### Validation of PDP Page (With / Without Media) [P2] (from DVW-T45)

**Objective:** Product information, media carousel (if applicable), license length options, pricing by geo, mini cart / flyout CTAs, responsive layout.

**Preconditions:**

- Guest user; product with and without media as applicable; Figma reference.

**Steps:**

1. Navigate to PDP as guest → Expected: PDP displayed.
2. For products with media → Expected: Carousel navigates images/videos smoothly.
3. Observe name, SKU, description → Expected: Match catalog.
4. If license lengths exist → Expected: Options visible.
5. Check retail prices per option → Expected: Currency matches guest geolocation.
6. Select license option → Expected: Selected state highlighted; others greyed.
7. Open mini cart from icon → Expected: Slide-out opens.
8. Responsive check tablet/phone/desktop → Expected: Layout matches Figma.

---

### PDP - Validation of Cart Slide out menu [P2] (from DVW-T46)

**Objective:** Slide-out after add-to-cart: messaging, counts, CTAs, quantity edits, max errors, scroll, empty state.

**Preconditions:**

- Guest or registered PDP session.

**Steps:**

1. Add item from PDP → Expected: Slide-out shows; "Item Added to Cart" visible.
2. Verify mini cart badge → Expected: Total item quantity updated.
3. Add another item → Expected: Slide-out appears again.
4. Open slide-out → Expected: Lines show name, qty, price, remove.
5. Verify subtotal under lines → Expected: Subtotal correct.
6. Header has X → Expected: X closes slide-out.
7. Reopen via mini cart → Expected: Slide-out opens.
8. Product imagery → Expected: Matches Figma.
9. Responsiveness of slide-out → Expected: Usable on breakpoints.
10. In slide-out, raise qty above max → Expected: Error about max quantity / contact sales.
11. Set qty to 1 → Expected: Trash icon appears left of qty.
12. Clear qty to 0 → Expected: Defaults to 0 and removes line from slide-out.
13. Enter negative/special/alpha in qty → Expected: Not accepted.
14. Trash remove → Expected: Line removed; badge and subtotal update.
15. On PDP set max qty → Expected: Inline max error on PDP as specified.
16. Remove all in slide-out → Expected: "Your cart is empty"; only Continue Shopping.
17. After empty, add again to max then increment → Expected: Add to cart enabled; qty 2 in mini cart and slide-out.
18. Add 4+ distinct products → Expected: Scrollbar in slide-out list.
19. Guest: open slide-out → Expected: CTAs "Sign In to Checkout" and "View Cart".
20. Registered: open slide-out → Expected: CTAs "Proceed to Checkout" and "View Cart".

---

### Validation of Quantity Field in PDP Page [P2] (from DVW-T68)

**Objective:** +/- controls, min/max validation, alphanumeric rejection, error clearing.

**Preconditions:**

- PDP with purchasable SKU.

**Steps:**

1. Click "+" → Expected: Quantity increases by 1 each click.
2. Enter 0 → Expected: Red border; "Quantity must be at least 1"; Add to Cart disabled.
3. At minimum, click "-" → Expected: "-" disabled.
4. Increase from error state → Expected: Error clears when valid qty restored.
5. Enter qty exceeding max (e.g. cart threshold scenario in Zephyr) → Expected: Red border, max message, Add to Cart disabled.
6. Reduce below violation → Expected: Error clears.
7. Type letters/specials → Expected: Field stays numeric-only.

---

### Navigation from PDP for Registered / Guest User [P2] (from DVW-T69)

**Objective:** Registered: View cart, Proceed to checkout, account switcher, account id in URL. Guest: slide-out CTAs to TID and guest cart.

**Preconditions:**

- Signed-in user with roles/accounts; guest session for latter steps.

**Steps:**

1. Sign in from flyout; return to PDP as registered → Expected: PDP shown; currency reflects account.
2. Add products with quantities → Expected: Prices/qty updated.
3. Add to cart → Expected: Slide-out with line details; Proceed + View cart for registered path.
4. Click Proceed to checkout → Expected: Checkout page.
5. Click View cart → Expected: Registered cart page; URL includes account id when applicable.
6. Account switcher section → Expected: (Placeholder header in Zephyr) follow multi-account steps.
7. Log out; open PDP as guest → Expected: Anonymous PDP.
8. Multi-role user: sign in from flyout → Expected: Default account name/role in flyout.
9. Open flyout account chevron → Expected: Switcher; pick role; cart URL reflects account.
10. Guest: add items → Expected: Slide-out with Sign in to checkout + View cart.
11. Guest: Sign in to checkout → Expected: TID login.
12. Guest: View cart → Expected: Guest shopping cart page.

---

### Validation of Add to Cart & Hidden "View Cart" (ProjectSight) [P3] (from DVW-T118)

**Objective:** ProjectSight PDP in US/Canada: add to cart works; "View Cart" hidden in flyout for anonymous and registered.

**Preconditions:**

- Geolocation or test setup for USA/Canada; ProjectSight product.

**Steps:**

1. Scenario: Anonymous customer (US/CA) → Expected: Section header only; geo verified.
2. On ProjectSight PDP, Add to cart → Expected: Product in flyout cart.
3. Check flyout → Expected: "View Cart" not shown for anonymous.
4. Scenario: Registered US/CA → Expected: Header for registered branch.
5. Sign in as registered with US/CA address → Expected: Customer context correct.
6. Open ProjectSight PDP → Expected: Page accessible.
7. Add to cart → Expected: Item in flyout.
8. Flyout for registered → Expected: "View Cart" still hidden per spec.

---

### PDP - User in ROW Regions: ProjectSight Flows [P1] (from DVW-T123)

**Objective:** Outside US/Canada: PDP hides purchase controls until eligible; sign-in scenarios (new user, AO/SAO, PU/LI, ROW, merges, currency popups) per ProjectSight matrix. (Zephyr lists many scenario headers; core expectations below.)

**Preconditions:**

- ROW VPN or geo; TID users per scenario (new, AO/SAO, PU/LI, with/without cart, with Connect+SketchUp in cart).

**Steps:**

1. Anonymous ROW on ProjectSight PDP → Expected: No Add to Cart, qty, or pricing card; variant pricing card visible; after TID login still no purchase card if still ROW-ineligible.
2. Repeat ROW checks after login per scenario blocks → Expected: Same visibility rules until country eligibility changes.
3. Anonymous no cart → sign in / create account flows → Expected: Cart in slide-out if existing; Add to cart + pricing when eligible; account creation may redirect to checkout or geo-empty cart per address country.
4. Anonymous with Connect+SketchUp → sign-in variants (US/Canada AO/SAO, PU/LI, ROW with/without items) → Expected: Slide-out shows combined cart; merges and currency mismatch / quantity / subscription popups appear when applicable; checkout or cart destination per scenario.
5. Add ProjectSight SKU when allowed → Expected: Line appears; proceed to checkout behaves per role.
6. For each "Repeat steps 1–7" blocks in Zephyr → Expected: Re-validate controls visibility, popups, and navigation for that user+cart combo.

---

### PDP - Account Selector [P3] (from DVW-T143)

**Objective:** When multiple purchasing accounts exist, selector and merge behavior; single-role shortcuts; Switch Account in cart; Trimble home parity.

**Preconditions:**

- Test accounts listed in Zephyr (single role, multi AO/SAO, PU/LI mixes).

**Steps:**

1. Single purchasing role user: anonymous cart → sign in → Expected: No account selector; cart merges if provisioned.
2. Only non-purchasing roles: anonymous cart → sign in to checkout → Expected: No selector; lowest PU/LI auto-selected; anonymous cart kept for same user.
3. Multi purchasing accounts: anonymous cart → sign in → Expected: Selector shown; pick account → Confirm → merge and currency update.
4. Top flyout → Expected: Account selector entry opens selector.
5. Cart slide-out → Expected: "Switch Account" opens selector.
6. Selector: Set as Default → Expected: Visible and clickable for eligible purchasing accounts.
7. Mixed purchasing + non-purchasing (example roles in Zephyr) → Expected: PU/LI entries disabled as specified.
8. DVW AO/SAO, EP AO/SAO, DVR AO/SAO, trial AO/SAO examples → Expected: Selector appears when ≥2 AO/SAO from allowed combinations.
9. After selecting DVW purchasable AO/SAO → Expected: Cart merges; applicable popups display.
10. From Trimble home: repeat anonymous cart + navigation → Expected: Same selector/merge behavior.

---

## Key Validations

- Slide-out content, CTAs, and mini cart counts stay synchronized after add/qty/remove.
- Quantity rules on PDP match cart/slide-out max messaging.
- Geo-gated SKUs (e.g. ProjectSight) hide or show purchase UI consistently for anonymous vs registered and ROW vs US/CA.
- Multi-account flows preserve correct `account` query parameter and merge+currency warnings.

## Common Preconditions

- QA (or target) storefront base URL; TID credentials for guest→registered transitions.
- Products with media vs without; ProjectSight when testing DVW-T118/T123.
- Figma links in Zephyr for visual comparison when automated checks are partial.
