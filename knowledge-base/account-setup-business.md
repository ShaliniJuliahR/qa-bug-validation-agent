# Feature: Account Setup - Business Info & Tax

## Overview

Business information, VAT/GST exemption and verification, purchase type toggles, marketing consent, edit/collapse behavior, CT/CDH integration after completion, error scenarios, and Fonoa validation (including Admin Console cross-checks where specified).

## Test Flows

### Testing Integration of CT and CDH post Account Completion [P1] (from DVW-T5)

**Objective:** Complete Account Setup triggers processing UI; success lands on Carts; verify CT and CDH; failure and retry paths.

**Preconditions:**

- Full account setup completable with all mandatory sections filled (Not_Automated in Zephyr—plan manual time).

**Steps:**

1. Verify Complete Account Setup enabled when required fields filled. → Expected: Button enabled.
2. Click Complete Account Setup. → Expected: Grey overlay; processing popup while account is set up.
3. After success. → Expected: Redirect to Carts Page.
4. Click Complete Account Setup after long delay (~1 hour scenario). → Expected: Account still creates; Carts shown.
5. Verify CDH account created. → Expected: CDH record present.
6. Verify Commerce Tools account created. → Expected: CT record present.
7. If CDH or CT integration fails on submit. → Expected: Error "We encountered an error processing your account. Please try again in a few minutes. If you receive this error again, please contact customer support."
8. Verify CDH not created on failure path. → Expected: GetAccounts returns empty / not created per spec.
9. Verify CT not created on failure path. → Expected: Not created in CT.
10. Retry Complete Account Setup after minutes. → Expected: Resubmit works; Landing if created.
11. Follow CDH verification as in step 5 when successful. → Expected: CDH present.

---

### Field validation of Business Information Section [P2] (from DVW-T23)

**Objective:** Company name variants, size, country, VAT question for non-US, Continue enablement, toggling business No.

**Preconditions:**

- Account Setup Step 1 with contact auto-filled; phone country code + valid number.

**Steps:**

1. Login; reach Account Setup Profile; fill contact (auto name/email; phone with country code). → Expected: Step 1 ready.
2. Select Yes for business purchase. → Expected: Business Information section shown.
3. Verify Company Name, Company Size, Country/Territory displayed. → Expected: All three fields present.
4. Enter alphanumeric company name. → Expected: No error.
5. Enter numbers in company name. → Expected: No error.
6. Enter special characters in company name. → Expected: No error.
7. Enter alphanumeric with spaces. → Expected: No error.
8. Select company size. → Expected: No error.
9. Select country. → Expected: No error.
10. US country → Continue to Account Address enabled. Non-US → VAT/GST registration question appears with default Yes/No. → Expected: Per country rules.
11. Select No on VAT/GST question (non-US). → Expected: Continue to Account Address enabled.
12. Select No on business purchase. → Expected: Business section hidden; Continue to Account Address enabled.

---

### Field validation of Business Information Section [P3] (from DVW-T24)

**Objective:** Company name length (200 chars), size, country, VAT question, same enablement rules as T23.

**Preconditions:**

- Same as DVW-T23 entry state.

**Steps:**

1. Login; Account Setup Profile complete through phone. → Expected: Ready for business question.
2. Select Yes for business purchase. → Expected: Business section visible.
3. Verify Company Name, Company Size, Country/Territory. → Expected: Fields displayed.
4. Enter company name up to 200 chars (alphanumeric/specials/spaces). → Expected: Truncated or capped at 200; no error.
5. Select company size. → Expected: No error.
6. Select country. → Expected: No error.
7. US vs non-US: Continue vs VAT/GST question behavior. → Expected: Same as DVW-T23 steps 10–11.
8. Select No on VAT/GST when shown. → Expected: Continue enabled.
9. Select No on business purchase. → Expected: Business hidden; Continue enabled.

---

### Validation of Profile details collapsed view (business Yes, VAT/GST No) [P3] (from DVW-T25)

**Objective:** Collapsed summary shows contact + business lines correctly after Continue to Company address enabled.

**Preconditions:**

- Profile and business completed with business Yes and VAT/GST No.

**Steps:**

1. Complete profile with business Yes and business section with VAT/GST No. → Expected: Continue to Company address enabled.
2. Profile details displayed on Account Setup. → Expected: Page state correct.
3. Verify Contact and Business in collapsed view. → Expected: Collapsed sections visible.
4. Contact summary: bold name, email, phone, business purchase Yes. → Expected: Content and emphasis match spec.
5. Business summary: company name, size, country; US → Continue to Account Address; non-US → VAT question and No path enables Continue. → Expected: Text and controls per rules.

---

### Selecting 'Yes' for Tax ID Verification [P3] (from DVW-T27)

**Objective:** Non-US VAT Yes path: VAT field, verify valid ID, collapsed sections, proceed to account address.

**Preconditions:**

- Account Setup with business Yes.

**Steps:**

1. Login; fill contact with phone. → Expected: Step 1 ready.
2. Select Yes for business purchase. → Expected: Business section shown.
3. Verify Company Name, Size, Country fields. → Expected: Fields present.
4. Enter valid company name, size, country (US skips VAT question). → Expected: US: no VAT question; non-US: VAT question with No default.
5. For non-US, verify VAT/GST question shown. → Expected: Question visible.
6. Select Yes on VAT/GST. → Expected: VAT/GST ID field under verification prompt.
7. Enter valid VAT/GST for country; click Verify. → Expected: Validates and shows verified state when valid.
8. Verify collapsed Contact and Business sections. → Expected: Collapsed view correct.
9. Complete Step 1 mandatory fields; verify Complete Account Address enabled. → Expected: Button enabled (label per build).
10. Verify Complete Account Address enabled (duplicate check per Zephyr). → Expected: Enabled.
11. Click Complete Account Address; continue to Account Address section. → Expected: User proceeds to address entry.

---

### UI Validation when Yes is selected for business purchase [P3] (from DVW-T28)

**Objective:** Business section layout: mandatory markers, placeholders, Continue disabled until required data, VAT question text.

**Preconditions:**

- Prerequisite: DVW-181 linked cases completed.

**Steps:**

1. Follow prerequisite (DVW-181 tests). → Expected: Prerequisite state satisfied.
2. Select Yes for business purchase question. → Expected: Yes selected.
3. Business section shows Company Name, Size, Country/Territory. → Expected: Fields displayed.
4. Company name and country mandatory; company size optional. → Expected: Markers correct.
5. Company name open text with placeholder. → Expected: Placeholder shown.
6. Company Size dropdown placeholder "Number of Employees". → Expected: Placeholder correct.
7. Country dropdown placeholder "Select Country or Territory". → Expected: Placeholder correct.
8. Continue to Account Address visible but disabled until mandatory filled. → Expected: Disabled when empty.
9. Verify VAT/GST question in Business Information (Zephyr text references tax exempt wording—confirm current copy vs release). → Expected: Question block shown with default No.

---

### Validation of Profile details collapsed view (business Yes, VAT/GST Yes) [P3] (from DVW-T31)

**Objective:** Collapsed summary when VAT Yes includes verified VAT and Continue rules.

**Preconditions:**

- Business Yes; VAT/GST Yes with valid VAT entered.

**Steps:**

1. Complete profile with business Yes and VAT/GST Yes including valid VAT. → Expected: Continue to Company address enabled.
2. Profile details on Account Setup. → Expected: Page shown.
3. Contact and Business collapsed. → Expected: Collapsed layout.
4. Contact: name, email, phone, business Yes. → Expected: Summary correct.
5. Business: company, size, country; US shows tax exempt question variant; non-US VAT Yes requires VAT ID before Continue to Account Address enabled. → Expected: Matches Zephyr matrix.

---

### Marketing Consent Validation - Non Trial User [P3] (from DVW-T34)

**Objective:** Marketing checkbox under Privacy Notice; survives profile edit round-trip.

**Preconditions:**

- User not opted into marketing.

**Steps:**

1. Login as user not opted into marketing. → Expected: Account Creation Page.
2. Complete Profile details; Continue to account address enabled. → Expected: Ready for address.
3. Continue to Account Address. → Expected: Address page shown.
4. Fill mandatory address fields; reach currency/complete state. → Expected: Complete Account Setup enabled when all required filled.
5. Verify marketing consent checkbox under Privacy Notice. → Expected: Checkbox present.
6. Check marketing consent; minor profile edit; return to Account Address. → Expected: Consent selection retained.

---

### Validation of Purchase Type change [P3] (from DVW-T35)

**Objective:** Switching business Yes → No from collapsed edit updates steps and address view.

**Preconditions:**

- Completed profile with business Yes, VAT/GST No, reached company/account address step.

**Steps:**

1. Complete profile business Yes, business info with VAT No. → Expected: Business block complete.
2. Continue to Account Address enabled; step 2 address visible with collapsed contact+business. → Expected: Layout as specified.
3. Edit from collapsed view; set business purchase to No. → Expected: Business section hides; Continue to billing/account address enabled.
4. Continue to Account Address. → Expected: Address page; collapsed view shows only contact information.

---

### Validation of Edit functionality in Account Setup page [P3] (from DVW-T36)

**Objective:** Edit toggles between business and personal paths; address resets when purchase type changes.

**Preconditions:**

- User can complete profile and address sections.

**Steps:**

1. Complete profile with business Yes; Continue to Company Address. → Expected: Profile collapsed; company address expanded.
2. Verify Edit button. → Expected: Edit visible.
3. Select country and valid address fields. → Expected: Continue enabled.
4. Edit; change phone and/or company fields; Continue to account address. → Expected: Collapsed summary updates.
5. Edit; switch business to No. → Expected: Business section hidden.
6. Continue to account address. → Expected: Billing address expanded; prior company-only address fields empty; only contact collapsed.
7. Fill address; Complete Account Setup enabled. → Expected: Enabled when valid.
8. Edit; switch business back to Yes. → Expected: Profile expands with business section.
9. Complete business again; Continue to account address. → Expected: Company address expanded empty; prior billing not shown.

---

### VAT/GST Exemption Validation for Non - US User [P2] (from DVW-T38)

**Objective:** Non-US VAT paths: ID vs certificate options, verify gating, EP migration spot checks, network validation of VAT with spaces.

**Preconditions:**

- Non-US TID user; Figma scenario G for reference.

**Steps:**

1. Login as Non-US user. → Expected: Profile Details section.
2. Verify no default selected for business purchase question (per Zephyr). → Expected: No pre-selection until user acts.
3. Verify First/Last/Email auto-filled; enter valid non-US phone. → Expected: Contact valid.
4. Select Yes for business purchase; fill company name, size, country. → Expected: Business fields complete.
5. Select No on "Is your business or organization registered for VAT/GST?" → Expected: Continue to Company Address enabled.
6. Select Yes on business VAT/GST exempt question path (wording per Zephyr step). → Expected: Continue to Company Address disabled until verification resolved.
7. Verify "Accepted verification options" shows ID Number and Certificate. → Expected: Both options offered.
8. Select ID Number. → Expected: VAT/GST ID field mandatory; notice that tax ID must be verified before proceed.
9. Complete account setup when enabled. → Expected: Setup completes.
10. Verify UI vs Figma. → Expected: Visual match.
11. Complete Account Address disabled until VAT entered (when Yes path). → Expected: Disabled until valid entry.
12. Continue to company address enabled; profile collapsed (intermediate checkpoint per Zephyr). → Expected: State verified.
13. Enter VAT ID; click Verify. → Expected: Green check when valid.
14. Enter VAT with spaces; verify in network tab validation POST and Account Container API. → Expected: VAT passed without spaces in validation call.
15. Continue; collapsed view shows VAT verified under business column. → Expected: Summary shows verified state.
16. EP individual migrated account: name/email fetched in DVW setup. → Expected: Data correct (when applicable).
17. EP org without VAT migrated. → Expected: Company fields correct.
18. EP org with VAT migrated. → Expected: Company, country, VAT correct.

---

### Account Setup Error Scenarios [P3] (from DVW-T106)

**Objective:** Toasters and fallbacks for profile fetch, Address Doctor, typeahead, VAT verify, Fonoa country, throttling (INFEASIBLE label in Zephyr—confirm tooling).

**Preconditions:**

- Ability to simulate network failure, slow network, invalid VAT, unsupported Fonoa country.

**Steps:**

1. Scenario header: failure to fetch name/email from Trimble Cloud. → Expected: (See step 2.)
2. Accounts endpoint: Trimble Cloud profile failure. → Expected: Toaster "refresh the page as we are currently unable to access your profile details."
3. Scenario: failure to connect Address Doctor. → Expected: (See step 4.)
4. Address Validation API failure. → Expected: No Try Again; user message that validation unavailable; user may proceed with typed address.
5. Address typeahead load failure. → Expected: Error toaster when suggestions cannot load.
6. Address Doctor no match. → Expected: Copy about no match + recommended "Address not found. Try another or use original."
7. VAT ID error scenarios header. → Expected: (See following steps.)
8. Invalid VAT format/ID; click Verify. → Expected: Error toaster.
9. Fonoa unsupported country (e.g. Fiji). → Expected: Error toaster in UI.
10. Throttle network to 2G while entering VAT; wait. → Expected: Per Zephyr, no erroneous toast (confirm current product behavior).
11. VAT with special character on Verify. → Expected: Toaster includes "Something went wrong while processing" (or current copy).

---

### Fonoa VAT validation - Commerce [P1] (from DVW-T170)

**Objective:** Account Creation and Admin Console: supported vs unsupported countries, Fonoa valid/invalid/skip, India/Mexico/Malaysia extra fields, tax ID edit/remove, country sync (condensed from Zephyr indices 0–35).

**Preconditions:**

- Valid TID; access to Admin Console Account Settings for console legs; ecommerce country selection for sync tests.

**Steps:**

1. Section: Account Creation Page (header step). → Expected: Enter flow context.
2. Login with valid TID. → Expected: Account Creation page displayed.
3. Observe Contact Information. → Expected: First/Last/Email prefilled and read-only.
4. Select Yes for business purchase. → Expected: Business Information enabled.
5. Enter company name; select company size. → Expected: Company name mandatory; size captured.
6. Select country. → Expected: Tax registration question appears.
7. Select Yes for VAT/GST registration. → Expected: Tax ID number field displayed.
8. Section: Tax ID validation — Fonoa supported countries. → Expected: Supported country (e.g. Germany/UK/FR) shows Tax ID field.
9. Enter valid registered Tax ID; click Verify Details. → Expected: Backend validationStatus valid; success UI.
10. Enter valid format but not tax-registered ID. → Expected: validationStatus invalid or equivalent failure.
11. Enter invalid Tax ID and verify. → Expected: Error e.g. ACCOUNTS_1346 / VAT invalid message.
12. Section: Unsupported countries (validation skipped). → Expected: Header context.
13. Select Canada as unsupported-for-Fonoa example in script. → Expected: Field shown per spec.
14. Enter valid or invalid Tax ID for unsupported country. → Expected: HTTP 200 with validationStatus skipped; Continue to account address allowed.
15. Section: Additional mandatory fields — India, Mexico, Malaysia. → Expected: Header context.
16. India: State Name dropdown in tax section mandatory. → Expected: Cannot proceed without state.
17. Mexico: Zip/Postal rules—no specials; max length 10; valid zip required. → Expected: Errors for violations.
18. Malaysia: ID Type dropdown (Army, Passport, NRIC, BRN) and Business Registration Number with character rules. → Expected: Special chars blocked with message.
19. Summary step: India/Malaysia/Mexico supported → Fonoa triggers; valid accepted, invalid blocked. → Expected: Matches policy statement in Zephyr.
20. Section: Valid Tax ID — verification and continue. → Expected: Header context.
21. Enter valid Tax ID for supported or unsupported country; Continue to Account Address. → Expected: VAT ID shows Verified in summary.
22. Section: Admin Console tax validation. → Expected: Header context.
23. Navigate Admin Console → Account Settings. → Expected: Account Settings page loads.
24. Observe Business Information after ecommerce country selection. → Expected: Country matches ecommerce; company and tax sections visible.
25. Tax ID status when no Tax ID in ecommerce. → Expected: Shows Not registered; No selected; Edit available.
26. Tax ID status when valid ID saved in ecommerce for supported/unsupported mix. → Expected: Verified where applicable per rules.
27. Remove existing Tax ID via Edit → No → Save. → Expected: Tax ID removed; status Not registered.
28. Edit Tax ID — supported country valid ID. → Expected: Fonoa valid response; Verified.
29. Edit Tax ID — supported country invalid/unregistered. → Expected: invalid response; save blocked.
30. Edit Tax ID — unsupported country. → Expected: validationStatus skipped; tax ID saved.
31. Verify India additional fields (reduced retest). → Expected: State dropdown mandatory.
32. Verify Mexico zip rules (reduced retest). → Expected: Same as step 17.
33. Verify Malaysia ID Type + BRN (reduced retest). → Expected: Same as step 18.
34. Verify Country Sync after order with specific country. → Expected: Admin country matches ecommerce; no mismatch.

---

## Key Validations

- Business block: mandatory company + country; optional company size; US vs non-US tax question differences.
- VAT/GST Yes: verify gating, error toasters, verified state in collapsed summary, Continue enablement.
- Purchase type edits reconfigure visible sections and may clear address data (DVW-T35, DVW-T36).
- Post-submit: processing overlay, Cart redirect, CT/CDH creation or structured failure (DVW-T5).
- Error layer: profile, typeahead, Address Doctor, VAT, Fonoa unsupported country (DVW-T106).
- Fonoa matrix: supported vs skipped validation, regional extra fields, Admin Console parity (DVW-T170).

## Common Preconditions

- TID users for new setup vs migrated EP accounts where noted.
- Test companies, VAT IDs, and countries per Zephyr (valid/invalid per region).
- Admin Console access for DVW-T170 console portions.
- Network manipulation capability only for scenarios that require throttling or forced API failures.
