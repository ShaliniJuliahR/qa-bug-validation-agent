# Feature: Account Setup - Address

## Overview

Billing and account address validation including country/territory selection, address fields, Address Doctor suggestions, state rules by country, and Fonoa/tax-adjacent currency flow after address.

## Test Flows

### Validation of Billing Address Setup & Profile Details Edit [P3] (from DVW-T3)

**Objective:** Minor profile edits do not wipe billing address; country from business info stays frozen moving to account address.

**Preconditions:**

- Account setup with business country selected and user progressed toward account address.

**Steps:**

1. Select Country/Territory in business information; proceed toward step 2 account address. → Expected: Country remains fixed/frozen when moving to account address.
2. Click Edit in profile details. → Expected: Step 1 Profile Details expanded.
3. Confirm Account Address section collapsed but still shows prior country/territory and street. → Expected: Collapsed summary retains values.
4. Minor profile edit (e.g. update phone). → Expected: Phone updated.
5. Click Continue to Account Address. → Expected: Continue enabled and works.
6. Profile collapsed; updated phone in summary; Account Address expanded. → Expected: Layout and data as specified.
7. Verify country and street address still match prior entries. → Expected: Prior address data persists.

---

### Validation of Billing Address Page [P1] (from DVW-T9)

**Objective:** Mandatory fields, dropdowns, placeholders, typeahead, APIs, errors, marketing consent, Figma parity.

**Preconditions:**

- User on Billing/Account Address during account setup; valid phone already in profile (US or non-US examples in Zephyr).

**Steps:**

1. Enter valid phone in profile (US or non-US per test data). → Expected: Accepted; no error.
2. Select No for business purchase; click Continue to Account Address. → Expected: Profile collapses; billing address enabled.
3. Verify Continue to account address disabled until requirements met (if applicable mid-flow). → Expected: Button state correct.
4. Verify mandatory markers: Country/Territory, Street, City, State/Province, Zip/Postal. → Expected: All marked mandatory.
5. Country/Territory and State/Province are dropdowns. → Expected: Dropdown behavior.
6. Until country chosen, "Enter Country or Territory" style placeholder shown. → Expected: Placeholder as specified.
7. Street Address open text with "Enter Street Address" placeholder. → Expected: Matches spec.
8. City open text with "Enter City" placeholder. → Expected: Matches spec.
9. State placeholder and mandatory/optional rules follow country (states optional for countries without states). → Expected: Rules applied.
10. Zip/Postal open text with "Enter Zip or Postal Code" placeholder. → Expected: Matches spec.
11. Partial street entry shows typeahead (~5 suggestions, scrollable). → Expected: Suggestions appear and scroll.
12. Address Doctor popup: same address validated via suggestion path. → Expected: Validation consistent with popup flow.
13. Call /getCountry and /getState; verify address data loads correctly. → Expected: APIs drive UI correctly.
14. Disconnect network; verify error toaster when verification unavailable. → Expected: "Refresh the page as we are currently unable to verify your address" (or equivalent).
15. Leave fields incomplete; verify field-level errors. → Expected: Errors map to empty/invalid fields.
16. Open Privacy Notice link. → Expected: Opens in new window.
17. Marketing consent checkbox only if customer not already opted in. → Expected: Checkbox rules correct.
18. Marketing consent visible for Trial Users. → Expected: Shown when applicable.
19. UI matches Figma (exclude Norton badge). → Expected: Font, color, alignment match.

---

### Validation of Autofill Address from Browser [P3] (from DVW-T16)

**Objective:** Browser autofill interacts correctly with Address Doctor and field population.

**Preconditions:**

- Account setup address step; browser autofill data saved (US-style example in Zephyr).

**Steps:**

1. Clear Country, Street, City, State, Zip once then empty all. → Expected: Fields empty.
2. Enter invalid UK-style address (per test data). → Expected: Data entered for negative path.
3. Click Complete account setup. → Expected: Address Doctor shows no match / 0% similarity message with Try Another.
4. Click Try Another in popup. → Expected: Popup closes; user can edit address on setup page.
5. Click Use this address on Address Doctor when applicable. → Expected: Can complete setup.
6. Open Country/Territory field again. → Expected: Dropdown list; browser may not suggest country field.
7. Select country (e.g. USA). → Expected: Country field filled.
8. Focus Street Address; pick browser autofill suggestion. → Expected: Street fills (e.g. 2A, Colorado).
9. Focus City; pick autofill. → Expected: City fills (e.g. Colorado Springs).
10. Focus State/Province; select from dropdown (e.g. Colorado). → Expected: State filled.
11. Focus Zip/Postal; pick autofill. → Expected: Zip fills (e.g. 123456).
12. Verify Complete Account Setup enabled when all required filled. → Expected: Button enabled.
13. Continue to currency / Address Doctor validation per flow. → Expected: Popup and Accept/Use this address behavior as specified.

---

### Validation of Account Address Fields [P1] (from DVW-T17)

**Objective:** Full billing/company address validation—mandatory rules, character limits, special characters, typeahead, Address Doctor variants, India/Mexico/Malaysia cases, Figma, optional state countries.

**Preconditions:**

- Account setup with address section reachable; test addresses from Zephyr (US, Canada, Brazil, UK, invalid combos).

**Steps (ordered by Zephyr index; lower-priority localization/admin bullets 38–41 omitted here—track in release checklist):**

1. No address fields entered → Continue to currency settings disabled. → Expected: Button disabled.
2. Exceed per-field character limits. → Expected: Error when length exceeded (counters: Street 40, Line 2 40, City 40, Zip 10 per Zephyr).
3. Enter Street/City/Zip before Country. → Expected: Error "Select a country/territory prior to entering your street address" under street.
4. Select country from dropdown. → Expected: Placeholder clears; prior error clears.
5. Enter valid alphanumeric street; placeholder clears. → Expected: No error; field accepts input.
6. Type street; choose typeahead suggestion. → Expected: Suggestion fills street field.
7. Enter city with alphanumeric; placeholder clears. → Expected: City accepts input.
8. Select state; verify mandatory when country has states. → Expected: State rules enforced.
9. Enter numeric zip; placeholder clears. → Expected: Zip accepted.
10. Valid Canada address path (example in Zephyr). → Expected: No popup when address valid.
11. Invalid street name variant after typeahead selection; Continue to currency. → Expected: Use this address / Accept in review popup per spec.
12. Invalid city for state-mandatory countries after typeahead. → Expected: Invalid address message with Edit this address and Accept.
13. Invalid state variant. → Expected: Same class of error and actions.
14. Invalid zip variant. → Expected: Same class of error and actions.
15. Edit this address repeatedly then valid address. → Expected: Popup clears when valid; currency section reachable.
16. Invalid city/state/zip combo; Accept on multi-match / not found scenarios. → Expected: Messages per address examples (Brazil multi-match vs India not found text in Zephyr).
17. India account: State mandatory in setup, checkout, admin change payment (per Zephyr). → Expected: State marked required where specified.
18. Complete Account Setup enabled when valid. → Expected: Button enabled.
19. Submit with all fields from typeahead only. → Expected: No Address Doctor popup.
20. Invalid zip rules: spaces, charset, hyphens, length (see Zephyr matrix). → Expected: Errors for bad patterns; valid patterns enable Complete.
21. Invalid address → Complete Account Setup → Address Doctor with Try Another / Use this address. → Expected: Popup behavior and Try Another returns to form.
22. Use this address path. → Expected: Setup can complete.
23. Navigate back; fill billing; trigger popup without typeahead selections. → Expected: Popup shown; page greyed/non-editable behind modal.
24. Popup buttons: single vs multiple suggestions; Accept disabled until selection when required; highlight outline on selection. → Expected: Button rules per suggestion count.
25. Complete account creation via Accept/Use this address. → Expected: Account created.
26. Re-enter setup; fill address again; test invalid zip edit CG6T and other zip rules. → Expected: Field-level errors per matrix.
27. Clear street, city, zip; re-enter valid US-style line. → Expected: Errors clear appropriately.
28. Street special characters from forbidden set. → Expected: Error listing forbidden characters.
29. Correct street to remove specials. → Expected: Error clears.
30. City with forbidden specials. → Expected: City error with forbidden list.
31. Correct city. → Expected: Error clears.
32. Invalid zip with specials. → Expected: "Please enter a valid zip/postal code."
33. Valid zip alphanumeric with hyphens. → Expected: Error clears; Complete enabled.
34. UI matches Figma for setup and Address Doctor dialogs. → Expected: Visual parity.
35. Select country without states. → Expected: State and zip marked optional per country rules.

---

### Validation of country/ Territory field in Company Address Page [P2] (from DVW-T32)

**Objective:** Company address country auto-filled from business info and locked.

**Preconditions:**

- Profile with business Yes; business section completed.

**Steps:**

1. Complete profile with business Yes and complete Business Information. → Expected: Continue to Account/Company Address enabled.
2. Click Continue to Company Address. → Expected: Profile collapsed; company address expanded.
3. Verify Contact and Business sections collapsed summary. → Expected: Collapsed view shown.
4. Verify Country/Territory auto-populated from business and not editable. → Expected: Field locked to business country.
5. Further field validations per DVW-182 linked cases. → Expected: Cross-reference linked tests.

---

### Integrate Fonoa with tax questions and currency list by account address country [P2] (from DVW-T78)

**Objective:** After address path: tax questions by country, VAT verify, timeout, currency dropdown, edit-from-collapsed, full completion.

**Preconditions:**

- QA shop URL in Zephyr; new TID; account setup with business Yes path.

**Steps:**

1. Navigate to URL; create TID. → Expected: Logged in.
2. Redirect to account setup. → Expected: Setup page.
3. Greyed name/email; phone entered; business purchase Yes. → Expected: Profile state correct.
4. Business: company name, size, country dropdown. → Expected: Section complete.
5. Tax question for Canada path (verbiage per Zephyr). → Expected: Correct question text.
6. Tax question for non-US (VAT/GST registered). → Expected: Wording verified.
7. Yes/No radios visible and clickable. → Expected: Interactive.
8. Yes on VAT/GST → ID required; Verify shows green tick when valid. → Expected: Validation UI.
9. Invalid VAT verify. → Expected: Toaster "Please enter a valid ID number."; Continue to company address disabled.
10. Invalid alphanumeric VAT path. → Expected: Same error class; Continue disabled.
11. Valid VAT verify. → Expected: Continue enabled; green tick.
12. Remove VAT; wait ~30s. → Expected: "Response Timeout. Please try again" (or current copy).
13. VAT question not for US region. → Expected: No VAT question for US.
14. UK registered/unregistered VAT mix. → Expected: Valid shows Verified; invalid shows error toaster.
15. Profile and Account Address collapsed with Edit CTA after progression. → Expected: Collapsed sections.
16. Continue to Currency Settings expands section with title and info message. → Expected: Currency section UX.
17. Preferred Currency placeholder "Select Currency"; Complete disabled until currency chosen. → Expected: Enablement rules.
18. Edit collapsed sections from currency step and proceed. → Expected: No blockage after edits.
19. Complete setup after currency. → Expected: Account setup completes.

---

### State/Province Field Validation Based on Country [Unspecified] (from DVW-T153)

**Objective:** US/Canada/Brazil require state; other countries optional; typeahead-without-state behavior; tax payload behavior.

**Preconditions:**

- Account creation or add payment address flows; countries per step test data.

**Steps:**

1. Start account creation or add payment address; country United States. → Expected: State/Province required; cannot proceed blank.
2. US address with state left blank. → Expected: User not allowed to proceed.
3. Repeat for Canada. → Expected: State required (mirror step 2 expectations).
4. Country Brazil. → Expected: State/Province required.
5. Country other than US, Canada, Brazil (e.g. Germany, UK, France). → Expected: State/Province optional; can proceed to currency without state.
6. Typeahead address without state for non-US/Canada/Brazil. → Expected: No red error on state; user can proceed without state.
7. Place order with address outside US/Canada/Brazil; state code omitted in tax payload. → Expected: Tax still calculated from postal code per Salesforce/Fonoa rules in Zephyr.

---

## Key Validations

- Mandatory address field set, placeholders, and country-driven state optional/mandatory rules.
- Address Doctor: suggestions, no-match, multi-suggestion Accept/Use this address, Try Another, grey overlay.
- Persistence when editing profile (DVW-T3) and browser autofill compatibility (DVW-T16).
- Company address country locked to business country (DVW-T32).
- Fonoa/tax and currency section gating after address (DVW-T78).
- Regional state requirements and checkout/tax implications (DVW-T153).

## Common Preconditions

- Account setup reachable after TID login (new or partial account per case).
- Network available unless testing offline toaster paths.
- Browser autofill data configured for DVW-T16 when running that case.
- Valid test addresses per country in Zephyr for Address Doctor positive/negative paths.
