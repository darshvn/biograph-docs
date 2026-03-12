# Patient Registration

## User Story

> As a **healthcare administrator**, I want to register new patients so that their demographics and medical history are captured in the system.

> As a **patient**, I want to self-register via the portal so that I can book appointments online.

## Entry Point

- **Desk**: Healthcare workspace > Patient > New
- **Portal**: `/patient-registration` web form (guest access)

## Steps

1. **Fill demographics**
   - Screen: Patient form (Desk) or `/patient-registration` (Portal)
   - User action: Enter name, DOB, gender, contact, address
   - Data changed: Patient record created

2. **Add medical history** *(optional)*
   - User action: Record allergies, medications, surgeries
   - Data changed: Allergy, Medication History Item, Surgery History Item child records

3. **Link user account** *(admin only)*
   - User action: Set `Patient.user_id` to a website user email
   - Data changed: Patient linked to user, enabling portal access
   - API: Standard Frappe CRUD

4. **Duplicate check**
   - Automatic: `check_patient_duplicates(patient)` runs on save
   - Error: Warns if matching patient found

## Error States

- Duplicate patient detected → warning shown, admin decides to proceed or merge
- Missing required fields → validation error on save

## Permissions

- **Healthcare Administrator**: Can create patients from Desk
- **Guest**: Can submit `/patient-registration` web form
- **Patient role**: Assigned after linking user account

## Related Code

- Backend: `healthcare/healthcare/doctype/patient/patient.py`
- Web form: `healthcare/healthcare/web_form/patient_registration/`
- Dedup: `healthcare/healthcare/utils.py` → `check_patient_duplicates()`
