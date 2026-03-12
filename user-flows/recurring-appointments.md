# Recurring Appointments

## User Story

> As a **receptionist**, I want to schedule recurring appointments so that patients with ongoing treatment have consistent booking.

## Entry Point

Patient Appointment form > Recurring Appointments

## Steps

1. **Configure recurrence**
   - User action: Set frequency, start/end dates, practitioner, patient
   - Data changed: Recurrence parameters set

2. **Preview dates**
   - API: `recuring_appointment_handler.get_recurring_appointment_dates(data)`
   - Shows: List of proposed appointment dates

3. **Check availability**
   - API: `recuring_appointment_handler.get_availability(scheduled_details, practitioner, service_unit)`
   - Shows: Which dates/slots are available

4. **Get service unit**
   - API: `recuring_appointment_handler.get_service_unit_values(selected_practitioner)`
   - Returns: Service units assigned to the practitioner

5. **Book all appointments**
   - API: `recuring_appointment_handler.book_appointments(data)` or `create_recurring_appointments(data)`
   - Data changed: Multiple Patient Appointment records created in batch

## Error States

- Practitioner unavailable on some dates → those dates flagged/skipped
- Slot conflicts → batch booking skips conflicting slots

## Permissions

- **Healthcare Administrator**: Can create recurring appointments

## Related Code

- Backend: `healthcare/healthcare/doctype/patient_appointment/recuring_appointment_handler.py`
