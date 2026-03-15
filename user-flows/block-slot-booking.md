# Block & Slot Appointment Booking

## User Story

> As a **front-desk staff**, I want to see available time slots for a practitioner so that I can book the right appointment for the patient.

> As a **healthcare administrator**, I want to configure slot capacity and overlap rules per service unit so that the schedule matches our operational capacity.

## Entry Point

- **Desk**: Patient Appointment > New > Check Availability button
- **Portal**: Patient Portal > Book Appointment wizard (Vue 3 SPA)

## Concepts

### Predefined Slots
Time slots are defined in **Practitioner Schedule** — a reusable template with day-wise time ranges. Each practitioner links to one or more schedules via the **Practitioner Service Unit Schedule** child table, which pairs a schedule with a service unit.

### Block Booking
A manual override that lets staff create a custom time block (instead of choosing a predefined slot). Used for unavailability blocks or special appointments outside regular slots.

### Slot Capacity
Each Healthcare Service Unit can allow overlapping appointments up to a configured capacity. This supports multi-chair clinics or shared consultation rooms.

## Steps — Predefined Slot Booking

1. **Select department and practitioner**
   - Screen: Check Availability dialog
   - User action: Choose Department → Practitioner → Date
   - JS triggers `check_and_set_availability()` (line 509)

2. **Fetch available slots**
   - API: `frappe.call("healthcare.healthcare.doctype.patient_appointment.patient_appointment.get_availability_data", { date, practitioner })`
   - Backend flow:
     1. Validates practitioner has a Practitioner Schedule assigned
     2. Calls `check_employee_wise_availability(date, practitioner_doc)` (line 1247):
        - If practitioner is Employee: checks for holidays via `is_holiday()`
        - If HRMS installed: checks for approved Leave Applications
        - Throws "Not Available" if on leave or holiday
     3. Calls `get_available_slots(practitioner_doc, date)` (line 1280)
   - Returns: `slot_details[]` array + `fee_validity` status

3. **Slot calculation logic** (`get_available_slots`, line 1280–1378)
   - For each entry in `practitioner_doc.practitioner_schedules`:
     - Validates entry has a `service_unit` assigned
     - Fetches matching `time_slots` from Practitioner Schedule where `day` = current weekday
     - Reads service unit config:
       - `allow_overlap`: from Healthcare Service Unit (line 1309–1313)
       - `service_unit_capacity`: max concurrent appointments (line 1312)
     - Fetches all existing appointments for that date (not Cancelled) — line 1323–1328
     - Fetches all unavailable appointments — line 1332–1342
     - Returns per-slot: `slot_name`, `service_unit`, `avail_slot` (time windows), `appointments`, `allow_overlap`, `service_unit_capacity`, `tele_conf`

4. **Render slot buttons** (JS `get_slots()`, line 1093–1242)
   - For each service unit in slot_details:
     - Displays schedule name and service unit with capacity badge
     - Shows video camera icon if `tele_conf` enabled
   - For each time slot window:
     - Generates buttons at intervals matching appointment duration
     - **Disabling logic:**
       - Past time slots → disabled (line 1136–1137)
       - Overlaps with "Unavailable" appointment → disabled + tooltip "Practitioner unavailable at this time" (line 1154–1159)
       - If `allow_overlap = 0`: disabled if any appointment already booked in that slot (line 1178–1202)
       - If `allow_overlap = 1`: counts booked appointments against `service_unit_capacity`; disabled when full
       - If `maximum_appointments` set on time slot: shows remaining count badge; disabled when reached (line 1212–1228)

5. **Select slot and book**
   - User action: Click an available slot button
   - Sets `appointment_time` and `duration` on the form
   - User completes remaining fields (patient, type) and saves

## Steps — Block Booking (Custom Time)

1. **Enable block booking**
   - User action: Check "Block Booking" checkbox in availability dialog (JS line 540–550)
   - UI: Shows `from_time` and `to_time` fields instead of slot buttons

2. **Select custom time range**
   - User action: Set start and end times
   - JS calculates `duration_minutes` from time difference (line 605–611)
   - Validates: not in the past (line 612–616), start < end (line 581)

3. **Check conflicts**
   - API: `check_unavailability_conflicts(filters)` (JS line 635)
   - If conflicts: shows dialog listing affected appointments
   - If none: creates the appointment/unavailability block

## Configuration Points

### Practitioner Schedule
| Field | Type | Purpose |
|-------|------|---------|
| `schedule_name` | Data | Unique name for the schedule template |
| `time_slots` | Table | Day-wise time slots (day, from_time, to_time, duration, maximum_appointments) |
| `allow_video_conferencing` | Check | Enables teleconference for this schedule |
| `disabled` | Check | Deactivates the schedule |

### Healthcare Service Unit (Capacity)
| Field | Type | Purpose |
|-------|------|---------|
| `allow_appointments` | Check | Whether this unit accepts appointments |
| `overlap_appointments` | Check | Allow multiple concurrent appointments |
| `service_unit_capacity` | Int | Max concurrent appointments (required if overlap=1) |

### Healthcare Schedule Time Slot
| Field | Type | Purpose |
|-------|------|---------|
| `day` | Select | Sunday–Saturday |
| `from_time` | Time | Slot window start |
| `to_time` | Time | Slot window end |
| `duration` | Float | Auto-calculated slot length |
| `maximum_appointments` | Int | Cap per slot window (optional) |

## Error States

- Practitioner has no schedule assigned → "Add a Practitioner Schedule" error
- Practitioner on leave/holiday → "Not Available" error
- All slots full → all buttons disabled, no booking possible
- Block booking time in past → validation error
- Block booking conflicts with existing appointments → conflict dialog

## Permissions

- **Healthcare Administrator / Physician**: Can use block booking and see all slots
- **Patient (Portal)**: Can only see and select available predefined slots
- Service unit capacity applies equally to all users

## Related Code

- Backend: `healthcare/healthcare/doctype/patient_appointment/patient_appointment.py`
  - `get_availability_data()` — lines 1197–1244
  - `get_available_slots()` — lines 1280–1378
  - `check_employee_wise_availability()` — lines 1247–1277
  - `validate_practitioner_schedules()` — lines 1381–1400
- Frontend: `healthcare/healthcare/doctype/patient_appointment/patient_appointment.js`
  - `check_and_set_availability()` — lines 509–1000
  - `get_slots()` — lines 1093–1242
- Config: `healthcare/healthcare/doctype/practitioner_schedule/practitioner_schedule.py`
- Config: `healthcare/healthcare/doctype/healthcare_service_unit/healthcare_service_unit.json`
