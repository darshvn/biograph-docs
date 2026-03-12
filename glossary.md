# Glossary

## Healthcare Terms

| Term | Definition |
|------|-----------|
| **Patient Encounter** | A clinical consultation record (doctor's visit). Contains diagnoses, prescriptions, and orders. |
| **Service Request** | An HL7 FHIR-aligned order for a lab test, clinical procedure, or therapy session. Created when an encounter is submitted. |
| **Medication Request** | An HL7 FHIR-aligned prescription order for drugs. Created from encounter drug prescriptions. |
| **Observation** | An individual clinical measurement or lab result. FHIR-aligned. Can be numeric, text, or select. |
| **Diagnostic Report** | A group of approved observations forming a complete lab/diagnostic result report. |
| **Sample Collection** | Tracks the collection of biological samples (blood, urine, etc.) for lab testing. |
| **Inpatient Record** | The admission-to-discharge record for a hospitalized patient. Tracks bed assignments, medications, and charges. |
| **Treatment Counselling** | Pre-admission counselling and consent documentation before inpatient admission. |
| **Healthcare Service Unit** | A physical location in the hospital hierarchy: building > floor > ward > bed. Represented as a tree. |
| **Fee Validity** | A window during which follow-up appointments with the same practitioner are free of charge. |
| **Medical Code** | A code from a standard system (ICD-10, CPT, SNOMED, LOINC) linked to diagnoses, procedures, or observations. |
| **Practitioner Schedule** | A weekly time slot template defining when a practitioner is available for appointments. |
| **Appointment Type** | Categories of appointments (Walk-in, Online, Follow-up) with associated billing items. |
| **Insurance Payor** | An insurance company or provider that reimburses healthcare costs. |
| **Insurance Claim** | A request for reimbursement submitted to an insurance payor for covered services. |
| **ABDM** | Ayushman Bharat Digital Mission — India's national health ID system for interoperable health records. |
| **Nursing Task** | A task assigned to nursing staff, often generated from checklist templates. |
| **Patient Assessment** | A structured evaluation using scored parameters (e.g., pain scales, functional assessments). |

## Frappe / ERPNext Terms

| Term | Definition |
|------|-----------|
| **DocType** | Frappe's data model. Defines schema (JSON), controller (Python), and client UI (JS). Each DocType maps to a database table. |
| **Whitelisted Method** | A Python function decorated with `@frappe.whitelist()`, making it callable from the frontend via RPC (`frappe.call()`). |
| **Bench** | Frappe's CLI tool for managing sites, apps, and deployments. Commands: `bench start`, `bench migrate`, `bench get-app`, etc. |
| **Site** | A Frappe instance with its own database, users, and configuration. Multiple sites can run on one bench. |
| **ERPNext** | Open-source ERP system that Biograph extends. Provides Accounts (billing), Stock (inventory), HR, and Assets modules. |
| **Frappe UI** | A Vue 3 component library by Frappe for building modern UIs. Used by the patient portal. |
| **Web Form** | Frappe's built-in portal page generator. Creates public-facing forms from DocType definitions. |
| **Workspace** | A configurable sidebar section in Frappe Desk that organizes shortcuts, charts, and links for a module. |
| **Print Format** | A template defining how a DocType renders when printed (PDF/HTML). Uses Jinja templates. |
| **Singleton / Single DocType** | A DocType that only has one record (e.g., Healthcare Settings). Accessed via `frappe.get_single()`. |
| **Child Table** | A DocType embedded as rows within another DocType (parent). Stored with `parent`, `parenttype`, `parentfield` links. |
| **Tree DocType** | A hierarchical DocType with parent-child relationships (e.g., Healthcare Service Unit). |
| **Document Event** | A lifecycle hook (validate, on_submit, on_cancel, etc.) that triggers Python code when a document changes state. |
| **Scheduler Event** | A background task that runs on a schedule (every minute, daily, weekly, etc.) via Frappe's job queue. |
| **`frappe.call()`** | The frontend JavaScript function to call a whitelisted Python method via AJAX. |
| **`run_doc_method`** | A Frappe endpoint to call instance methods on a specific document: `POST /api/method/run_doc_method`. |
