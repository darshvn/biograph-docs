# Patient Appointment Type Setup

## Overview

Appointment Types categorize appointments (e.g., "Consultation", "Follow Up", "Emergency", "Dental Checkup"). Each type defines default duration, calendar color, booking scope, and billing items with OP/IP pricing.

Navigate to: Healthcare workspace > Appointment Type > New

## Fields

### Basic Configuration

| Field | Type | Description |
|-------|------|-------------|
| `appointment_type` | Data | Unique name (e.g., "Consultation", "Follow Up") |
| `default_duration` | Int | Duration in minutes. Used to calculate slot intervals. |
| `color` | Color | Calendar display color (hex, e.g., `#4CAF50`) |
| `allow_booking_for` | Select | **Practitioner** (default), **Department**, or **Service Unit** — determines what the user selects when booking |

### Billing Items

The `items` child table (**Appointment Type Service Item**) lets you define different billing items for different departments or service units:

| Field | Type | Description |
|-------|------|-------------|
| `dt` | Link → DocType | Scope: "Medical Department" or "Healthcare Service Unit" (blank = generic/default) |
| `dn` | Dynamic Link | The specific department or service unit |
| `op_consulting_charge_item` | Link → Item | ERPNext Item for outpatient charge |
| `op_consulting_charge` | Currency | Outpatient fee amount |
| `inpatient_visit_charge_item` | Link → Item | ERPNext Item for inpatient charge |
| `inpatient_visit_charge` | Currency | Inpatient fee amount |

**Example configuration:**

| Scope | OP Item | OP Charge | IP Item | IP Charge |
|-------|---------|-----------|---------|-----------|
| *(generic)* | Consultation Fee | ₹500 | IP Visit Fee | ₹300 |
| Cardiology Dept | Cardio Consultation | ₹800 | Cardio IP Visit | ₹500 |
| Ortho Dept | Ortho Consultation | ₹700 | Ortho IP Visit | ₹450 |

### Medical Coding

| Field | Type | Description |
|-------|------|-------------|
| `codification_table` | Table (Codification Table) | Medical codes linked to this appointment type (e.g., CPT codes) |

### Price List Integration

| Field | Type | Description |
|-------|------|-------------|
| `price_list` | Link → Price List | ERPNext Price List for auto-creating Item Prices |

## Billing Lookup Logic

When an appointment is booked, `get_billing_details()` (appointment_type.py line 41) resolves the correct charges:

1. **Check department-specific items first**: If the practitioner's department matches an entry in the `items` table with `dt = "Medical Department"`, use those items
2. **Fall back to generic items**: If no department match, use entries where `dt` is blank
3. **Final fallback**: If no items defined on the appointment type, use Practitioner-level charges, then Healthcare Settings defaults

## Auto Item Price Creation

On `validate()` (appointment_type.py line 10):
- If `price_list` is set and `items` have charges configured
- Calls `make_item_price()` (line 75) for each charge item
- Creates/updates Item Price documents in the specified Price List
- This ensures appointment charges appear correctly in billing

## The "Unavailable" Type

The system auto-creates a special "Unavailable" appointment type for blocking time:
- Created by `setup_appointment_type_for_unavailability()`
- `default_duration`: 60 minutes
- `color`: `#ff5858` (red)
- `price`: 0
- See [Appointment Unavailability](user-flows/appointment-unavailability.md)

## Related Code

- DocType: `healthcare/healthcare/doctype/appointment_type/appointment_type.json`
- Python: `healthcare/healthcare/doctype/appointment_type/appointment_type.py`
  - `validate()` — line 10
  - `get_billing_details()` — line 41
  - `make_item_price()` — line 75
- Child table: `healthcare/healthcare/doctype/appointment_type_service_item/appointment_type_service_item.json`
