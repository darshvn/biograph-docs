# Appointment Booking (Patient Portal)

## User Story

> As a **patient**, I want to book appointments online so that I don't need to call the clinic.

## Entry Point

`/patient-portal` > Book Appointment button

## Steps

1. **Select department**
   - Screen: BookAppointmentModel > DepartmentSelector
   - User action: Click a department card
   - API: `patient_portal.get_departments()`
   - Note: Skipped if only one department has `show_in_portal=1`

2. **Select practitioner**
   - Screen: PractitionerSelector
   - User action: Click a practitioner card (shows photo, name, designation)
   - API: `patient_portal.get_practitioners(department)`

3. **Select date**
   - Screen: Calendar component
   - User action: Pick a date from calendar

4. **Select time slot**
   - Screen: Time slot grid
   - User action: Click an available slot
   - API: `patient_portal.get_slots(practitioner, date)`
   - Logic: Excludes booked slots and past times for today

5. **Select patient** *(if has relations)*
   - Screen: Patient picker
   - User action: Choose self or a dependent
   - API: `patient_portal.get_patients()`
   - Note: Auto-selects if user has no patient relations

6. **Review fees**
   - Screen: Confirmation dialog
   - API: `patient_portal.get_fees(practitioner, date)`
   - Shows: Billing item, amount, currency

7. **Confirm and book**
   - User action: Click "Book"
   - API: `patient_portal.make_appointment(practitioner, patient, date, slot)`
   - Data changed: Patient Appointment created (status: Scheduled)

8. **Pay** *(if payment gateway configured)*
   - API: `patient_portal.get_payment_link(doctype, docname, ...)`
   - User action: Redirected to Razorpay/Stripe
   - Data changed: Healthcare Payment Record created

## Error States

- Past date selected → `"Cannot fetch slots for past dates"` error
- No slots available → returns null, UI shows "no slots"
- Payment failure → patient redirected back, payment record stays pending

## Permissions

- **Patient role** required (linked via `Patient.user_id`)

## Related Code

- Frontend: `patient_portal/src/components/BookAppointmentModel.vue`
- Backend: `healthcare/healthcare/api/patient_portal.py`
