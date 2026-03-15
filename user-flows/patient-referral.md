# Patient Referral Workflow

## User Story

> As a **physician**, I want to refer a patient to another practitioner or department so that the patient receives specialized care with a proper audit trail.

> As a **referred practitioner**, I want to see incoming referrals so that I can schedule and act on them.

## Entry Point

- **Desk**: Patient Encounter form > References child table

## How It Works

Referrals in Biograph are implemented through the **Service Request** DocType. When a physician adds a referral in the Patient Encounter's References table and submits the encounter, the system creates a Service Request with `template_dt = "Appointment Type"` and links it to the referred practitioner. The Service Request then follows a standard lifecycle from draft through active to completed or revoked.

## Steps

1. **Create a Patient Encounter**
   - Screen: Patient Encounter form
   - User action: Complete the clinical consultation (diagnosis, prescriptions, etc.)

2. **Add referral in References table**
   - User action: Add a row to the "References" child table with:
     - `appointment_type`: Type of appointment to refer for (e.g., "Consultation", "Follow Up")
     - `refer_to`: The practitioner to refer to (Link to Healthcare Practitioner)
     - `referral_note`: Free text describing the reason for referral

3. **Submit the encounter**
   - User action: Submit the Patient Encounter
   - Trigger: `create_patient_referral(encounter, references)` (patient_encounter.py line 817–843)

4. **System creates Service Request(s)**
   - For each reference row in the encounter:
     - Creates a Service Request document with:
       - `order_date` / `order_time`: from encounter date/time
       - `company`: from encounter
       - `status`: "draft-Request Status"
       - `source_doc`: "Patient Encounter"
       - `order_group`: encounter name (links back to source)
       - `patient`: from encounter
       - `practitioner`: from encounter (the referring physician)
       - `template_dt`: "Appointment Type"
       - `template_dn`: ref.appointment_type OR defaults to "Consultation"
       - `quantity`: 1
       - `order_description`: ref.referral_note
       - `referred_to_practitioner`: ref.refer_to
     - Inserts with `ignore_permissions=True`
     - Submits immediately → status transitions to "active-Request Status"

5. **Referred practitioner acts on the referral**
   - The Service Request appears in the referred practitioner's queue
   - Actions available:
     - Create a new appointment for the patient
     - Create a lab test, clinical procedure, or therapy session
     - Put the request on hold
     - Revoke the request (if cancelled)

6. **Service Request completion**
   - When the related document (appointment, lab test, etc.) is created and linked
   - Service Request status updates to "completed-Request Status"
   - Billing status tracks separately: Pending → Partly Invoiced → Invoiced

## Service Request Status Lifecycle

```
┌──────────┐     submit     ┌──────────┐
│  Draft    │ ─────────────→│  Active   │
│  Request  │               │  Request  │
│  Status   │               │  Status   │
└──────────┘               └────┬──┬───┘
                                │  │
                    ┌───────────┘  └──────────┐
                    ▼                          ▼
             ┌──────────┐              ┌──────────┐
             │ On Hold  │              │ Completed │
             │ Request  │              │ Request   │
             │ Status   │              │ Status    │
             └──────────┘              └──────────┘
                    │
                    ▼  (if cancelled)
             ┌──────────┐
             │ Revoked  │
             │ Request  │
             │ Status   │
             └──────────┘
```

## Key Service Request Fields

| Field | Type | Description |
|-------|------|-------------|
| `template_dt` | Link → DocType | "Appointment Type" for referrals |
| `template_dn` | Dynamic Link | The specific appointment type |
| `referred_to_practitioner` | Link → Healthcare Practitioner | Target of the referral |
| `practitioner` | Link → Healthcare Practitioner | The referring physician |
| `order_group` | Link → Patient Encounter | Source encounter |
| `order_description` | Text | Referral reason/notes |
| `billing_status` | Select | Pending / Partly Invoiced / Invoiced |
| `qty_invoiced` | Int | Quantity already billed |

## Error States

- No appointment type specified → defaults to "Consultation"
- Referred practitioner not set → Service Request created without `referred_to_practitioner` (still valid)
- Encounter cancelled → Service Request stays active (must be manually revoked)

## Permissions

- **Physician**: Can create referrals within Patient Encounters
- **Healthcare Administrator**: Can view and manage all Service Requests
- **Referred Practitioner**: Can see Service Requests assigned to them

## Related Code

- Backend: `healthcare/healthcare/doctype/patient_encounter/patient_encounter.py`
  - `create_patient_referral()` — lines 817–843
- Service Request: `healthcare/healthcare/doctype/service_request/service_request.py`
- Controller: `healthcare/healthcare/controllers/service_request_controller.py`
  - `before_submit()` — lines 17–31
  - `on_cancel()` — lines 41–47
