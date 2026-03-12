# API Reference

All whitelisted methods are callable via Frappe's RPC: `POST /api/method/{dotted.path}`. Auth required unless noted.

## Calling Conventions

```bash
# Module-level function
POST /api/method/healthcare.healthcare.doctype.patient_appointment.patient_appointment.update_status

# Instance method (on a document)
POST /api/method/run_doc_method
Body: { "dt": "Inpatient Record", "dn": "IP-00001", "method": "admit", "args": {...} }

# Standard CRUD (all DocTypes)
GET    /api/resource/{DocType}              # List
GET    /api/resource/{DocType}/{name}       # Read
POST   /api/resource/{DocType}              # Create
PUT    /api/resource/{DocType}/{name}       # Update
DELETE /api/resource/{DocType}/{name}       # Delete

# Auth header
Authorization: token api_key:api_secret
```

---

## Patient Portal API

Module: `healthcare.healthcare.api.patient_portal`

| Method | Params | Description |
|--------|--------|-------------|
| `get_logged_in_patient` | — | Resolve session user to Patient record |
| `get_appointments` | — | List appointments for patient + relations |
| `get_departments` | — | List portal-visible departments |
| `get_practitioners` | department | List practitioners by department |
| `get_patients` | — | List accessible patients (self + relations) |
| `get_settings` | — | Return Healthcare Settings singleton |
| `get_slots` | practitioner, date | Available time slots |
| `make_appointment` | practitioner, patient, date, slot | Book appointment |
| `get_fees` | practitioner, date | Consultation fee details |
| `get_print_format` | doctype, name | Print format + letter head info |
| `get_orders` | — | Diagnostic test orders + results |
| `get_payment_link` | doctype, docname, title, amount, total_amount, currency, patient, redirect_to | Payment gateway URL |

## Patient Appointment

Module: `healthcare.healthcare.doctype.patient_appointment.patient_appointment`

| Method | Params | Description |
|--------|--------|-------------|
| `get_availability_data` | date, practitioner, appointment_type, ... | Slot availability |
| `check_payment_reqd` | patient | Check if payment required |
| `invoice_appointment` | appointment_name, discount_%, discount_amt | Invoice the appointment |
| `update_fee_validity` | appointment | Update fee validity after booking |
| `update_status` | appointment_id, status | Change appointment status |
| `make_encounter` | source_name | Create Patient Encounter |
| `get_events` | start, end, filters | Calendar events |
| `get_procedure_prescribed` | patient | List prescribed procedures |
| `create_therapy_sessions` | appointment, therapy_types | Create linked therapy sessions |
| `check_unavailability_conflicts` | filters | Check for scheduling conflicts |
| `create_unavailability_appointment` | data | Block practitioner time |

### Recurring Handler

Module: `healthcare.healthcare.doctype.patient_appointment.recuring_appointment_handler`

| Method | Params | Description |
|--------|--------|-------------|
| `create_recurring_appointments` | data | Create recurring appointment series |
| `book_appointments` | data | Batch book appointments |
| `get_recurring_appointment_dates` | data | Preview recurring dates |
| `get_availability` | scheduled_details, practitioner, service_unit | Check availability |
| `get_service_unit_values` | selected_practitioner | Practitioner's service units |

## Patient Encounter

Module: `healthcare.healthcare.doctype.patient_encounter.patient_encounter`

| Method | Params | Description |
|--------|--------|-------------|
| `get_applicable_treatment_plans` | encounter | Matching treatment plan templates |
| `set_treatment_plans` | *(instance)* treatment_plans | Apply treatment plan |
| `add_clinical_note` | *(instance)* note, note_type | Add SOAP note |
| `edit_clinical_note` | *(instance)* note, note_name | Edit note |
| `delete_clinical_note` | *(instance)* note_name | Delete note |
| `get_clinical_notes` | *(instance)* patient | Get all notes |
| `load_patient_history` | *(instance)* | Load medical history |
| `make_ip_medication_order` | source_name | Create IP medication order |
| `create_service_request` | encounter | Create service requests from prescriptions |
| `create_medication_request` | encounter | Create medication requests |
| `get_medications` | medication | Medication details |
| `cancel_request` | doctype, request | Cancel a request |
| `create_service_request_from_widget` | encounter, data | Sidebar widget ordering |
| `get_encounter_details` | doc | Formatted encounter details |
| `create_patient_referral` | encounter, references | Create referral |

## Inpatient Record

Module: `healthcare.healthcare.doctype.inpatient_record.inpatient_record`

| Method | Params | Description |
|--------|--------|-------------|
| `admit` | *(instance)* service_unit, check_in, ... | Admit patient |
| `discharge` | *(instance)* | Discharge patient |
| `transfer` | *(instance)* service_unit, check_in, ... | Transfer bed/ward |
| `add_service_unit_rent_to_billable_items` | *(instance)* | Add daily rent charges |
| `create_insurance_coverage` | *(instance)* | Create IP insurance coverage |
| `schedule_inpatient` | admission_order | Schedule admission |
| `schedule_discharge` | discharge_order | Schedule discharge |
| `set_ip_order_cancelled` | inpatient_record, reason | Cancel IP order |
| `make_discharge_summary` | source_name | Create discharge summary |
| `create_treatment_counselling` | ip_order | Create pre-admission counselling |
| `create_stock_entry` | items, inpatient_record | Create stock entry for consumables |

## Observation

Module: `healthcare.healthcare.doctype.observation.observation`

| Method | Params | Description |
|--------|--------|-------------|
| `get_observation_details` | docname | Full observation data |
| `edit_observation` | observation, data_type, result | Edit result |
| `add_observation` | **args | Create new observation |
| `record_observation_result` | values | Record result values |
| `add_note` | note, observation | Add note to observation |
| `get_observation_result_template` | template_name, observation | Get result template |
| `set_observation_status` | observation, status, reason | Set approval status |

## Therapy Plan + Session

### Therapy Plan — `healthcare.healthcare.doctype.therapy_plan.therapy_plan`

| Method | Params | Description |
|--------|--------|-------------|
| `set_therapy_details_from_template` | *(instance)* | Populate from template |
| `get_invoiced_details` | *(instance)* | Invoicing status |
| `make_therapy_session` | therapy_plan, patient, therapy_type, company | Create session |
| `make_sales_invoice` | reference_name, patient, company, items | Invoice plan |
| `make_patient_appointment` | source_name | Create appointment |

### Therapy Session — `healthcare.healthcare.doctype.therapy_session.therapy_session`

| Method | Params | Description |
|--------|--------|-------------|
| `consume_stocks` | *(instance)* | Consume consumables |
| `verify_stock` | *(instance)* | Check stock availability |
| `make_material_receipt` | *(instance)* submit | Create material receipt |
| `create_therapy_session` | source_name | Create from appointment |
| `invoice_therapy_session` | source_name | Invoice session |
| `validate_no_of_session` | therapy_plan | Check session limit |
| `get_therapy_consumables` | therapy_type | List consumables |

## Clinical Procedure

Module: `healthcare.healthcare.doctype.clinical_procedure.clinical_procedure`

| Method | Params | Description |
|--------|--------|-------------|
| `start_procedure` | *(instance)* | Start procedure |
| `complete_procedure` | *(instance)* | Complete procedure |
| `make_material_receipt` | *(instance)* submit | Material receipt |
| `get_procedure_consumables` | procedure_template | List consumables |
| `make_stock_entry` | doc | Create stock entry |
| `make_procedure` | source_name | Create from service request |

## Service Request

Module: `healthcare.healthcare.doctype.service_request.service_request`

| Method | Params | Description |
|--------|--------|-------------|
| `set_service_request_status` | service_request, status | Update status |
| `make_clinical_procedure` | service_request | Create procedure |
| `make_lab_test` | service_request | Create lab test |
| `make_therapy_session` | service_request | Create therapy session |
| `make_observation` | service_request | Create observation |
| `make_appointment` | source_name | Create appointment |

## Lab Test

Module: `healthcare.healthcare.doctype.lab_test.lab_test`

| Method | Params | Description |
|--------|--------|-------------|
| `update_status` | status, name | Update test status |
| `create_multiple` | doctype, docname | Batch create tests |
| `get_lab_test_prescribed` | patient | List prescribed tests |

## Insurance

| Module | Method | Params | Description |
|--------|--------|--------|-------------|
| `insurance_payor` | `has_active_contract` | insurance_payor, company, on_date | Check active contract |
| `patient_insurance_coverage` | `create_insurance_eligibility` | doc | Create eligibility check |
| `insurance_claim` | `get_coverages` | *(instance)* | Get applicable coverages |
| `insurance_claim` | `create_payment_entry` | doc | Create payment for claim |

## Billing Utilities

Module: `healthcare.healthcare.utils`

| Method | Params | Description |
|--------|--------|-------------|
| `get_healthcare_services_to_invoice` | patient, customer, company, link_customer | Aggregate all billable services |
| `get_appointment_billing_item_and_rate` | doc | Billing item + rate for appointment |
| `get_drugs_to_invoice` | encounter, customer, link_customer | Drugs to invoice |
| `get_patient_vitals` | patient, from_date, to_date | Vital signs records |
| `render_doc_as_html` | doctype, docname, exclude_fields | Render document as HTML |
| `get_medical_codes` | template_dt, template_dn, code_standard | Medical codes for template |
| `generate_barcodes` | in_val | Barcode image generation |
| `add_node` | — | Add Healthcare Service Unit tree node |
| `check_patient_duplicates` | patient | Duplicate patient detection |

## Desk Page APIs

### Patient History — `healthcare.healthcare.page.patient_history.patient_history`

| Method | Params | Description |
|--------|--------|-------------|
| `get_feed` | name, document_types, date_range, start, page_length | Paginated medical timeline |
| `get_feed_for_dt` | doctype, docname | Single document feed |
| `get_patient_history_doctypes` | — | Available doc types |

### Patient Progress — `healthcare.healthcare.page.patient_progress.patient_progress`

| Method | Params | Description |
|--------|--------|-------------|
| `get_therapy_sessions_count` | patient | Total/completed counts |
| `get_patient_heatmap_data` | patient, date | Activity heatmap |
| `get_therapy_sessions_distribution_data` | patient, field | Distribution by type |
| `get_therapy_progress_data` | patient, therapy_type, time_span | Progress over time |
| `get_patient_assessment_data` | patient, assessment_template, time_span | Assessment scores |
| `get_therapy_assessment_correlation_data` | patient, assessment_template, time_span | Correlation analysis |
| `get_assessment_parameter_data` | patient, parameter, time_span | Parameter trends |

## Dashboard Chart Sources

| Source | Description |
|--------|-------------|
| `department_wise_patient_appointments` | Appointments by department |
| `insurance_claim_status` | Insurance claim status breakdown |
| `service_unit_type_wise_admission_status` | Admissions by unit type |
