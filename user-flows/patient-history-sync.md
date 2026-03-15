# Patient History Sync in Patient Encounter

## User Story

> As a **physician**, I want the patient's medical history (allergies, surgeries, medications, comorbidities) to automatically appear in the encounter form so that I have full clinical context during consultation.

> As a **physician**, I want changes I make to the patient's history during an encounter to sync back to the patient record so that the master record stays up to date.

## Entry Point

- **Automatic**: Triggered when creating a new Patient Encounter (loads history from Patient)
- **Automatic**: Triggered when submitting a Patient Encounter (syncs history back to Patient)
- **Manual**: "Load Patient History" button in Patient Encounter form

## How It Works

Patient history sync is **bidirectional**:

1. **Load** (Patient → Encounter): When a new encounter is created, the system copies the patient's current medical history into the encounter form. This gives the physician a working copy to review and modify.

2. **Save back** (Encounter → Patient): When the encounter is submitted, any changes the physician made to the history fields are written back to the Patient master record.

This ensures the Patient record always reflects the latest clinical information from the most recent encounter.

## Steps — Loading History into Encounter

1. **Create new Patient Encounter**
   - User action: Select a Patient on a new Patient Encounter form
   - Trigger: `load_history_from_patient()` (patient_encounter.py line 341–424)

2. **System copies history fields**
   - Copies these fields from Patient → Encounter:

   | Patient Field | Encounter Field | Type |
   |--------------|----------------|------|
   | `allergies` | `allergies` | Small Text |
   | `surgical_history` | `surgical_history` | Small Text |
   | `medication` | `medication` | Small Text |
   | `patient_madical_history` | `patient_madical_history` | Table (diagnosis links) |
   | `family_medical_history` | `family_medical_history` | Table (diagnosis links) |

   - Clears existing child table rows before loading to avoid duplicates (line 369–372)
   - Copies diagnosis entries from patient's comorbidities (line 375–380)
   - Calls `_deduplicate_child_tables()` to remove any remaining duplicates (line 420)
   - Wrapped in try/except — errors are logged but don't block encounter creation (line 422–424)

3. **Physician reviews and modifies**
   - The history fields are now editable in the encounter form
   - Physician can add new allergies, update medication, add comorbidities, etc.

## Steps — Syncing History Back to Patient

1. **Submit the Patient Encounter**
   - Trigger: `update_patient_history()` (patient_encounter.py line 426–518)

2. **System writes back to Patient**
   - Updates these fields from Encounter → Patient:

   | Encounter Field | Patient Field | Sync Method |
   |----------------|--------------|-------------|
   | `allergies` | `allergies` | Direct overwrite |
   | `surgical_history` | `surgical_history` | Direct overwrite |
   | `medication` | `medication` | Direct overwrite |
   | `patient_madical_history` | `patient_madical_history` | Clear + repopulate |
   | `family_medical_history` | `family_medical_history` | Clear + repopulate |

   - For child tables (comorbidities, family history): clears all existing rows, then copies from encounter (line 449–458, 500–509)
   - Saves Patient with `ignore_permissions=True` (line 512)
   - Shows success message (line 514)

3. **Medical Record created**
   - After encounter submission, `create_medical_record()` hook fires
   - Creates a **Patient Medical Record** entry linking to this encounter
   - Used by the Patient History page for timeline display

## Patient History Settings

The **Patient History Settings** DocType controls which documents appear in the Patient History page.

### Configuration Fields
- `standard_doctypes`: Pre-configured healthcare DocTypes (Patient Encounter, Lab Test, Clinical Procedure, etc.)
- `custom_doctypes`: User-added DocTypes to include in history
- Each entry specifies:
  - `document_type`: The DocType to track
  - `date_fieldname`: Which date field to use for ordering (must be Date or Datetime type)
  - `selected_fields`: JSON array of field names to display in the history summary

### Medical Record Creation
- Hook: `create_medical_record(doc)` (patient_history_settings.py line 75–100)
- Fires as `after_insert` for healthcare documents
- Creates Patient Medical Record with: patient, subject (rendered from selected_fields), communication_date, reference_doctype, reference_name

### Patient History Page API
- `get_feed(name, document_types, date_range, start, page_length)` (patient_history.py line 12–25)
  - Returns Patient Medical Records for the patient
  - Optional filters: document types array, date range
  - Paginated (default: 20 per page)
  - Sorted by `communication_date DESC`

## Fields Synced

| Field Name | Type | Description | Note |
|-----------|------|-------------|------|
| `allergies` | Small Text | Free-text allergy list | Direct text overwrite |
| `surgical_history` | Small Text | Free-text surgery history | Direct text overwrite |
| `medication` | Small Text | Free-text current medications | Direct text overwrite |
| `patient_madical_history` | Table | Codified comorbidities (diagnosis links) | Spelled with typo in field name |
| `family_medical_history` | Table | Codified family conditions (diagnosis links) | Clear + repopulate on sync |

> **Note**: The field name `patient_madical_history` contains a typo ("madical" instead of "medical"). This is in the original codebase and is used consistently across Patient and Patient Encounter DocTypes.

## Error States

- Patient not set on encounter → history load skipped
- Load fails → error logged, encounter creation continues (non-blocking)
- Encounter amend → history loaded from patient again (fresh copy)
- Multiple encounters submitted same day → last submitted encounter's history overwrites Patient record

## Permissions

- **Physician**: Can view and edit history fields in encounters
- **Healthcare Administrator**: Can configure Patient History Settings
- **Patient (Portal)**: Can view their own history through Patient History page

## Related Code

- Backend: `healthcare/healthcare/doctype/patient_encounter/patient_encounter.py`
  - `load_history_from_patient()` — lines 341–424
  - `update_patient_history()` — lines 426–518
  - `load_patient_history()` — lines 520–540 (manual reload whitelist method)
- History Settings: `healthcare/healthcare/doctype/patient_history_settings/patient_history_settings.py`
  - `create_medical_record()` — lines 75–100
  - `update_medical_record()` — lines 103–117
- History Page: `healthcare/healthcare/page/patient_history/patient_history.py`
  - `get_feed()` — lines 12–25
  - `get_patient_history_doctypes()` — lines 60–70
