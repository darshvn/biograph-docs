# Mark Patient Appointment Unavailable

## User Story

> As a **healthcare administrator**, I want to mark time blocks as unavailable for a practitioner or service unit so that patients cannot book appointments during those periods.

> As a **front-desk staff**, I want to cancel an unavailability block so that the practitioner's schedule opens back up.

## Entry Point

- **Desk**: Healthcare workspace > Patient Appointment > Check Availability > Block Booking checkbox
- **Calendar**: Calendar view of Patient Appointment

## How It Works

Biograph treats unavailability as a special appointment. When a block is marked unavailable, the system creates a Patient Appointment with `appointment_type = "Unavailable"` and `status = "Unavailable"`. This appointment then blocks regular bookings through overlap validation.

## Steps

1. **Open the availability dialog**
   - Screen: Patient Appointment form > "Check Availability" button
   - User action: Select practitioner and date
   - API: `get_availability_data(date, practitioner)` fetches existing slots

2. **Enable Block Booking**
   - User action: Check the "Block Booking" checkbox in the availability dialog
   - UI change: Shows `from_time` and `to_time` fields instead of slot selection
   - Validation (JS line 581): `from_time` must be before `to_time`
   - Validation (JS line 612–616): Cannot create blocks in the past

3. **Check for conflicts**
   - API: `frappe.call("healthcare.healthcare.doctype.patient_appointment.patient_appointment.check_unavailability_conflicts", { filters })`
   - Checks if any existing appointments overlap the selected time range
   - If conflicts found: shows conflict dialog listing affected appointments
   - If no conflicts: proceeds to create the block

4. **Create the unavailability appointment**
   - API: `frappe.call("healthcare.healthcare.doctype.patient_appointment.patient_appointment.create_unavailability_appointment", { data })`
   - Parameters:
     - `unavailability_for`: "Practitioner" or "Service Unit"
     - `practitioner` / `service_unit`: the resource to block
     - `date`, `from_time`, `to_time`: the block period
     - `reason`: optional text note
   - Backend actions:
     - Sets up "Unavailable" appointment type if it doesn't exist (`setup_appointment_type_for_unavailability()` — duration 60min, color `#ff5858`, price 0)
     - Creates Patient Appointment with `status = "Unavailable"`
     - Calculates duration from time range
     - Creates a linked calendar **Event** (red color `#ff5858`, private, no Google Calendar sync)

5. **Regular bookings are now blocked**
   - When any new appointment is saved, `validate_overlaps()` (line 522–679) runs
   - Fetches all unavailable appointments for the practitioner on that date
   - Checks 4 overlap conditions: start-during, end-during, contains, exact-match
   - Throws `OverlapError` if the new appointment overlaps an unavailable block
   - In the slot selection UI (JS line 1154–1159): unavailable slots appear **disabled** with tooltip "Practitioner unavailable at this time"

6. **Cancel the unavailability**
   - Screen: Open the unavailable appointment
   - User action: Click "Cancel Unavailability" button (JS line 97–100)
   - Confirmation dialog shown
   - API: `frappe.call("healthcare.healthcare.doctype.patient_appointment.patient_appointment.cancel_unavailability_appointment", { appointment_name })`
   - Backend: Directly sets status to "Cancelled" via `frappe.db.set_value()` (bypasses set_only_once validation)
   - Cancels linked calendar Event if it exists

## Unavailable Appointment Fields

| Field | Type | Description |
|-------|------|-------------|
| `appointment_type` | Link | Set to "Unavailable" |
| `status` | Select | Set to "Unavailable" |
| `appointment_date` | Date | Block date |
| `appointment_time` | Time | Block start time |
| `end_time` | Time | Calculated from duration |
| `duration` | Int | Minutes, calculated from time range |
| `notes` | Small Text | Reason for unavailability |
| `event` | Link → Event | Linked calendar event (red, private) |

## Error States

- Time range in the past → validation error (JS)
- `from_time` >= `to_time` → validation error (JS)
- Conflicts with existing appointments → conflict dialog shown, user must resolve
- Overlap with another unavailable block → `OverlapError` thrown

## Permissions

- **Healthcare Administrator**: Can create and cancel unavailability blocks
- **Physician**: Can mark their own schedule as unavailable
- Patients cannot see unavailable blocks — slots simply appear disabled in the portal

## Related Code

- Backend: `healthcare/healthcare/doctype/patient_appointment/patient_appointment.py`
  - `create_unavailability_appointment()` — lines 1905–2050
  - `cancel_unavailability_appointment()` — lines 1855–1888
  - `setup_appointment_type_for_unavailability()` — lines 1890–1902
  - `validate_overlaps()` — lines 522–679
  - `insert_calendar_event()` — lines 223–345
- Frontend: `healthcare/healthcare/doctype/patient_appointment/patient_appointment.js`
  - `check_and_set_availability()` — lines 509–1000
  - Cancel button — lines 97–100
  - Slot disable logic — lines 1154–1159
