# Healthcare Settings

## Overview

Healthcare Settings is a **Single** DocType — there's exactly one instance. It controls global feature toggles, default items, billing accounts, lab configuration, SMS templates, and patient duplicate checking.

Navigate to: `/app/healthcare-settings` or Search bar > Healthcare Settings

## Configuration Sections

### Tab: OP & IP

#### Outpatient Settings

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `patient_name_by` | Select | Patient Name | How patient records are named: Patient Name / Naming Series / Auto Name |
| `link_customer_to_patient` | Check | ✓ | Auto-create ERPNext Customer when creating a Patient (required for billing) |
| `default_code_system` | Link → Code System | — | Default medical coding system (e.g., ICD-10) |
| `default_google_calendar` | Link → Google Calendar | — | Default calendar for appointment sync |
| `collect_registration_fee` | Check | ✗ | Charge a fee when registering a new patient |
| `registration_item` | Link → Item | — | ERPNext Item for registration fee (visible when collect_registration_fee=1) |
| `registration_fee` | Currency | — | Fee amount |
| `show_payment_popup` | Check | ✗ | Show payment dialog during appointment booking |
| `enable_free_follow_ups` | Check | ✗ | Allow free follow-up visits within a time window |
| `max_visits` | Int | — | Number of free follow-ups allowed |
| `valid_days` | Int | — | Window (in days) for free follow-ups |

#### Inpatient Settings

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `allow_discharge_despite_unbilled_services` | Check | ✗ | Allow patient discharge even if services haven't been invoiced |
| `do_not_bill_inpatient_encounters` | Check | ✗ | Skip encounter-based billing for inpatients |
| `validate_nursing_checklists` | Check | ✗ | Require nursing checklists to be completed before procedures |

### Tab: Laboratory

#### Lab Settings

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `create_lab_test_on_si_submit` | Check | ✓ | Auto-create Lab Test when Sales Invoice with lab items is submitted |
| `create_observation_on_si_submit` | Check | ✗ | Auto-create Observation when Sales Invoice with observation items is submitted |
| `create_sample_collection_for_lab_test` | Check | ✗ | Auto-create Sample Collection document for new lab tests |
| `lab_test_approval_required` | Check | ✗ | Require approval workflow for lab test results |
| `employee_name_and_designation_in_print` | Check | ✓ | Show signing physician's name and designation in lab report print |
| `custom_signature_in_print` | Small Text | — | Custom signature text (alternative to employee name) |

#### Lab SMS Alerts

| Field | Type | Description |
|-------|------|-------------|
| `sms_printed` | Small Text | SMS template when lab report is printed. Variables: `{patient_name}`, `{lab_test}`, `{result}` |
| `sms_emailed` | Small Text | SMS template when lab report is emailed |

### Tab: Billing & Accounts

#### Default Healthcare Service Items

| Field | Type | Description |
|-------|------|-------------|
| `inpatient_visit_charge_item` | Link → Item | Default Item for inpatient visit charges |
| `op_consulting_charge_item` | Link → Item | Default Item for outpatient consultation charges |
| `clinical_procedure_consumable_item` | Link → Item | Default Item for clinical procedure consumables |

These are fallback defaults — Appointment Type and Practitioner-level items take precedence.

#### Default Accounts

| Field | Type | Description |
|-------|------|-------------|
| `income_account` | Table (Party Account) | Default income accounts per company |
| `receivable_account` | Table (Party Account) | Default receivable accounts per company |

### Tab: Orders

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `process_service_request_only_if_paid` | Check | ✗ | Require payment before processing Service Requests |
| `validate_medication_quantity_in_invoice` | Check | ✗ | Validate prescribed medication quantity matches invoice |
| `default_intent` | Link → Code Value | — | Default intent for orders (e.g., "order", "proposal") |
| `default_priority` | Link → Code Value | — | Default priority for orders (e.g., "routine", "urgent") |

### Tab: Alerts

#### Out Patient SMS Alerts

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `send_registration_msg` | Check | ✗ | Send SMS on patient registration |
| `registration_msg` | Small Text | — | Template with variables |
| `send_appointment_confirmation` | Check | ✗ | Send SMS on appointment booking |
| `appointment_confirmation_msg` | Small Text | — | Template with variables |
| `avoid_confirmation` | Check | ✗ | Skip confirmation SMS for same-day appointments |
| `send_appointment_reminder` | Check | ✗ | Send reminder SMS before appointment |
| `appointment_reminder_msg` | Small Text | — | Template with variables |
| `remind_before` | Time | — | How long before the appointment to send reminder |

### Tab: Patient Duplicate Check

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `enable_patient_duplicate_check` | Check | ✗ | Enable automatic duplicate detection on patient creation |
| `patient_duplicate_check_rules` | Table | — | Links to Patient Duplicate Check Rule Configuration documents |

See [Duplicate Checker Setup](admin-masters/duplicate-checker-setup.md) for full details.

## Item Hierarchy (Billing Precedence)

When determining which ERPNext Item to use for billing, the system checks in this order:

1. **Appointment Type** items (department-specific) → highest priority
2. **Appointment Type** items (generic, no department filter)
3. **Practitioner** charge items
4. **Healthcare Settings** default items → lowest priority (fallback)

## Related Code

- DocType: `healthcare/healthcare/doctype/healthcare_settings/healthcare_settings.json`
- Python: `healthcare/healthcare/doctype/healthcare_settings/healthcare_settings.py`
- JavaScript: `healthcare/healthcare/doctype/healthcare_settings/healthcare_settings.js`
