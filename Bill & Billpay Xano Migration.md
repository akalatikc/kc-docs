# Bill and BillPay Discovery Document
## KleerCard MVP Bubble + Xano Application

**Generated:** February 6, 2026  
**App ID:** kleercardmvp (dedicated: d115)  
**Version:** test

---

## Table of Contents
1. [Overview](#overview)
2. [Data Model](#data-model)
3. [Bills Section](#bills-section)
4. [BillPay Section](#billpay-section)

---

## Overview

This document provides a comprehensive analysis of the bill and billpay features within the KleerCard MVP application, which operates on a Bubble frontend with Xano backend architecture. The system handles expense/bill management and payment processing workflows with integration to accounting systems (QuickBooks Online) and third-party services.

### Platform Architecture
- **Frontend/Logic:** Bubble (bubble.io)
- **Backend/Data:** Xano (xano.kleercard.com)
- **Third-party Services:** 
  - Photon (receipt/bill parsing)
  - ForwardMX & Mailparser (email ingestion)
  - QuickBooks Online (accounting integration)
  - Dwolla (payment processing)

---

## Data Model

### Primary Data Types (Bubble)

#### expense (Data Type: `bill`)
**Display Name:** expense  
**Platform:** Bubble  
**Purpose:** Core bill/expense record

**Key Fields:**
- **Identifiers:**
  - `invoice_number` (Text) - Invoice number from vendor
  - `reference_number` (Text) - Internal reference
  - `photon_key` (Text) - Photon parsing service identifier
  - `xano_id` (Text) - Xano database identifier
  - `AcctIntID` (Text) - Accounting integration ID (QBO)

- **Relationships:**
  - `_company` (company) - Parent company
  - `_vendor` (vendor) - Associated vendor
  - `_user_assignedTo` (User) - Assigned user
  - `_approval_flow` (approval_flow) - Associated approval chain
  - `_attachment` (attachment) - Main attachment/document
  - `_lineItems` (List of expense_line_item) - Line items
  - `_bill_payment_items` (List of BillPayment_item) - Payment allocations
  - `_expense_request` (expense_request) - Source request (if from inbox)
  - `_payment_method` (payment_method) - Payment method/account
  - `z-2025-11-03_customFields` (List of z-2025-11-03_expense_custom_field)
  - `z-2025-11-03_taxItems` (List of z-2025-11-03_expense_tax_item)

- **Amounts:**
  - `amount_total` (Number) - Total bill amount
  - `amount_subtotal` (Number) - Subtotal before tax/shipping
  - `amount_tax` (Number) - Tax amount
  - `amount_shipping` (Number) - Shipping charges
  - `amount_discount` (Number) - Discount applied
  - `amount_tip` (Number) - Tip/gratuity
  - `amount_cashback` (Number) - Cash back amount
  - `amount_balance_due` (Number) - Outstanding balance

- **Status Fields:**
  - `status` (Option: Status | Bill) - Overall bill status (Draft, Submitted, Approved, Paid, etc.)
  - `status_approval` (Option: Status | Bill Approval) - Approval workflow status
  - `status_scanning_attachment` (Boolean) - Indicates if attachment is being scanned/parsed
  - `is_exported` (Boolean) - Exported to accounting system
  - `mark_as_paid` (Boolean) - Manually marked as paid

- **Dates:**
  - `date_created` (Date) - Bill creation date
  - `date_due` (Date) - Payment due date
  - `date_ship` (Date) - Shipment date
  - `date_delivered` (Date) - Delivery date
  - `date_service_start` (Date) - Service period start
  - `date_service_end` (Date) - Service period end
  - `date_exported` (Date) - Export timestamp
  - `payment_date` (Date) - Payment date

- **Vendor Information:**
  - `vendor_name` (Text) - Vendor name
  - `vendor_raw_name` (Text) - Unparsed vendor name
  - `vendor_address` (Text) - Vendor address
  - `vendor_city` (Text)
  - `vendor_state` (Text)
  - `vendor_zip` (Text)
  - `vendor_country` (Text)
  - `vendor_email` (Text)
  - `vendor_phone` (Text)
  - `vendor_fax` (Text)
  - `vendor_website` (Text)
  - `vendor_type` (Text)
  - `vendor_account_number` (Text)
  - `vendor_bank_name` (Text)
  - `vendor_bank_number` (Text)
  - `vendor_bank_swift` (Text)
  - `vendor_iban` (Text)
  - `vat_number` (Text)
  - `vendor_abn_number` (Text)

- **Bill To Information:**
  - `bill_to_name` (Text)
  - `bill_to_recipient` (Text)
  - `bill_to_address` (Text)
  - `bill_to_address_line` (Text)
  - `bill_to_city` (Text)
  - `bill_to_state` (Text)
  - `bill_to_zip` (Text)
  - `bill_to_email` (Text)
  - `bill_to_vat_number` (Text)

- **Ship To Information:**
  - `ship_to_name` (Text)
  - `ship_to_address` (Text)

- **Additional Fields:**
  - `type` (Option: Type of Expense) - Expense classification
  - `notes` (Text) - Bill notes/comments
  - `comments_rejection` (Text) - Rejection reason
  - `payment_terms` (Text) - Payment terms
  - `payment_type` (Text) - Payment method type
  - `payment_display_name` (Text)
  - `po_number` (Text) - Purchase order number
  - `category` (Text) - Segment 3 mapping
  - `currency_code` (Text)
  - `document_type` (Text)
  - `tracking_number` (Text)
  - `carrier` (Text)
  - `check_number` (Number)
  - `card_number` (Text)
  - `phone_number` (Text)
  - `account_number` (Text)
  - `line-items-json` (List of Text) - JSON representation of line items
  - `alert_possible_duplicate` (Boolean)
  - `Alerts Dismissed` (List of Option: Alerts - Bill)
  - `all_email_addresses` (Text)
  - `error_list_acct_integration` (List of Text) - Integration errors
  - `IntErrorCode` (Text) - Error code
  - `qbo_print_check` (Boolean) - QuickBooks print check flag
  - `remit_to_name` (Text)
  - `remit_to_address` (Text)

#### expense_line_item (Data Type: `bill_line_item`)
**Display Name:** expense_line_item  
**Platform:** Bubble  
**Purpose:** Individual line items within a bill

**Key Relationships:**
- Parent expense record
- Chart of accounts mapping
- Tax and custom field associations

#### Bill_request (Data Type: `inbox` / `bill_request`)
**Display Name:** Bill_request  
**Platform:** Bubble  
**Purpose:** Bills received via email before acceptance

**Note:** There are two data types for bill requests:
- `inbox` - Primary inbox record
- `bill_request` - Legacy/alternate structure

#### expense_request (Data Type: `reimbursement_request`)
**Display Name:** expense_request  
**Platform:** Bubble  
**Purpose:** Employee reimbursement requests that can become bills

#### BillPayment (Data Type: `billpayment`)
**Display Name:** BillPayment  
**Platform:** Bubble  
**Purpose:** Payment records for bills

**Key Relationships:**
- Links to expense records
- Payment method
- Dwolla payment processing

#### payment_method (Data Type: `billpayee`)
**Display Name:** payment_method  
**Platform:** Bubble  
**Purpose:** Vendor payment methods (bank accounts, cards, etc.)

**Status Options:**
- Via option set: `Status | Payment Method` (`status_of_bill_payee`)

#### vendor (Data Type: `vendor`)
**Display Name:** vendor  
**Platform:** Bubble  
**Purpose:** Vendor/payee master records

### Related Data Types

#### attachment (Data Type: `attachment`)
**Purpose:** File attachments for bills (PDFs, images)

**Type Options:**
- Via option set: `Type of Attachment` (`type_of_attachment`)

#### approval_flow (Data Type: `approval_chain`)
**Purpose:** Multi-step approval chains for bills

#### approval_flow_step (Data Type: `approval_step`)
**Purpose:** Individual steps in approval workflow

#### approval_request (Data Type: `approval_request`)
**Purpose:** Individual approval requests to approvers

**Status Options:**
- Via option set: `Status | Approval Request` (`status_of_approval_request`)

---

## Bills Section

### 1. Bill Creation Flows

The system supports three primary methods for creating bills:

#### 1.1 Manual Bill Entry

**Page:** `p-bill-upload` (Reusable Element)  
**Trigger:** Bills > Add bill > Manual entry  
**Platform:** Bubble

**Process:**
1. User navigates to Bills page (`pg-bill-list`)
2. Clicks "Add bill" button
3. Selects "Manual entry" option
4. Opens `p-bill-upload` popup
5. User enters bill details manually
6. **Backend Workflow:** Creates new expense record with minimal fields:
   - Company reference
   - Status set to "Draft"
   - User assignment
   - Basic amount fields

**Data Created:**
- New `expense` record (Bubble)

**Initial Status:** Draft

---

#### 1.2 Bill Upload with Document Parsing

**Reusable Element:** `w-bill-btnAddNew`  
**Trigger:** Bills > Add bill > Upload  
**Platforms:** Bubble → Xano → Photon (3rd party) → Xano → Bubble

**Detailed Flow:**

**Step 1: User Initiates Upload**
- **Component:** `w-bill-btnAddNew` (reusable element)
- **Page Context:** `pg-bill-list` or other bill-related pages
- **Action:** User clicks "Upload" button and selects file

**Step 2: Create Initial Expense Record**
- **Platform:** Bubble
- **Action:** Create new `expense` record with:
  - `_company` = Current user's company
  - `_user_assignedTo` = Current user
  - `status` = "Draft" (option: `bill_status`)
  - `status_scanning_attachment` = true
  - `date_created` = Current date/time
  - Minimal identifying fields

**Step 3: Create Attachment Record**
- **Platform:** Bubble
- **Action:** Create new `attachment` record:
  - Link to parent `expense`
  - Store uploaded file
  - Set `type` (option: `type_of_attachment`)

**Step 4: Trigger Xano Bill Parse**
- **API Call:** POST to Xano `/bill/parse`
- **Platform:** Xano
- **Endpoint:** `https://xano.kleercard.com/api:[namespace]/bill/parse`
- **Payload:**
  ```json
  {
    "expense_bubble_id": "<expense unique id>",
    "attachment_id": "<attachment unique id>",
    "file_url": "<attachment file URL>",
    "company_id": "<company bubble id>"
  }
  ```

**Step 5: Xano Processing**
- **Platform:** Xano
- **Process:**
  1. Receives file URL from Bubble
  2. Calls Photon API (3rd party receipt/bill parsing service)
  3. Photon extracts:
     - Vendor information
     - Invoice number
     - Dates (invoice, due, service)
     - Amounts (subtotal, tax, total)
     - Line items
     - Payment terms
     - Addresses
  4. Xano stores parsed data
  5. Prepares webhook payload

**Step 6: Xano Webhook Callback to Bubble**
- **Backend Workflow:** `bill_update` (Bubble API Workflow)
- **Endpoint:** POST to Bubble webhook URL
- **Workflow Name:** bill_update
- **wf_name:** `bill_update`
- **Trigger:** Xano webhook callback

**Workflow Actions in `bill_update`:**

1. **Update Expense Record:**
   - Locate expense by `xano_id` or bubble ID
   - Update fields from Xano payload:
     - `vendor_name`
     - `invoice_number`
     - `reference_number`
     - `date_due`
     - `date_service_start`
     - `date_service_end`
     - `amount_subtotal`
     - `amount_tax`
     - `amount_total`
     - `amount_balance_due`
     - `payment_terms`
     - `currency_code`
     - Vendor address fields
     - Bill-to address fields
     - Ship-to address fields
     - `photon_key` (Photon's unique identifier)
     - Various metadata fields

2. **Process Line Items:**
   - **Backend Workflow (on list):** `bill_item_create`
   - **Purpose:** Create expense_line_item records
   - **Input:** List of line items from Xano payload
   - **For each line item:**
     - Create new `expense_line_item` record
     - Link to parent `expense`
     - Set:
       - `description`
       - `quantity`
       - `unit_price`
       - `amount`
       - Chart of accounts mappings (if available)
       - Tax codes
       - Other line-level details

3. **Update Status:**
   - Set `status_scanning_attachment` = false
   - Keep `status` = "Draft"

4. **Error Handling:**
   - If parsing fails, log error
   - Set error flags on expense record
   - Notify user (optional)

**Data Created:**
- `expense` record (Bubble)
- `attachment` record (Bubble)
- `expense_line_item` records (Bubble) - one per line item
- Parsed data stored in Xano

**Backend Workflows Involved:**
- **bill_update** (wf_name: `bill_update`) - Main webhook receiver
- **bill_item_create** (wf_name: `bill_item_create`) - Line item creation (scheduled on list)

**Related Reusable Elements:**
- `w-bill-btnAddNew` - Upload button component
- `input-file-uploader` or `input_file_uploader` - File upload component
- `input-receipt-uploader` - Alternate upload component

**Status Progression:**
- Initial: Draft (with `status_scanning_attachment` = true)
- After parsing: Draft (with `status_scanning_attachment` = false)

---

#### 1.3 Bills Received via Email

**Email Provider:** ForwardMX + Mailparser  
**Platforms:** Email → ForwardMX → Mailparser → Xano → Bubble

**Detailed Flow:**

**Step 1: Email Received**
- **Service:** ForwardMX
- **Process:**
  1. Company has unique forwarding email address (e.g., `bills-companyid@kleercard.com`)
  2. Vendor sends bill PDF/invoice to this address
  3. ForwardMX receives email
  4. ForwardMX forwards to Mailparser

**Step 2: Email Parsing**
- **Service:** Mailparser (3rd party)
- **Process:**
  1. Extracts email components:
     - Sender email address
     - Subject line
     - Email body text
     - Attachments (PDFs, images)
  2. Applies parsing rules
  3. Sends parsed data to Xano webhook

**Step 3: Xano Receives Email Data**
- **Endpoint:** POST to Xano `/bill-request`
- **Platform:** Xano
- **Payload from Mailparser:**
  ```json
  {
    "from_email": "vendor@example.com",
    "subject": "Invoice #12345",
    "body_text": "...",
    "attachments": [
      {
        "filename": "invoice.pdf",
        "url": "https://..."
      }
    ],
    "company_forwarding_email": "bills-companyid@kleercard.com"
  }
  ```

**Step 4: Xano Processing**
- **Platform:** Xano
- **Process:**
  1. Identifies company from forwarding email
  2. Stores email metadata
  3. Prepares data for Bubble
  4. Calls Bubble backend workflow

**Step 5: Create Bill Request in Bubble**
- **Backend Workflow:** `request_bill`
- **Workflow Name:** request_bill
- **wf_name:** `request_bill`
- **Platform:** Bubble
- **Trigger:** Xano webhook

**Workflow Actions in `request_bill`:**

1. **Create Request Record:**
   - Create new `request` record:
     - `_company` = Identified company
     - Type = Bill request
     - Status = Pending review

2. **Create Expense Request Record:**
   - Create new `expense_request` (data type: `reimbursement_request`):
     - Link to parent `request`
     - `_company` = Company
     - Store email metadata:
       - Sender email
       - Subject line
       - Received date
     - Status = In inbox

**Step 6: User Reviews in Inbox**
- **Page:** `pg-bill-inbox` (reusable element)
- **Navigation:** Bills > Inbox
- **Display:**
  - List of `expense_request` records where status = "In inbox"
  - Shows sender, subject, date received
  - Preview of attachment
  - Accept or Reject buttons

**Step 7: User Accepts Bill from Inbox**
- **Page:** `pg-bill-inbox`
- **Button:** Accept
- **Trigger:** User clicks Accept button
- **Backend Workflow:** `bill_create_from_req`
- **Workflow Name:** bill_create_from_req
- **wf_name:** `bill_create_from_req`

**Workflow Actions in `bill_create_from_req`:**

1. **Create Expense Record:**
   - Create new `expense` record:
     - `_company` = Company from request
     - `_user_assignedTo` = Current user or requester
     - `_expense_request` = Link to source request
     - `status` = "Draft"
     - `status_scanning_attachment` = true
     - `vendor_email` = From email address
     - Copy any available data from request

2. **Create Attachment from Email:**
   - Create new `attachment` record:
     - Link to newly created `expense`
     - Use attachment URL from email
     - Set `type` appropriately

3. **Trigger Bill Parsing:**
   - **API Call:** POST to Xano `/bill/parse`
   - **Process:** Same as "Bill Upload" flow (Step 4-6 from section 1.2)
   - Calls Photon to parse the document
   - Xano processes results
   - Webhook back to Bubble `bill_update` workflow
   - Updates expense with parsed data
   - Creates line items via `bill_item_create`
   - Sets `status_scanning_attachment` = false

4. **Update Request Status:**
   - Update `expense_request`:
     - Status = Accepted
     - Link to created `expense`

**Data Created:**
- `request` record (Bubble)
- `expense_request` record (Bubble)
- `expense` record (Bubble) - after acceptance
- `attachment` record (Bubble) - after acceptance
- `expense_line_item` records (Bubble) - after parsing

**Backend Workflows Involved:**
- **request_bill** (wf_name: `request_bill`) - Initial email receipt
- **bill_create_from_req** (wf_name: `bill_create_from_req`) - Accept from inbox
- **bill_update** (wf_name: `bill_update`) - Parse callback (reused)
- **bill_item_create** (wf_name: `bill_item_create`) - Line items (reused)

**Pages/Reusables:**
- `pg-bill-inbox` - Inbox listing page
- May use `m-bill` or similar modals for preview

**Email Configuration:**
- Each company gets unique forwarding address
- Stored in company settings
- Can be created/managed in settings:
  - **Reusable:** `w-billSettings-createForwardingEmail`
  - **Settings Page:** `preferences_bill_management` or `settings_bill_management`

---

### 2. Bill Update Flows

Once a bill exists in the system, users can modify it through various interfaces.

#### 2.1 Save Bill (Edit Details)

**Reusable Element:** `fg-bill-viewDetails`  
**Component Type:** Floating Group (detail view/editor)  
**Trigger:** User opens bill from list and makes edits  
**Platform:** Bubble

**User Interface:**
- **Primary Component:** `fg-bill-viewDetails` (floating group reusable element)
- **Contains:** Form fields for all bill properties
- **Also Used By:** `pg-bill-list` and other pages displaying bill details
- **Related Component:** `cmp-bill-lineItems` - Line item editor within detail view

**Save Action Flow:**

**Step 1: User Edits Fields**
- User modifies any expense fields in `fg-bill-viewDetails`:
  - Header fields (vendor, dates, amounts, etc.)
  - Notes, PO number, payment terms
  - Chart of accounts assignments
  - Custom fields
  - Tax items

**Step 2: Update Expense Record**
- **Action:** Modify `expense` record (Bubble)
- **Modified Fields:** Any changed by user
- **Common Updates:**
  - `vendor_name`
  - `amount_total`, `amount_subtotal`, `amount_tax`
  - `date_due`, `payment_date`
  - `notes`
  - `_vendor` (vendor relationship)
  - Chart of accounts mappings
  - Status (if transitioning)

**Step 3: Process Line Items**
- **Backend Workflow (on list):** `reimbursement_create_line`
- **Alternative Name:** May be `bill_line_create` or similar
- **Input:** JSON-based virtual bill lines
- **Process:** For each line item in the modified list:

  **If line item was deleted:**
  - Delete corresponding `expense_line_item` record

  **If line item is new:**
  - Create new `expense_line_item` record:
    - Link to parent `expense`
    - Set description, quantity, unit_price, amount
    - Set chart of accounts mappings
    - Set tax assignments

  **If line item was updated:**
  - Modify existing `expense_line_item` record:
    - Update description, amounts
    - Update chart of accounts
    - Update tax codes

**Step 4: Recalculate Totals**
- If line items changed:
  - Recalculate `amount_subtotal`
  - Recalculate `amount_tax`
  - Update `amount_total`
  - Update `amount_balance_due`

**Step 5: Trigger Database Event**
- Database trigger: "Bill Updated" (if significant changes)
- May trigger:
  - Approval flow updates
  - Duplicate detection
  - Integration sync

**Backend Workflows Involved:**
- **reimbursement_create_line** (wf_name may vary) - Scheduled on list for line items
- Possible other validation workflows

**Related Reusables:**
- `fg-bill-viewDetails` - Main detail/edit view
- `cmp-bill-lineItems` - Line item table/editor component
- `cmp-bill-dateInputs` - Date field component
- `cmp-bill-billToData` - Bill-to address component
- `cmp-bill-shipToData` - Ship-to address component

**Validation:**
- Required field checks
- Amount validations
- Date logic (due date, service dates)
- Line item totals match header amounts

**Status Considerations:**
- If bill is in "Draft", stays in Draft
- If bill is "Submitted" or later, changes may require re-approval
- Status changes handled separately (see approval flows)

---

#### 2.2 Add New Attachment

**Reusable Element:** `fg-bill-viewDetails`  
**Component:** File uploader within detail view  
**Trigger:** User uploads additional document  
**Platform:** Bubble

**Flow:**

1. **User Action:**
   - Within `fg-bill-viewDetails`
   - Clicks "Add Attachment" or similar
   - Selects file from device

2. **Create Attachment Record:**
   - Create new `attachment` (Bubble):
     - Link to parent `expense`
     - Store uploaded file
     - Set `type` = Supporting document or similar
     - Created date

3. **Optional Processing:**
   - May trigger OCR/parsing if it's a bill-related document
   - Update expense with additional parsed data

**Related Components:**
- `input-file-uploader` or `input_file_uploader`
- `input-receipt-uploader`
- `w-file-preview` - For viewing attachments

---

#### 2.3 Delete Attachment

**Reusable Element:** `w-file-preview`  
**Component:** File preview/viewer with delete option  
**Trigger:** User clicks delete on attachment  
**Platform:** Bubble

**Flow:**

1. **User Action:**
   - Views attachment in `w-file-preview`
   - Clicks delete icon

2. **Confirmation:**
   - Popup asks for confirmation

3. **Delete Attachment Record:**
   - Delete `attachment` record (Bubble)
   - File is removed from storage

4. **Update Expense:**
   - If main attachment was deleted:
     - Update `_attachment` field to next available attachment
     - Or set to empty if no attachments remain

**Note:** Cannot delete last attachment if bill was created from upload/email

---

#### 2.4 Delete Bill

**Popup:** `p-bill-delete` (reusable element)  
**Menu:** `m-bill` (reusable element - bill action menu)  
**Trigger:** User selects "Delete" from bill menu  
**Platform:** Bubble

**Flow:**

**Step 1: User Initiates Delete**
- **Page Context:** Bill list (`pg-bill-list`) or detail view
- **Menu:** `m-bill` (bill action menu)
- **Action:** Click "Delete" option
- **Popup Opens:** `p-bill-delete`

**Step 2: Confirmation**
- `p-bill-delete` displays:
  - Bill details (vendor, amount, invoice number)
  - Warning message
  - Confirm and Cancel buttons

**Step 3: Execute Delete**
- **Backend Workflow:** `bill-delete`
- **Workflow Name:** bill-delete (note: hyphenated, not underscore)
- **wf_name:** `bill-delete`

**Workflow Actions in `bill-delete`:**

1. **Soft Delete Expense:**
   - **Action:** Modify `expense` record (Bubble)
   - **Update:** `status` = "Deleted" (option from `bill_status`)
   - **Note:** This is a soft delete; record remains in database

2. **Update Related Records:**
   - Approval flows: Mark as cancelled/void
   - Payment allocations: Remove or mark as cancelled

3. **Integration Handling:**
   - If bill was synced to accounting system:
     - May trigger reverse sync
     - Or mark for deletion in next sync

**Step 4: Database Triggers**
- **Trigger Event:** "Bill Deleted"
- **Actions:**
  1. Delete bill from QuickBooks (if synced):
     - Calls `qbo-bill-delete` workflow
  2. Integration handling:
     - Workflow: `int-bill-all` or similar
     - Marks for deletion in integrated systems

**Backend Workflows Involved:**
- **bill-delete** (wf_name: `bill-delete`) - Main delete workflow
- **qbo-bill-delete** (wf_name: `qbo-bill-delete`) - QBO integration delete

**Database Triggers:**
- **Bill Deleted** - Triggered when status set to Deleted
- **Bill Deleted Retain** - Alternate trigger for terminal status retention

**Related Reusables:**
- `m-bill` - Bill action menu
- `p-bill-delete` - Delete confirmation popup

**Status After Delete:**
- `status` = "Deleted"
- Record retained for audit trail
- Filtered out of standard bill lists

**Permissions:**
- Only users with appropriate permissions can delete
- May require admin or approval based on bill status
- Bills in "Paid" status may have restricted deletion

---

#### 2.5 Duplicate Bill (Single)

**Menu:** `m-bill` (reusable element)  
**Workflow:** `wf-bill` (reusable element with workflows)  
**Trigger:** User selects "Duplicate" from bill menu  
**Platform:** Bubble

**Flow:**

**Step 1: User Initiates Duplicate**
- **Menu:** `m-bill` (bill action menu)
- **Option:** "Duplicate" or "Copy bill"
- **Workflow Container:** `wf-bill`

**Step 2: Trigger Duplicate Workflow**
- **Workflow:** Custom event "duplicate-bill" in `wf-bill`
- **Input:** Source expense unique ID

**Workflow Actions:**

1. **Copy Line Items:**
   - Copy all `expense_line_item` records from source bill:
     - Create new line items (not yet linked)
     - Preserve:
       - Descriptions
       - Quantities
       - Unit prices
       - Amounts
       - Chart of accounts mappings
       - Tax codes

2. **Create New Expense:**
   - Create new `expense` record:
     - Copy from source:
       - `_company`
       - `_vendor`
       - `vendor_name` and all vendor fields
       - All amount fields
       - Payment terms
       - Notes (with "Duplicated from..." prefix)
       - Chart of accounts defaults
     - Set new values:
       - `_user_assignedTo` = Current user
       - `status` = "Draft"
       - `status_approval` = Empty/initial
       - `date_created` = Current date/time
       - `invoice_number` = Empty (to be filled by user)
       - `date_due` = Empty or calculated from terms
     - Do NOT copy:
       - `_approval_flow`
       - `_attachment` (attachments not duplicated)
       - Payment-related fields
       - Sync/export fields
       - AcctIntID

3. **Link Line Items:**
   - Update copied line items to link to new expense

4. **Create Approval Flow:**
   - **Backend Workflow:** `approval_flow_create`
   - **Workflow Name:** approval_flow_create
   - **wf_name:** `approval_flow_create`
   - Generates new approval chain based on:
     - Company approval rules
     - Bill amount
     - User who created bill

**Backend Workflows Involved:**
- Custom event in `wf-bill` - "duplicate-bill"
- **approval_flow_create** (wf_name: `approval_flow_create`)

**Related Reusables:**
- `m-bill` - Bill action menu
- `wf-bill` - Workflow container

**Result:**
- New expense in Draft status
- User can edit before submitting
- Original bill unchanged

---

#### 2.6 Duplicate Bill (Bulk)

**Backend Workflow:** `bill-duplicateListOfBills`  
**Workflow Name:** bill-duplicateListOfBills (hyphenated)  
**wf_name:** `bill-duplicateListOfBills`  
**Trigger:** Bulk action from bill list  
**Platform:** Bubble

**Flow:**

**Step 1: User Selects Multiple Bills**
- **Page:** `pg-bill-list`
- **Action:** Checkbox selection of multiple bills
- **Bulk Menu:** Appears with "Duplicate" option

**Step 2: Initiate Bulk Duplicate**
- **Backend Workflow:** `bill-duplicateListOfBills`
- **Input:** List of expense unique IDs

**Workflow Actions in `bill-duplicateListOfBills`:**

1. **Schedule Duplication (on list):**
   - **Backend Workflow (scheduled):** `bill-duplicatebill`
   - **Workflow Name:** bill-duplicatebill
   - **wf_name:** `bill-duplicatebill` (note: lowercase, no hyphens)
   - **Scheduled for each:** Expense ID in the list
   - **Process:** Same as single duplicate (section 2.5)

**Backend Workflows Involved:**
- **bill-duplicateListOfBills** (wf_name: `bill-duplicateListOfBills`) - Orchestrator
- **bill-duplicatebill** (wf_name: `bill-duplicatebill`) - Per-bill duplication

**Performance:**
- Scheduled workflows run asynchronously
- User notified when complete
- May take time for large lists

**Related Pages:**
- `pg-bill-list` - Bill listing with bulk actions

---

#### 2.7 Submit Bill for Approval

**Backend Workflow:** `bill_submitted_for_approval`  
**Workflow Name:** bill_submitted_for_approval  
**wf_name:** `bill_submitted_for_approval`  
**Trigger:** User clicks "Submit" button on bill  
**Platform:** Bubble

**Flow:**

**Step 1: User Submits Bill**
- **Page Context:** `fg-bill-viewDetails` or bill detail page
- **Button:** "Submit for Approval"
- **Validation:** Check required fields completed

**Step 2: Update Expense Status**
- **Backend Workflow:** `bill_submitted_for_approval`

**Workflow Actions in `bill_submitted_for_approval`:**

1. **Modify Expense Record:**
   - Update `expense`:
     - `status` = "Submitted" (option from `bill_status`)
     - `status_approval` = "Pending" or "In Progress" (option from `bill_approval_status`)
     - Submission timestamp

2. **Route to Approval Flow:**
   - **Backend Workflow:** `approval-flow-router`
   - **Workflow Name:** approval-flow-router (hyphenated)
   - **Purpose:** Determines approval path
   - **Process:**
     - Checks approval rules for:
       - Bill amount thresholds
       - Department/segment
       - Vendor type
       - User role
     - Creates or updates approval flow:
       - Calls `approval_flow_create` if needed
       - Or uses existing approval flow
     - Creates approval requests:
       - One per required approver
       - Sets order/sequence
       - Sends notifications

3. **Create Approval Requests:**
   - For each approver in the chain:
     - Create `approval_request` record:
       - Link to `expense`
       - Link to `approval_flow_step`
       - `_user_assignedTo` = Approver
       - Status = "Pending" or "Waiting"
       - Due date (if applicable)

4. **Send Notifications:**
   - Email to first approver(s)
   - In-app notification
   - Mobile push (if enabled)

**Step 3: Database Triggers**
- **Trigger Event:** "Bill Submitted"
- **Actions:**
  - Update `status_approval`
  - May trigger external systems
  - Activity log entry

**Backend Workflows Involved:**
- **bill_submitted_for_approval** (wf_name: `bill_submitted_for_approval`)
- **approval-flow-router** (hyphenated) - Routes to appropriate approval chain
- **approval_flow_create** (wf_name: `approval_flow_create`) - May be called

**Related Database Triggers:**
- **Bill Submitted** - Primary submission trigger
- **Bill Updated & New Approval Flow** - If bill edited after submission

**Related Reusables:**
- `fg-bill-viewDetails` - Detail view with submit button
- `w-approval-chain-flex` or `w-approval-chain-legacy` - Approval chain display

**Approval Flow Options:**
- Configured per company in approval rules
- Can be:
  - Auto-approve (for low amounts or certain users)
  - Single approver
  - Multi-level approval chain
  - Parallel approvers

**Status Progression:**
- Before: Draft
- After: Submitted (status) + Pending (status_approval)
- Next: Approved/Declined (handled in approval workflows)

---

### 3. Bill Integration Flows

#### 3.1 Sync to Accounting Integration (QuickBooks Online)

**Backend Workflows:**
- **int-bill-all** - Master integration sync workflow
- **qbo-bill-create** (wf_name: `qbo-bill-create`) - Create bill in QBO
- **qbo-bill-update** (wf_name: `qbo-bill-update`) - Update existing bill in QBO
- **qbo-bill-delete** (wf_name: `qbo-bill-delete`) - Delete bill from QBO

**Platform:** Bubble → Xano → QuickBooks Online API

**Flow Overview:**

The system maintains bi-directional sync with QuickBooks Online (QBO), allowing bills to be created, updated, and deleted in the accounting system.

#### Sync Trigger Scenarios:

1. **Manual Sync:**
   - User clicks "Sync to QBO" button
   - Bulk sync from bill list

2. **Automatic Sync:**
   - Bill status changes to "Approved"
   - Scheduled sync (periodic)
   - Mark as Exported action

3. **Database Triggers:**
   - Bill Created → may auto-sync
   - Bill Updated → syncs changes
   - Bill Deleted → removes from QBO

---

#### 3.1.1 Create Bill in QBO

**Backend Workflow:** `qbo-bill-create`  
**Workflow Name:** qbo-bill-create  
**wf_name:** `qbo-bill-create`  
**Trigger:** Bill approved or manually synced (first time)

**Workflow Actions:**

1. **Validate Prerequisites:**
   - Check integration is connected
   - Verify expense has required fields:
     - Vendor (must be mapped to QBO vendor)
     - Amount total
     - Date created/due
   - Ensure not already synced (no `AcctIntID`)

2. **Map Vendor:**
   - Look up vendor mapping:
     - Bubble `vendor` → QBO Vendor ID
   - If not mapped:
     - May auto-create in QBO
     - Or pause sync pending manual mapping

3. **Map Chart of Accounts:**
   - For each line item:
     - Map Bubble account → QBO Account ID
   - Use default mappings if available
   - May use Xano integration service

4. **Prepare QBO Payload:**
   - Build JSON for QBO Bill API:
     ```json
     {
       "VendorRef": {"value": "<qbo_vendor_id>"},
       "TxnDate": "<date_created>",
       "DueDate": "<date_due>",
       "TotalAmt": <amount_total>,
       "Line": [
         {
           "DetailType": "AccountBasedExpenseLineDetail",
           "Amount": <line_amount>,
           "AccountBasedExpenseLineDetail": {
             "AccountRef": {"value": "<qbo_account_id>"},
             "ClassRef": {"value": "<qbo_class_id>"},
             "CustomerRef": {"value": "<qbo_customer_id>"}
           },
           "Description": "<line_description>"
         }
       ],
       "PrivateNote": "<notes>",
       "DocNumber": "<invoice_number>"
     }
     ```

5. **Call QBO API:**
   - **Method:** POST
   - **Endpoint:** `/v3/company/<realmId>/bill`
   - **Via:** Xano integration service or direct
   - **Auth:** OAuth 2.0 token (managed by Xano)

6. **Handle Response:**
   - **Success:**
     - Store QBO Bill ID:
       - Update expense `AcctIntID` = QBO Bill ID
     - Update `is_exported` = true
     - Update `date_exported` = current timestamp
     - Clear any previous errors:
       - Empty `error_list_acct_integration`
       - Clear `IntErrorCode`
   - **Failure:**
     - Log error:
       - Add to `error_list_acct_integration`
       - Set `IntErrorCode`
     - Notify user (optional)
     - May retry automatically

**Related Components:**
- Integration settings pages:
  - `pg-integrations-accounting`
  - `pg-integrations-accounting-self`
- Mapping pages:
  - `p-qbo-mapAccounts`
  - `p-qbo-mapVendors`
  - `p-qbo-importAccounts`
  - `p-qbo-importVendors`

---

#### 3.1.2 Update Bill in QBO

**Backend Workflow:** `qbo-bill-update`  
**Workflow Name:** qbo-bill-update  
**wf_name:** `qbo-bill-update`  
**Trigger:** Expense updated after initial sync

**Workflow Actions:**

1. **Validate Prerequisites:**
   - Check expense has `AcctIntID` (QBO Bill ID)
   - Verify integration still connected

2. **Determine Changes:**
   - Compare current expense fields to last synced state
   - Identify modified fields:
     - Amount changes
     - Line item changes
     - Date changes
     - Vendor changes
     - Memo/notes

3. **Retrieve QBO Bill:**
   - **Method:** GET
   - **Endpoint:** `/v3/company/<realmId>/bill/<AcctIntID>`
   - **Purpose:** Get current SyncToken (required for updates)

4. **Prepare Update Payload:**
   - Same structure as create
   - Include:
     - `Id`: AcctIntID
     - `SyncToken`: From GET response
     - Updated fields

5. **Call QBO API:**
   - **Method:** POST (with existing ID is an update)
   - **Endpoint:** `/v3/company/<realmId>/bill`
   - **Payload:** Full bill JSON with Id and SyncToken

6. **Handle Response:**
   - **Success:**
     - Update `date_exported` = current timestamp
     - Clear errors
   - **Failure:**
     - Log error in `error_list_acct_integration`
     - If SyncToken conflict, retry with fresh GET

**Note:** QBO bills cannot be updated after payment applied

---

#### 3.1.3 Delete Bill from QBO

**Backend Workflow:** `qbo-bill-delete`  
**Workflow Name:** qbo-bill-delete  
**wf_name:** `qbo-bill-delete`  
**Trigger:** Expense deleted in Bubble (database trigger)

**Workflow Actions:**

1. **Check if Synced:**
   - Verify expense has `AcctIntID`
   - If not synced, skip (nothing to delete)

2. **Check QBO Status:**
   - Retrieve bill from QBO to check if paid
   - Cannot delete paid bills

3. **Delete from QBO:**
   - **Method:** POST (operation=delete)
   - **Endpoint:** `/v3/company/<realmId>/bill?operation=delete`
   - **Payload:**
     ```json
     {
       "Id": "<AcctIntID>",
       "SyncToken": "<current_sync_token>"
     }
     ```

4. **Handle Response:**
   - **Success:**
     - Clear `AcctIntID` in Bubble
     - Update `is_exported` = false
   - **Failure:**
     - Log error
     - Bill remains in QBO

**Database Trigger:**
- **Bill Deleted** - Automatically calls this workflow

---

#### 3.1.4 Bulk Integration Sync

**Backend Workflow:** `bill-bulk-sync-integration`  
**Workflow Name:** bill-bulk-sync-integration  
**wf_name:** `bill-bulk-sync-integration`  
**Related Workflows:**
- `bill-bulk-sync-integration_bill_create` (wf_name: `bill-bulk-sync-integration_bill_create`)
- `bill-bulk-sync-integration_bill_update` (wf_name: `bill-bulk-sync-integration_bill_update`)
- `bill-bulk-sync-integration_billpay_create` (wf_name: `bill-bulk-sync-integration_billpay_create`)

**Purpose:** Bulk sync multiple bills to integration

**Flow:**

1. **Orchestration:**
   - `bill-bulk-sync-integration` receives list of expense IDs
   - Schedules sub-workflows:
     - For bills not yet synced: `bill-bulk-sync-integration_bill_create`
     - For bills already synced: `bill-bulk-sync-integration_bill_update`
     - For bill payments: `bill-bulk-sync-integration_billpay_create`

2. **Processing:**
   - Each sub-workflow processes one bill
   - Calls appropriate integration workflow:
     - `qbo-bill-create`
     - `qbo-bill-update`
     - `qbo-billpayment-create`

3. **Progress Tracking:**
   - Updates sync status in Bubble
   - User sees progress indicator
   - Final summary of successes/errors

**Related Pages:**
- `pg-bill-list` - Bulk sync action
- `pg-integrations-accounting` - Integration management

**Possible Updates to Expense Fields:**
- `AcctIntID` - QuickBooks Bill ID
- `is_exported` - Export flag
- `date_exported` - Export timestamp
- `error_list_acct_integration` - List of errors
- `IntErrorCode` - Error code

---

### 4. Database Triggers

The Bubble application uses database triggers (Data Thing Changed workflows) to automate responses to bill record changes. These run automatically when expense records are created, updated, or deleted.

#### 4.1 Bill Changelog

**Platform:** Bubble → Xano  
**Trigger:** Any change to expense record  
**Purpose:** Audit trail and external system notification

**Process:**
1. Detect expense record change
2. Capture changed fields
3. POST to Xano changelog endpoint
4. Xano stores historical data

**Xano Endpoint:**
- POST to Xano changelog API
- Stores who, what, when for audit

---

#### 4.2 Bill Created

**Trigger Event:** New expense record created  
**Backend Workflow:** `activity-log-bill-create`  
**Workflow Name:** activity-log-bill-create  
**wf_name:** `activity-log-bill-create`

**Workflow Actions:**
1. Create activity log entry
2. Record:
   - User who created
   - Company
   - Timestamp
   - Initial values
3. May trigger notifications

---

#### 4.3 Bill Deleted

**Trigger Event:** expense status changed to "Deleted"  
**Actions:**
1. **Delete from QuickBooks:**
   - Calls `qbo-bill-delete` workflow
2. **Integration cleanup:**
   - Workflow: `int-bill-all` or integration-specific deletion
   - Removes from all connected systems

---

#### 4.4 Bill Deleted Retain

**Trigger Event:** Alternate deletion trigger  
**Actions:**
1. **Terminate workflow:** Stops further processing
2. **Retain terminal status:** Keeps status as "Deleted"
3. **Purpose:** Prevents cascading deletions in certain scenarios

---

#### 4.5 Bill Submitted

**Trigger Event:** expense status_approval changed to submitted state  
**Actions:**
1. **Update status_approval:** Ensures consistency
2. May trigger approval notifications

---

#### 4.6 Bill Updated & New Approval Flow

**Trigger Event:** Expense modified after initial creation  
**Actions:**

1. **Duplicate Detection:**
   - **Set duplicate alert:**
     - Checks for similar bills:
       - Same vendor
       - Similar amount
       - Similar date
     - If found, set `alert_possible_duplicate` = true
   - **Delete duplicate alert:**
     - If no longer a duplicate, clear flag

2. **Approval Flow Management:**
   - **Backend Workflow:** `approval_flow_create`
   - **Workflow Name:** approval_flow_create
   - **wf_name:** `approval_flow_create`
   - **Triggers when:**
     - Bill amount changes significantly
     - Vendor changes
     - Status changes requiring new approval
   - **Process:**
     - Creates new approval chain
     - Based on updated values
     - May invalidate previous approvals

3. **Calculate Balance Due:**
   - Recalculates `amount_balance_due`:
     - `amount_total` - sum of payments
   - Updates expense record

---

### 5. Bill-Related Pages and Reusable Elements

#### Main Pages

1. **pg-bill-list** (Reusable Element)
   - **Purpose:** Main bill listing/table
   - **Features:**
     - Filter bills by status, vendor, date
     - Search functionality
     - Bulk actions
     - Sorting
   - **Uses:** `f-bills` filter component

2. **pg-bill-inbox** (Reusable Element)
   - **Purpose:** Email-received bills pending acceptance
   - **Features:**
     - List of expense_request records
     - Preview attachments
     - Accept/Reject actions
   - **Workflows:** Calls `bill_create_from_req`

3. **pg-vendor-bills** (Reusable Element)
   - **Purpose:** View bills for specific vendor
   - **Context:** Within vendor detail pages

#### Key Reusable Components

1. **fg-bill-viewDetails** (Floating Group)
   - **Purpose:** Bill detail view and editor
   - **Features:**
     - All bill fields editable
     - Line item management
     - Attachment viewer
     - Approval status display
   - **Contains:**
     - `cmp-bill-lineItems`
     - `cmp-bill-dateInputs`
     - `cmp-bill-billToData`
     - `cmp-bill-shipToData`

2. **w-bill-btnAddNew** (Reusable)
   - **Purpose:** Add bill button/upload trigger
   - **Features:**
     - Manual entry option
     - File upload option
     - Initiates bill creation flows

3. **wf-bill** (Reusable - Workflow Container)
   - **Purpose:** Contains custom bill workflows
   - **Workflows:**
     - duplicate-bill event
     - Other bill-specific custom events

4. **m-bill** (Reusable - Menu)
   - **Purpose:** Bill action menu
   - **Actions:**
     - Edit
     - Delete
     - Duplicate
     - Submit for approval
     - Sync to integration
     - Export
     - Other contextual actions

5. **m-bill-action** (Reusable)
   - **Purpose:** Additional bill actions menu
   - **May be alternate/additional action menu**

6. **cmp-bill-lineItems** (Reusable)
   - **Purpose:** Line item editor component
   - **Features:**
     - Add/edit/delete lines
     - Quantity, description, amount
     - Chart of accounts per line
     - Tax code assignment

7. **table-bill-lines** (Reusable)
   - **Purpose:** Display-only line item table
   - **Used in:** Read-only views, reports

#### Popup/Modal Components

1. **p-bill-upload** (Reusable)
   - **Purpose:** Bill upload modal
   - **Features:**
     - File uploader
     - Manual entry form
     - Initial bill creation

2. **p-bill-delete** (Reusable)
   - **Purpose:** Delete confirmation
   - **Features:**
     - Shows bill details
     - Confirmation buttons
     - Warnings

3. **p-bill-approve** (Reusable)
   - **Purpose:** Approve bill modal
   - **Features:**
     - Approve action
     - Add approval notes
     - May set payment date

4. **p-bill-reject** (Reusable)
   - **Purpose:** Reject bill modal
   - **Features:**
     - Rejection reason input
     - Notify submitter option

5. **p-bill-withdraw** (Reusable)
   - **Purpose:** Withdraw bill from approval
   - **Features:**
     - Submitter can withdraw
     - Returns to Draft status

6. **p-bill-markAsPaid** (Reusable)
   - **Purpose:** Manually mark bill as paid
   - **Features:**
     - Mark as paid flag
     - Payment date input
     - Bypasses payment processing

7. **p-bill-schedulePayment** (Reusable)
   - **Purpose:** Schedule bill payment
   - **Features:**
     - Select payment method
     - Set payment date
     - Creates bill payment record

#### Filter and Search Components

1. **f-bills** (Reusable)
   - **Purpose:** Bill list filter component
   - **Features:**
     - Status filter
     - Date range
     - Vendor filter
     - Amount range
     - Assigned user

2. **f-template** (Reusable)
   - **Purpose:** Template filter (if bills use templates)

#### Approval-Related Components

1. **w-approval-chain-flex** (Reusable)
   - **Purpose:** Display approval chain (new version)
   - **Features:**
     - Shows approval steps
     - Current status per step
     - Approver names
   - **Used in:** `fg-bill-viewDetails`

2. **w-approval-chain-legacy** (Reusable)
   - **Purpose:** Display approval chain (legacy version)

3. **p-approval-addStep-flex** (Reusable)
   - **Purpose:** Add approval step to chain
   - **Context:** Approval rule management

4. **p-approval-removeStep-flex** (Reusable)
   - **Purpose:** Remove approval step

5. **wf-approval-flex** (Reusable - Workflow Container)
   - **Purpose:** Approval workflow container (new)

6. **wf-approval-legacy** (Reusable - Workflow Container)
   - **Purpose:** Approval workflow container (legacy)

#### Settings and Configuration

1. **pg-billpay-settings** (Reusable)
   - **Purpose:** Bill pay feature settings
   - **Features:**
     - Enable/disable billpay
     - Payment method setup
     - Default settings

2. **preferences_bill_management** (Reusable)
   - **Purpose:** Bill management preferences
   - **Features:**
     - Email forwarding setup
     - Default approval rules
     - Export settings

3. **settings_bill_management** (Reusable)
   - **Purpose:** Alternate bill settings page

4. **w-billSettings-createForwardingEmail** (Reusable)
   - **Purpose:** Create bill forwarding email address
   - **Features:**
     - Generate unique email
     - Configure forwarding rules

#### Export Components

1. **p-export-bills** (Reusable)
   - **Purpose:** Export bills to CSV/Excel
   - **Features:**
     - Select date range
     - Filter by status/vendor
     - Choose columns

2. **icon-bills-markAsExported** (Reusable)
   - **Purpose:** Icon/button to mark bills as exported
   - **Action:** Sets `is_exported` flag

#### Badge and Status Components

1. **bdg_status_bill-dep** (Reusable - Deprecated)
   - **Purpose:** Bill status badge display
   - **Shows:** Visual indicator of status

2. **cmp-badge-main** (Reusable)
   - **Purpose:** Main badge component (may display bill status)

#### Drawer Components

1. **drawer-bill-logs** (Reusable)
   - **Purpose:** View bill activity/changelog
   - **Features:**
     - Show history of changes
     - Who made changes
     - Timestamps

---

### 6. Bill Status Flow

**Option Set:** `Status | Bill` (`bill_status`)

**Status Values:**
1. **Draft** - Initial creation, not submitted
2. **Submitted** - Sent for approval
3. **Approved** - Approved by all required approvers
4. **Paid** - Payment completed
5. **Partially Paid** - Some payment made, balance remains
6. **Scheduled** - Payment scheduled but not yet sent
7. **Rejected** - Declined by approver
8. **Void** - Cancelled/voided
9. **Deleted** - Soft deleted

**Approval Status Option Set:** `Status | Bill Approval` (`bill_approval_status`)

**Approval Status Values:**
1. **Pending** - Awaiting approval
2. **In Progress** - Some approvers have acted
3. **Approved** - All required approvals received
4. **Declined** - Rejected by an approver
5. **Withdrawn** - Submitter withdrew request

---

### 7. Bill Approval Workflows

(Note: Approval workflows are extensive and could be a separate section. Key workflows are listed here.)

#### Key Approval Backend Workflows

1. **bill_submitted_for_approval** (wf_name: `bill_submitted_for_approval`)
   - Submits bill for approval
   - Creates approval flow

2. **approval-flow-router** (hyphenated)
   - Routes bill to appropriate approval chain

3. **approval_flow_create** (wf_name: `approval_flow_create`)
   - Creates new approval flow for bill

4. **bill_approved** (wf_name: `bill_approved`)
   - Handles final approval
   - Transitions to Approved status

5. **bill_approval_declined** (wf_name: `bill_approval_declined`)
   - Handles rejection
   - Notifies submitter

6. **bill_create_approval_chain** (wf_name: `bill_create_approval_chain`)
   - Specific workflow for creating approval chain

#### Approval Pages

1. **pg-approvalRequests-list** (Reusable)
   - Lists approval requests for current user

2. **pg-approvalRule-list** (Reusable)
   - Manages approval rules

3. **pg-approvalRule-settings** (Reusable)
   - Configure approval rule settings

4. **f-approval-request** (Reusable)
   - Filter component for approval requests

---

### 8. Backend Workflow Summary (Bills)

**Workflow Naming Convention:**
- **Underscores (_):** Standard Bubble workflows (e.g., `bill_update`)
- **Hyphens (-):** Some workflows use hyphens (e.g., `bill-delete`)
- **wf_name:** API endpoint slug, usually matches workflow name

#### Core Bill Workflows

| Workflow Name | wf_name | Purpose | Platform |
|---------------|---------|---------|----------|
| bill_create | bill_create | Create new bill | Bubble |
| bill_update | bill_update | Update bill from Xano parse | Bubble |
| bill_item_create | bill_item_create | Create line item | Bubble |
| bill-delete | bill-delete | Delete bill (soft delete) | Bubble |
| bill-duplicateBill | bill-duplicateBill | Duplicate single bill | Bubble |
| bill-duplicateListOfBills | bill-duplicateListOfBills | Bulk duplicate orchestrator | Bubble |
| bill_submitted_for_approval | bill_submitted_for_approval | Submit for approval | Bubble |
| bill_approved | bill_approved | Bill approved | Bubble |
| bill_approval_declined | bill_approval_declined | Bill rejected | Bubble |
| bill_create_from_req | bill_create_from_req | Create from inbox request | Bubble |
| request_bill | request_bill | Create bill request from email | Bubble |
| bill_create_approval_chain | bill_create_approval_chain | Create approval chain | Bubble |
| bill_mark_as_paid | bill_mark_as_paid | Manually mark paid | Bubble |
| bill_paid | bill_paid | Handle payment completion | Bubble |
| bill_export | bill_export | Export bills | Bubble |
| bill_export_create_file | bill_export_create_file | Generate export file | Bubble |
| bill-by-company | bill-by-company | Get bills for company | Bubble |
| bill_info | bill_info | Get bill details | Bubble |
| mobile_bill_get | mobile_bill_get | Mobile app bill retrieval | Bubble |
| BILL /mobile-bill-get-v2 | (API endpoint) | Mobile app bill v2 | Bubble |
| bill-line-create | bill-line-create | Create bill line | Bubble |
| bill-lines-process | bill-lines-process | Process line items | Bubble |
| bill-set-list-of-line-items | bill-set-list-of-line-items | Set line item list | Bubble |
| activity-log-bill-create | (triggered) | Log bill creation | Bubble |
| bill-approval-history | bill-approval-history | Get approval history | Bubble |

#### Integration Workflows (QuickBooks)

| Workflow Name | wf_name | Purpose | Platform |
|---------------|---------|---------|----------|
| qbo-bill-create | qbo-bill-create | Create bill in QBO | Bubble |
| qbo-bill-update | qbo-bill-update | Update bill in QBO | Bubble |
| qbo-bill-delete | qbo-bill-delete | Delete bill from QBO | Bubble |
| bill-bulk-sync-integration | bill-bulk-sync-integration | Bulk sync orchestrator | Bubble |
| bill-bulk-sync-integration_bill_create | bill-bulk-sync-integration_bill_create | Bulk create in integration | Bubble |
| bill-bulk-sync-integration_bill_update | bill-bulk-sync-integration_bill_update | Bulk update in integration | Bubble |
| z_251020_bill-bulk-sync-integration | (legacy/test) | Legacy bulk sync | Bubble |

#### Approval Workflows

| Workflow Name | wf_name | Purpose | Platform |
|---------------|---------|---------|----------|
| approval-flow-router | (hyphenated) | Route to approval chain | Bubble |
| approval_flow_create | approval_flow_create | Create approval flow | Bubble |

#### Backfill/Admin Workflows

| Workflow Name | wf_name | Purpose | Platform |
|---------------|---------|---------|----------|
| &backfill_export_bill_1 | &backfill_export_bill_1 | Backfill export data (1) | Bubble |
| &backfill_export_bill_2 | &backfill_export_bill_2 | Backfill export data (2) | Bubble |

---

### 9. Xano API Endpoints (Bills)

**Base URL:** `https://xano.kleercard.com`

#### Bill Parsing

| Endpoint | Method | Purpose |
|----------|--------|---------|
| /api:[namespace]/bill/parse | POST | Parse bill document via Photon |

**Note:** Specific API namespace may vary per environment.

---

### 10. Third-Party Services (Bills)

#### Photon (Receipt/Bill Parsing)
- **Purpose:** OCR and data extraction from bill PDFs/images
- **Used In:** Bill upload and email ingestion flows
- **Fields Extracted:**
  - Vendor information
  - Invoice number and dates
  - Line items (description, quantity, price)
  - Amounts (subtotal, tax, total)
  - Payment terms
  - Addresses

#### ForwardMX
- **Purpose:** Email forwarding service
- **Used In:** Bill email ingestion
- **Function:** Routes emails to Mailparser

#### Mailparser
- **Purpose:** Email parsing service
- **Used In:** Bill email ingestion
- **Extracts:** Email metadata, attachments, sender info
- **Webhook:** Sends parsed data to Xano

---

## BillPay Section

### 1. BillPay Overview

The BillPay feature enables companies to pay approved bills electronically through ACH or check payments. The system integrates with Dwolla (payment processor) to execute payments to vendors.

**Key Features:**
- Schedule bill payments
- ACH and check payment methods
- Vendor payment method management
- Payment status tracking
- Integration with accounting systems

---

### 2. BillPay Data Model

#### BillPayment (Data Type: `billpayment`)
**Display Name:** BillPayment  
**Platform:** Bubble  
**Purpose:** Represents a payment transaction for one or more bills

**Key Relationships:**
- Links to multiple bills via `BillPayment_item` records
- Associated with payment method (`payment_method`)
- Company reference
- Dwolla transaction tracking

**Status Option Set:** `Status | Bill Payment` (`bill_payment_status`)

**Status Values:**
- Scheduled - Payment scheduled but not processed
- Pending - Payment initiated, awaiting processing
- Processing - Payment in progress
- Completed - Payment successfully sent
- Failed - Payment failed
- Cancelled - Payment cancelled before sending

#### BillPayment_item (Data Type: `billpaymentcharge`)
**Display Name:** BillPayment_item  
**Platform:** Bubble  
**Purpose:** Links bills to payments (many-to-many relationship)

**Key Fields:**
- Link to parent `BillPayment`
- Link to `expense` (bill)
- Amount allocated to this bill
- Payment date
- Status

#### payment_method (Data Type: `billpayee`)
**Display Name:** payment_method  
**Platform:** Bubble  
**Purpose:** Vendor payment method (bank account, check address)

**Key Fields:**
- Link to `vendor`
- Payment method type (ACH, check, wire)
- Bank account details (encrypted)
- Routing number
- Account number
- Check mailing address
- Dwolla funding source ID
- Status (active, pending verification, inactive)

**Type Option Set:** `Type of Payment Method` (`type_of_bill_payment`)

**Type Values:**
- ACH
- Check
- Wire Transfer
- Credit Card (if supported)

---

### 3. BillPay Creation Flows

#### 3.1 Schedule Payment from Bill

**Popup:** `p-bill-schedulePayment` (reusable element)  
**Trigger:** User clicks "Schedule Payment" on approved bill  
**Platform:** Bubble → Xano → Dwolla

**Flow:**

**Step 1: User Initiates Payment**
- **Page Context:** Bill detail view (`fg-bill-viewDetails`)
- **Button:** "Schedule Payment" (only visible on Approved bills)
- **Popup Opens:** `p-bill-schedulePayment`

**Step 2: Select Payment Details**
- **Popup Displays:**
  - Bill details (vendor, amount, due date)
  - Payment method dropdown:
    - Lists vendor's payment methods
    - Filter by active status
  - Payment date picker:
    - Minimum: Today
    - Default: Bill due date
  - Payment amount (pre-filled, may be editable)
  - Payment memo/notes

**Step 3: Validate Payment Method**
- Check if vendor has payment method:
  - If NO: Prompt to create payment method
    - Link to `p-payment-method` popup
  - If YES: Verify status is Active
    - If Pending: Show verification required
    - If Inactive: Show error

**Step 4: Create Bill Payment**
- **Backend Workflow:** `billpayment_create`
- **Workflow Name:** billpayment_create
- **wf_name:** `billpayment_create`

**Workflow Actions in `billpayment_create`:**

1. **Create BillPayment Record:**
   - Create new `BillPayment` (Bubble):
     - `_company` = Bill's company
     - `payment_method` = Selected payment method
     - `_vendor` = Bill's vendor
     - Status = "Scheduled"
     - `payment_date` = Selected date
     - `amount_total` = Payment amount
     - Created by = Current user

2. **Create BillPayment_item:**
   - Create `BillPayment_item`:
     - Link to `BillPayment`
     - Link to `expense` (bill)
     - `amount` = Allocated amount
     - May partially pay bill

3. **Update Bill Status:**
   - Modify `expense`:
     - If full payment: Status = "Scheduled"
     - If partial: Status = "Partially Scheduled"
     - `amount_balance_due` -= payment amount

4. **Schedule Payment Processing:**
   - **Backend Workflow:** `billpayment_schedule`
   - **Workflow Name:** billpayment_schedule
   - **wf_name:** `billpayment_schedule`
   - **Scheduled for:** Payment date at processing time (e.g., 6 AM)

**Data Created:**
- `BillPayment` record (Bubble)
- `BillPayment_item` record(s) (Bubble)
- Updated `expense` status

**Backend Workflows Involved:**
- **billpayment_create** (wf_name: `billpayment_create`) - Create payment
- **billpayment_schedule** (wf_name: `billpayment_schedule`) - Schedule processing

**Related Reusables:**
- `p-bill-schedulePayment` - Schedule payment popup
- `p-payment-method` - Add/edit payment method

**Status Progression:**
- Bill: Approved → Scheduled (or Partially Scheduled)
- Payment: Created with status Scheduled

---

#### 3.2 Create Payment from BillPay Page

**Page:** `pg-billpay` (reusable element)  
**Popup:** `p-billpayment-create` (reusable element)  
**Trigger:** User navigates to BillPay and clicks "Create Payment"  
**Platform:** Bubble

**Flow:**

**Step 1: Navigate to BillPay**
- **Page:** `pg-billpay`
- **Features:**
  - List of all bill payments
  - Filter by status, vendor, date
  - "Create Payment" button

**Step 2: Open Create Payment Popup**
- **Popup:** `p-billpayment-create`
- **Displays:**
  - Vendor dropdown/search
  - Bill selector (multi-select):
    - Shows Approved bills for selected vendor
    - Displays amount, due date, invoice number
  - Payment method dropdown
  - Payment date picker
  - Total payment amount (calculated)

**Step 3: Select Bills to Pay**
- User selects one or more bills
- May pay partially or in full for each
- Total amount calculated automatically

**Step 4: Create Payment**
- **Backend Workflow:** `billpayment_create` (same as section 3.1)
- **Additional:** May create multiple `BillPayment_item` records
- Updates each bill's `amount_balance_due`

**Backend Workflows Involved:**
- **billpayment_create** (wf_name: `billpayment_create`)

**Related Pages:**
- `pg-billpay` - Main BillPay page
- `p-billpayment-create` - Create payment popup

---

#### 3.3 Bulk Payment Creation

**Page:** `pg-bill-list`  
**Trigger:** Select multiple approved bills, bulk action "Pay Bills"  
**Backend Workflow:** Potentially custom bulk payment workflow

**Flow:**

1. **Select Multiple Bills:**
   - User checks multiple bills in list
   - Bills must be Approved status
   - Same vendor recommended (or group by vendor)

2. **Bulk Pay Action:**
   - Opens payment modal
   - Groups bills by vendor
   - Creates one payment per vendor

3. **Process:**
   - Calls `billpayment_create` for each vendor group
   - Creates `BillPayment_item` for each bill

**Note:** Specific bulk payment workflow may exist, needs confirmation

---

### 4. Payment Processing Flows

#### 4.1 Scheduled Payment Processing

**Backend Workflow:** `billpayment_schedule`  
**Workflow Name:** billpayment_schedule  
**wf_name:** `billpayment_schedule`  
**Trigger:** Scheduled workflow runs on payment date  
**Platform:** Bubble → Xano → Dwolla

**Flow:**

**Step 1: Retrieve Scheduled Payments**
- Workflow runs daily (e.g., 6 AM)
- Query `BillPayment` records:
  - Status = "Scheduled"
  - `payment_date` = Today

**Step 2: Validate Payment**
- For each payment:
  - Check company has sufficient funds
  - Verify payment method is still active
  - Confirm bills still Approved
  - Check for any holds or restrictions

**Step 3: Initiate Payment**
- **Backend Workflow:** `billpay_send_ach` (if ACH)
- **Workflow Name:** billpay_send_ach
- **wf_name:** `billpay_send_ach`
- **Or:** Check printing workflow (if check payment)

**Workflow Actions in `billpay_send_ach`:**

1. **Update Payment Status:**
   - Modify `BillPayment`:
     - Status = "Processing"
     - Processing timestamp

2. **Call Xano Payment Service:**
   - **API Call:** POST to Xano payment endpoint
   - **Endpoint:** May be `/api:billpay/ach` or similar
   - **Payload:**
     ```json
     {
       "billpayment_bubble_id": "<BillPayment unique ID>",
       "payment_method_bubble_id": "<payment_method unique ID>",
       "amount": <amount_total>,
       "memo": "<payment notes>",
       "company_id": "<company bubble ID>"
     }
     ```

**Step 4: Xano Processing**
- **Platform:** Xano
- **Process:**
  1. Retrieve payment details from Bubble
  2. Get company's Dwolla customer ID
  3. Get vendor's Dwolla funding source ID
  4. Prepare Dwolla transfer request

**Step 5: Call Dwolla API**
- **Service:** Dwolla (3rd party payment processor)
- **API Call:** POST to Dwolla `/transfers`
- **Endpoint:** `https://api.dwolla.com/transfers` (or sandbox)
- **Auth:** OAuth token (managed by Xano)
- **Payload:**
  ```json
  {
    "_links": {
      "source": {
        "href": "https://api.dwolla.com/funding-sources/<company_funding_source_id>"
      },
      "destination": {
        "href": "https://api.dwolla.com/funding-sources/<vendor_funding_source_id>"
      }
    },
    "amount": {
      "currency": "USD",
      "value": "<amount>"
    },
    "metadata": {
      "billpayment_id": "<BillPayment unique ID>",
      "memo": "<payment memo>"
    }
  }
  ```

**Step 6: Dwolla Response**
- **Success:**
  - Dwolla returns transfer ID
  - Transfer status: "pending"
- **Failure:**
  - Error code and message

**Step 7: Xano Callback to Bubble**
- **Backend Workflow:** `billpayment_update`
- **Workflow Name:** billpayment_update
- **wf_name:** `billpayment_update`
- **Trigger:** Xano webhook or immediate API response

**Workflow Actions in `billpayment_update`:**

1. **Update BillPayment:**
   - Modify `BillPayment`:
     - If success:
       - Status = "Pending" (awaiting Dwolla confirmation)
       - Dwolla transfer ID stored
     - If failure:
       - Status = "Failed"
       - Error message stored
       - Calls `bill_payment_error` workflow

2. **Update Bills:**
   - For each `BillPayment_item`:
     - If payment successful:
       - Update related `expense`:
         - Status = "Pending Payment"
     - If payment failed:
       - Revert `expense` status to "Approved"
       - Restore `amount_balance_due`

**Step 8: Dwolla Webhook (Payment Completion)**
- **Service:** Dwolla sends webhook to Xano
- **Webhook Types:**
  - `customer_transfer_completed` - Payment successful
  - `customer_transfer_failed` - Payment failed
  - `customer_transfer_cancelled` - Payment cancelled

**Step 9: Xano Processes Dwolla Webhook**
- **Platform:** Xano
- **Endpoint:** Xano webhook receiver for Dwolla events
- **Process:**
  1. Verify webhook signature
  2. Parse transfer status
  3. Call Bubble to update payment

**Step 10: Final Update in Bubble**
- **Backend Workflow:** `bill-payment-complete` or `billpayment_update`
- **Workflow Name:** bill-payment-complete
- **wf_name:** `bill-payment-complete`

**Workflow Actions in `bill-payment-complete`:**

1. **Update BillPayment:**
   - Modify `BillPayment`:
     - Status = "Completed" (or "Failed")
     - Completion date
     - Final Dwolla status

2. **Update Bills:**
   - For each `BillPayment_item`:
     - Update related `expense`:
       - Status = "Paid"
       - `payment_date` = Completion date
       - `mark_as_paid` = true
       - `amount_balance_due` = 0 (if fully paid)

3. **Trigger Bill Paid Workflow:**
   - **Backend Workflow:** `bill_paid`
   - **Workflow Name:** bill_paid
   - **wf_name:** `bill_paid`
   - **Actions:**
     - May sync to accounting integration
     - Send payment confirmation email
     - Update reporting/analytics

4. **Send Notifications:**
   - Email to bill submitter: Payment completed
   - Email to vendor (optional): Payment sent
   - In-app notification

**Data Updated:**
- `BillPayment`: Status = Completed/Failed
- `expense`: Status = Paid (or reverted if failed)
- `BillPayment_item`: Status updated
- Dwolla transfer ID stored

**Backend Workflows Involved:**
- **billpayment_schedule** (wf_name: `billpayment_schedule`) - Scheduled orchestrator
- **billpay_send_ach** (wf_name: `billpay_send_ach`) - ACH payment initiation
- **billpayment_update** (wf_name: `billpayment_update`) - Payment status updates
- **bill-payment-complete** (wf_name: `bill-payment-complete`) - Finalization
- **bill_paid** (wf_name: `bill_paid`) - Bill paid handler
- **bill_payment_error** (wf_name: `bill_payment_error`) - Error handler

**Related Workflows (Check Payments):**
- May have separate workflow for check payments
- Generates check PDF
- Marks for mailing

**Integration:**
- May sync to QuickBooks via `qbo-billpayment-create`

**Status Progression:**
- Payment: Scheduled → Processing → Pending → Completed/Failed
- Bill: Approved → Scheduled → Pending Payment → Paid

---

#### 4.2 Check Payment Processing

**Note:** Check payments follow similar flow but:
- Generate check PDF instead of Dwolla transfer
- Mark as "Pending Mailing"
- User confirms mailing
- Status updates to "Mailed" then "Cleared"

**Potential Workflows:**
- Check generation workflow
- Check printing workflow
- Check tracking workflow

**Related Fields:**
- `check_number` on expense
- `qbo_print_check` flag for QuickBooks integration

---

#### 4.3 Payment Failure Handling

**Backend Workflow:** `bill_payment_error`  
**Workflow Name:** bill_payment_error  
**wf_name:** `bill_payment_error`  
**Trigger:** Payment fails at any stage

**Workflow Actions:**

1. **Update Payment Status:**
   - Modify `BillPayment`:
     - Status = "Failed"
     - Error message
     - Error code

2. **Revert Bill Status:**
   - Update `expense`:
     - Status back to "Approved"
     - Restore `amount_balance_due`

3. **Notify Users:**
   - Email to payment creator
   - Email to AP team
   - In-app notification
   - Error details provided

4. **Log Error:**
   - Create error log entry
   - Store full error details for troubleshooting

5. **Retry Logic (Optional):**
   - May automatically retry
   - Or flag for manual review

**Common Failure Reasons:**
- Insufficient funds
- Invalid payment method
- Bank account closed
- Vendor account issues
- Network/API errors
- Compliance holds

**User Actions After Failure:**
- Review error details
- Update payment method if needed
- Reschedule payment
- Or cancel and recreate

---

### 5. Payment Management Flows

#### 5.1 View Payment Details

**Page:** `pg-billpayment-detail` (reusable element)  
**Popup:** `p-billpayment-details` (reusable element)  
**Trigger:** User clicks on payment from list  
**Platform:** Bubble

**Displays:**
- **Payment Information:**
  - Payment ID
  - Status (with badge)
  - Amount
  - Payment date
  - Created by, created date
  - Payment method details

- **Associated Bills:**
  - Table of `BillPayment_item` records
  - Shows each bill:
    - Vendor name
    - Invoice number
    - Amount allocated
    - Bill status
  - Link to bill details

- **Transaction Details:**
  - Dwolla transfer ID (if available)
  - Transaction timeline
  - Status updates history

- **Actions:**
  - Cancel payment (if not yet processed)
  - Modify date (if scheduled)
  - View in QuickBooks (if synced)
  - Download payment confirmation

**Related Reusables:**
- `pg-billpayment-detail` - Detail page
- `p-billpayment-details` - Detail popup
- `w-card-detail-billpayment` - Card/widget displaying payment info
- `drawer-payment-details` - Drawer component for payment details

---

#### 5.2 Modify Payment Date

**Popup:** `p-billpayment-modifyDate` (reusable element)  
**Trigger:** User clicks "Change Date" on scheduled payment  
**Platform:** Bubble

**Flow:**

1. **Validate:**
   - Payment must be in "Scheduled" status
   - Cannot modify if already processing

2. **Open Popup:**
   - Shows current date
   - Date picker for new date
   - Minimum: Today

3. **Update Payment:**
   - **Backend Workflow:** `billpayment_update`
   - Modify `BillPayment`:
     - `payment_date` = New date
   - Reschedule `billpayment_schedule` workflow

4. **Update Bills:**
   - May update bill `payment_date` field

**Backend Workflows Involved:**
- **billpayment_update** (wf_name: `billpayment_update`)

**Related Reusables:**
- `p-billpayment-modifyDate` - Date modification popup

---

#### 5.3 Cancel Payment

**Popup:** `p-billpayment-cancel` (reusable element)  
**Trigger:** User clicks "Cancel" on scheduled/pending payment  
**Platform:** Bubble → Xano → Dwolla (if needed)

**Flow:**

**Step 1: Validate Cancellation**
- Check payment status:
  - Can cancel: Scheduled, Pending (early stage)
  - Cannot cancel: Processing, Completed
- Get confirmation from user

**Step 2: Cancel in Dwolla (if needed)**
- If payment already initiated with Dwolla:
  - Call Dwolla API to cancel transfer
  - May not be possible if too far along

**Step 3: Update Payment**
- **Backend Workflow:** `billpayment_update` or specific cancel workflow
- Modify `BillPayment`:
  - Status = "Cancelled"
  - Cancellation date
  - Cancelled by user

**Step 4: Revert Bills**
- For each `BillPayment_item`:
  - Update related `expense`:
    - Status back to "Approved"
    - Restore `amount_balance_due`
    - Clear `payment_date`

**Step 5: Notify Users**
- Email to payment creator: Cancellation confirmed
- May notify AP team

**Backend Workflows Involved:**
- **billpayment_update** (wf_name: `billpayment_update`)
- Possible specific cancellation workflow

**Related Reusables:**
- `p-billpayment-cancel` - Cancellation confirmation popup
- Alternate: `z_p-billpayment-cancel` - Possibly legacy version

---

#### 5.4 Payment Method Management

**Page:** `pg-vendor-payment-methods` (reusable element)  
**Popup:** `p-payment-method` (reusable element)  
**Trigger:** User manages vendor payment methods  
**Platform:** Bubble → Xano → Dwolla

**Add/Edit Payment Method Flow:**

**Step 1: Open Payment Method Popup**
- **Popup:** `p-payment-method`
- **Context:** Within vendor detail page
- **Forms:**
  - Payment method type selector (ACH, check)
  - **If ACH:**
    - Bank name
    - Routing number
    - Account number
    - Account type (checking/savings)
  - **If Check:**
    - Mailing address
    - Check payable to name

**Step 2: Create Payment Method**
- Create `payment_method` (Bubble):
  - Link to `vendor`
  - Type (ACH or check)
  - Store bank details (encrypted)
  - Status = "Pending Verification" (if ACH)

**Step 3: Create Dwolla Funding Source (ACH only)**
- **API Call:** POST to Xano Dwolla service
- **Endpoint:** Create Funding Source
- **API Connector:** Dwolla (Xano) > Create Funding Source
- **Payload:**
  ```json
  {
    "payment_method_bubble_id": "<payment_method unique ID>",
    "routing_number": "<routing>",
    "account_number": "<account>",
    "account_type": "checking/savings",
    "name": "<bank account name>"
  }
  ```

**Step 4: Xano/Dwolla Processing**
- Xano calls Dwolla API
- Dwolla creates funding source
- Returns funding source ID

**Step 5: Update Payment Method**
- **Backend Workflow:** Response handler or `billpayment_update`
- Update `payment_method`:
  - Store Dwolla funding source ID
  - Status = "Pending Verification" (ACH requires micro-deposits)

**Step 6: Micro-Deposit Verification (ACH)**
- Dwolla sends 2 small deposits to bank account
- Vendor receives deposits in 1-3 business days
- Vendor enters deposit amounts to verify

**Step 7: Verify Micro-Deposits**
- **Popup:** User enters two deposit amounts
- **API Call:** POST to Xano Dwolla service
- **API Connector:** Dwolla (Xano) > Micro-deposits
- **Payload:**
  ```json
  {
    "funding_source_id": "<dwolla_funding_source_id>",
    "amount1": <deposit_amount_1>,
    "amount2": <deposit_amount_2>
  }
  ```

**Step 8: Verification Complete**
- Dwolla verifies amounts
- Update `payment_method`:
  - Status = "Active"
- Now eligible for payments

**Remove Payment Method:**
- **Popup:** Confirmation
- **Backend Workflow:** May call Dwolla to remove funding source
- **API Connector:** Dwolla (Xano) > Remove Funding Source
- Update `payment_method`:
  - Status = "Inactive"
  - Or soft delete

**Backend Workflows Involved:**
- Payment method creation/update workflows
- Dwolla funding source management workflows

**Related Reusables:**
- `p-payment-method` - Add/edit payment method popup
- `pg-vendor-payment-methods` - Payment methods list page
- `input_payment_account` or `input-payment-account` - Payment account input
- `m-payment-method` - Payment method action menu

**API Connectors (Xano/Dwolla):**
- **Create Funding Source** - Add bank account to Dwolla
- **Micro-deposits** - Initiate or verify micro-deposits
- **Remove Funding Source** - Deactivate bank account

---

### 6. BillPay Integration

#### 6.1 QuickBooks Online Bill Payment Sync

**Backend Workflow:** `qbo-billpayment-create`  
**Workflow Name:** qbo-billpayment-create  
**wf_name:** `qbo-billpayment-create`  
**Trigger:** Bill payment completed  
**Platform:** Bubble → Xano → QuickBooks Online API

**Flow:**

1. **Trigger:**
   - After `BillPayment` status = "Completed"
   - May be automatic or manual sync

2. **Validate:**
   - Check integration is connected
   - Verify bill was synced to QBO (has `AcctIntID`)
   - Check payment method is mapped

3. **Map Payment Account:**
   - Identify QBO bank account for payment
   - Use company's payment account mapping

4. **Prepare QBO Payload:**
   - Build JSON for QBO BillPayment API:
     ```json
     {
       "VendorRef": {"value": "<qbo_vendor_id>"},
       "PayType": "Check" or "CreditCard",
       "TxnDate": "<payment_date>",
       "TotalAmt": <amount_total>,
       "Line": [
         {
           "Amount": <line_amount>,
           "LinkedTxn": [
             {
               "TxnId": "<qbo_bill_id>",
               "TxnType": "Bill"
             }
           ]
         }
       ],
       "BankAccountRef": {"value": "<qbo_account_id>"}
     }
     ```

5. **Call QBO API:**
   - **Method:** POST
   - **Endpoint:** `/v3/company/<realmId>/billpayment`

6. **Handle Response:**
   - **Success:**
     - Store QBO BillPayment ID
     - Mark bills as paid in QBO
   - **Failure:**
     - Log error
     - May retry

**Backend Workflows Involved:**
- **qbo-billpayment-create** (wf_name: `qbo-billpayment-create`)
- **bill-bulk-sync-integration_billpay_create** - Bulk sync variant

---

### 7. BillPay-Related Pages and Reusable Elements

#### Main Pages

1. **pg-billpay** (Reusable Element)
   - **Purpose:** Main bill payments listing
   - **Features:**
     - List of all payments
     - Filter by status, vendor, date
     - Search functionality
     - Create payment button
   - **Uses:** `f-bill-payments` filter component

2. **pg-billpay-marketing** (Reusable Element)
   - **Purpose:** Marketing page for BillPay feature
   - **Audience:** Companies not yet using BillPay
   - **Features:**
     - Feature overview
     - Benefits
     - Enable BillPay button

3. **pg-billpay-settings** (Reusable Element)
   - **Purpose:** BillPay settings and configuration
   - **Features:**
     - Enable/disable BillPay
     - Default payment account
     - Payment approval settings
     - Notification preferences

4. **pg-billpayment-detail** (Reusable Element)
   - **Purpose:** Payment detail view
   - **Features:**
     - Full payment information
     - Associated bills
     - Transaction history
     - Actions (cancel, modify)

5. **pg-vendor-payment-methods** (Reusable Element)
   - **Purpose:** Vendor payment methods list
   - **Context:** Within vendor detail pages

#### Key Reusable Components

1. **wf-billPayment** (Reusable - Workflow Container)
   - **Purpose:** Contains BillPay workflows
   - **Workflows:**
     - Payment-related custom events

2. **w-card-detail-billpayment** (Reusable)
   - **Purpose:** Payment card/widget display
   - **Features:**
     - Summary information
     - Status badge
     - Quick actions

3. **drawer-payment-details** (Reusable)
   - **Purpose:** Payment details drawer
   - **Features:**
     - Side panel with full details
     - Transaction timeline

#### Popup/Modal Components

1. **p-billpayment-create** (Reusable)
   - **Purpose:** Create new payment
   - **Features:**
     - Vendor selection
     - Bill selection (multi-select)
     - Payment method dropdown
     - Date picker

2. **p-billpayment-details** (Reusable)
   - **Purpose:** Payment details popup
   - **Features:**
     - View full payment info
     - Associated bills
     - Actions

3. **p-billpayment-cancel** (Reusable)
   - **Purpose:** Cancel payment confirmation
   - **Features:**
     - Shows payment details
     - Cancellation warning
     - Confirm/cancel buttons

4. **z_p-billpayment-cancel** (Reusable)
   - **Purpose:** Possibly legacy cancel popup

5. **p-billpayment-modifyDate** (Reusable)
   - **Purpose:** Change payment date
   - **Features:**
     - Current date display
     - New date picker

6. **p-billpay-enableBillpayFeature** (Reusable)
   - **Purpose:** Enable BillPay for company
   - **Features:**
     - Feature explanation
     - Terms acceptance
     - Setup wizard

7. **p-payment-method** (Reusable)
   - **Purpose:** Add/edit payment method
   - **Features:**
     - Bank account form
     - Check address form
     - Verification steps

8. **p-billpayment-account** (Reusable)
   - **Purpose:** Select payment account
   - **Features:**
     - Company bank account dropdown

#### Filter and Table Components

1. **f-bill-payments** (Reusable)
   - **Purpose:** Payment list filters
   - **Features:**
     - Status filter
     - Date range
     - Vendor filter
     - Amount range

2. **table-payments** (Reusable)
   - **Purpose:** Payment list table
   - **Features:**
     - Sortable columns
     - Status badges
     - Action buttons

3. **table-payment-lines** (Reusable)
   - **Purpose:** Payment line items table
   - **Features:**
     - Associated bills
     - Amounts per bill

#### Settings Components

1. **pg-billpay-settings** (Reusable)
   - **Purpose:** BillPay configuration
   - **Features:**
     - Enable/disable feature
     - Payment defaults
     - Approval rules

2. **w-billpay-account-type** (Reusable)
   - **Purpose:** Bank account type selector
   - **Options:**
     - Checking
     - Savings

#### Status and Badge Components

1. **bdg_status_bill_payment-dep** (Reusable - Deprecated)
   - **Purpose:** Payment status badge display

2. **bdg_payment_method-dep** (Reusable - Deprecated)
   - **Purpose:** Payment method badge

#### Input Components

1. **input_payment_account** (Reusable)
   - **Purpose:** Payment account input/selector

2. **input-payment-account** (Reusable)
   - **Purpose:** Possibly alternate version

---

### 8. BillPay Status Flow

**Payment Status Option Set:** `Status | Bill Payment` (`bill_payment_status`)

**Status Values:**
1. **Scheduled** - Payment created, scheduled for future date
2. **Processing** - Payment initiated, sent to processor
3. **Pending** - Payment sent to bank, awaiting clearance
4. **Completed** - Payment successfully completed
5. **Failed** - Payment failed
6. **Cancelled** - Payment cancelled before processing
7. **Returned** - Payment returned/rejected by bank

**Status Transitions:**
- Scheduled → Processing (on payment date)
- Processing → Pending (sent to Dwolla)
- Pending → Completed (Dwolla webhook confirms)
- Pending → Failed (Dwolla webhook indicates failure)
- Scheduled/Processing → Cancelled (user cancellation)

---

### 9. Backend Workflow Summary (BillPay)

#### Core BillPay Workflows

| Workflow Name | wf_name | Purpose | Platform |
|---------------|---------|---------|----------|
| billpayment_create | billpayment_create | Create bill payment | Bubble |
| billpayment_update | billpayment_update | Update payment status | Bubble |
| billpayment_schedule | billpayment_schedule | Schedule payment processing | Bubble |
| billpay_send_ach | billpay_send_ach | Send ACH payment via Dwolla | Bubble |
| bill_payment_ach | bill_payment_ach | Handle ACH payment | Bubble |
| bill-payment-complete | bill-payment-complete | Finalize completed payment | Bubble |
| bill_paid | bill_paid | Handle bill paid event | Bubble |
| bill_payment_mark_as_paid | bill_payment_mark_as_paid | Manually mark bill paid | Bubble |
| bill_mark_as_paid | bill_mark_as_paid | Mark bill as paid (alternate) | Bubble |
| bill_payment_error | bill_payment_error | Handle payment errors | Bubble |
| bill_payment_check_update | bill_payment_check_update | Update check payment status | Bubble |
| billpayment_by_company | billpayment_by_company | Get payments by company | Bubble |
| billpayment_companies_to_process | billpayment_companies_to_process | Process pending companies | Bubble |
| billpayment_card_details | billpayment_card_details | Get payment card details | Bubble |
| email_ach_bill_payment_completed | email_ach_bill_payment_completed | Send ACH completion email | Bubble |
| create_billpayment_charge | create_billpayment_charge | Create payment charge | Bubble |
| POST /payment-billpay-create | payment-billpay-create | API endpoint for create | Bubble |
| POST /payment-billpay-update | payment-billpay-update | API endpoint for update | Bubble |

#### Integration Workflows (QuickBooks)

| Workflow Name | wf_name | Purpose | Platform |
|---------------|---------|---------|----------|
| qbo-billpayment-create | qbo-billpayment-create | Create bill payment in QBO | Bubble |
| bill-bulk-sync-integration_billpay_create | bill-bulk-sync-integration_billpay_create | Bulk sync payments to QBO | Bubble |

#### Admin/Processing Workflows

| Workflow Name | wf_name | Purpose | Platform |
|---------------|---------|---------|----------|
| &backfill_billpay_CLEAR_markaspaid | (backfill) | Admin backfill workflow | Bubble |

---

### 10. Xano API Endpoints (BillPay)

**Base URL:** `https://xano.kleercard.com`

#### BillPay Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| /api:[namespace]/billpay/ach | POST | Initiate ACH payment |
| /api:[namespace]/payment | POST | Create payment (Xano Payment API) |
| /api:[namespace]/payment/[bubble_payment_id]/ledger | GET | Get payment ledger |

**Note:** Specific API namespaces may vary per environment.

#### Dwolla Endpoints (via Xano)

**API Connector Group:** Dwolla (Xano)

| Call Name | Method | Purpose |
|-----------|--------|---------|
| Create Funding Source | POST | Add vendor bank account |
| Micro-deposits | POST | Initiate or verify micro-deposits |
| Remove Funding Source | DELETE | Deactivate bank account |

---

### 11. Third-Party Services (BillPay)

#### Dwolla (Payment Processor)
- **Purpose:** ACH payment processing
- **Used In:** All electronic bill payments
- **Functions:**
  - Customer account management
  - Funding source management (bank accounts)
  - Transfer/payment processing
  - Micro-deposit verification
  - Webhook notifications

**Dwolla Integration Flow:**
1. Company onboards to Dwolla (during setup)
2. Vendors add bank accounts (funding sources)
3. Payments initiated via Dwolla API
4. Dwolla processes ACH transfers
5. Webhooks notify of status changes

**Dwolla Status Events:**
- `customer_transfer_created`
- `customer_transfer_pending`
- `customer_transfer_completed`
- `customer_transfer_failed`
- `customer_transfer_cancelled`

---

### 12. BillPay Settings and Configuration

#### Enable BillPay Feature

**Popup:** `p-billpay-enableBillpayFeature`  
**Trigger:** Company wants to start using BillPay  
**Flow:**

1. **User Navigates to BillPay:**
   - If not enabled, sees marketing page (`pg-billpay-marketing`)

2. **Enable Feature:**
   - Opens `p-billpay-enableBillpayFeature` popup
   - User reviews terms
   - Accepts agreement

3. **Company Setup:**
   - Update `company_setting_bill` or `company_setting`:
     - BillPay enabled flag = true
   - May trigger Dwolla customer onboarding
   - Set up company bank account for payments

4. **Redirect to Settings:**
   - Opens `pg-billpay-settings`
   - Configure default payment account
   - Set up approval rules (if any)

**Related Reusables:**
- `p-billpay-enableBillpayFeature` - Enable popup
- `pg-billpay-marketing` - Marketing page
- `pg-billpay-settings` - Settings page

#### Company Payment Account Setup

**Purpose:** Designate which bank account pays bills

**Flow:**
1. Navigate to `pg-billpay-settings`
2. Select "Payment Account"
3. Choose from company's linked bank accounts
4. Save settings

**Related Data:**
- `company_setting_bill` or `company_setting`
- Default payment account field
- Links to `bank_account` records

---

### 13. Bill and BillPay Relationship

**Key Connections:**

1. **Expense → BillPayment:**
   - One bill can have multiple partial payments
   - Linked via `BillPayment_item` (many-to-many)

2. **BillPayment → Multiple Bills:**
   - One payment can pay multiple bills
   - Grouped by vendor
   - Linked via `BillPayment_item`

3. **Status Synchronization:**
   - Bill status updates when payment is scheduled
   - Bill status updates when payment completes
   - Payment status updates on failure revert bill status

4. **Balance Tracking:**
   - `expense.amount_balance_due` tracks outstanding amount
   - Decremented when payment scheduled
   - Restored if payment cancelled/failed
   - Set to zero when fully paid

5. **Integration Sync:**
   - Bill must be synced to QBO before payment
   - Payment syncs to QBO after completion
   - QBO links bill and payment via `LinkedTxn`

---

## Summary

### Bill Feature Summary
The Bills feature provides comprehensive bill/expense management with:
- Multiple bill creation methods (manual, upload, email)
- OCR/parsing via Photon for automatic data extraction
- Multi-level approval workflows
- Chart of accounts mapping
- Vendor management
- Accounting integration (QuickBooks Online)
- Status tracking and audit trail

### BillPay Feature Summary
The BillPay feature enables electronic bill payments with:
- ACH and check payment methods
- Payment scheduling
- Dwolla integration for payment processing
- Vendor bank account management with verification
- Payment status tracking and notifications
- QuickBooks Online payment sync
- Multi-bill payment support
- Payment failure handling and retry logic

### Key Platforms
- **Bubble:** Frontend logic, UI, workflows, database
- **Xano:** Backend services, API orchestration, data storage
- **Photon:** Bill/receipt parsing (OCR)
- **ForwardMX/Mailparser:** Email ingestion
- **Dwolla:** Payment processing
- **QuickBooks Online:** Accounting integration

### Data Flow Overview
```
User Action
    ↓
Bubble Workflow
    ↓
Xano API (optional)
    ↓
Third-party Service (Photon, Dwolla, QBO)
    ↓
Xano Processing
    ↓
Webhook back to Bubble
    ↓
Bubble Updates Data
    ↓
UI Refresh
```

---

## Appendix

### A. Option Sets Reference

#### Bill-Related Option Sets
- **Status | Bill** (`bill_status`)
- **Status | Bill Approval** (`bill_approval_status`)
- **Type of Expense** (`type_of_bill`)
- **Alerts - Bill** (`alert___bills`)
- **Filter | Bill Status** (`filter___bill_status`)

#### BillPay-Related Option Sets
- **Status | Bill Payment** (`bill_payment_status`)
- **Type of Payment Method** (`type_of_bill_payment`)
- **Status | Payment Method** (`status_of_bill_payee`)
- **Bill Payee Options** (`bill_payee_options`)
- **Bill Payee Option Types** (`bill_payment_approval_method`)

#### Approval-Related Option Sets
- **Status | Approval Request** (`status_of_approval_request`)
- **Status | Approval Flow Step** (`status_of_approval_flow_step`)
- **Approval Flow Step Type** (`approval_flow_step_type`)
- **Approval Types** (`user_multiselect_types`)
- **Approver Action Types** (`approver_action_types`)

#### Integration-Related Option Sets
- **IntegrationSetting-Accounting Options** (`integration_options`)
- **OS IntegrationEntity** (`os_integrationentity`)
- **OS IntegrationClass** (`integration_class`)

### B. Database Triggers Reference

#### Bill Triggers
1. **Bill Changelog** - Audit logging to Xano
2. **Bill Created** - Activity log entry
3. **Bill Deleted** - QBO deletion, integration cleanup
4. **Bill Deleted Retain** - Terminal status retention
5. **Bill Submitted** - Status approval update
6. **Bill Updated & New Approval Flow** - Duplicate detection, approval flow creation, balance calculation

#### BillPay Triggers
- May have triggers on BillPayment status changes
- May have triggers on payment completion
- Integration-related triggers for QBO sync

### C. Key Workflows Quick Reference

#### Bill Creation
- `bill_create` - Manual creation
- `bill_update` - Update from parse
- `bill_item_create` - Line items
- `bill_create_from_req` - From inbox
- `request_bill` - Email receipt

#### Bill Updates
- `bill-delete` - Delete bill
- `bill-duplicateBill` - Duplicate single
- `bill-duplicateListOfBills` - Bulk duplicate
- `bill_submitted_for_approval` - Submit
- `bill_approved` - Approve
- `bill_approval_declined` - Reject

#### Bill Integration
- `qbo-bill-create` - Create in QBO
- `qbo-bill-update` - Update in QBO
- `qbo-bill-delete` - Delete from QBO
- `bill-bulk-sync-integration` - Bulk sync

#### BillPay
- `billpayment_create` - Create payment
- `billpayment_schedule` - Schedule processing
- `billpay_send_ach` - Send ACH
- `bill-payment-complete` - Finalize
- `bill_paid` - Bill paid handler
- `bill_payment_error` - Error handling

#### BillPay Integration
- `qbo-billpayment-create` - Create in QBO

### D. Page and Reusable Index

#### Bill Pages
- `pg-bill-list` - Bill list
- `pg-bill-inbox` - Email inbox
- `pg-vendor-bills` - Vendor bills

#### BillPay Pages
- `pg-billpay` - Payment list
- `pg-billpay-settings` - Settings
- `pg-billpay-marketing` - Marketing
- `pg-billpayment-detail` - Payment detail
- `pg-vendor-payment-methods` - Payment methods

#### Key Bill Reusables
- `fg-bill-viewDetails` - Detail editor
- `w-bill-btnAddNew` - Add button
- `m-bill` - Action menu
- `cmp-bill-lineItems` - Line items

#### Key BillPay Reusables
- `p-billpayment-create` - Create payment
- `p-payment-method` - Payment method
- `wf-billPayment` - Workflow container

---

**Document Ends**
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5MTQ1MDk1NjEsLTE3NjIwNDIyLC0xNz
YyMDQyMl19
-->