# Feature: Notifications & Dunning

## Overview

Email notifications for orders, account management, subscription reminders, D-country holds, dunning failure/revoke scenarios for yearly and monthly subscriptions, and EEUC pre-expiration notifications.

## Test Flows

### Account Management Email Notifications [P2] (from DVW-T101)

**Objective:** Address and payment update emails; first-time add behaviors; SAO update routing.

**Preconditions:**

- Commerce and AXP access; secondary owner scenario data per Zephyr.

**Steps:**

1. (Section) Customer updates address. → Expected: Header.
2. First account creation: no address-update email. → Expected: No address notification on first signup.
3. After account creation, each commerce address update sends email matching template doc. → Expected: Email generated and matches document.
4. Manage account button in email → AXP /settings. → Expected: Opens settings with updated address/payment.
5. (Section) Customer updates payment. → Expected: Header.
6. First time adding payment method: no email. → Expected: No notification on first add.
7. Each subsequent payment change (PayPal↔CC permutations) sends email per template doc. → Expected: Email matches document.
8. Manage account from payment email → /settings. → Expected: Correct navigation.
9. (Section) SAO updates address/payment. → Expected: Header.
10. SAO updates Abel's account address/payment while logged as Adam. → Expected: Email to account owner and signed-in user per rules (see Zephyr narrative).

---

### Order Email Notifications [P2] (from DVW-T103)

**Objective:** First-order email sequence and template compliance; currency; payment display; VAT variants; secondary role email routing.

**Preconditions:**

- Inbox access for purchaser.

**Steps:**

1. Place first order with one or more products; verify success and emails triggered. → Expected: Order succeeds; notifications fire.
2. Inbox order: Order Confirmation, then Payment Confirmation, then Account Statement. → Expected: Sequence correct.
3. Order confirmation email vs Google Doc template (link in Zephyr). → Expected: Matches template.
4. Payment confirmation email vs template doc. → Expected: Matches template.
5. Account statement email vs template doc. → Expected: Matches template.
6. Switzerland TEBV order: statement and AXP invoice show VAT CHE-167.888.242 MWST. → Expected: VAT string displayed.
7. Liechtenstein TEBV: same VAT as CH reference. → Expected: VAT CHE-167.888.242 MWST.
8. Netherlands TEBV: VAT NL003257447B01. → Expected: Displayed on statement and invoice.
9. UK non-TEBV: entity VAT on statement and invoice. → Expected: Entity VAT shown.
10. Currency in all emails matches order currency. → Expected: Currency consistent.
11. Order number displayed correctly. → Expected: Matches order.
12. Line items, subtotal, tax, total with correct decimals. → Expected: Totals accurate.
13. Payment method: masked card or PayPal label per template. → Expected: Display matches doc.
14. Order from secondary account owner role: emails to logged-in user only. → Expected: Routing per Zephyr.

---

### D-country hold & Email Notification [P2] (from DVW-T104)

**Objective:** D-country check in order saga; DCOUNTRYHOLD status; email on hold; EEUC link behavior; embargo overlap.

**Preconditions:**

- D-country list spreadsheet access; DataDog access optional.

**Steps:**

1. (Section) D-country pass. → Expected: Header.
2. Precondition: account without D-country (e.g., USD, UK). → Expected: Account ready.
3. First order non–D-country; verify DataDog D-country check false; order succeeds. → Expected: IsDCountry false in logs; confirmation with View Orders.
4. (Section) D-country hold prerequisite. → Expected: Header with D-country list link.
5. Place order for D-country account; GetOrders; DataDog. → Expected: Status DCOUNTRYHOLD; D-country true in logs; order not sent to processing queue.
6. DCOUNTRYHOLD: compliance check should not run. → Expected: No compliance for DCOUNTRYHOLD-only path.
7. Country both D-country and Embargo. → Expected: D-country check true; still DCOUNTRYHOLD; no compliance processing.
8. When order on hold, email to account owner per template. → Expected: Email generated (see attachment reference in Zephyr).
9. SAO places D-country order: hold + email to logged-in owner email. → Expected: Same hold behavior; email to owner email.
10. EEUC form link opens in new tab. → Expected: New tab behavior.
11. Manage account from email → AXP settings endpoint. → Expected: Correct redirect.

---

### Subscription Adjustment Email Notifications [P2] (from DVW-T110)

**Objective:** Emails for add seats, upgrade, downgrade schedule and completion, cancel — with template docs and Order ID / Account ID.

**Preconditions:**

- Account with SKP-GO, SKP-PRO, SKP-STUDIO purchased in commerce.

**Steps:**

1. (Prerequisite) Account with three SKUs purchased. → Expected: Setup complete.
2. AXP: add seats on SKP-GO; checkout; place order. → Expected: Add-seats template email; Account ID and Order ID present.
3. Verify also order subscription + order invoice mails for add seats. → Expected: Additional mails with IDs per template links.
4. AXP: upgrade SKP-GO → SKP-PRO; place order. → Expected: Upgrade template email with IDs.
5. Verify order subscription + order invoice mails for upgrade. → Expected: Mails per linked docs.
6. AXP: downgrade SKP-PRO → SKP-GO; schedule downgrade. → Expected: Downgrade scheduled mail per template.
7. Advance Aria VC to end of term for downgrade processing. → Expected: Downgrade processed mail + order invoice per templates.
8. AXP: cancel SKP-STUDIO with reason. → Expected: Cancel subscription mail per template.

---

### Yearly Dunning Failure Scenario & Notification Email + Dunning to Paid [P1] (from DVW-T112)

**Objective:** Yearly dunning when PayPal insufficient / billing agreement cancelled or CC renewal dunning; Aria steps; emails; paid recovery.

**Preconditions:**

- PayPal dunning account or CC renewal scenario.

**Steps:**

1. Order enters dunning (PayPal insufficient / agreement cancelled). → Expected: Dunning status; collection group true in Aria; no collection yet.
2. GetOrders invoice status INVOICE_UNPAID_SENT. → Expected: Status present.
3. SF subscriptions/entitlements/assets created; invoice Unpaid. → Expected: Unpaid invoice state.
4. getPayments collectionFlag true in dunning. → Expected: API shows collection true.
5. CC renewal also enters dunning. → Expected: Verified for CC renewals.
6. Day 1: payment update email. → Expected: Notification sent.
7. Day 6: second collection attempt; email; Aria dunning Step 2. → Expected: Step 2 state.
8. Day 12: third attempt; email; Aria dunning Step 3. → Expected: Step 3 state.
9. Update payment (PayPal/CC combinations). → Expected: Invoice becomes Paid; MIDs correct for CC updates.

---

### Dunning Revoke (Yearly & Monthly)- with Aria, DB, AXP, EMS, Salesforce Validation [P1] (from DVW-T122)

**Objective:** Revocation after dunning for first order, subsequent, add seats, upgrade, mixed monthly/yearly, double dunning timelines — with negative SF orders, NOT_PAID GetOrders, AXP subscription removal.

**Preconditions:**

- Dunning PayPal account; SKUs SKP-PRO, SKP-STUDIO, SKP-STDO, SKP-GO per scenario; Aria stage control.

**Steps:**

*(Full case has 80 indexed steps; below preserves step order with concise wording. See DVW-T122 in Zephyr for exact dates/SKU variants.)*

1. (Section) First order dunning revoke. → Expected: Header.
2. First order with assign license using dunning PayPal. → Expected: Order placed; licenses assigned.
3. Yearly: GetOrders Dunning; wait 14d or set Aria dunning stage 4. → Expected: Dunning state; revoke path available.
4. Monthly: GetOrders Dunning; wait 7d or stage 4. → Expected: Shorter dunning window behavior.
5. Subscription revoked; plan balance 0. → Expected: Balance zero in Plans.
6. AXP subscription Cancelled. → Expected: Status cancelled.
7. Transactions: write-off record. → Expected: Write-off visible.
8. AXP: no subscriptions listed. → Expected: Subscriptions empty.
9. GetOrders order status NOT_PAID. → Expected: API status.
10. SF: new negative order. → Expected: Negative order created.
11. SF negative order: subtotal/total/tax negative. → Expected: Negative amounts.
12. Negative order payment status Unpaid. → Expected: Unpaid on negative order.
13. (Section) Subsequent order. → Expected: Header.
14. First order SKP-PRO with license. → Expected: Success.
15. Subsequent SKP-STDO with dunning PayPal; subsequent in dunning. → Expected: Subsequent enters dunning.
16. After 14d: only STDO revoked; PRO remains. → Expected: Partial revocation per scenario.
17. (Section) Add seats — all assigned (3 seats). → Expected: Header.
18. First order SKP-PRO qty 3 assigned. → Expected: Baseline.
19. Add 3 seats dunning; all assigned. → Expected: Add seats in dunning.
20. After 14d: only new 3 seats removed; original 3 users intact. → Expected: Targeted revocation.
21. Tax/subtotal/total after revocation accurate in SF and GetOrders. → Expected: Amounts adjusted.
22. Only users on new seats removed from AXP. → Expected: Assignment cleanup correct.
23. (Section) Add seats — 3 assigned + 1 unassigned. → Expected: Header.
24. First order SKP-PRO qty 3 assigned. → Expected: Baseline.
25. Add 4 seats: 3 assigned, 1 unassigned; dunning. → Expected: Totals 7 licenses (6 assigned, 1 available) before revoke.
26. After 14d: 4 new seats revoked; 3 base remain all assigned. → Expected: Counts per case.
27. (Section) Add seats — 2 normal + 1 new TID user. → Expected: Header.
28–35. Repeat add-seats dunning variants and Aria validations from Zephyr indices 32–37 (upgrade path setup). → Expected: Per DVW-T122 steps 32–37 (follow steps 4–11 from first revoke case where stated).
36. After 14d: all 3 add-seat licenses revoked. → Expected: AXP shows 3 assigned, 0 available post revoke.
37. (Section) Upgrade SKP-PRO → SKP-STDO. → Expected: Header.
38. First order SKP-PRO qty 3 (2 assigned, 1 unassigned). → Expected: Baseline.
39. Upgrade all 3 to SKP-STDO with dunning PayPal. → Expected: Upgrade succeeds; 3 licenses on STDO.
40. After 14d (7d wait note for monthly): no STDO subscriptions in AXP. → Expected: All STDO revoked.
41. Revoke 2 assigned + 1 unassigned STDO licenses. → Expected: All upgraded licenses removed.
42–43. Follow first-order revoke validations 4–11. → Expected: Negative order/invoice pattern repeats.
44. (Section) Renewals and downgrade revoke. → Expected: Header.
45. First order SKP-STUDIO qty 2 with assign to me. → Expected: Baseline.
46. Downgrade STUDIO → PRO staged. → Expected: Downgrade queued.
47. Subsequent SKP-GO with card expiring soon. → Expected: Subsequent placed.
48. Renewals: downgrade + SKP-GO renewal with expired card → both dunning. → Expected: Staged downgrade and renewal in dunning.
49. After 14d without payment update: downgraded PRO and SKP-GO revoked. → Expected: Both gone.
50. AXP: no active subscriptions. → Expected: Empty subscriptions tab.
51–61. Double-order dunning calendar scenarios (Dec/Jan example): SKP-STUDIO subsequent dunning ends Jan 8; SKP-PRO renewal dunning Jan 15; earliest end revokes all active dunning orders together. → Expected: Per DVW-T122 indices 52–60 (confirm cross-order revoke when first dunning period ends).
62–79. Mixed monthly/yearly purchase orders, renewal dunning alignment, same-day email behavior, add-seats + renewal overlap calendars. → Expected: Execute timing matrix in Zephyr indices 62–79 (monthly-only revoke vs yearly-only revoke; combined renewal dunning; notification separation).
80. Final mixed scenario (yearly Jan 9, monthly Dec 9, add seats Jan 5): single renewal notification with add-seat steps; revoke timing Jan 11. → Expected: Per step 79 description in DVW-T122.

---

### Renewal/Dunning without payment method [P2] (from DVW-T131)

**Objective:** Renewal when payment method deleted in Aria; dunning; plan cancel after dunning steps; recovery when payment fixed.

**Preconditions:**

- CC order placed; payment method manually removed in Aria.

**Steps:**

1. (Pre-requisite) Place order with CC; delete payment in Aria. → Expected: No PM in Aria.
2. Trigger renewal via VC/bill lag. → Expected: Renewal processes but plan enters dunning; COLLECTION_FAILED true in Aria CDF.
3. Advance dunning steps 1→4 in Aria. → Expected: Plan cancelled in Aria; asset expired in SF; negative order and invoices in SF.
4. Repeat for PayPal. → Expected: Same outcomes for PayPal path.
5. When in dunning, add correct payment method. → Expected: collection_failed false; out of dunning; plan active; asset active in SF.

---

### Subscription Reminder Notifications - YEARLY Subscriptions [P2] (from DVW-T133)

**Objective:** 30-day auto-renewal and subscription-expiration emails; 7-day repeat; multi-subscription combinations; DataDog clean; template links.

**Preconditions:**

- Yearly auto-renewable subscriptions; Aria VC control.

**Steps:**

1. (Section) Yearly auto-renewal reminder. → Expected: Header.
2. At least one yearly subscription; VC 30 days before renewal. → Expected: Auto-renewal reminder in inbox; multi-SKU combined per rules.
3. DataDog: no errors for job (30 min cadence note). → Expected: Clean logs.
4. Email: unit price per subscription column; payment method; template match (Google Doc link). → Expected: Matches template.
5. VC 7 days before renewal again. → Expected: Same reminder fires again at 7-day mark.
6. (Section) Yearly expiration reminder (cancelled). → Expected: Header.
7. Cancel yearly; VC 30 days before renewal date. → Expected: Expiration reminder email; multiple yearly can be combined.
8. DataDog clean. → Expected: No error logs.
9. Email content matches expiration template (Google Doc link). → Expected: Template match.
10. (Section) More than one yearly. → Expected: Header.
11. Three yearly subs: cancel Connect Business; keep Premium and ProjectSight; VC −30d. → Expected: Auto-renew email lists Premium + ProjectSight; expiration email lists cancelled SKU.
12. After 30-day emails, cancel ProjectSight; VC −7d. → Expected: Only Premium auto-renew at 7d; expiration for others per step.
13. (Section) Subscription expired notification. → Expected: Header.
14. Yearly subscription cancelled; VC on renewal date. → Expected: Subscription expired email per template; multi-cancel combined.

---

### Subscription Reminder Notification - Monthly Sku [P2] (from DVW-T134)

**Objective:** Monthly auto-renew 7-day reminder; monthly non-renew expiration reminders and expiry-day email; locale-specific timing (UAT vs QA/SIT).

**Preconditions:**

- New customer account; valid payment on setup.

**Steps:**

1. Create account. → Expected: Account created.
2. Account setup with valid payment. → Expected: Payment saved.
3. Monthly subscription order auto-renew on. → Expected: Activated.
4. Simulate 7 days before renewal. → Expected: Auto-renew email ~6:30 AM IST for India region (per Zephyr).
5. Verify auto-renew email content vs template link. → Expected: Accurate details.
6. Monthly subscription auto-renew off. → Expected: Order activated.
7. Simulate 7 days before expiration. → Expected: Expiration reminder sent.
8. Verify expiration reminder template. → Expected: Matches doc.
9. On exact expiration day (auto-renew off). → Expected: Expiration subscription email sent.
10. Verify expiration day email template. → Expected: Matches doc.
11. Validate registered email receives mail. → Expected: Correct recipient.
12. Cancel subscription after 7 days from purchase: no further notifications. → Expected: No emails post rule.
13. UAT: monthly cancel flow — expiring soon emails 3 & 1 days before anniversary; expired on anniversary (vs QA/SIT 7-day rule). → Expected: Environment-specific schedule in Zephyr table.

---

### Monthly Dunning Failure Scenario & Notification Email + Dunning to Paid [P1] (from DVW-T135)

**Objective:** Monthly subscription dunning with PayPal insufficient/cancelled agreement; faster step cadence than yearly; dunning to paid.

**Preconditions:**

- Monthly subscription order with dunning PayPal.

**Steps:**

1. Monthly order; PayPal insufficient / billing agreement cancelled → dunning. → Expected: Dunning; collection true in Aria; no collection initially.
2. GetOrders INVOICE_UNPAID_SENT. → Expected: Invoice status.
3. SF entities created; invoice Unpaid. → Expected: Unpaid.
4. getPayments collectionFlag true. → Expected: True in dunning.
5. Aria Day 1 dunning Step 1. → Expected: Dunning In Progress step 1.
6. Day 3: second attempt; email; Step 2 (note: description says 3 calendar days vs title "2nd time"). → Expected: Step 2 in Aria.
7. Day 5: third attempt; email; Step 3. → Expected: Step 3 in Aria.
8. Update payment across PayPal/CC combinations. → Expected: Invoice Paid; MIDs for CC.

---

### D-country hold Renewal order - Mail Notification to User [P2] (from DVW-T136)

**Objective:** D-country hold emails go to user, not support test group; EEUC validity paths; renewal after expiry; dunning email locale.

**Preconditions:**

- D-country account; test support mailbox dcountrytestcommerce-ug@trimble.com for negative check.

**Steps:**

1. Create account in D-country region. → Expected: Account valid for D-country hold.
2. Complete account setup and checkout → EEUC form. → Expected: Redirect to EEUC.
3. Accept EEUC. → Expected: Validity 1 year from acceptance.
4. Place first order after acceptance. → Expected: No D-country hold email to support; order confirmation only to user.
5. Verify notification logs: no D-country admin email. → Expected: Logs clean of admin success to support address.
6. Renewal within EEUC validity. → Expected: Renewal succeeds; no D-country hold email.
7. (Contradictory step in Zephyr index 7) Verify no dcountry email to admin and user — treat as log check per test data. → Expected: Interpret per Zephyr test data note.
8. DB: set EEUCExpirationDate past; renewal after expiry. → Expected: D-country hold email NOT to support group; IS to user personal email per template.
9. (Section) Dunning. → Expected: Header.
10. Place order that goes dunning (expired card). → Expected: Order in dunning.
11. Dunning email to user in English. → Expected: English dunning mail.
12. Change profile locale → email language follows locale. → Expected: Localized dunning content.

---

### RMsg Received During Renewal – Remove Payment Method and Move to Dunning InProgress [P2] (from DVW-T155)

**Objective:** Cybersource RMsg (e.g., stolen card) on renewal removes PM from Aria and sets dunning InProgress without further retries on that PM.

**Preconditions:**

- SketchUp Go Annual (or test SKU); Aria access to rates and bill lag; Cybersource portal.

**Steps:**

1. Place valid SKU order in UI. → Expected: Order success.
2. Aria: Account Overview for customer. → Expected: Overview loads.
3. Select master plan for subscription. → Expected: Plan context.
4. Edit Rates → custom rate 1004 RPU (or SKU-specific). → Expected: Custom rate saved.
5. Bill Lag Days −400 or −30 per SKU. → Expected: Field updated.
6. Generate Pending invoice with account. → Expected: Pending invoice created.
7. Save invoice; let renewal/transaction run. → Expected: Renewal attempt occurs.
8. Recent invoices show failed invoice (stolen/lost style). → Expected: Red failed invoice visible.
9. Plans: dunning InProgress (1/4). → Expected: Dunning state shown.
10. Cybersource: locate transaction for renewal invoice. → Expected: Transaction found.
11. Map RMsg (e.g., 205 stolen) to mapping sheet. → Expected: Code matches sheet.
12. Aria Payment Methods: card removed. → Expected: PM no longer listed.
13. No further capture attempts on removed PM. → Expected: No successful retries on old PM.

---

### Refund UI Updates - History in Support Portal, Email Notification [P1] (from DVW-T159)

**Objective:** Support portal refund UI, dropdowns, partial refund math, emails, Aria refund record, Salesforce credit note, upgrade refund line behavior.

**Preconditions:**

- Support portal access; placed order with known SKU (e.g., Connect Business Premium).

**Steps:**

1. (Section) Create account and place order. → Expected: Header.
2. Place order; capture Account ID and Order ID on success page. → Expected: IDs available.
3. (Section) Support portal. → Expected: Header.
4. Support Portal → Refund → search Account ID + Order ID → open details. → Expected: Details load; Initiate Refund disabled until dropdowns filled.
5. Credit Note Cause/Reason/Other Reason dropdowns; enable Initiate Refund when all set. → Expected: Button enables per spreadsheet lists.
6. Refund history dropdown lists prior refunds for SKU. → Expected: History entries with date/amount/tax/total.
7. Formatting: subtotal/tax/total show two decimals. → Expected: .00 padding consistent.
8. Enter partial or full refund amount. → Expected: System calculates tax and total refund.
9. Auto calculation display matches proportional rules. → Expected: Refund total = subtotal + tax logic.
10. (Section) Refund email. → Expected: Header.
11. Inbox: "Your Trimble refund confirmation" with subtotal/tax/total/date. → Expected: Email received with fields.
12. (Section) Aria. → Expected: Header.
13. Aria Payments & Credits → refund record. → Expected: Refund amount, reason, method, refund id shown.
14. (Section) Salesforce. → Expected: Header.
15. SF Credit Note tab: cause/reason/situation/status/date. → Expected: Credit note matches portal selection.
16. Upgrade refund: negative line for unused portion of original SKU; credit note linkage. → Expected: SF handles upgrade refund shape.
17. Repeat for multi-line same order, renewals, upgrades, returns combinations. → Expected: Each refund row in portal/Aria/SF; email each time.

---

### EEUC Pre- Expiration Notification [P1] (from DVW-T169)

**Objective:** EEUC reminder emails at 90/60/30/15 days before expiry; acceptance stops lower-tier reminders; localization API; staged cancel/downgrade accounts; post-renewal links to AXP EEUC.

**Preconditions:**

- Accounts with EEUC expiry set; active Aria plan; notification API curl access for localization test.

**Steps:**

1. EEUC expiry exactly 90 days out. → Expected: Email with acceptance link; accept updates expiry; no 60/30/15 if accepted early; only accounts with active Aria plan; AXP EEUC form still reachable after day 90.
2. Expiry 60 days out. → Expected: Email + accept; suppresses 30/15; active plan rule; AXP form accessible after day 60.
3. Expiry 30 days out. → Expected: Email + accept; suppresses 15-day; other rules same.
4. Expiry 15 days out. → Expected: Email only if prior three not accepted; other rules same.
5. Call notifications API for EEUC_PRE_EXPIRATION across locales (sample curl in Zephyr). → Expected: Templates valid per locale list.
6. Account with multiple lines and staged cancel/downgrade but active in Aria. → Expected: Email still triggers without error.
7. Post-renewal EEUC emails: links open AXP EEUC page moved from commerce. → Expected: Links work to AXP.

---

## Key Validations

- Email templates, IDs (order/account), and timing (VC-relative days) match linked Google Docs per case.
- Dunning state in Aria aligns with GetOrders invoice flags and collection APIs before and after payment remediation.
- D-country and EEUC notification routing respects support vs user recipients per current product rules.

## Common Preconditions

- Mailbox access (user and test distribution lists); Aria virtual clock; ability to simulate network throttling where required.
- Support portal permissions for refund scenarios in DVW-T159.
