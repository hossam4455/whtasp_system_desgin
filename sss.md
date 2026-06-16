# 📋 Comprehensive Report — Rent Contract Actions

> Full documentation for every action in the "Contract Actions" menu — what was implemented, why, and which files were modified.

> Note: workflow and status labels are translated to English here for readability, while DocType names, file paths, and commit references remain exactly as implemented.

---

## 🗺️ Action Map

| # | Action | DocType | Available Statuses |
|---|--------|---------|--------------------|
| 1 | 🏛️ File Case | `Case` | All statuses |
| 2 | 💸 Financial Claim | `Financial Claim` | `Active`, `Ended` (excluding archived) |
| 3 | 💰 Rent Raise | `Rent Raise Request` | `Active` |
| 4 | 🚫 Non-Renewal | `Non Renewal Request` | `Active`, `Registered` |
| 5 | 🔄 Auto Renewal | `Auto Renewal Request` | `Active` |
| 6 | ⚖️ Contract Settlement | `Rent Contract Cancellation Request` | `Active` |
| 7 | 📝 Contract Termination Notice | `Contract Termination Notice` | `Active`, `Ended`, `Registered` |
| 8 | 💵 Create Receipt Voucher | `Tenant Payment Receipt` | `Active` |
| 9 | 🔑 Unit Receive | `Unit Receive Request` | End of contract |
| 10 | 🚪 Unit Deliver | `Unit Deliver Request` | Before activation (automatic) |

---

## 1️⃣ File Case 🏛️

### Purpose
Create a legal case against a tenant who violated the contract terms. The action moves from the property officer -> property supervisor -> lawyer until completion.

### What Was Implemented
- A dedicated `Dialog`, `create_case_dialog(frm)`, inside `rent_contract.js` opens a modal for selecting the case type from a dynamic list.
- **Step tracking** — the `Case` DocType is linked to a workflow (`Draft` -> `Pending Approval` -> `Approved` -> `Completed`).
- A separate `workflow_state` handles lawyer assignment and supervisor approval.

### Modified Files

| File | Change |
|------|--------|
| `nofouth_property_managment/doctype/rent_contract/rent_contract.js` | Added button + `create_case_dialog` |
| `nofouth_property_managment/doctype/case/case.js` | Validation + lawyer assignment |
| `nofouth_property_managment/doctype/case/case.json` | Fields: `case_type`, `description`, `lawyer`, `status` |
| `nofouth_property_managment/doctype/case/case.py` | `after_insert` -> link with contract |
| `fixtures/workflow.json` | Workflow state machine for `Case` |

### Related Commits

| Commit | Date | Description |
|--------|------|-------------|
| `52ed670` | 2026-04-01 | new updates cases, property owner — added types + owner linkage |
| `34d67b3` | 2026-02-24 | updates in cases and fix flexible contracts payment commission |
| `d4a4f4d` | 2026-03-16 | new updates from nofouth dev 15-03 |
| `221b2cd` | 2026-04-05 | update sales invoice — linked invoice with case |

---

## 2️⃣ Financial Claim 💸

### Purpose
Notify the tenant that there is an unpaid outstanding amount. This action is available from:
- Inside the rent contract
- Collection and overdue tables

### What Was Implemented

#### UI (Dialog Box)
- Select **payment number** from a dropdown showing payments with statuses `Not Due`, `Due`, `Partially Paid`.
- **Letter date** (default = today, editable).
- **Notification types** — 4 checkboxes:
  - WhatsApp message
  - SMS
  - Email
  - Attach letter PDF
- **Payment details preview** (read-only): period from/to, rent amount, expenses, tax, paid amount, remaining amount.

#### Financial Claim Screen (After Creation)
- **Contract and tenant data** — contract number, type, start/end, tenant name, ID, phone.
- **Unit and property data** — property, units, owner, property officer.
- **Payments table** — editable.
- **Notification options** — editable.
- **Notification template** — Jinja template that generates the claim text.
- **Outputs** — PDF / Word (checkbox).
- **Auto-save PDF** in the contract folder.

### Important Implemented Constraints
1. `Financial Claim` has **no workflow** — final creation happens after approval from the review screen.
2. **Prevent creating a claim for an archived property** — check `workflow_state === 'Archived'` before showing the button.
3. **Default delivery channel** — WhatsApp + SMS.

### Modified Files

| File | Change |
|------|--------|
| `doctype/rent_contract/rent_contract.js:578+` | Button + full dialog + payment filter |
| `doctype/financial_claim/financial_claim.js` | UI logic (83 added lines) |
| `doctype/financial_claim/financial_claim.json` | 343 additions — fields + sections |
| `doctype/financial_claim/financial_claim.py` | 291 additions — create + attach + send |
| `doctype/financial_claim_payments/` | Child table for payments |
| `print_format/financial_claim_letter/` | Jinja print format |
| `controllers/property_transaction.py` | GL posting if linked to invoice |
| `apis/notifications_api.py` | Delivery channels |
| `apis/letters_api.py` | Letter text generation |

### Related Commits

| Commit | Date | Description |
|--------|------|-------------|
| `80ba98d` | 2026-03-26 | **contract termination notice + financial claim updates** — foundation |
| `b184aea` | 2026-03-31 | payment selection improvements + overdue report linkage |
| `66def7f` | — | schedule in financial claim |
| `fa8a3e5` | — | Apply finance changes |

---

## 3️⃣ Rent Raise 💰

### Purpose
Notify the tenant of a rent increase 30 days before contract end. If the tenant chooses to renew, the new price is applied.

### What Was Implemented

#### Inputs
- **New rent amount** (validation: must be greater than the current one).
- **Expense raise table** — child table:
  - Expense type (`Link`)
  - New amount (validation: must be greater than the current amount)
  - No duplicate expense type
- **Send letter 30 days before end date** — Yes/No.

#### Critical Validations
```python
# Server-side in rent_raise_request.py
if new_rent_amount < contract.annual_rent:
    throw("The new rent amount is lower than the current contract rent amount")

for expense in new_expenses:
    if expense.amount < contract_expense.amount:
        throw(f"The new expense amount for type {expense.type} is lower than the current amount")
```

#### 3-State Workflow
```text
Draft (Property Officer)
      ↓
Pending Property Supervisor Approval
      ↓
Approved
```

### Approval Impact
- **At contract renewal** -> server-side validation prevents renewal with an amount lower than the approved rent raise.

### Modified Files

| File | Change |
|------|--------|
| `doctype/rent_contract/rent_contract.js:511` | Added button |
| `doctype/rent_contract/rent_contract.py` | `create_rent_raise_request` method |
| `doctype/rent_raise_request/rent_raise_request.json` | All fields |
| `doctype/rent_raise_request/rent_raise_request.py` | Validation + save + send |
| `doctype/rent_raise_request/rent_raise_request.js` | UI + notification options |
| `doctype/raise_rent_request_expense/` | Child table for expenses |
| `doctype/rent_raise_units/` | Child table for units |
| `doctype/letter_of_rent_raise/` | Companion DocType for the letter |
| `print_format/letter_of_rent_raise_pf/` | Print format |
| `fixtures/workflow.json` | 3-state workflow |

### Commits

| Commit | Date | Description |
|--------|------|-------------|
| `463749f` | 2026-03-26 | updates of exit admin + workflow |
| `5202984` | — | Update Phase 1 — UX improvements |
| `4000668` | — | updates with fix |

---

## 4️⃣ Non-Renewal 🚫

### Purpose
Notify the tenant that the contract will not be renewed when it ends, with a request to hand over the unit.

### What Was Implemented
- **Letter date** (automatic + editable).
- **Text notes**.
- **Units table** — auto-fetched from the contract.
- **Notification options** — WhatsApp, SMS, Email, PDF.
- **3-state workflow** (same as rent raise).

### Post-Approval Validations
```python
# In rent_raise_request.py and auto_renewal_request.py
if has_approved_non_renewal(contract):
    throw("You cannot create a rent raise or auto-renewal after a non-renewal request has been approved")
```

### Modified Files

| File | Change |
|------|--------|
| `doctype/rent_contract/rent_contract.js:557` | Button |
| `doctype/rent_contract/rent_contract.py` | `create_non_renewal_request` method |
| `doctype/non_renewal_request/non_renewal_request.json` | Fields |
| `doctype/non_renewal_request/non_renewal_request.py` | Logic + email/WhatsApp |
| `doctype/non_renewal_request/non_renewal_request.js` | UI |
| `print_format/non_renwal/` | Print format |

### Commits

| Commit | Date | Description |
|--------|------|-------------|
| `463749f` | 2026-03-26 | updates of exit admin |
| `5202984` | — | Update Phase 1 |
| `4000668` | — | updates with fix |
| `1aa58c8` | — | unit handling |
| `946d6b7` | — | Updating |

---

## 5️⃣ Auto Renewal 🔄

### Purpose
Create an auto-renewal request for the contract using the same current terms.

### What Was Implemented
- **Mapped Doc** — `frappe.model.open_mapped_doc` copies the contract data.
- **Validations**:
  - Contract status must be `Active` only.
  - There must be no approved `non_renewal_request`.
  - There must be no approved `rent_raise_request` because the new values must be applied instead.

### Modified Files

| File | Change |
|------|--------|
| `doctype/rent_contract/rent_contract.js:519` | Button |
| `doctype/rent_contract/rent_contract.py` | `create_auto_renewal_request` method |
| `doctype/auto_renewal_request/auto_renewal_request.json` | Fields |
| `doctype/auto_renewal_request/auto_renewal_request.py` | Logic |
| `doctype/auto_renewal_request/auto_renewal_request.js` | UI |
| `print_format/auto_renewal_request_pf/` | Print format |

### Commits

| Commit | Date | Description |
|--------|------|-------------|
| `079d806` | 2026-03-27 | print format |
| `463749f` | 2026-03-26 | updates of exit admin |
| `5202984` | — | Update Phase 1 |

---

## 6️⃣ Contract Settlement ⚖️

### Purpose
Create a settlement request for an active contract — calculate due payments and adjust their statuses.

### What Was Implemented

#### DocType Structure
- **`Rent Contract Cancellation Request`** — master
- **`Rent Contract Cancel Request Unit`** — child table for units
- **`Rent Contract Canellation Payment`** — child table for payments

#### Flow
1. Fetch all payments from the contract.
2. Calculate refund / receivable for each payment.
3. Employee approves -> a `Journal Entry` is created for settlement.
4. Contract status -> `Cancelled`.

### Validations
- You cannot settle a contract that has an open financial claim.
- You cannot settle a contract after a termination event.

### Modified Files

| File | Change |
|------|--------|
| `doctype/rent_contract/rent_contract.js:528` | Button |
| `doctype/rent_contract/rent_contract.py` | `create_cancel_request` method |
| `doctype/rent_contract_cancellation_request/` | Master doctype |
| `doctype/rent_contract_cancel_request_unit/` | Child table for units |
| `doctype/rent_contract_canellation_payment/` | Child table for payments |
| `doctype/rent_contract_reconcile_payment/` | Reconciliation logic |
| `print_format/cancelation_request/` | Print format |

### Commits

| Commit | Date | Description |
|--------|------|-------------|
| `4cf641d` | 2026-04-05 | reconsilation — cancellation reconcile logic |
| `2c44601` | — | updates — cancellation |

---

## 7️⃣ Contract Termination Notice 📝

### Purpose
Formal notice to the tenant that the contract is being terminated due to a violation.

### What Was Implemented

#### Inputs
- **Contract number** (automatic, read-only).
- **Letter date** (automatic, editable).
- **Violation type** (text).
- **Explanation** (text).

#### Contract Data (Read-only)
Contract number, tenant, owner, property, units, annual rent, annual services, dates, property officer.

#### Channels
WhatsApp, SMS, Email + attach PDF.

#### 3-State Workflow
```text
Draft (Property Officer)
      ↓
Pending Property Supervisor Approval
      ↓
Approved
```

### Approval Impact
```python
# In auto_renewal_request.py and rent_contract.py
if has_approved_termination_notice(contract):
    throw("The contract cannot be renewed while an approved termination notice exists")
```

### Modified Files — Full Detail

| File | Change |
|------|--------|
| `doctype/rent_contract/rent_contract.js:502` | Button |
| `doctype/rent_contract/rent_contract.py` | `create_contract_termination_notice` |
| `doctype/contract_termination_notice/__init__.py` | Module creation |
| `doctype/contract_termination_notice/contract_termination_notice.json` | 356 lines — full field definition |
| `doctype/contract_termination_notice/contract_termination_notice.py` | 306 lines — Python logic |
| `doctype/contract_termination_notice/contract_termination_notice.js` | 54 UI lines |
| `doctype/contract_termination_notice/test_contract_termination_notice.py` | Tests |
| `controllers/property_transaction.py` | 65 lines — GL linkage |
| `doc_events/whatsapp_message.py` | 27 lines — WhatsApp message event linkage |
| `doc_events/sms_log.py` | 13 lines — SMS tracking |
| `hooks.py` | 12 lines — `doc_events` registration |
| `print_format/contract_termination_notice_pf/` | Print format |

### Main Commit
**`80ba98d`** (2026-03-26): `contract termination notice + financial claim updates`
- 14 modified files
- +1,500 added lines
- This is the full feature foundation.

---

## 8️⃣ Create Receipt Voucher 💵

### Purpose
Create a `Tenant Payment Receipt` automatically linked to the contract.

### What Was Implemented

#### Flow (Without Dialog)
1. Click the button -> call `check_duplicate_tpr` API.
2. If a draft/open TPR exists for the same payment -> creation is blocked.
3. Redirect to the `Tenant Payment Receipt` form with pre-filled values:
   - `rent_contract` = current contract
   - `tenant` = tenant

### Critical Validation
```python
# rent_contract.py:check_duplicate_tpr
def check_duplicate_tpr(rent_contract):
    open_tpr = frappe.db.exists('Tenant Payment Receipt', {
        'rent_contract': rent_contract,
        'docstatus': ['!=', 2],  # not cancelled
        'workflow_state': ['in', ['Draft', 'Pending Approval']]
    })
    if open_tpr:
        frappe.throw(f"An open receipt voucher already exists ({open_tpr})")
```

### Modified Files

| File | Change |
|------|--------|
| `doctype/rent_contract/rent_contract.js:535` | Button + duplicate check |
| `doctype/rent_contract/rent_contract.py` | `check_duplicate_tpr` method |
| `doctype/tenant_payment_receipt/` | TPR doctype (multiple improvements) |
| `doctype/tenant_payment_receipt_unit/` | Child table |
| `controllers/property_transaction.py` | GL posting |
| `print_format/tenant_payment_receipt/` | Print format |

### Commits

| Commit | Date | Description |
|--------|------|-------------|
| `57ccaa1` | 2026-04-18 | debugging TPR Submitting Process |
| `edd1ffa` | 2026-04-18 | edits |
| `00b155d` | 2026-04-06 | commission on services from property not contract in TPR |
| `bd0f5e4` | 2026-04-06 | fix |
| `dc409e3` | 2026-04-02 | opd handling logic |
| `87b4ad0` | 2026-02-25 | tenant payment receipt |

---

## 9️⃣ Unit Receive 🔑

### Purpose
Receive the unit from the tenant after the contract ends.

### Inputs
- Receive date
- Receive notes
- Receive ratings (multi-rating)
- Receive image attachments (image child table)

### Approval Impact
- Contract status -> `Archived`.
- **No impact on payment statuses** (outstanding dues remain unchanged).

### Modified Files

| File | Change |
|------|--------|
| `doctype/rent_contract/rent_contract.js:487` | Button |
| `doctype/unit_receive_request/unit_receive_request.json` | Fields |
| `doctype/unit_receive_request/unit_receive_request.py` | Archive on approval |
| `print_format/` | Print format |

---

## 🔟 Unit Deliver 🚪

### Purpose
Deliver the unit to the tenant — **created automatically** when the contract is created, and automatically approved when the contract itself is approved.

### Inputs
- Deliver date
- Deliver notes
- Deliver ratings
- Deliver image attachments

### Integration
- `after_insert` on `Rent Contract` -> automatically creates `Unit Deliver Request`.
- The contract's first status change to `Active` -> approves the deliver request.

### Modified Files

| File | Change |
|------|--------|
| `doctype/rent_contract/rent_contract.py` | Auto-create in `after_insert` |
| `doctype/rent_contract/rent_contract.js:852` | Button (for special cases) |
| `doctype/unit_deliver_request/unit_deliver_request.json` | 14 updated lines |
| `doctype/unit_deliver_request/unit_deliver_request.js` | 20 logic lines |
| `doctype/unit_deliver_request/unit_deliver_request.py` | Validation |
| `api.py` | `create_unit_deliver_request_api` |

### Commits

| Commit | Date | Description |
|--------|------|-------------|
| `463749f` | 2026-03-26 | updates of exit admin |
| `ce1b2b1` | 2026-03-11 | fix |
| `2ca9dbe` | — | excluding validation of partners |
| `9e08b5d` | — | Fix |
| `cc30d66` | — | Enhancing Code |

---

## 🔗 Interdependencies Between Actions

```text
                    ┌─────────────────────────┐
                    │     Rent Contract       │
                    │        (Active)         │
                    └────────────┬────────────┘
                                 │
            ┌──────────┬──────────┼──────────┬──────────┬──────────┐
            │          │          │          │          │          │
         File Case  Financial   Rent Raise  Non-Renewal  Auto     Termination
                      Claim                               Renewal    Notice
                                 │
                                 ↓ (after approval)
            ┌─────────────────────────────────────────┐
            │ Blocks: auto-renewal + rent raise       │
            └─────────────────────────────────────────┘
                                 │
                                 ↓ (after deliver/receive)
                       ┌─────────────────┐
                       │    Archived     │
                       └─────────────────┘
```

---

## 📅 Full Development Timeline

### Q1 2026 (January - March)

| Date | Action | Details |
|------|--------|---------|
| 2026-01-20 | Reports, procedures, taxes, fixes | `be3bd57` — start of the revamp |
| 2026-01-26 | update validation of commission | `65e1f7b` |
| 2026-02-03 | Updates in workflow | `960dee8` — workflow state definition |
| 2026-02-07 | Taxes NEW VERSION | `88019f5` + `bf1d8d5` — new property-related doctypes |
| 2026-02-11 | fix invoice cleared validation | `d77d1c3` |
| 2026-02-24 | updates in cases | `34d67b3` — flexible contracts commission fix |
| 2026-02-25 | tenant payment receipt | `87b4ad0` + `4ba4cfd` |
| 2026-03-16 | major updates from dev branch | `d4a4f4d` + `110dabb` + `415c8ca` |
| **2026-03-26** | **🌟 Contract Termination + Financial Claim** | **`80ba98d` — complete foundation** |
| 2026-03-26 | updates of exit admin | `463749f` — workflow enhancements |
| 2026-03-27 | print format | `079d806` — Auto Renewal |
| 2026-03-31 | new updates in reports and financial claim | `b184aea` — improvements |

### Q2 2026 (April - Present)

| Date | Action | Details |
|------|--------|---------|
| 2026-04-01 | new updates cases, property owner | `52ed670` — case types |
| 2026-04-05 | reconsilation | `4cf641d` — contract settlement |
| 2026-04-06 | commission on services from property | `00b155d` |
| 2026-04-18 | debugging TPR | `57ccaa1` |
| 2026-04-20 | fix contract fees GL reverse | `007e777` |

---

## 🐛 Most Important Bugs That Were Fixed

### 1. **Financial Claim on Archived Property**
- **Problem**: It was possible to create a claim on an archived property.
- **Fix**: `rent_contract.js:568-577` — check `workflow_state === 'Archived'` and remove the button.
- **Commit**: `b184aea`.

### 2. **TPR Duplicate**
- **Problem**: More than one open receipt voucher could be created for the same contract.
- **Fix**: `check_duplicate_tpr` API.
- **Commit**: `edd1ffa`, `57ccaa1`.

### 3. **Contract Fees GL Reversed**
- **Problem**: GL entries for fees were reversed (`credit` / `debit`).
- **Fix**: Adjusted accounting logic in `property_transaction.py`.
- **Commit**: `007e777`, `a8bf134`, `bbaed1c`.

### 4. **Flexible Contracts Commission**
- **Problem**: Flexible contracts were calculating commission incorrectly.
- **Fix**: Revamped commission calculation.
- **Commit**: `34d67b3`.

### 5. **Invoice Cleared Validation**
- **Problem**: Validation failed when the invoice was already cleared.
- **Commit**: `d77d1c3`.

### 6. **Tax Percent Auto-Calculation**
- **Problem**: Tax percentage was not being calculated automatically.
- **Commit**: `f3078d4`, `88019f5`.

### 7. **Validation of Partners**
- **Problem**: Property partners were causing blocking validations.
- **Fix**: Excluded them from unit deliver validation.
- **Commit**: `2ca9dbe`.

### 8. **Cancellation Reconciliation**
- **Problem**: Contract settlement left GL entries hanging.
- **Fix**: `rent_contract_reconcile_payment` doctype.
- **Commit**: `4cf641d`.

---

## 🗂️ Core Files — Full Map

### Frontend
```text
nofouth_property_managment/doctype/rent_contract/
├── rent_contract.js          ← all action buttons (~900+ lines)
└── rent_contract_list.js     ← list view
```

### Backend
```text
nofouth_property_managment/doctype/rent_contract/
└── rent_contract.py          ← methods:
                                - create_contract_termination_notice
                                - create_rent_raise_request
                                - create_auto_renewal_request
                                - create_cancel_request
                                - create_non_renewal_request
                                - create_unit_deliver_request
                                - check_duplicate_tpr
                                - create_case_dialog logic
```

### Child / Related DocTypes (10 new)
```text
nofouth_property_managment/doctype/
├── case/                                ← File Case
├── financial_claim/                     ← Financial Claim (+ financial_claim_payments)
├── rent_raise_request/                  ← Rent Raise (+ raise_rent_request_expense, rent_raise_units)
├── non_renewal_request/                 ← Non-Renewal
├── auto_renewal_request/                ← Auto Renewal
├── rent_contract_cancellation_request/  ← Settlement (+ cancel_request_unit, canellation_payment)
├── contract_termination_notice/         ← Termination
├── tenant_payment_receipt/              ← Receipt Voucher (+ tenant_payment_receipt_unit)
├── unit_receive_request/                ← Receive
└── unit_deliver_request/                ← Deliver
```

### Print Formats
```text
nofouth_property_managment/print_format/
├── contract_termination_notice_pf/
├── non_renwal/
├── financial_claim_letter/
├── cancelation_request/
├── tenant_payment_receipt/
├── letter_of_rent_raise_pf/
└── auto_renewal_request_pf/
```

### APIs + Helpers
```text
nofouth_property_management/
├── api.py                         ← create_unit_deliver_request_api
├── apis/
│   ├── notifications_api.py       ← channel routing (WhatsApp/SMS/Email)
│   ├── letters_api.py             ← notification text generation
│   └── reports_api.py             ← overdue tenants, undue reports
├── controllers/
│   └── property_transaction.py    ← GL posting for actions
└── doc_events/
    ├── sales_invoice.py
    ├── sms_log.py                 ← tracking
    └── whatsapp_message.py        ← tracking
```

### Workflows + Fixtures
```text
nofouth_property_management/fixtures/
├── workflow.json                  ← 3-state workflows
├── custom_field.json              ← added fields
└── property_setter.json           ← UI customizations
```

---

## 📊 Overall Statistics

| Item | Value |
|------|-------|
| **Implemented actions** | 10 |
| **New DocTypes** | 10 main + ~7 child tables |
| **Whitelisted APIs** | 15+ |
| **Print Formats** | 7 |
| **Commits (12 months)** | 80+ |
| **Modified files (estimated)** | 50+ files |

---

## 🎯 Summary

A full "Contract Actions" management system was built, including:
1. ✅ **10 independent actions**, each with its own DocType
2. ✅ **3-state workflows** (`Draft` -> `Pending` -> `Approved`) for major actions
3. ✅ **Multi-channel notifications** (WhatsApp / SMS / Email / PDF) for each action
4. ✅ **Cross-validations** between actions (for example: prevent renewal after termination)
5. ✅ **Professional print formats** using Jinja templates
6. ✅ **GL integration** for every financial action
7. ✅ **Complete notification tracking** (`delivered` / `read` / `failed`)

---

## 📌 Quick Review Commands

```bash
# To see all changes for a specific action (example: Financial Claim):
cd /home/frappe/frappe-bench/apps/nofouth_property_management
git log --oneline --all -- "nofouth_property_management/nofouth_property_managment/doctype/financial_claim/"

# To inspect a specific commit in full:
git show 80ba98d

# To search for when a specific function was added:
git log --all -S "create_contract_termination_notice" --oneline
```
