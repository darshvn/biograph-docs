# Data Models

Biograph contains **134 DocTypes** across 15 functional domains. All support standard Frappe CRUD via `/api/resource/{DocType}/{name}`.

**Type legend**: Master (reference data), Document (transactional), Child (child table), Single (singleton config), Tree (hierarchical), Log (audit trail)

---

## Core Patient Management (10)

| DocType | Type | Purpose |
|---------|------|---------|
| Patient | Master | Core patient record — demographics, history, linked user |
| Patient Relation | Child | Patient family/relation links (enables portal access for dependents) |
| Patient Medical Record | Log | Auto-generated timeline of all clinical events |
| Patient History Settings | Single | Configures which DocTypes appear in patient history |
| Patient History Standard Document Type | Child | Standard doc types in history |
| Patient History Custom Document Type | Child | Custom doc types in history |
| Patient Care Type | Master | Types of patient care |
| Patient Duplicate Check Rule Configuration | Single | Deduplication rules |
| Patient Duplicate Check Rule Link | Child | Rule links |
| Patient Duplicate Check Field | Child | Fields used for matching |

## Appointments & Scheduling (10)

| DocType | Type | Purpose |
|---------|------|---------|
| Patient Appointment | Document | Core appointment record |
| Appointment Type | Master | Types (Walk-in, Online, Follow-up) |
| Appointment Type Service Item | Child | Billing items per type |
| Patient Appointment Therapy | Child | Therapy links |
| Practitioner Schedule | Master | Weekly time slot templates |
| Healthcare Schedule Time Slot | Child | Individual time slots |
| Practitioner Service Unit Schedule | Child | Practitioner-to-unit mapping |
| Practitioner Availability | Document | Date-specific availability overrides |
| Fee Validity | Document | Free follow-up windows |
| Fee Validity Reference | Child | Appointment references |

## Clinical / Encounters (11)

| DocType | Type | Purpose |
|---------|------|---------|
| Patient Encounter | Document | Clinical consultation (core clinical document) |
| Patient Encounter Diagnosis | Child | Diagnoses recorded |
| Patient Encounter Symptom | Child | Symptoms/complaints |
| Drug Prescription | Child | Medications prescribed |
| Lab Prescription | Child | Lab tests ordered |
| Procedure Prescription | Child | Procedures ordered |
| Clinical Note | Child | SOAP-style clinical notes |
| Clinical Note Type | Master | Note categories |
| Doctor Advice Template | Master | Reusable advice text |
| Discharge Summary | Document | Discharge documentation |
| Vital Signs | Document | Vital sign recordings |

## Laboratory (18)

| DocType | Type | Purpose |
|---------|------|---------|
| Lab Test | Document | Test order and results |
| Lab Test Template | Master | Test definitions |
| Lab Test Group Template | Child | Grouped test items |
| Lab Test Sample | Master | Sample type definitions |
| Lab Test UOM | Master | Units of measurement |
| Normal Test Result | Child | Numeric results |
| Normal Test Template | Child | Numeric test definitions |
| Descriptive Test Result | Child | Text-based results |
| Descriptive Test Template | Child | Text test definitions |
| Sample Collection | Document | Sample tracking |
| Sample Type | Master | Biological sample types |
| Specimen | Master | Specimen definitions |
| Sensitivity | Master | Antibiotic sensitivity vocab |
| Sensitivity Test Result | Child | Sensitivity results |
| Organism | Master | Organism vocab |
| Organism Test Item | Child | Organism entries |
| Organism Test Result | Child | Organism results |
| Antibiotic | Master | Antibiotic vocab |

## Observations & Diagnostics (6)

| DocType | Type | Purpose |
|---------|------|---------|
| Observation | Document | Individual measurement/result (FHIR-aligned) |
| Observation Template | Master | Observation definitions with reference ranges |
| Observation Component | Child | Sub-observations in panels |
| Observation Reference Range | Child | Normal ranges by age/gender |
| Observation Sample Collection | Child | Sample links |
| Diagnostic Report | Document | Groups observations into a report |

## Clinical Procedures (3)

| DocType | Type | Purpose |
|---------|------|---------|
| Clinical Procedure | Document | Procedure execution record |
| Clinical Procedure Template | Master | Procedure definitions |
| Clinical Procedure Item | Child | Consumable items |

## Medications & Prescriptions (16)

| DocType | Type | Purpose |
|---------|------|---------|
| Medication | Master (Tree) | Drug definitions (hierarchical) |
| Medication Class | Master | Drug classification |
| Medication Class Interaction | Child | Class-level interactions |
| Medication Ingredient | Child | Active ingredients |
| Medication Linked Item | Child | Links to ERPNext stock items |
| Medication Request | Document | Prescription order (FHIR) |
| Medication History Item | Child | Patient medication history |
| Mediciation Override Reason Code | Master | Override reasons |
| Drug Interaction | Master | Drug-drug interactions |
| Drug Prescription | Child | Prescription entries |
| Lab Prescription | Child | Lab order entries |
| Procedure Prescription | Child | Procedure order entries |
| Dosage Form | Master | Forms (tablet, syrup, injection) |
| Dosage Strength | Child | Strength options |
| Prescription Dosage | Master | Frequency patterns (1-0-1) |
| Prescription Duration | Master | Duration options (5 days, 2 weeks) |

## Inpatient Management (11)

| DocType | Type | Purpose |
|---------|------|---------|
| Inpatient Record | Document | Admission-to-discharge record |
| Inpatient Record Item | Child | Billable items |
| Inpatient Occupancy | Child | Bed occupancy tracking |
| Inpatient Medication Order | Document | IP medication orders |
| Inpatient Medication Order Entry | Child | Order entries |
| Inpatient Medication Entry | Document | Medication administration log |
| Inpatient Medication Entry Detail | Child | Administration details |
| Discharge Summary | Document | Discharge documentation |
| Nursing Task | Document | Nursing task tracking |
| Nursing Checklist Template | Master | Checklist templates |
| Nursing Checklist Template Task | Child | Checklist items |

## Therapy & Rehabilitation (16)

| DocType | Type | Purpose |
|---------|------|---------|
| Therapy Type | Master | Therapy definitions |
| Therapy Plan | Document | Patient treatment plans |
| Therapy Plan Detail | Child | Plan items |
| Therapy Plan Template | Master | Reusable plan definitions |
| Therapy Plan Template Detail | Child | Template items |
| Therapy Session | Document | Individual session records |
| Exercise Type | Master | Exercise definitions |
| Exercise Type Step | Child | Exercise steps |
| Exercise | Child | Exercise entries in therapy types |
| Exercise Difficulty Level | Master | Difficulty levels |
| Patient Assessment | Document | Structured assessments |
| Patient Assessment Template | Master | Assessment form definitions |
| Patient Assessment Sheet | Child | Assessment sheets |
| Patient Assessment Detail | Child | Assessment items |
| Patient Assessment Parameter | Master | Assessment parameters |
| Body Part / Body Part Link | Master/Child | Anatomical reference |

## Insurance (8)

| DocType | Type | Purpose |
|---------|------|---------|
| Insurance Payor | Master | Insurance company records |
| Insurance Payor Contract | Child | Contract terms |
| Insurance Payor Eligibility Plan | Child | Eligibility plans |
| Patient Insurance Policy | Document | Patient's insurance details |
| Patient Insurance Coverage | Document | Coverage determination |
| Item Insurance Eligibility | Child | Per-item eligibility |
| Insurance Claim | Document | Claim records |
| Insurance Claim Coverage | Child | Claim coverage details |

## Treatment & Packages (7)

| DocType | Type | Purpose |
|---------|------|---------|
| Treatment Counselling | Document | Pre-admission counselling |
| Treatment Plan Template | Master | Treatment plan definitions |
| Treatment Plan Template Item | Child | Template items |
| Treatment Plan Template Practitioner | Child | Assigned practitioners |
| Healthcare Package | Master | Bundled service packages |
| Healthcare Package Item | Child | Package items |
| Package Subscription | Document | Patient subscriptions |

## Service Requests (4)

| DocType | Type | Purpose |
|---------|------|---------|
| Service Request | Document | Generic service order (FHIR) |
| Service Request Category | Master | Request categories |
| Service Request Reason | Child | Order reasons |
| Doctor Advice Template | Master | Advice text templates |

## Coding & Classification (5)

| DocType | Type | Purpose |
|---------|------|---------|
| Code System | Master | Coding systems (ICD-10, CPT, SNOMED, LOINC) |
| Code Value | Master | Individual codes |
| Code Value Set | Master | Grouped code sets |
| Codification Table | Child | Code links on templates |
| Medical Code | Master | Searchable codes |

## Infrastructure & Settings (10)

| DocType | Type | Purpose |
|---------|------|---------|
| Healthcare Settings | Single | Global configuration |
| Healthcare Service Unit | Tree | Hospital hierarchy (building > floor > ward > bed) |
| Healthcare Service Unit Type | Master | Unit type definitions |
| Service Unit Type Item | Child | Billing items per type |
| Medical Department | Master | Clinical departments |
| Healthcare Practitioner | Master | Doctor/nurse/therapist records |
| Practitioner Service Unit Schedule | Child | Practitioner-unit mapping |
| Healthcare Activity | Log | Activity log |
| Healthcare Payment Record | Document | Portal payment tracking |
| Surgery History Item | Child | Surgical history entries |

## ABDM — India Regional (2)

| DocType | Type | Purpose |
|---------|------|---------|
| ABDM Settings | Single | ABDM integration config |
| ABDM Request | Document | API request logs |

---

## Entity Relationships

```
Patient ──────┬──→ Patient Appointment ──→ Patient Encounter
              │         │                       │
              │         ↓                       ├──→ Service Request ──→ Lab Test
              │    Fee Validity                 │                   ──→ Clinical Procedure
              │                                 │                   ──→ Observation
              │                                 ├──→ Medication Request
              │                                 └──→ Diagnosis / Symptoms
              │
              ├──→ Inpatient Record
              │         ├──→ IP Medication Order ──→ IP Medication Entry ──→ Stock Entry
              │         ├──→ Inpatient Occupancy (bed tracking)
              │         └──→ Discharge Summary
              │
              ├──→ Therapy Plan ──→ Therapy Session
              │
              ├──→ Insurance Policy ──→ Coverage ──→ Insurance Claim
              │
              ├──→ Sample Collection ──→ Observation ──→ Diagnostic Report
              │
              └──→ Patient Medical Record (auto-generated from all submitted docs)

Healthcare Practitioner ──→ Practitioner Schedule ──→ Time Slots
         │
         └──→ Healthcare Service Unit (tree: building > floor > ward > bed)

All billable services ──→ Sales Invoice (ERPNext) ──→ Payment Entry (ERPNext)
```
