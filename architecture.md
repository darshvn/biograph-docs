# Architecture

## System Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                        BIOGRAPH HIS                              │
│                                                                  │
│  ┌─────────────────┐   ┌─────────────────┐   ┌──────────────┐  │
│  │  Frappe Desk UI  │   │  Patient Portal  │   │  REST API    │  │
│  │  (jQuery/Frappe)  │   │  (Vue 3 SPA)     │   │  (Clients)   │  │
│  └────────┬─────────┘   └────────┬─────────┘   └──────┬───────┘  │
│           │                      │                      │         │
│           ▼                      ▼                      ▼         │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │              Frappe RPC Layer / REST API                    │  │
│  │  frappe.call() / POST /api/method/ / /api/resource/        │  │
│  └────────────────────────────┬───────────────────────────────┘  │
│                               │                                   │
│  ┌────────────────────────────▼───────────────────────────────┐  │
│  │                  Biograph Backend (Python)                  │  │
│  │                                                             │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐  │  │
│  │  │ Patient  │ │ Clinical │ │   Lab    │ │  Inpatient   │  │  │
│  │  │ Mgmt     │ │ (OPD)    │ │          │ │  (IPD)       │  │  │
│  │  ├──────────┤ ├──────────┤ ├──────────┤ ├──────────────┤  │  │
│  │  │ Therapy  │ │Insurance │ │ Billing  │ │  ABDM (India)│  │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────────┘  │  │
│  └────────────────────────────┬───────────────────────────────┘  │
│                               │                                   │
│  ┌────────────────────────────▼───────────────────────────────┐  │
│  │                   ERPNext Integration                       │  │
│  │  Sales Invoice │ Payment Entry │ Stock Entry │ HR / Assets  │  │
│  └────────────────────────────────────────────────────────────┘  │
│                               │                                   │
│  ┌────────────────────────────▼───────────────────────────────┐  │
│  │          Frappe Framework (ORM, Auth, Queue, Files)         │  │
│  └────────────────┬────────────────────┬──────────────────────┘  │
│                   │                    │                          │
│              ┌────▼────┐          ┌────▼────┐                    │
│              │ MariaDB │          │  Redis  │                    │
│              └─────────┘          └─────────┘                    │
└──────────────────────────────────────────────────────────────────┘
```

## Architecture Type

**Modular monolith** — single Frappe app with domain-organized DocTypes + a standalone Vue 3 SPA for the patient portal. Not microservices.

## Components

### Backend (Python / Frappe)

| Property | Value |
|----------|-------|
| **Tech stack** | Python 3.10+, Frappe Framework, python-barcode |
| **Location** | `healthcare/` |
| **Entry point** | `healthcare/hooks.py` |
| **Purpose** | All clinical, administrative, and billing logic |
| **Scale** | 134 DocTypes, 120+ whitelisted methods, 1600-line utils module |

Organized into domains: Patient Management, Appointments & Scheduling, Patient Encounters, Laboratory, Observations & Diagnostics, Clinical Procedures, Medications, Inpatient, Therapy & Rehabilitation, Insurance, Billing, Medical Coding, ABDM (India).

### Patient Portal (Vue 3 SPA)

| Property | Value |
|----------|-------|
| **Tech stack** | Vue 3.2, Frappe UI, Tailwind CSS 3.4, Vite 4.4, Feather Icons |
| **Location** | `patient_portal/` |
| **Entry point** | `patient_portal/src/patient_portal.js` |
| **Purpose** | Patient self-service: appointments, diagnostics, payments |
| **Scale** | 7 components, 12 API calls, tab-based navigation |
| **Served at** | `/patient-portal` (Frappe static files) |

### ERPNext Integration Layer

| Property | Value |
|----------|-------|
| **Location** | `healthcare/healthcare/custom_doctype/`, document event hooks |
| **Purpose** | Bridge healthcare services to ERP billing, stock, and payments |

Key integration points:
- `HealthcareSalesInvoice` class overrides Sales Invoice for healthcare billing
- Payment Entry hooks update paid amounts on healthcare documents
- Stock Entry creation for clinical procedure / therapy consumables
- Company hooks create Healthcare Service Unit tree roots

## Component Communication

| From | To | Method | Use Case |
|------|----|--------|----------|
| Desk UI | Backend | `frappe.call()` (RPC) | All desk operations |
| Patient Portal | Backend | `POST /api/method/healthcare.healthcare.api.patient_portal.*` | Portal features |
| External clients | Backend | REST API with token auth | Integrations |
| Backend | ERPNext | Python imports + hooks | Billing, stock, payments |
| Portal | Backend | Socket.IO | Realtime updates |

## Shared Infrastructure

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Database** | MariaDB | All DocType data (via Frappe ORM) |
| **Cache** | Redis | Session cache, background job queue |
| **File Storage** | Frappe file system | Attachments, images, print formats |
| **Background Jobs** | Frappe Scheduler (Redis Queue) | Appointment reminders, daily tasks |
| **Email** | Frappe email queue | Notifications, reminders |

## Authentication & Authorization

### Auth Methods

| Method | Used By | Mechanism |
|--------|---------|-----------|
| **Session (cookie)** | Desk UI, Patient Portal | `frappe.auth` login → session cookie |
| **Token** | API clients | `Authorization: token api_key:api_secret` header |
| **Guest** | 1 endpoint only | `get_context_for_dev()` — dev mode only |

### Roles

| Role | Access Scope |
|------|-------------|
| Healthcare Administrator | Full access to all healthcare DocTypes and settings |
| Physician | Encounters, prescriptions, service requests, referrals |
| Laboratory User | Lab tests, sample collection, observations, diagnostic reports |
| Nursing User | Nursing tasks, inpatient medication entries |
| Patient | Portal only — filtered to own records + relations |

### Portal Security

- All patient portal API methods require authentication
- Data filtered by `Patient.user_id == frappe.session.user`
- Patient Relations (dependents) are also accessible
- Web form permissions use custom `has_website_permission` handlers
- Print format access restricted to 3 DocTypes only

## Data Flow

### Appointment → Invoice

```
Patient books appointment (Portal or Desk)
    │
    ▼
Patient Appointment (status: Open)
    ├── Fee Validity checked (free follow-up?)
    │
    ▼
Practitioner creates Patient Encounter
    ├── Symptoms + Diagnoses recorded
    ├── Drug Prescriptions → Medication Request
    ├── Lab Prescriptions → Service Request → Lab Test
    ├── Procedure Orders → Service Request → Clinical Procedure
    └── Therapy Orders → Service Request → Therapy Session
            │
            ▼
    Lab: Sample Collection → Observation → Diagnostic Report
            │
            ▼
    get_healthcare_services_to_invoice() aggregates all billable items
            │
            ▼
    Sales Invoice (ERPNext) → Payment Entry (ERPNext)
    hooks update paid_amount on healthcare docs
```

### Inpatient Flow

```
Treatment Counselling (pre-admission consent)
    │
    ▼
Inpatient Record created → admit(service_unit, check_in)
    │
    ├── Inpatient Occupancy tracks bed assignment
    ├── IP Medication Order → IP Medication Entry → Stock Entry
    ├── Daily scheduler adds service unit rent to billables
    ├── transfer(new_service_unit) if bed change needed
    │
    ▼
discharge() → Discharge Summary
    │
    ▼
All billable items → Sales Invoice
```
