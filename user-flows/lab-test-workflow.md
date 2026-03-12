# Lab Test Workflow

## User Story

> As a **laboratory user**, I want to process lab orders from collection to results so that diagnostic reports are available to physicians and patients.

## Entry Point

- Service Request created from Patient Encounter (on submit)
- Direct: Desk > Lab Test > New

## Steps

1. **Service request created**
   - Trigger: Patient Encounter submitted with lab prescriptions
   - API: `patient_encounter.create_service_request(encounter)`
   - Data changed: Service Request (status: active)

2. **Create lab test**
   - User action: From Service Request, click "Create > Lab Test"
   - API: `service_request.make_lab_test(service_request)`
   - Data changed: Lab Test record created

3. **Collect sample**
   - Screen: Sample Collection form
   - User action: Record sample details (type, collection time)
   - Data changed: Sample Collection record

4. **Create observations**
   - User action: Select samples and create observations
   - API: `sample_collection.create_observation(selected, sample_collection, component_observations)`
   - Data changed: Observation records created (one per test item)

5. **Record results**
   - User action: Enter test values (numeric, text, or select)
   - API: `observation.record_observation_result(values)`
   - Data changed: Observation result fields populated

6. **Approve observations**
   - User action: Set status to "Approved"
   - API: `observation.set_observation_status(observation, "Approved")`
   - Data changed: Observation status updated

7. **Generate diagnostic report**
   - Automatic or manual: Groups approved observations
   - Data changed: Diagnostic Report record
   - API: `diagnostic_report.set_observation_status(docname)`

8. **Update lab test status**
   - API: `lab_test.update_status(status, name)`
   - Data changed: Lab Test status → Completed/Approved

## Error States

- Sample not collected → cannot create observations
- Observation not approved → not included in diagnostic report
- Missing reference ranges → results shown without interpretation

## Permissions

- **Laboratory User**: Full lab access (tests, samples, observations, reports)
- **Physician**: Can view results, order tests
- **Patient**: Can view results in portal

## Related Code

- Backend: `healthcare/healthcare/doctype/lab_test/lab_test.py`
- Sample: `healthcare/healthcare/doctype/sample_collection/sample_collection.py`
- Observation: `healthcare/healthcare/doctype/observation/observation.py`
- Report: `healthcare/healthcare/doctype/diagnostic_report/diagnostic_report.py`
