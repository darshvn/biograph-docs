# E2E Insurance Management Module Setup

## Overview

Biograph's insurance module covers the full lifecycle: payor onboarding → contract negotiation → eligibility configuration → patient policy assignment → coverage pre-approval → invoice integration → claim aggregation → payment settlement.

The module consists of **8 DocTypes** that work together:

```
┌─────────────────────────────────────────────────────────────────┐
│                     MASTER DATA (Setup Once)                     │
│                                                                  │
│  Insurance Payor ──→ Insurance Payor Contract (per company)      │
│       │                                                          │
│       └──→ Insurance Payor Eligibility Plan                      │
│                    │                                              │
│                    └──→ Item Insurance Eligibility (coverage %)   │
├─────────────────────────────────────────────────────────────────┤
│                     PATIENT-LEVEL                                │
│                                                                  │
│  Patient Insurance Policy ──→ Patient Insurance Coverage         │
│  (patient + payor + plan)     (pre-approval per service)         │
├─────────────────────────────────────────────────────────────────┤
│                     BILLING & CLAIMS                             │
│                                                                  │
│  Sales Invoice (items linked to coverages)                       │
│       │                                                          │
│       └──→ Insurance Claim ──→ Payment Entry                     │
│            (aggregates invoiced coverages)                        │
└─────────────────────────────────────────────────────────────────┘
```

## Setup Order

Configure these in sequence:

1. Insurance Payor (the insurance company)
2. Insurance Payor Contract (per company, with date range)
3. Insurance Payor Eligibility Plan (coverage plan template)
4. Item Insurance Eligibility (which services get what coverage %)
5. Patient Insurance Policy (assign patient to payor+plan)
6. Patient Insurance Coverage (pre-approve specific services)
7. Sales Invoice (billing with coverage amounts split)
8. Insurance Claim → Payment Entry (claim and settle)

---

## Part 1: Insurance Payor

The insurance company master. Auto-creates a linked ERPNext **Customer** for receivables tracking.

Navigate to: Healthcare workspace > Insurance Payor > New

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `insurance_payor_name` | Data | Unique name (required) |
| `customer` | Link → Customer | Auto-created, read-only |
| `insurance_claim_credit_days` | Int | Payment terms (days) for claims |
| `disabled` | Check | Deactivate payor |
| `code_system` | Link → Code System | Default code system for claims |
| `image` | Attach Image | Payor logo |
| `website` | Data | Payor website URL |
| `claims_receivable_accounts` | Table (Party Account) | Receivable account per company |
| `rejected_claims_expense_accounts` | Table (Party Account) | Expense account for rejected claims per company |

### Auto Customer Creation

On `on_update()` (insurance_payor.py):
- Creates Customer Group "Insurance Payor" if it doesn't exist
- Creates a Customer record linked to the payor
- Syncs account changes to the Customer's party accounts

### Key API Methods

```python
# Check if payor has an active contract for a company on a date
frappe.call(
    "healthcare.healthcare.doctype.insurance_payor.insurance_payor.has_active_contract",
    { "insurance_payor": "ACME Insurance", "company": "Hospital Corp", "on_date": "2024-01-15" }
)
# Returns: True/False

# Get payor's receivable account for a company
# Internal: get_insurance_payor_details(insurance_payor, company)
# Returns: { customer, claims_receivable_account }

# Get expense account for rejected claims
# Internal: get_insurance_payor_expense_account(insurance_payor, company)
```

---

## Part 2: Insurance Payor Contract

Defines the agreement period between a payor and a hospital company. Only **one active contract** allowed per payor+company at a time.

Navigate to: Insurance Payor Contract > New

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `naming_series` | Data | PAYOR-CONT-.YYYY.- |
| `insurance_payor` | Link | Required |
| `company` | Link | Required |
| `start_date` | Date | Contract start (required) |
| `end_date` | Date | Contract end (required) |
| `is_active` | Check | Active flag |
| `default_price_list` | Link → Price List | Payor-negotiated price list |
| `posting_date` | Date | Document date |

**Submittable**: No

### Validation (insurance_payor_contract.py)

- `start_date` must be before `end_date`
- Overlap detection prevents multiple active contracts for the same payor+company in overlapping date ranges (uses Frappe Query Builder)

---

## Part 3: Insurance Payor Eligibility Plan

A plan template that groups coverage rules. One payor can offer multiple plans (e.g., "Gold Plan", "Basic Plan").

Navigate to: Insurance Payor Eligibility Plan > New

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `insurance_plan_name` | Data | Unique name (required) |
| `insurance_payor` | Link | Parent payor (required) |
| `price_list` | Link → Price List | Plan-specific price list (overrides contract default) |
| `is_active` | Check | Active flag |

---

## Part 4: Item Insurance Eligibility

The **coverage matrix** — defines what percentage each service/item is covered at, and whether approval is automatic or manual.

Navigate to: Item Insurance Eligibility > New

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `eligibility_for` | Select | **Service** (healthcare template) or **Item** (ERPNext item) |
| `template_dt` | Link → DocType | DocType of the service template (if Service) |
| `template_dn` | Dynamic Link | Specific template (e.g., "Blood Test - CBC") |
| `item_code` | Link → Item | Auto-populated from template, or manually set for Item type |
| `insurance_plan` | Link | Optional — scope to a specific plan. Blank = applies to all plans. |
| `coverage` | Percent | Coverage percentage (1–100), required |
| `discount` | Percent | Additional discount (only if coverage < 100%) |
| `mode_of_approval` | Select | **Automatic** (default) or **Manual** |
| `is_active` | Check | Active (default: ✓) |
| `valid_from` | Date | Start of validity (required, default: today) |
| `valid_till` | Date | End of validity (optional — blank = perpetual) |
| `code_system` | Link | Medical code system |
| `code_value` | Link | Medical code |

### Coverage Lookup Logic

`get_insurance_eligibility(item_code, insurance_plan, template_dt, template_dn, on_date)` — the core lookup function:

1. First tries: matching item_code + insurance_plan + active + `valid_from ≤ on_date ≤ valid_till`
2. Falls back to: perpetual rules (no `valid_till`) if date-bounded match not found
3. Returns: `{ coverage, discount, mode_of_approval, insurance_plan }`

### Validation

- Coverage must be 1–100%. If coverage = 100%, discount is forced to 0.
- Overlap detection prevents duplicate active eligibilities for the same item+plan in overlapping date ranges

### Example Configuration

| Service | Plan | Coverage | Discount | Approval |
|---------|------|----------|----------|----------|
| Lab Test: CBC | Gold Plan | 100% | 0% | Automatic |
| Lab Test: CBC | Basic Plan | 80% | 5% | Automatic |
| Clinical Procedure: Biopsy | Gold Plan | 90% | 0% | Manual |
| Consultation (Item) | *(all plans)* | 70% | 10% | Automatic |

---

## Part 5: Patient Insurance Policy

Assigns a patient to an insurance payor and plan. This is the patient-facing record that staff reference during appointments and billing.

Navigate to: Patient Insurance Policy > New

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `naming_series` | Data | INS-PLCY-.YYYY.- |
| `patient` | Link | Required |
| `insurance_payor` | Link | Required |
| `insurance_plan` | Link → Eligibility Plan | Optional plan assignment |
| `policy_number` | Data | Unique per patient (required) |
| `policy_expiry_date` | Date | Required — must be in the future |
| `date` | Date | Issue date (default: today) |
| Patient details | Various | Auto-fetched: patient_name, gender, birth_date, email, mobile, country |

**Submittable**: Yes

### Validation

- `policy_expiry_date` must be in the future
- No overlapping policies for the same patient+payor+plan
- Policy number must be unique per patient

### Key API Methods

```python
# Check if policy is still valid
frappe.call(
    "healthcare.healthcare.doctype.patient_insurance_policy.patient_insurance_policy.is_insurance_policy_valid",
    { "insurance_policy": "INS-PLCY-2024-00001", "on_date": "2024-06-15" }
)
# Returns: True/False

# Get price lists for insurance pricing
frappe.call(
    "healthcare.healthcare.doctype.patient_insurance_policy.patient_insurance_policy.get_insurance_price_lists",
    { "insurance_policy": "INS-PLCY-2024-00001" }
)
# Returns: { plan_price_list, default_price_list }
```

---

## Part 6: Patient Insurance Coverage (Pre-Approval)

A per-service pre-approval record. Created when a patient needs a specific service — the system checks eligibility and calculates coverage amounts.

Navigate to: Patient Insurance Coverage > New (or created via API)

### Fields

**Core**

| Field | Type | Description |
|-------|------|-------------|
| `naming_series` | Data | INS-COV-.YYYY.- |
| `patient` | Link | Required |
| `insurance_policy` | Link | Required |
| `insurance_payor` | Link | Fetched from policy (read-only) |
| `company` | Link | Required |
| `status` | Select | Draft / Approved / Rejected / Cancelled / Invoiced / Partly Invoiced |
| `mode_of_approval` | Select | Manual / Automatic (read-only, from eligibility) |

**Service Definition**

| Field | Type | Description |
|-------|------|-------------|
| `template_dt` | Link → DocType | Service type (Lab Test Template, Clinical Procedure Template, etc.) |
| `template_dn` | Dynamic Link | Specific service template |
| `item_code` | Link → Item | Resolved item for billing |
| `qty` | Float | Quantity (required) |

**Coverage Calculation (all read-only, auto-calculated)**

| Field | Type | Description |
|-------|------|-------------|
| `coverage` | Percent | From eligibility lookup |
| `discount` | Percent | From eligibility lookup |
| `price_list_rate` | Currency | Rate from insurance price list |
| `discount_amount` | Currency | `price_list_rate × discount%` |
| `amount` | Currency | `price_list_rate - discount_amount` |
| `coverage_amount` | Currency | `amount × coverage%` — what insurance pays |
| `patient_payable` | Currency | `amount - coverage_amount` — what patient pays |
| `coverage_validity_end_date` | Date | From eligibility rule |

**Invoice Tracking**

| Field | Type | Description |
|-------|------|-------------|
| `qty_invoiced` | Float | Quantity already billed |
| `coverage_amount_invoiced` | Currency | Amount already claimed |
| `approved_amount` | Currency | Amount approved by payor |
| `paid_amount` | Currency | Amount settled |

**Submittable**: Yes

### Coverage Calculation Flow

On `validate()`:

1. **Validate policy** — checks policy exists and isn't expired
2. **Resolve service item** — extracts `item_code` from the service template
3. **Look up eligibility** — calls `get_insurance_eligibility()` with item_code + plan
4. **Get price** — checks plan price list → contract price list → default selling price list
5. **Calculate amounts**:
   - `discount_amount = price_list_rate × discount%`
   - `amount = price_list_rate - discount_amount`
   - `coverage_amount = amount × coverage%`
   - `patient_payable = amount - coverage_amount`

### Status Flow

```
                    ┌──→ Approved ──→ Partly Invoiced ──→ Invoiced
Draft ──→ submit ──┤
                    └──→ Rejected

Approved/Rejected can be set manually (via custom buttons) or automatically
(if mode_of_approval = "Automatic" and coverage_amount > 0 → auto-Approved)
```

### Creating Coverage via API

```python
# Create coverage from a policy (validates active contract, calculates amounts)
frappe.call(
    "healthcare.healthcare.doctype.patient_insurance_coverage.patient_insurance_coverage.make_insurance_coverage",
    {
        "insurance_policy": "INS-PLCY-2024-00001",
        "template_dt": "Lab Test Template",
        "template_dn": "Blood Test - CBC",
        "qty": 1,
        "company": "Hospital Corp"
    }
)
# Returns: Coverage doc (auto-submitted if Automatic approval)
```

---

## Part 7: Sales Invoice Integration

When a healthcare service with insurance coverage is invoiced, the system splits amounts between insurance and patient.

### Custom Fields on Sales Invoice

**Invoice level:**

| Field | Type | Description |
|-------|------|-------------|
| `total_insurance_coverage_amount` | Currency | Sum of all item coverage amounts (read-only) |
| `patient_payable_amount` | Currency | `outstanding_amount - total_insurance_coverage_amount` (read-only) |

**Item level:**

| Field | Type | Description |
|-------|------|-------------|
| `insurance_coverage` | Link → Patient Insurance Coverage | Links item to pre-approved coverage |
| `coverage_rate` | Currency | Approved rate |
| `coverage_qty` | Float | Approved quantity |
| `coverage_percentage` | Percent | Coverage % |
| `insurance_coverage_amount` | Currency | `amount × coverage_percentage%` |
| `insurance_payor` | Link | Payor reference |
| `patient_insurance_policy` | Data | Policy number |

### Calculation Logic (sales_invoice.py)

`calculate_patient_insurance_coverage()`:
- For each item with `insurance_coverage` set:
  - Calculates `insurance_coverage_amount = amount × coverage_percentage%`
- Sums to `total_insurance_coverage_amount`
- Sets `patient_payable_amount = outstanding_amount - total_insurance_coverage_amount`

### Invoice Hooks

```python
# hooks.py
"Sales Invoice": {
    "validate": "healthcare.healthcare.utils.manage_invoice_validate",
    "on_submit": "healthcare.healthcare.utils.manage_invoice_submit_cancel",
    "on_cancel": "healthcare.healthcare.utils.manage_invoice_submit_cancel",
}
```

On submit: updates `qty_invoiced` and `coverage_amount_invoiced` in the linked Patient Insurance Coverage documents, transitioning their status to Partly Invoiced or Invoiced.

---

## Part 8: Insurance Claim

Aggregates invoiced coverages into a single claim document for submission to the payor. Creates Payment Entries for settlement.

Navigate to: Insurance Claim > New

### Fields

**Header**

| Field | Type | Description |
|-------|------|-------------|
| `naming_series` | Data | INS-CLAIM-.YYYY.- |
| `insurance_payor` | Link | Required |
| `patient` | Link | Required |
| `insurance_policy` | Link | Required |
| `company` | Link | Required |
| `posting_date` | Date | Default: today |
| `from_date` / `to_date` | Date | Date range filter for fetching coverages |
| `posting_date_based_on` | Select | Insurance Coverage / Sales Invoice — which date to filter on |
| `mode_of_payment` | Link | Required — how payor will pay |
| `payment_account` | Link → Account | Fetched from mode_of_payment |
| `due_date` | Date | Payment due date (required) |

**Coverage Lines**

| Field | Type | Description |
|-------|------|-------------|
| `coverages` | Table (Insurance Claim Coverage) | Populated via "Get Coverages" button, editable after submit |

**Totals (all read-only)**

| Field | Type | Description |
|-------|------|-------------|
| `insurance_claim_amount` | Currency | Total claimed |
| `approved_amount` | Currency | Total approved by payor |
| `rejected_amount` | Currency | Total rejected |
| `outstanding_amount` | Currency | Approved but unpaid |
| `paid_amount` | Currency | Settled amount |
| `status` | Select | Draft / Submitted / Completed / Cancelled / Error |

**Submittable**: Yes

### Workflow

1. **Create claim** — select payor, patient, policy, date range
2. **Fetch coverages** — click "Get Coverages" button
   - API: `get_coverages()` — complex SQL JOIN across Patient Insurance Coverage + Sales Invoice Items + Journal Entry Accounts
   - Fetches all coverages with status Partly Invoiced or Invoiced within the date range
   - Excludes coverages already in another claim
   - Populates the `coverages` child table
3. **Submit claim** — sets status to "Submitted", updates each coverage's claim link
4. **Review & approve** — use "Update Claim Status" button to batch-set coverage statuses (Approved/Rejected/Error)
   - For each coverage row, set `approved_amount` or `rejected_amount`
   - Claim recalculates totals automatically
5. **Create payment** — click "Create Payment Entry" for approved amounts
   - API: `create_payment_entry()` — builds Payment Entry referencing Journal Entry rows
   - Sets party = Insurance Payor's linked Customer
   - Payment amount = outstanding_amount
6. **Payment reconciliation** — on Payment Entry submit:
   - Hook: `validate_payment_entry_and_set_claim_fields()` — links PE references to claim coverages
   - Hook: `update_claim_paid_amount()` — updates paid_amount per coverage row
   - Claim status transitions to **Completed** when all approved amounts are paid

### Claim Status State Machine

```
Draft ──→ Submitted ──┬──→ Completed  (all approved + fully paid)
                      ├──→ Error      (all coverages rejected)
                      ├──→ Cancelled  (manual cancellation)
                      └──→ Submitted  (mixed statuses or pending)
```

Implemented in `before_update_after_submit()` — recalculates on every coverage status change.

### Cancellation Rules

- Cannot cancel if status is "Completed"
- Cannot cancel if `paid_amount > 0`
- On cancel: all coverage rows reset to "Cancelled" status

### Payment Entry Hooks

```python
# hooks.py
"Payment Entry": {
    "validate": "healthcare.healthcare.doctype.insurance_claim.insurance_claim.validate_payment_entry_and_set_claim_fields",
    "on_submit": [
        "healthcare.healthcare.custom_doctype.payment_entry.manage_payment_entry_submit_cancel",
        "healthcare.healthcare.custom_doctype.payment_entry.set_paid_amount_in_healthcare_docs",
    ],
    "on_cancel": [
        "healthcare.healthcare.custom_doctype.payment_entry.manage_payment_entry_submit_cancel",
        "healthcare.healthcare.custom_doctype.payment_entry.set_paid_amount_in_healthcare_docs",
    ],
}
```

### Custom Fields for Payment Tracking

| DocType | Field | Type | Description |
|---------|-------|------|-------------|
| Journal Entry | `insurance_coverage` | Link → Patient Insurance Coverage | Links JE to coverage |
| Payment Entry Reference | `insurance_claim` | Link → Insurance Claim | Links payment to claim |
| Payment Entry Reference | `insurance_claim_coverage` | Link → Insurance Claim Coverage | Links payment to coverage row |

---

## Insurance Workspace & Dashboard

Located at: Healthcare workspace > Insurance section

### Dashboard Components

| Component | Type | Description |
|-----------|------|-------------|
| Insurance Claim Status | Chart | Visual breakdown of claim statuses |
| Outstanding Claims | Number Card | Count of unpaid claims |
| Outstanding Claims Value | Number Card | Sum of outstanding amounts |
| Paid Claims | Number Card | Count of settled claims |
| Paid Claims Value | Number Card | Sum of paid amounts |

### Quick Access Shortcuts

- Insurance Claims
- Patient Insurance Coverage
- Eligibility Plans
- Sales Invoices (filtered to insurance)

---

## Complete API Reference

| Method | DocType | Purpose |
|--------|---------|---------|
| `has_active_contract()` | Insurance Payor | Check if active contract exists for company+date |
| `is_insurance_policy_valid()` | Patient Insurance Policy | Validate policy hasn't expired |
| `get_insurance_price_lists()` | Patient Insurance Policy | Get plan and default price lists |
| `make_insurance_coverage()` | Patient Insurance Coverage | Create and auto-approve coverage |
| `create_insurance_eligibility()` | Patient Insurance Coverage | Generate eligibility rule from coverage |
| `get_insurance_eligibility()` | Item Insurance Eligibility | Look up coverage terms for item/service |
| `get_coverages()` | Insurance Claim | Fetch invoiced coverages for claim |
| `create_payment_entry()` | Insurance Claim | Generate Payment Entry for approved claims |
| `validate_payment_entry_and_set_claim_fields()` | Insurance Claim | Link PE to claim on validate |
| `update_claim_paid_amount()` | Insurance Claim | Update paid amounts on PE submit/cancel |

---

## Price List Precedence

When calculating coverage amounts, the system resolves the price list in this order:

1. **Plan price list** — `Insurance Payor Eligibility Plan.price_list`
2. **Contract price list** — `Insurance Payor Contract.default_price_list`
3. **Default selling price list** — ERPNext global default

---

## Error Handling

| Error | Trigger | Resolution |
|-------|---------|------------|
| No active contract | Creating coverage when no valid contract exists | Create/activate a contract for the payor+company |
| Policy expired | Coverage creation with expired policy | Renew the policy (new document) |
| No eligibility found | Item/service has no matching eligibility rule | Create an Item Insurance Eligibility record |
| Overlap (contract) | Multiple active contracts for same payor+company | Deactivate one contract |
| Overlap (policy) | Duplicate policies for same patient+payor+plan | Cancel the duplicate |
| Overlap (eligibility) | Duplicate eligibilities for same item+plan | Deactivate or adjust date ranges |
| Claim already exists | Coverage already linked to another claim | Check existing claims for the coverage |
| Cannot cancel paid claim | Cancelling claim with paid_amount > 0 | Reverse the payment entry first |

---

## Related Code

**DocTypes:**
- Insurance Payor: `healthcare/healthcare/doctype/insurance_payor/insurance_payor.py`
- Contract: `healthcare/healthcare/doctype/insurance_payor_contract/insurance_payor_contract.py`
- Eligibility Plan: `healthcare/healthcare/doctype/insurance_payor_eligibility_plan/`
- Item Eligibility: `healthcare/healthcare/doctype/item_insurance_eligibility/item_insurance_eligibility.py`
- Patient Policy: `healthcare/healthcare/doctype/patient_insurance_policy/patient_insurance_policy.py`
- Patient Coverage: `healthcare/healthcare/doctype/patient_insurance_coverage/patient_insurance_coverage.py`
- Insurance Claim: `healthcare/healthcare/doctype/insurance_claim/insurance_claim.py`
- Claim Coverage (child): `healthcare/healthcare/doctype/insurance_claim_coverage/insurance_claim_coverage.json`

**Integration:**
- Custom fields: `healthcare/healthcare/healthcare.py`
- Sales Invoice override: `healthcare/healthcare/custom_doctype/sales_invoice.py`
- Payment Entry hooks: `healthcare/healthcare/custom_doctype/payment_entry.py`
- Invoice utilities: `healthcare/healthcare/utils.py`
- Hook registration: `healthcare/hooks.py`

**Workspace:**
- `healthcare/healthcare/workspace/insurance/insurance.json`
