# Feature: Accounts Service API

## Overview

API tests for account creation, retrieval (GetAccounts), address validation, address editing, DVR-to-DVW provisioning, and default account management.

## Test Flows

### GetAccounts API - Error cases [P1] (from DVW-T2)

**Objective:** Validate GET `/accounts` error handling for bad URL, missing auth, invalid/expired tokens, and server errors.

**Preconditions:**

- Base URL: `https://api.trimble.com/account-experience` (or environment equivalent); endpoint `/accounts`, method GET.

**Steps:**

1. GET invalid base URL (`https://api.trimble.com/invalid`) with valid Bearer token → Expected: 404.
2. GET valid `/accounts` without token → Expected: 401 Unauthorized.
3. GET `/accounts` with invalid token → Expected: appropriate error for invalid token.
4. GET `/accounts` with expired token (modify expiration on valid token) → Expected: 401 or appropriate invalid-token handling.
5. Simulate server error (e.g. DB) and hit endpoint → Expected: 500 with meaningful message.

---

### Verify Create Account API [P1] (from DVW-T10)

**Objective:** Verify Create Account creates records in CDH and Commerce Tools (CT) for Organization and Individual accounts, mandatory fields, tax exemption variants, and country/currency/legal-entity alignment.

**Preconditions:**

- Valid user access token; reference Create Account API docs for payload shape.
- Legal-entity / country–currency mapping sheet for multi-region cases.

**Steps:**

1. POST Create Account (Organization) with full `businessInfo`, mandatory address (line1, city, state, stateName, country, countryName, zip), optional line2, `currency` → Expected: 200; response includes id, name from company, role ACCOUNT_OWNER, address, sourceSystems DVW, currency.
2. POST Create Account (Individual) with address + currency, no businessInfo → Expected: 200; name from TID first/last name.
3. Verify Individual account name comes from TID first and last name → Expected: matches TID.
4. Create multiple Individual/Organization accounts with same token → Expected: allowed.
5. Organization without `businessInfo` → Expected: 400; Business Info required.
6. Individual without address block → Expected: 400; Address required.
7. On success, verify accounts exist in CDH and CT → Expected: both created.
8. Verify CDH: source DVW, reference = CT customer id; type Organization vs Individual → Expected: correct mapping.
9. CT getCustomer by ID: CDH account number, user UUID in externalId, email pattern `cdhAccNo_TIDemail` → Expected: as specified.
10. Organization with `taxExemptionType` TAX_EXEMPTION (empty VAT id example) → Expected: 200; account created.
11. Organization VAT_EXEMPTION with VAT id → Expected: 200.
12. Organization NO_EXEMPTION → Expected: 200.
13. Create accounts with UK+GBP, US+USD, AU+AUD, EU+EUR, etc. per legal-entity sheet → Expected: 200 when country/code/currency valid.
14. (Section divider / exemption matrix in source test.)
15. Individual without exemption → Expected: GetAccounts returns tax exemption NO_EXEMPTION.
16. Organization without tax exemption → Expected: GetAccounts returns NO_EXEMPTION.
17. Organization with VAT id (VAT exempt) → Expected: GetAccounts returns VAT_EXEMPTED.

---

### Verify GetAccounts API (Happy paths) [P1] (from DVW-T11)

**Objective:** Validate GET `/accounts` success paths: payload shape, role filtering, sourceSystem filter, address/phone behavior, and Create Account mandatory-field errors.

**Preconditions:**

- TID users with DX-provisioned products; multi-account and multi-role scenarios as needed.

**Steps:**

1. GET `/accounts` with Bearer token → Expected: 200; JSON with id, name, role, address, sourceSystems, currency, phoneNumber where applicable.
2. User with multiple DX accounts/roles; GET without role param → Expected: array of accounts; highest-precedence role per account (priority: TCP_SECONDARY_ACCOUNT_OWNER, TCP_COMPANY_ADMIN, TCP_TEAM_MANAGER, TCP_END_USER, TCP_ACCOUNT_OWNER — note: verify product-specific precedence in env).
3. GET with role query param matching user’s role → Expected: accounts filtered to that role.
4. User with no DX-provisioned products → Expected: 200; empty array.
5. Create Account missing mandatory blocks (User, Account Type, BusinessInfo for Org, Address except line2) → Expected: validation errors.
6. Address in response matches creation address → Expected: match.
7. Address line2 optional → Expected: empty string if omitted at creation.
8. Edit address via Edit API → Expected: GetAccounts and CDH show updated address.
9. Edit API: country not updatable; other fields updatable → Expected: country unchanged; others updated.
10. GET with `sourceSystem=DVW` for user with DVW and TDX → Expected: only DVW accounts.
11. GET with `sourceSystem=TDX` → Expected: only TDX accounts.
12. GET with Trial Service source param → Expected: only Trial Service accounts.
13. Token with EP/other sources + DVW/TDX/Trial; filter DVW/TDX/Trial → Expected: 200; null/empty array per rules in test.
14. Same user without sourceSystem param → Expected: all accounts; non-DVW/TDX/Trial may show empty address/source.
15. Verify phoneNumber with DVW source → Expected: populated in response.
16. Verify phoneNumber with DVW and TDX → Expected: populated.

---

### Verify Create Account API - Error handling [P1] (from DVW-T12)

**Objective:** Validate Create Account rejects invalid payloads, enums, phone/address/legal-entity mismatches, and auth failures.

**Preconditions:**

- Valid token for positive paths; ability to omit or corrupt token for negative cases.

**Steps:**

1. POST with payload, no token (Org/Individual) → Expected: 401 JWT not present.
2. POST invalid/expired token → Expected: 401 JWT not present / invalid.
3. Missing user block → Expected: 400 User required.
4. Empty/missing accountType → Expected: 400 account type validation.
5. Empty phone → Expected: 400 User.PhoneNumber rules.
6. Empty payload `{}` → Expected: 400 User and Address required.
7. Organization without BusinessInfo → Expected: 400 BusinessInfo/address errors.
8. Org with missing/empty address fields (line1, city, state, names, country, zip) → Expected: 400 address validation.
9. Org/Individual without address block → Expected: 400 Address required.
10. Individual payload including businessInfo → Expected: 400 Business Info must be empty.
11. Invalid accountType string → Expected: 400 JSON conversion error on accountType.
12. Phone length under 7 or over 15 → Expected: 400 phone validation.
13. Org VAT_EXEMPTION without VAT id where required → Expected: 400.
14. Invalid taxExemptionType → Expected: 400 enum conversion.
15. Company name over 200 characters → Expected: 400 length error.
16. Address field empty (non-line2) → Expected: error per validation.
17. Country/currency mismatch vs legal entity (matrix cases) → Expected: 400 Legal entity not found for country and currency.
18. Empty currency → Expected: 400 currency / legal entity errors.
19. Valid country name + wrong country code → Expected: 400 legal entity not found.

---

### API Test: Address Validate API - Address Doctor [P3] (from DVW-T56)

**Objective:** Validate address suggestion API for US/UK inputs, disambiguation, invalid inputs, and auth/account errors.

**Preconditions:**

- Valid access token and account id for happy paths.

**Steps:**

1. Valid US address in body + valid token + account id → Expected: 200; suggestions with line1–2, city, state, stateCode, county, country, countryCode, postalCode.
2. Valid UK address → Expected: 200; appropriate UK suggestion shape.
3. Misspelled country/city/state → Expected: corrected suggestion where applicable.
4. Invalid US freeform → Expected: corrected suggestion with postal code.
5. Invalid non-US → Expected: suggestions for correctable non-US address.
6. Ambiguous address (e.g. Washington state vs DC) → Expected: most probable match from other components.
7. Expired/invalid token → Expected: 401 Invalid JWT.
8. Invalid account id → Expected: 403 Forbidden.
9. Invalid address (wrong field usage) → Expected: `{ suggestions: [] }`.
10. Non-US/non-UK country → Expected: empty suggestions.

---

### API Test: Address Edit PATCH API [P2] (from DVW-T59)

**Objective:** Validate PATCH edit billing address for DVW accounts, role behavior, validation, phone field rules, and non-DVW restrictions.

**Preconditions:**

- DVW source account; token with appropriate role; `actionType` e.g. `BillToAddressEdit`.

**Steps:**

1. Valid payload + account id + token (DVW user) → Expected: 200; updated address in response.
2. Without/invalid account id → Expected: 403.
3. User with multiple roles: update as primary of own account → Expected: only that primary address updated.
4. Primary + product user linked → Expected: address updates per rules.
5. SAO + linked accounts → Expected: linked accounts get updated per rule.
6. Primary token + account id affecting all linked roles → Expected: all three roles’ addresses updated per test.
7. Empty payload / missing mandatory fields (except line2) → Expected: 400.
8. Wrong `actionType` (e.g. `DVWAccountProvision` with edit payload) → Expected: 404 unable to provision / wrong action handling.
9. Invalid actionType value → Expected: 400 conversion error.
10. `BillToAddressEdit` case-insensitive → Expected: 200.
11. Special characters/long street → Expected: 200 if accepted.
12. Same address repeated → Expected: 200.
13. Non-DVW source (tdx, ebs, ep, trial) → Expected: 404/400 not a DVW account.
14. DVW BillTo false edge account → Expected: 404 not DVW / cannot update.
15. TDX-then-DVW provisioned token+id combination → Expected: error; no update.
16. PATCH without `phone` where required by API version → Expected: 400 (per test) OR optional path in step 19 — follow environment contract.
17. PATCH with `phone` → Expected: 200; `phoneNumber` in response.
18. DVR-sourced account → Expected: 400; cannot update.
19. PATCH without phone (alternate step) → Expected: optional success per test data.
20. Valid DVW update → Expected: CDH and Aria reflect address.
21. TDX origin with both TDX+DVW source systems → Expected: 400 update only for DVW accounts message.

---

### DVW provisioning account API cases [P2] (from DVW-T66)

**Objective:** Validate PATCH provisioning for DVR (TDX) accounts to DVW commerce, error paths, and post-provision flows.

**Preconditions:**

- DVR/TDX accounts with CDH `isBillTo` true/false; UK vs US addresses per cases; matrix spreadsheet referenced in Zephyr.

**Steps:**

1. PATCH `/accounts` with token + id of DVR (TDX) account with `isBillTo=true` in CDH → Expected: 200; id, name, address, sourceSystems include DVW+TDX.
2. CDH GET after success → Expected: copy of bill-to address with source DVW; reference id matches CT.
3. PATCH for DVW account (already commerce) → Expected: 422.
4. PATCH for DVR with `isBillTo=false` → Expected: 422.
5. Invalid actionType in provision payload → Expected: 400.
6. GET commerce `/accounts` for provisioned account → Expected: sourceSystems TDX and DVW.
7. Add payment method to provisioned user → Expected: success.
8. Create/get/update cart, legal entity, payments, POST order → Expected: same as native DVW.
9. UI login → Expected: redirect to checkout as applicable.
10. Product user token + owner account id → Expected: 401.
11. Primary owner token → Expected: 200 provision response.
12. SAO token → Expected: 200.
13. Invalid token → Expected: 401.
14. Valid token, invalid account id → Expected: 403.
15. User with both AO and product roles → Expected: 200 provision.
16. Multiple isBillTo in CDH → Expected: first copied with DVW source.
17. DVR US address → Expected: 422 UK-only provisioning rule.
18. Execute remaining matrix rows per linked spreadsheet → Expected: per matrix.
19–20. DVR as SAO/AO to DVW without provisioning → Expected: cannot perform DVW actions until provisioned.
21. Address country vs currency mismatch (e.g. UK + USD) → Expected: 422.

---

### APIs for Storing, Fetching Default Account [P3] (from DVW-T105)

**Objective:** Validate `isDefaultAccount` on GetAccounts, SetDefaultAccount API, and multi-role account visibility.

**Preconditions:**

- User with ability to create Individual account and call SetDefaultAccount with account id header.

**Steps:**

1. Create Individual account (valid token, address, currency GBP example) → Expected: 200; response includes `isDefaultAccount: false`, `originalSystem`, tax fields.
2. GET accounts before default set → Expected: `isDefaultAccount` false until Set Default called.
3. Set default via SetDefaultAccount with header account id → Expected: 200 `{ status: success }`.
4. Multi-account setup: AO, SAO, product user, company admin mapped to one commerce user; GET accounts → Expected: all mapped accounts listed.
5. Set another account default → Expected: 200; that account `isDefaultAccount` true in subsequent GET.
6. Sign out Postman, re-auth, GET accounts → Expected: previous default remains true (not cleared).

---

## Key Validations

- GET `/accounts` auth, role filtering, and `sourceSystem` filtering behave per contract; empty array when no DX accounts.
- Create Account enforces type-specific rules (Org needs BusinessInfo; Individual must not send businessInfo); CDH/CT alignment and tax exemption fields on read.
- Address validate returns structured suggestions or empty arrays; edit PATCH restricted to DVW bill-to scenarios; provisioning only for eligible DVR accounts.
- Default account flag survives re-authentication after explicit set.

## Common Preconditions

- Trimble identity (TID) access token and known `x-account-id` / CDH account id for the environment under test.
- Account-experience and commerce API base URLs for target tier (dev/qa/stg).
- Test users covering AO, SAO, product user, and multi-source (DVW/TDX) where flows require it.
