# Frontend Map

## Vue 3 Patient Portal

**Location**: `patient_portal/`
**Served at**: `/patient-portal`
**Tech**: Vue 3.2, Frappe UI 0.1, Tailwind CSS 3.4, Vite 4.4, Feather Icons
**Build output**: `/assets/healthcare/patient_portal/`

### Component Tree

```
PatientPortal.vue (root)
├── AppointmentModel.vue          # Lists appointments in card grid
│   └── BookAppointmentModel.vue  # Multi-step booking wizard (modal)
│       ├── DepartmentSelector    # Department grid with images
│       ├── PractitionerSelector  # Practitioner grid with photos
│       ├── Calendar              # Date picker
│       └── Payment               # Payment gateway integration
└── DiagnosticModel.vue           # Lists test orders and results
```

### API Usage by Component

| Component | API Method | Purpose |
|-----------|-----------|---------|
| Root | `get_logged_in_patient()` | Identify current patient |
| Root | `get_settings()` | Load Healthcare Settings |
| AppointmentModel | `get_appointments()` | Fetch appointment list |
| BookAppointment | `get_departments()` | Portal-enabled departments |
| BookAppointment | `get_practitioners(dept)` | Practitioners by dept |
| BookAppointment | `get_slots(prac, date)` | Available time slots |
| BookAppointment | `get_patients()` | Self + relations |
| BookAppointment | `get_fees(prac, date)` | Fee details |
| BookAppointment | `make_appointment(...)` | Create appointment |
| Payment | `get_payment_link(...)` | Initiate payment |
| DiagnosticModel | `get_orders()` | Test orders + results |
| DiagnosticModel | `get_print_format(dt, name)` | Print config |

### Navigation

Tab-based (no client-side router). Tabs: Appointments, Diagnostics.

### Realtime

Socket.IO via `patient_portal/src/socket.js` for live updates.

---

## Frappe Desk UI (Healthcare)

### Workspaces (sidebar navigation)

| Workspace | Key DocTypes / Pages |
|-----------|---------------------|
| Healthcare | Dashboard, Patient, Practitioner |
| Outpatient | Patient Appointment, Patient Encounter, Fee Validity |
| Inpatient | Inpatient Record, Nursing Task, Discharge Summary |
| Diagnostics | Lab Test, Sample Collection, Observation, Diagnostic Report |
| Insurance | Insurance Payor, Insurance Claim, Coverage |
| Rehabilitation | Therapy Plan, Therapy Session, Patient Assessment |
| Setup | Healthcare Settings, Service Units, Departments, Templates |

### Custom Desk Pages

| Page | Route | Purpose |
|------|-------|---------|
| Patient History | `/app/patient-history` | Paginated medical timeline for a patient |
| Patient Progress | `/app/patient-progress` | Therapy progress charts, heatmaps, assessments |

### JS Overrides

| ERPNext DocType | JS File | Adds |
|-----------------|---------|------|
| Sales Invoice | `public/js/sales_invoice.js` | "Get Healthcare Services to Invoice" button |
| Healthcare Practitioner | `public/js/healthcare_practitioner.js` | Extended practitioner form UI |

---

## Legacy Web Forms (Portal)

| Web Form | Route | Auth | Purpose |
|----------|-------|------|---------|
| Patient Registration | `/patient-registration` | Guest | New patient signup |
| Patient Appointments | `/patient-appointments` | Login | List appointments |
| Personal Details | `/personal-details` | Login | Edit patient profile |
| Prescription | `/prescription` | Login | View prescriptions (read-only) |
| Lab Test | `/lab-test` | Login | View lab results (read-only) |

All authenticated web forms use custom `has_website_permission` handlers that verify `Patient.user_id == session.user`.
