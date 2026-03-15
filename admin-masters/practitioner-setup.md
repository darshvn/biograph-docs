# Practitioner Master & Schedule Setup

## Overview

A **Healthcare Practitioner** represents a doctor, nurse, or other clinical staff member. Each practitioner can have one or more **Practitioner Schedules** that define when they're available for appointments, and at which service units.

## Healthcare Practitioner

### Creating a Practitioner

Navigate to: Healthcare workspace > Healthcare Practitioner > New

### Key Fields

**Identity & Type**

| Field | Type | Description |
|-------|------|-------------|
| `first_name` | Data | Required |
| `last_name` | Data | Optional |
| `practitioner_name` | Data | Auto-generated (first + last), read-only |
| `gender` | Link | Gender |
| `status` | Select | Active / Disabled |
| `practitioner_type` | Select | **Internal** (linked to Employee) or **External** (linked to Supplier) |

**Employee & User Linking**

| Field | Type | Description |
|-------|------|-------------|
| `employee` | Link → Employee | For internal practitioners — links to HR module |
| `user_id` | Link → User | Login account — enables desk access and auto-fills from Employee |
| `department` | Link → Department | Medical department |
| `designation` | Link → Designation | e.g., Senior Consultant, Resident |
| `supplier` | Link → Supplier | For external practitioners |

When `user_id` is set, the system automatically adds User Permissions so the practitioner can only see their own records.

When `employee` is set on the JS form (line 162), it auto-fetches `user_id`, names, and phone numbers from the Employee record.

**Consulting Charges**

| Field | Type | Description |
|-------|------|-------------|
| `op_consulting_charge_item` | Link → Item | ERPNext Item for outpatient consultation |
| `op_consulting_charge` | Currency | Fee amount |
| `inpatient_visit_charge_item` | Link → Item | ERPNext Item for inpatient visits |
| `inpatient_visit_charge` | Currency | Fee amount |

These can be overridden at the Appointment Type level for department-specific pricing.

**Schedule Assignment**

| Field | Type | Description |
|-------|------|-------------|
| `practitioner_schedules` | Table | Practitioner Service Unit Schedule (pairs a schedule with a service unit) |
| `google_calendar` | Link | Optional Google Calendar sync |

### Validation Rules (healthcare_practitioner.py)

- `validate_user_id()` (line 113): User must exist, be enabled, and not already assigned to another practitioner
- `validate_practitioner_schedules()` (line 83): If video conferencing is enabled on a schedule, validates prerequisites; checks for duplicate schedule+service_unit combinations
- `autoname()` (line 23): Document name = concatenation of first and last name

### API Methods

```python
# Get list of active practitioners (for dropdowns)
frappe.call("healthcare.healthcare.doctype.healthcare_practitioner.healthcare_practitioner.get_practitioner_list", {
    filters: { status: "Active" }
})

# Get supplier and user for a practitioner
frappe.call("healthcare.healthcare.doctype.healthcare_practitioner.healthcare_practitioner.get_supplier_and_user", {
    practitioner: "DR-001"
})
```

---

## Practitioner Schedule

### Creating a Schedule

Navigate to: Healthcare workspace > Practitioner Schedule > New

A Practitioner Schedule is a **reusable template** — one schedule can be assigned to multiple practitioners.

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `schedule_name` | Data | Unique name (e.g., "Morning OPD", "Evening Clinic") |
| `disabled` | Check | Deactivates the schedule |
| `allow_video_conferencing` | Check | Enables teleconsultation for slots in this schedule |
| `time_slots` | Table | Healthcare Schedule Time Slot entries |

### Time Slot Configuration

Each row in `time_slots` defines availability for one day:

| Field | Type | Description |
|-------|------|-------------|
| `day` | Select | Sunday through Saturday |
| `from_time` | Time | Slot window start (e.g., 09:00) |
| `to_time` | Time | Slot window end (e.g., 13:00) |
| `duration` | Float | Auto-calculated: to_time - from_time in minutes (read-only) |
| `maximum_appointments` | Int | Optional cap on appointments per slot window |

**Example**: A schedule with:
- Monday: 09:00–13:00 (4-hour window)
- Monday: 14:00–17:00 (3-hour window)
- Wednesday: 09:00–13:00

If the appointment duration is 15 minutes, the 09:00–13:00 window generates 16 slot buttons.

### Validation (practitioner_schedule.py line 10–27)

- Validates that time slot durations don't exceed available time in the window
- Checks for overlapping time slots on the same day

---

## Practitioner Service Unit Schedule (Linking)

This child table on the Healthcare Practitioner pairs a **schedule** with a **service unit**:

| Field | Type | Description |
|-------|------|-------------|
| `schedule` | Link → Practitioner Schedule | Which schedule to use |
| `service_unit` | Link → Healthcare Service Unit | Where the practitioner works during this schedule |

A practitioner can have multiple entries — e.g., "Morning OPD" at "Consultation Room 1" and "Evening Clinic" at "Consultation Room 3".

The `get_available_slots()` function iterates through these entries to calculate availability (see [Block & Slot Booking](user-flows/block-slot-booking.md)).

---

## Practitioner Availability (Overrides)

For one-off changes (time off, extra availability), use the **Practitioner Availability** DocType instead of modifying the schedule.

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `type` | Select | **Available** (add extra time) or **Unavailable** (block time) |
| `scope_type` | Select | Healthcare Practitioner / Healthcare Service Unit / Medical Department |
| `scope` | Dynamic Link | The specific practitioner, unit, or department |
| `status` | Select | Active / Expired |
| `start_date` / `end_date` | Date | Date range for the override |
| `start_time` / `end_time` | Time | Time range within each day |
| `duration` | Int | Auto-calculated (minutes), read-only |
| `reason` | Select | Time Off / Break / Training / Travel / Emergency (for unavailable type) |
| `repeat` | Select | Never / Daily / Weekly / Monthly |
| `monday`–`sunday` | Check | Repeat days (for weekly repeat) |
| `service_unit` | Link | Override location (for Available type only) |

### Validation (practitioner_availability.py)

- `validate_start_and_end()` (line 43): End time must be after start time
- `validate_availability_overlaps()` (line 55): Checks for overlapping availability/unavailability periods
- `validate_existing_appointments()` (line 153): Prevents creating unavailable blocks that conflict with existing patient appointments

## Related Code

- Practitioner: `healthcare/healthcare/doctype/healthcare_practitioner/healthcare_practitioner.py`
- Practitioner JS: `healthcare/healthcare/doctype/healthcare_practitioner/healthcare_practitioner.js`
- Schedule: `healthcare/healthcare/doctype/practitioner_schedule/practitioner_schedule.py`
- Availability: `healthcare/healthcare/doctype/practitioner_availability/practitioner_availability.py`
- Child table: `healthcare/healthcare/doctype/practitioner_service_unit_schedule/practitioner_service_unit_schedule.json`
- Time slots: `healthcare/healthcare/doctype/healthcare_schedule_time_slot/healthcare_schedule_time_slot.json`
