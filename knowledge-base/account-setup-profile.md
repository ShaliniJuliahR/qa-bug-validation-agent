# Feature: Account Setup - Profile & Sign-in

## Overview

Profile details step of account creation including TID sign-in flows, phone number validation, and redirection logic for new and existing CDH users.

## Test Flows

### Account Creation Page's Profile Step [P1] (from DVW-T1)

**Objective:** Verify landing on Account Creation after TID setup; profile fields, business question default, and collapsed account address until contact info is complete.

**Preconditions:**

- User completed TID setup and is on the Account Creation page.
- Test data (example): Username poonkuzhali_velmurugan+r1@trimble.com.test-google-a.com, Password Trimble@123.

**Steps:**

1. Verify you land on the Account Creation page after completing TID setup. → Expected: The Account Creation page is displayed.
2. Check that the Profile Details section is displayed. → Expected: The Profile Details section is visible.
3. Confirm Firstname, Lastname, and Email are pre-populated from TID and non-editable. → Expected: Fields correctly pre-populated and non-editable.
4. Verify phone number input exists and is mandatory. → Expected: Phone field present and marked mandatory.
5. Verify business purchase question default. → Expected: "Are you making this purchase for your business or organization?" shown, defaulted to No.
6. Verify Account address under Contact Information when mandatory fields incomplete and business = No. → Expected: Account address step collapsed and non-editable.
7. Compare UI to Figma (excluding Norton badge). → Expected: Font, color, alignment match mockup (Figma link in Zephyr test data).

---

### Validation of Fields in Profile Details Section Page [P1] (from DVW-T4)

**Objective:** Verify Firstname, Lastname, Email for US and international customers with special characters, numeric, HTML-like strings, and long TID inputs; network failure and refresh behavior.

**Preconditions:**

- TID accounts with varied profile data per Zephyr objective (special chars, long email, names with < > patterns, etc.), Password Trimble@123.

**Steps:**

1. Login to TID with given test data. → Expected: TID account created/logged in as specified.
2. Verify Profile Details section is displayed. → Expected: Profile details step shown.
3. Confirm Firstname and Lastname pre-populated with TID data including special characters. → Expected: Fields show TID values, non-editable.
4. Ensure Email is populated correctly. → Expected: Email field pre-populated.
5. If first name, last name, or email not fetched from Trimble Cloud. → Expected: Profile section disabled; toaster: "Refresh the page as we are currently unable to access your profile details".
6. If step 5 blocks editing, refresh page. → Expected: After refresh, name and email auto-populate and other fields enabled.
7. Logout and login TID with given test data. → Expected: Login succeeds.
8. Check Profile Details section again. → Expected: Section displayed.
9. Confirm Firstname and Lastname with numeric characters. → Expected: Shown as plain text, non-editable.
10. Logout and login again with TID test data. → Expected: Login succeeds.
11. Check Profile Details section. → Expected: Section displayed.
12. Verify fields show full long inputs without truncation; long text stays within accordion without overlap. → Expected: Full input visible; no overlap with other fields.
13. Logout and login TID with test data containing angle-bracket patterns in names (e.g. div/h1 text in TID). → Expected: TID login succeeds.
14. Verify Profile Details shows First and Last Name as stored without mangling angle brackets. → Expected: Display matches TID; no incorrect entity substitution.
15. Confirm First and Last Name visible as expected after the above login. → Expected: Names display correctly (verify visually per Zephyr).

---

### Phone Number Validation for US User [P1] (from DVW-T7)

**Objective:** Validate US phone field formatting, errors, Continue to Account Address enablement, and backspace behavior.

**Preconditions:**

- User on Profile Details step of Account Creation.

**Steps:**

1. Enter valid US phone (10 digits). → Expected: No error; field auto-formats (country code in parentheses, hyphens).
2. Select No for business purchase question. → Expected: No selected by default / confirmed.
3. Verify Continue to Account Address enabled when valid. → Expected: Button enabled.
4. Place cursor mid-number and backspace. → Expected: Digit deleted; cursor stays in position.
5. Remove phone number. → Expected: Number cleared; Continue disabled.
6. Enter invalid US phone with more than 10 digits. → Expected: Error "Phone number must be valid and contain 10 digits."; Continue disabled.
7. Enter invalid US phone fewer than 10 digits (e.g. 1234). → Expected: Error "Enter a valid phone number."; Continue disabled.
8. Select No for business question (repeat path if needed). → Expected: No selected.
9. Enter valid phone again. → Expected: Continue to Account Address enabled.

---

### TID Sign in - New CDH User [P1] (from DVW-T8)

**Objective:** New user without CDH (primary or secondary owner) is sent to Account Creation Profile; after completion, lands on Carts; new TID signup repeats the same CDH check and completion path.

**Preconditions:**

- TID user with no CDH account in primary or secondary owner roles (example: poonkuzhali_velmurugan+r1@trimble.com.test-google-a.com / Trimble@123).

**Steps:**

1. Login to TID. → Expected: Successful login.
2. Backend checks CDH for primary/secondary owner accounts. → Expected: If none, user navigated to Account Creation.
3. Verify redirect to Account Creation Profile details step. → Expected: Profile Details section displayed.
4. Complete the account creation steps. → Expected: Steps complete without errors.
5. Verify redirect after completion. → Expected: Redirected to Carts Page.
6. Logout. → Expected: User logged out; returns to TID Login Page.
7. Initiate TID account creation via "Create a Trimble ID" on TID page. → Expected: TID signup flow starts.
8. Complete TID account creation. → Expected: TID account creation finishes successfully.
9. Login with the created TID credentials. → Expected: Successful login.
10. Backend CDH check for primary/secondary roles. → Expected: If still no CDH account, user navigated to Account Creation.
11. Verify Account Creation page shows Profile details section. → Expected: Profile step displayed.
12. Complete the account creation steps again. → Expected: Completes without errors.
13. Verify redirect to Carts Page. → Expected: Carts Page displayed.

---

### TID sign in - Already has a CDH account [P1] (from DVW-T14)

**Objective:** User with existing CDH (primary or secondary owner) skips Account Creation and lands on Landing/Cart; refresh and long wait + refresh keep user on Landing.

**Preconditions:**

- TID user who already completed account creation / has CDH (example: farhiena_sa+r1@trimble.com.test-google-a.com / Trimble@123).

**Steps:**

1. Login to TID as user with existing CDH. → Expected: Successful login.
2. Backend CDH validation. → Expected: Account exists in primary or secondary role → navigate to Landing Page.
3. Verify not sent to Account Creation. → Expected: Direct to Landing (future Cart), not Account Creation.
4. Refresh browser (F5 or button). → Expected: Page loads successfully.
5. After refresh, verify same page. → Expected: Still on Landing Page.
6. Wait ~1 hour and refresh again. → Expected: Page loads successfully.
7. After refresh, verify still on Landing. → Expected: Remains on Landing Page.

---

### Phone Number Validation for Non - US User [P2] (from DVW-T15)

**Objective:** Non-US phone: country code dropdown, validation length/rules, interaction with business Yes/No and Continue button.

**Preconditions:**

- User on Profile Details; accounts endpoint phone UI with country prefix dropdown.

**Steps:**

1. Verify phone entry shows dropdown with two-letter country codes and scrollable list. → Expected: Country code dropdown behaves as specified.
2. Locate business purchase question; verify Yes/No radios. → Expected: Options visible and functional.
3. Verify Continue to Account Address enabled when requirements met. → Expected: Button enabled.
4. Remove valid phone number. → Expected: Phone cleared; Continue to Billing/Account Address disabled.
5. Enter invalid non-US phone over 13 digits. → Expected: Error "Enter a valid phone number."
6. Enter invalid non-US phone under 5 digits (e.g. +(12)345). → Expected: Message that number must be valid with 5–13 digits; special chars not allowed; with valid number and business No, Continue works.
7. Click Yes for business purchase. → Expected: Business information shown; Continue to Account Address disabled until business fields satisfied.
8. Verify Continue disabled when appropriate. → Expected: Button disabled per rules.

---

### User with Existing CDH Account Logging in via TID - Redirection to Landing Page [P2] (from DVW-T18)

**Objective:** Same as T14 subset: existing CDH → Cart/Landing; closed tab reopened shows Landing.

**Preconditions:**

- TID with CDH (TCP_ACCOUNT_OWNER or TCP_SECONDARY_ACCOUNT_OWNER or both).

**Steps:**

1. Login to TID as user who completed account creation. → Expected: Logged in with associated CDH.
2. Backend CDH check. → Expected: If any/both roles present → Landing Page.
3. Verify redirect to Cart Page. → Expected: User on Cart/Landing, not Account Creation.
4. Close browser tab. → Expected: Tab closed.
5. Reopen closed tab (e.g. Ctrl+Shift+T Chrome). → Expected: Landing Page displayed.
6. Verify redirect to Cart Page again if repeated. → Expected: Direct to Cart Page.

---

### New User TID Login - Account Creation Navigation [P1] (from DVW-T19)

**Objective:** No CDH → Account Creation Profile; full path including business Yes, VAT/GST, company fields, API country/state codes, collapsed sections, verbiage, and completion to Cart.

**Preconditions:**

- New TID user without CDH (example credentials in Zephyr).

**Steps:**

1. Login to TID. → Expected: Successful login.
2. Backend CDH check. → Expected: No account for both roles → Account Creation.
3. Verify Account Creation with Profile Details. → Expected: Profile section displayed.
4. Close tab; reopen. → Expected: Account Creation page still shown.
5. Complete account creation process. → Expected: Completes without errors.
6. Auto-populated name/email; enter phone; select Yes for business purchase. → Expected: Contact fields as specified; business path enabled.
7. In business info: company name, size, country dropdown. → Expected: Fields completed.
8. Verify company size optional. → Expected: Optional behavior confirmed.
9. Tax question for UK: "Is your business or organization registered for VAT/GST?" → Expected: Correct wording for UK.
10. Tax question for non-UK: "Is your business or organization tax exempt?" → Expected: Correct wording.
11. Verify Yes/No radios visible and clickable for tax question. → Expected: UI interactive.
12. Select Yes on tax question. → Expected: Only tax ID field shown; VAT certificate option removed; Continue to Account Address disabled until verified.
13. Invalid VAT (alphanumeric) verify. → Expected: Toaster "Please enter a valid ID number."; Continue disabled.
14. Valid VAT verify. → Expected: Verified; "Continue to company address" enabled (wording per build).
15. UK VAT numeric-only rule. → Expected: UK VAT digits validated per spec.
16. US and non-US VAT alphanumeric rule. → Expected: Alphanumeric allowed where specified.
17. Profile and Account Address collapsed with Edit CTA. → Expected: Collapsed summary with Edit.
18. Backend: Create/Get Account responses use country/state codes (e.g. US, UK, DE, AU). → Expected: API returns codes not full names.
19. Verify redirect to Carts after completion. → Expected: Cart Page.
20. Verify account setup verbiage vs latest Figma (sheet link in Zephyr). → Expected: Copy matches.
21. UI consistency mobile/laptop/desktop. → Expected: Consistent layouts.

---

### User with Existing Account - Redirects to Landing Screen Post Successful TID Login [P2] (from DVW-T20)

**Objective:** User who already completed account setup sees Cart/Landing after TID login (no blank/error screen).

**Preconditions:**

- TID user who already completed account creation.

**Steps:**

1. Log in to TID as user who already completed account creation. → Expected: TID login successful.
2. Verify redirect after login. → Expected: Redirected to Carts Page.

---

### Account Creation & Sign-in outside Shopping Cart Page [P2] (from DVW-T33)

**Objective:** From PDP or cart slide-out: sign-in, incomplete account → Profile; complete setup → return to PDP or registered cart; repeat login returns correctly.

**Preconditions:**

- On PDP Page or Cart Slide Out Menu (label INFEASIBLE in Zephyr—confirm environment support).

**Steps:**

1. Verify on PDP or Cart Slide Out. → Expected: Correct entry surface shown.
2. Click Sign In. → Expected: TID Login Page.
3. Verify TID Login displayed. → Expected: On TID Login Page.
4. Attempt login as user without completed Account Creation. → Expected: Navigated to Account Creation (Profile).
5. Verify Profile Page. → Expected: Profile Details displayed.
6. Click Trimble logo. → Expected: Navigation per spec (e.g. www.trimble.com).
7. Complete Account Creation; click Complete Account Setup. → Expected: Button enabled; return to sign-in origin flow.
8. Verify return to PDP or Registered Cart. → Expected: Correct post-setup destination.
9. Verify again on PDP or slide-out. → Expected: Entry context correct.
10. Sign In again. → Expected: TID Login.
11. Verify TID Login. → Expected: TID page shown.
12. Login as user who completed Account Creation. → Expected: Sign-in completes; navigated to Shopping Cart.
13. Verify return to page where login started (PDP or slide-out). → Expected: Correct return navigation.

---

### Account Sign-in & Sign Up from Shopping Cart Page [P1] (from DVW-T39)

**Objective:** Anonymous cart → sign in to checkout → incomplete account → Profile → Complete Account Setup → Registered cart; logout/login variations.

**Preconditions:**

- Anonymous Shopping Cart Page.

**Steps:**

1. Verify Anonymous Shopping Cart. → Expected: Page displayed.
2. Click "Sign in to checkout" in Order Summary. → Expected: TID Login Page.
3. Verify TID Login Page. → Expected: Correct page.
4. Login as user who completed Account Creation. → Expected: Back to Registered Shopping Cart.
5. Logout; verify Anonymous Cart. → Expected: Anonymous cart shown.
6. Click "Sign in to checkout" again. → Expected: TID Login.
7. Verify TID Login. → Expected: TID page.
8. Login as user without completed Account Creation. → Expected: Account Creation (Profile).
9. Verify Profile Page. → Expected: Profile displayed.
10. Complete Account Creation; click Complete Account Setup. → Expected: Enabled; navigates to Shopping Cart when done.
11. Verify Registered Shopping Cart after completion. → Expected: On registered cart.

---

### Implementation of Phone Number with Country Code [P2] (from DVW-T79)

**Objective:** Profile phone with country code dropdown, validation, placeholders, mandatory country code, Figma alignment.

**Preconditions:**

- Environment URL in Zephyr (e.g. Azure static apps link); TID account created.

**Steps:**

1. Navigate to URL and create TID account. → Expected: Account created; logged in.
2. Verify redirect to account setup. → Expected: Account setup page.
3. Verify Country Code field in Profile Details. → Expected: Field visible.
4. Select country code; verify highlight. → Expected: Selected code highlighted (e.g. blue).
5. Verify First/Last/Email greyed auto-filled; country code and phone mandatory. → Expected: As specified.
6. Verify placeholders (country code and numeric phone per spec). → Expected: Placeholders match design.
7. Enter phone without selecting country code. → Expected: Error; dropdown behavior as specified.
8. Verify Continue disabled without country code. → Expected: Button disabled.
9. Enter invalid phone e.g. 000000000000. → Expected: Error toaster.
10. Verify phone accepts 4–15 digits numeric. → Expected: Length rule applied.
11. Select No for business purchase question. → Expected: No selected.
12. Verify Continue enabled when valid. → Expected: Button enabled.
13. Backspace mid-number. → Expected: Cursor position preserved.
14. Remove phone number. → Expected: Continue disabled.
15. Verify UI matches Figma. → Expected: Layout matches design.

---

## Key Validations

- Profile: TID-sourced name/email read-only when loaded; phone and business question gating for Continue.
- CDH routing: no CDH → Account Creation Profile; existing CDH → skip to Cart/Landing (including refresh/tab reopen).
- US vs non-US phone rules, formatting, and error copy.
- Cart/PDP entry flows return users to the correct surface after account setup or completed login.

## Common Preconditions

- Valid TID credentials for new vs existing CDH scenarios (see Zephyr test data).
- Target environment (QA/Netlify/etc.) reachable; Trimble Cloud profile fetch working unless testing failure paths.
- For DVW-T4/DVW-T19: prepared TID accounts with special/long/HTML-like profile data as listed in Zephyr objectives.
