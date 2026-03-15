# Admin Master Setup

System administrators configure these master records before the hospital goes live. They define the operational rules — practitioner schedules, appointment types, service unit capacity, medical coding systems, and global settings.

## Masters Overview

| Master | Purpose | Key Config |
|--------|---------|------------|
| [Practitioner Setup](admin-masters/practitioner-setup.md) | Define practitioners and their schedules | Time slots, service unit assignment, video conferencing |
| [Healthcare Settings](admin-masters/healthcare-settings.md) | Global feature toggles and defaults | Registration fees, billing items, lab config, SMS alerts |
| [Patient Duplicate Checker](admin-masters/duplicate-checker-setup.md) | Configure duplicate detection rules | 24 pre-built rules, priority ordering, Warn vs Disallow |
| [Healthcare Service Unit](admin-masters/service-unit-setup.md) | Define hospital rooms, wards, clinics | Tree structure, capacity, overlap rules |
| [Appointment Type](admin-masters/appointment-type-setup.md) | Configure appointment categories | Duration, color, OP/IP pricing |
| [Medical Code Masters](admin-masters/medical-codes-masters.md) | Set up coding systems (ICD, SNOMED, LOINC) | Code System, Code Value, Codification Table |
| [Medication & Procedure Masters](admin-masters/medical-codes-masters.md#medication-master) | Define medications and procedure templates | Strength, dosage, consumables, item linking |

## Setup Order

For a new installation, configure masters in this order:

1. **Healthcare Settings** — Enable features, set default items and accounts
2. **Healthcare Service Unit** — Build the hospital's location hierarchy
3. **Practitioner Setup** — Create practitioners with schedules linked to service units
4. **Appointment Type** — Define appointment categories with pricing
5. **Medical Code Masters** — Import or configure coding systems
6. **Medication & Procedure Masters** — Set up clinical service templates
7. **Patient Duplicate Checker** — Enable and tune duplicate detection rules

## Navigation

All admin masters are accessible from:
- **Desk**: Healthcare workspace > Setup section
- **Search bar**: Type the DocType name directly
- **Healthcare Settings**: Single-page config at `/app/healthcare-settings`
