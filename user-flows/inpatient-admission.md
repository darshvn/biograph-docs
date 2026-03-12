# Inpatient Admission → Discharge

## User Story

> As a **physician**, I want to admit, manage, and discharge inpatients so that bed occupancy, daily charges, and medications are tracked.

## Entry Point

Patient Encounter > Admission Order → Treatment Counselling

## Steps

1. **Treatment counselling** *(pre-admission)*
   - User action: Create from IP order in encounter
   - API: `inpatient_record.create_treatment_counselling(ip_order)`
   - Data changed: Treatment Counselling record

2. **Schedule admission**
   - API: `inpatient_record.schedule_inpatient(admission_order)`
   - Data changed: Inpatient Record created (status: Admission Scheduled)

3. **Admit patient**
   - User action: Assign bed/ward and check-in
   - API: `inpatient_record.admit(service_unit, check_in, expected_discharge, currency, price_list)` (instance)
   - Data changed: Inpatient Record status → Admitted, Inpatient Occupancy created

4. **Daily billing** *(automatic)*
   - Scheduler: `add_occupied_service_unit_in_ip_to_billables` runs daily
   - Data changed: Service unit rent added to Inpatient Record billable items

5. **Medication orders**
   - API: `patient_encounter.make_ip_medication_order(source_name)`
   - See: [Inpatient Medication](inpatient-medication.md) flow

6. **Transfer** *(if bed change needed)*
   - API: `inpatient_record.transfer(new_service_unit, check_in, leave_from)` (instance)
   - Data changed: New Inpatient Occupancy entry, previous one closed

7. **Discharge**
   - API: `inpatient_record.discharge()` (instance)
   - Data changed: Inpatient Record status → Discharged

8. **Discharge summary**
   - API: `inpatient_record.make_discharge_summary(source_name)`
   - Data changed: Discharge Summary document created

9. **Create insurance coverage** *(if insured)*
   - API: `inpatient_record.create_insurance_coverage()` (instance)
   - Data changed: Patient Insurance Coverage record

## Error States

- No available beds → service unit occupancy check fails
- Cancel admission → `set_ip_order_cancelled(inpatient_record, reason)`

## Permissions

- **Healthcare Administrator**: Full IP access
- **Physician**: Admit, discharge, transfer
- **Nursing User**: View occupancy, medication tasks

## Related Code

- Backend: `healthcare/healthcare/doctype/inpatient_record/inpatient_record.py` (13 methods)
- Treatment: `healthcare/healthcare/doctype/treatment_counselling/treatment_counselling.py`
- Discharge: `healthcare/healthcare/doctype/discharge_summary/discharge_summary.py`
