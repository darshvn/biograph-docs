# Healthcare Service Unit Setup

## Overview

Healthcare Service Units represent the physical locations in a hospital — buildings, floors, wards, consultation rooms, labs, operation theaters. They form a **tree structure** (parent-child hierarchy) and control appointment scheduling, inpatient bed management, and stock warehouse linking.

## Tree Structure

Service units use Frappe's nested set model (`lft`/`rgt` fields) for hierarchical organization:

```
Hospital (is_group=1)
├── Outpatient Block (is_group=1)
│   ├── Consultation Room 1 (allow_appointments=1)
│   ├── Consultation Room 2 (allow_appointments=1, overlap=1, capacity=3)
│   └── Minor Procedure Room (allow_appointments=1)
├── Inpatient Block (is_group=1)
│   ├── General Ward (is_group=1)
│   │   ├── Bed GW-001 (inpatient_occupancy=1)
│   │   ├── Bed GW-002 (inpatient_occupancy=1)
│   │   └── Bed GW-003 (inpatient_occupancy=1)
│   └── ICU (is_group=1)
│       ├── Bed ICU-001 (inpatient_occupancy=1)
│       └── Bed ICU-002 (inpatient_occupancy=1)
└── Laboratory (allow_appointments=1)
```

## Healthcare Service Unit Fields

| Field | Type | Description |
|-------|------|-------------|
| `healthcare_service_unit_name` | Data | Required, display name |
| `is_group` | Check | If checked, this is a container (floor, wing, ward) — not bookable |
| `service_unit_type` | Link → Healthcare Service Unit Type | Required if not a group. Defines behavior. |
| `allow_appointments` | Check | Fetched from service_unit_type, read-only |
| `overlap_appointments` | Check | Fetched from service_unit_type, read-only |
| `service_unit_capacity` | Int | Max concurrent appointments. Required if overlap_appointments=1 |
| `inpatient_occupancy` | Check | Fetched from service_unit_type, read-only. Marks as a bed. |
| `occupancy_status` | Select | Vacant / Occupied (auto-managed, read-only) |
| `company` | Link → Company | Required. Limits visibility to company users. |
| `warehouse` | Link → Warehouse | ERPNext warehouse for stock consumption (e.g., procedure consumables) |
| `parent_healthcare_service_unit` | Link | Parent in the tree |

**Constraints**: `healthcare_service_unit_name` + `company` must be unique (enforced at database level, line 141).

## Healthcare Service Unit Type

Defines the **behavior template** for service units. Create types first, then assign to units.

| Field | Type | Description |
|-------|------|-------------|
| `service_unit_type` | Data | Unique name (e.g., "Consultation Room", "Ward Bed", "Lab") |
| `allow_appointments` | Check | Units of this type accept appointments |
| `overlap_appointments` | Check | Allow multiple simultaneous appointments (visible if allow_appointments=1) |
| `inpatient_occupancy` | Check | Units of this type are beds/rooms for inpatients (visible if allow_appointments=0) |
| `is_billable` | Check | Whether occupancy in this unit type generates charges (visible if inpatient_occupancy=1) |
| `disabled` | Check | Deactivate this type |

**If billable** (inpatient rooms that charge per stay):
| Field | Type | Description |
|-------|------|-------------|
| `item` | Link → Item | Auto-created ERPNext Item (read-only) |
| `item_code` | Data | Item code (read-only) |
| `item_group` | Link → Item Group | Required |
| `uom` | Link → UOM | Required (usually "Hour" or "Day") |
| `no_of_hours` | Int | Hours per UOM unit (for rate calculation) |
| `rate` | Currency | Charge rate per UOM |
| `description` | Data | Fetched from Item |

## Capacity & Overlap Rules

### No Overlap (default)
- `overlap_appointments = 0`
- Only one appointment per time slot
- Standard for consultation rooms

### Overlap with Capacity
- `overlap_appointments = 1`
- `service_unit_capacity = N` (required, must be > 0)
- Up to N appointments in the same time slot
- Used for: multi-chair dental clinics, physiotherapy rooms, group consultation
- The slot selection UI shows a badge with remaining capacity and disables the slot when full

### Validation (healthcare_service_unit.py)
`set_service_unit_properties()` (line 42):
- Copies `allow_appointments`, `overlap_appointments`, `inpatient_occupancy` from the service unit type
- Validates: if `overlap_appointments=1`, then `service_unit_capacity` must be > 0

## Bulk Creation

For wards with many beds, use the bulk creation API:

```python
frappe.call(
    "healthcare.healthcare.doctype.healthcare_service_unit.healthcare_service_unit.add_multiple_service_units",
    {
        "parent": "General Ward",
        "service_unit_type": "Ward Bed",
        "company": "Your Hospital",
        "count": 20,            # number of units to create
        "prefix": "GW-Bed"     # naming prefix (generates GW-Bed-0001 through GW-Bed-0020)
    }
)
```

This whitelist method (line 73) creates units with auto-incrementing names under the specified parent.

## Related Code

- Service Unit: `healthcare/healthcare/doctype/healthcare_service_unit/healthcare_service_unit.py`
  - `set_service_unit_properties()` — line 42
  - `add_multiple_service_units()` — line 73 (whitelisted)
  - `on_doctype_update()` — line 141 (unique constraint)
- Service Unit Type: `healthcare/healthcare/doctype/healthcare_service_unit_type/healthcare_service_unit_type.json`
