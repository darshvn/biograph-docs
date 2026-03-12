# Therapy & Rehabilitation

## User Story

> As a **physiotherapist**, I want to create therapy plans and conduct sessions so that patient rehabilitation progress is tracked and billed.

## Entry Point

- Service Request from Patient Encounter
- Direct: Desk > Therapy Plan > New

## Steps

1. **Create therapy plan**
   - User action: New Therapy Plan for patient
   - API: `therapy_plan.set_therapy_details_from_template()` (instance)
   - Data changed: Therapy Plan with detail items from template

2. **Schedule sessions**
   - API: `therapy_plan.make_therapy_session(therapy_plan, patient, therapy_type, company, appointment)`
   - Data changed: Therapy Session record created

3. **Create appointment** *(optional)*
   - API: `therapy_plan.make_patient_appointment(source_name)`
   - Data changed: Patient Appointment linked to therapy

4. **Conduct therapy session**
   - Screen: Therapy Session form
   - User action: Record exercises performed, duration, notes

5. **Consume stocks** *(if consumables used)*
   - API: `therapy_session.verify_stock()` → `consume_stocks()` (instance)
   - API: `therapy_session.make_stock_entry(doc)`
   - Data changed: Stock Entry (Material Issue)

6. **Assess patient** *(periodic)*
   - API: `patient_assessment.create_patient_assessment(source_name)`
   - Data changed: Patient Assessment with scored parameters

7. **Invoice sessions**
   - API: `therapy_session.invoice_therapy_session(source_name)` or `therapy_plan.make_sales_invoice(...)`
   - Data changed: Sales Invoice created

8. **Track progress**
   - Screen: Patient Progress desk page
   - API: `patient_progress.get_therapy_sessions_count(patient)`, `get_therapy_progress_data(patient, therapy_type, time_span)`
   - Shows: Heatmaps, distribution charts, assessment correlations

## Error States

- Session limit exceeded → `validate_no_of_session(therapy_plan)` prevents creation
- Stock insufficient → `verify_stock()` shows warning

## Permissions

- **Physician**: Full therapy access
- **Nursing User**: Can conduct sessions

## Related Code

- Plan: `healthcare/healthcare/doctype/therapy_plan/therapy_plan.py`
- Session: `healthcare/healthcare/doctype/therapy_session/therapy_session.py`
- Assessment: `healthcare/healthcare/doctype/patient_assessment/patient_assessment.py`
- Progress page: `healthcare/healthcare/page/patient_progress/patient_progress.py`
