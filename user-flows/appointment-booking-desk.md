# Appointment Booking (Desk)

## User Story

> As a **receptionist**, I want to book appointments for patients so that they can see the right practitioner at the right time.

## Entry Point

Desk > Patient Appointment > New

## Steps

1. **Select patient**
   - Screen: Patient Appointment form
   - User action: Search and select patient
   - Data changed: `patient` field set

2. **Select practitioner**
   - User action: Choose practitioner (filtered by department)
   - API: `get_practitioner_list` (standard query)

3. **Choose date and time**
   - User action: Pick date, system shows available slots
   - API: `patient_appointment.get_availability_data(date, practitioner, appointment_type, ...)`
   - Data changed: `appointment_date`, `appointment_time` set

4. **Check fee validity**
   - Automatic: `check_fee_validity(appointment)` determines if this is a free follow-up
   - Data changed: Fee Validity record updated if applicable

5. **Save appointment**
   - User action: Save
   - Data changed: Patient Appointment created (status: Open)

6. **Invoice** *(if payment required)*
   - User action: Click "Invoice" button
   - API: `invoice_appointment(appointment_name, discount_percentage, discount_amount)`
   - Data changed: Sales Invoice created, appointment marked as invoiced

7. **Create encounter** *(when patient arrives)*
   - User action: Click "Create > Patient Encounter"
   - API: `make_encounter(source_name)`
   - Data changed: Patient Encounter created, linked to appointment

## Error States

- No available slots → system shows "No slots available" message
- Double booking → overlap validation prevents save
- Patient not linked to customer → warning on invoice attempt

## Permissions

- **Healthcare Administrator**: Full appointment access
- **Physician**: Can create and view appointments

## Related Code

- Backend: `healthcare/healthcare/doctype/patient_appointment/patient_appointment.py`
- Fee validity: `healthcare/healthcare/doctype/fee_validity/fee_validity.py`
