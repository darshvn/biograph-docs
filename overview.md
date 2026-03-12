# Biograph — Product Overview

## What is Biograph?

Biograph is an open-source Hospital Information System (HIS) built on [Frappe](https://frappeframework.com/) and [ERPNext](https://erpnext.com/) for managing clinical workflows in healthcare organizations.

**By**: [Tacten](https://tacten.co/digitalhealth)
**License**: GNU GPL v3
**Origin**: Fork and enhancement of Marley Health

## Who Uses It?

| User Type | Role(s) | Access Point |
|-----------|---------|-------------|
| **Healthcare Administrators** | Healthcare Administrator | Frappe Desk (full access) |
| **Physicians / Doctors** | Physician | Desk — encounters, prescriptions, referrals |
| **Lab Technicians** | Laboratory User | Desk — tests, samples, observations, reports |
| **Nurses** | Nursing User | Desk — nursing tasks, medication administration |
| **Physiotherapists** | Physician | Desk — therapy plans, sessions, exercises |
| **Receptionists** | Healthcare Administrator | Desk — appointments, registration, billing |
| **Patients** | Patient | Patient Portal (`/patient-portal`) + web forms |

## Key Capabilities

| Module | Description |
|--------|-------------|
| **Patient Management** | Registration, demographics, medical history, relations, deduplication |
| **Outpatient (OPD)** | Appointments, encounters, prescriptions, referrals, fee validity |
| **Inpatient (IPD)** | Admission, bed management, transfers, discharge, medication orders |
| **Laboratory** | Test templates, sample collection, observations, diagnostic reports |
| **Clinical Procedures** | Procedure templates, scheduling, stock consumption tracking |
| **Rehabilitation** | Therapy types, plans, sessions, exercises, patient assessments |
| **Insurance** | Payor management, contracts, coverage, claims, payment integration |
| **Billing** | Service aggregation, invoicing, fee validity, package subscriptions |
| **Patient Portal** | Self-service appointments, lab results, online payments (Vue 3 SPA) |
| **Medical Coding** | ICD-10, CPT, SNOMED, LOINC code systems |
| **ABDM (India)** | Ayushman Bharat Digital Mission health ID integration |

## Tech Stack

| Component | Technology |
|-----------|-----------|
| **Backend** | Python 3.10+, Frappe Framework |
| **Frontend (Desk)** | Frappe JS Client (jQuery-based) |
| **Patient Portal** | Vue 3.2, Frappe UI, Tailwind CSS 3.4, Vite 4.4 |
| **Database** | MariaDB (via Frappe) |
| **Cache/Queue** | Redis (via Frappe) |
| **ERP Integration** | ERPNext (Accounts, Stock, HR, Assets) |
| **Standards** | HL7 FHIR (data model alignment) |
| **CI/CD** | GitHub Actions (9 workflows) |
| **Linting** | Ruff, Semgrep, Pre-commit |

## Scale

- **134 DocTypes** across 15 functional domains
- **120+ whitelisted API methods**
- **7 desk workspaces** (Healthcare, Outpatient, Inpatient, Diagnostics, Insurance, Rehabilitation, Setup)
- **6 custom reports** (Diagnosis Trends, Lab Test Report, Appointment Analytics, etc.)
- **5 web forms** for legacy patient portal
- **4 scheduled tasks** (appointment reminders, status updates, fee expiry, daily billing)

## Links

- **Repository**: [github.com/Tacten/biograph](https://github.com/Tacten/biograph)
- **Marketing**: [tacten.co/digitalhealth](https://tacten.co/digitalhealth)
- **Documentation**: [deepwiki.com/Tacten/biograph](https://deepwiki.com/Tacten/biograph)
- **Try it**: [Frappe Cloud](https://frappecloud.com/biograph/signup)
- **Community**: [Telegram Group](https://t.me/tactenbiograph)
