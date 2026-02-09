
# Bill and BillPay Enhanced Outline

## Bills

### 1. Bill creation-related flows

1.  **Bills > Add bill > Manual entry** (p-bill-upload)
    1.  Create a new expense record with minimal fields
2.  **Bills > Add bill > Upload** (w-bill-btnAddNew)
    1.  Create a new expense record with minimal fields
    2.  Set status_scanning_attachment = true
    3.  Create attachment for the expense
    4.  Bill Parse - Update (POST Xano /bill/parse → Photon OCR)
    5.  BWF bill_update (POST Bubble webhook back from Xano)
        1.  Update many of the expense record's fields as supplied by Photon
        2.  BWF on list: bill_item_create to create all line items
        3.  Set status_scanning_attachment = false
3.  **Bills via email**
    1.  Email received
        1.  ForwardMX (company forwarding email: bills-companyid@kleercard.com)
        2.  Mailparser (extract sender, subject, attachments)
        3.  POST Xano /bill-request
        4.  BWF request_bill
            1.  Create new request
            2.  Create a new expense_request
    2.  Inbox > Accept (pg-bill-inbox)
        1.  BWF bill_create_from_req
            1.  Create a new expense record from the request record
            2.  Create a new attachment from the request
            3.  Flow continues with Bill Parse - Update as we have in Bill Upload above

### 2. Bill update-related flows

1.  **Save bill** (fg-bill-viewDetails)
    1.  Make changes to Expense…
    2.  BWF on list: reimbursement_create_line (on JSON-based virtual bill lines)
        1.  Delete expense_line_item or
        2.  Create expense_line_item or
        3.  Update expense_line_item
    3.  Recalculate totals (amount_subtotal, amount_tax, amount_total, amount_balance_due)
2.  **New attachment** (fg-bill-viewDetails)
    1.  Create attachment…
    2.  Optional: Trigger OCR/parsing if bill-related document
3.  **Delete attachment** (w-file-preview)
    1.  Delete attachment…
    2.  Update _attachment field if main attachment deleted
4.  **Delete bill** (m-bill > p-bill-delete)
    1.  BWF bill-delete
        1.  MODIFY Expense: Status = Deleted (soft delete)
        2.  Update related records (approval flows, payment allocations)
        3.  Mark for integration deletion if synced
5.  **Duplicate bill - single** (m-bill)
    1.  Trigger duplicate-bill from wf-bill
        1.  Copy a list of expense_line_items…
        2.  Create a new expense (exclude: _approval_flow, _attachment, AcctIntID, payments)
        3.  BWF approval_flow_create
6.  **Duplicate bill - bulk** (BWF bill-duplicateListOfBills)
    1.  BWF on list: bill-duplicatebill (same as single duplicate)
7.  **Submit bill for approval**
    1.  BWF bill_submitted_for_approval
        1.  Make changes to expense (status = Submitted, status_approval = Pending)
        2.  BWF approval-flow-router
            1.  Check approval rules (amount, department, vendor, user role)
            2.  Create approval_request records for each approver
            3.  Send notifications to first approver(s)

### 3. Bill integration-related flows

1.  **Create bill in QBO** (qbo-bill-create)
    1.  Validate prerequisites (integration connected, vendor mapped, not already synced)
    2.  Map vendor (Bubble vendor → QBO Vendor ID)
    3.  Map chart of accounts for line items
    4.  Prepare QBO payload (VendorRef, TxnDate, DueDate, Line items)
    5.  POST QBO API: /v3/company/<realmId>/bill
    6.  Update Expense: AcctIntID = QBO Bill ID, is_exported = true, date_exported, clear errors
2.  **Update bill in QBO** (qbo-bill-update)
    1.  Retrieve QBO bill to get current SyncToken
    2.  Prepare update payload with Id and SyncToken
    3.  POST QBO API: /v3/company/<realmId>/bill
    4.  Update date_exported, clear errors
3.  **Delete bill from QBO** (qbo-bill-delete)
    1.  POST QBO API with operation=delete
    2.  Clear AcctIntID, set is_exported = false
4.  **Bulk sync** (bill-bulk-sync-integration)
    1.  BWF on list: bill-bulk-sync-integration_bill_create (for unsynced bills)
    2.  BWF on list: bill-bulk-sync-integration_bill_update (for synced bills)
    3.  Track progress, show summary of successes/errors

### 4. Database Triggers

1.  **Bill Changelog**
    1.  POST to Xano (audit trail: who, what, when)
2.  **Bill Created**
    1.  BWF activity-log-bill-create
3.  **Bill Deleted**
    1.  BWF qbo-bill-delete (delete from QuickBooks)
    2.  Integration: bill>delete
4.  **Bill Deleted Retain**
    1.  Terminate this workflow
    2.  Retain Terminal Status
5.  **Bill Submitted**
    1.  Update status_approval
6.  **Bill Updated & New Approval Flow**
    1.  Set duplicate alert (check same vendor, similar amount/date)
    2.  Delete duplicate alert (if no longer duplicate)
    3.  BWF approval_flow_create (if amount/vendor changes significantly)
    4.  Calculate Balance Due (amount_total - sum of payments)

----------

## BillPay

### 1. Payment creation-related flows

1.  **Schedule payment from bill** (p-bill-schedulePayment)
    1.  Validate vendor has active payment_method
    2.  BWF billpayment_create
        1.  Create BillPayment (status = Scheduled, payment_date, amount_total)
        2.  Create BillPayment_item (link to expense, amount allocated)
        3.  Update expense: status = Scheduled, amount_balance_due -= payment amount
    3.  Schedule BWF billpayment_schedule for payment_date
2.  **Create payment from BillPay page** (pg-billpay > p-billpayment-create)
    1.  Select vendor and multiple approved bills
    2.  BWF billpayment_create (same as above, multiple BillPayment_items)
3.  **Bulk payment creation** (pg-bill-list)
    1.  Select multiple approved bills, group by vendor
    2.  Create one BillPayment per vendor

### 2. Payment processing-related flows

1.  **Scheduled payment processing** (BWF billpayment_schedule - runs daily)
    1.  Query: status = Scheduled AND payment_date = Today
    2.  Validate: sufficient funds, payment_method active, bills still approved
    3.  **ACH Payment** (BWF billpay_send_ach)
        1.  Update BillPayment: status = Processing
        2.  POST Xano payment endpoint
        3.  Xano → Dwolla API: POST /transfers
            1.  Source: company funding source
            2.  Destination: vendor funding source
            3.  Metadata: billpayment_id, memo
        4.  Dwolla returns transfer ID
        5.  BWF billpayment_update (Xano webhook)
            1.  Update BillPayment: status = Pending, store Dwolla transfer ID
            2.  Update expense: status = Pending Payment
        6.  **Dwolla webhook** (customer_transfer_completed/failed)
            1.  POST to Xano webhook receiver
            2.  Xano processes and calls Bubble
        7.  BWF bill-payment-complete
            1.  Update BillPayment: status = Completed
            2.  Update expense: status = Paid, mark_as_paid = true, amount_balance_due = 0
            3.  BWF bill_paid (sync to QBO, send notifications)
    4.  **Check Payment** (alternative flow)
        1.  Generate check PDF
        2.  Mark as Pending Mailing
        3.  User confirms mailing → Mailed → Cleared
2.  **Payment failure handling** (BWF bill_payment_error)
    1.  Update BillPayment: status = Failed, store error message/code
    2.  Revert expense: status = Approved, restore amount_balance_due
    3.  Send notifications (payment creator, AP team)
    4.  Log error for troubleshooting
    5.  May trigger automatic retry or flag for manual review

### 3. Payment management-related flows

1.  **View payment details** (pg-billpayment-detail / p-billpayment-details)
    1.  Show: payment info, associated bills (BillPayment_item table), Dwolla transfer ID, transaction timeline
    2.  Actions: cancel, modify date, view in QBO, download confirmation
2.  **Modify payment date** (p-billpayment-modifyDate)
    1.  Validate: status must be Scheduled
    2.  Update BillPayment: payment_date = new date
    3.  Reschedule billpayment_schedule workflow
    4.  BWF billpayment_update
3.  **Cancel payment** (p-billpayment-cancel)
    1.  Validate: can cancel if Scheduled or early Pending
    2.  Call Dwolla API to cancel transfer (if already initiated)
    3.  Update BillPayment: status = Cancelled
    4.  Revert bills: status = Approved, restore amount_balance_due
    5.  BWF billpayment_update

### 4. Payment method management-related flows

1.  **Add payment method** (p-payment-method)
    1.  Create payment_method record (link to vendor)
    2.  **If ACH:**
        1.  POST Xano Dwolla service (Create Funding Source)
        2.  Xano → Dwolla API (creates funding source)
        3.  Update payment_method: store Dwolla funding_source_id, status = Pending Verification
        4.  Dwolla sends micro-deposits (2 small amounts, 1-3 days)
    3.  **If Check:**
        1.  Store mailing address
        2.  Set status = Active
2.  **Verify micro-deposits** (p-payment-method)
    1.  User enters two deposit amounts
    2.  POST Xano Dwolla service (Micro-deposits verify)
    3.  Dwolla verifies amounts
    4.  Update payment_method: status = Active
3.  **Remove payment method** (m-payment-method)
    1.  Confirmation popup
    2.  Call Dwolla API (Remove Funding Source)
    3.  Update payment_method: status = Inactive

### 5. Payment integration-related flows

1.  **Sync to QuickBooks Online** (qbo-billpayment-create)
    1.  Validate: integration connected, bill synced to QBO (has AcctIntID)
    2.  Map payment account to QBO bank account
    3.  Prepare QBO BillPayment payload (VendorRef, TxnDate, Line with LinkedTxn to bill)
    4.  POST QBO API: /v3/company/<realmId>/billpayment
    5.  Store QBO BillPayment ID
2.  **Bulk payment sync** (bill-bulk-sync-integration_billpay_create)
    1.  Process multiple completed payments
    2.  Call qbo-billpayment-create for each

### 6. BillPay configuration flows

1.  **Enable BillPay feature** (p-billpay-enableBillpayFeature)
    1.  User sees pg-billpay-marketing (if not enabled)
    2.  User reviews terms and accepts
    3.  Update company_setting_bill: BillPay enabled = true
    4.  Trigger Dwolla customer onboarding
    5.  Setup company bank account
    6.  Configure default payment account (pg-billpay-settings)
2.  **Create forwarding email** (w-billSettings-createForwardingEmail)
    1.  Generate unique email: bills-companyid@kleercard.com
    2.  Store in company settings

----------

## Key Data Relationships

### Expense (bill) Data Type Fields

-   **Identifiers:** invoice_number, reference_number, photon_key, xano_id, AcctIntID
-   **Relationships:** _company, _vendor, _user_assignedTo, _approval_flow, _attachment, _lineItems, _bill_payment_items, _expense_request, _payment_method
-   **Amounts:** amount_total, amount_subtotal, amount_tax, amount_shipping, amount_discount, amount_balance_due
-   **Status:** status (Draft/Submitted/Approved/Paid/Deleted), status_approval (Pending/Approved/Declined), status_scanning_attachment
-   **Dates:** date_created, date_due, date_service_start/end, date_exported, payment_date
-   **Vendor Info:** vendor_name, vendor_address, vendor_email, vendor_phone, vendor_bank details
-   **Integration:** AcctIntID (QBO Bill ID), is_exported, error_list_acct_integration, IntErrorCode

### BillPayment Data Type Fields

-   **Relationships:** Links to bills via BillPayment_item, payment_method, vendor, company
-   **Amounts:** amount_total
-   **Status:** Status (Scheduled/Processing/Pending/Completed/Failed/Cancelled)
-   **Dates:** payment_date, completion_date
-   **Integration:** Dwolla transfer ID, QBO BillPayment ID

### Payment_method Data Type Fields

-   **Relationships:** Belongs to vendor
-   **Type:** ACH, Check, Wire Transfer
-   **ACH Details:** routing_number, account_number, Dwolla funding_source_id
-   **Check Details:** mailing address
-   **Status:** Active, Pending Verification, Inactive

----------

## Status Progressions

### Bill Status

Draft → Submitted → Approved → Scheduled → Pending Payment → Paid

### Payment Status

Scheduled → Processing → Pending → Completed (or Failed/Cancelled)

### Payment Method Status (ACH)

Created → Pending Verification (micro-deposits) → Active

----------

## Platform Architecture

**Bubble:** Frontend UI, workflows, database  
**Xano:** Backend orchestration, webhooks, Dwolla management  
**Photon:** Bill OCR/parsing  
**ForwardMX/Mailparser:** Email ingestion  
**Dwolla:** ACH payment processing  
**QuickBooks Online:** Accounting integration

----------

## Key Backend Workflows

**Bill:** bill_create, bill_update, bill_item_create, bill-delete, bill-duplicateBill, bill_submitted_for_approval, bill_approved, bill_create_from_req, request_bill, qbo-bill-create/update/delete, bill-bulk-sync-integration

**BillPay:** billpayment_create, billpayment_schedule, billpay_send_ach, billpayment_update, bill-payment-complete, bill_paid, bill_payment_error, qbo-billpayment-create

**Approval:** approval_flow_create, approval-flow-router

----------

## Key Pages & Reusables

**Bill:** pg-bill-list, pg-bill-inbox, fg-bill-viewDetails, w-bill-btnAddNew, m-bill, p-bill-upload, p-bill-delete, cmp-bill-lineItems

**BillPay:** pg-billpay, pg-billpayment-detail, p-billpayment-create, p-bill-schedulePayment, p-billpayment-cancel, p-payment-method, pg-vendor-payment-methods

**Settings:** preferences_bill_management, pg-billpay-settings, w-billSettings-createForwardingEmail
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxNDU3MTkwOTksLTIwNTMxNzQxMjIsLT
E5MTQ1MDk1NjEsLTE3NjIwNDIyLC0xNzYyMDQyMl19
-->