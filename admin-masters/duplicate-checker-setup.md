# Patient Duplicate Checker Setup

## Overview

The duplicate checker automatically detects potential duplicate patients when a new patient is registered. It uses configurable rules that match field combinations and take actions (Disallow, Warn, or Allow).

For how the checker runs at runtime, see [Patient Duplicate Checking](user-flows/patient-duplicate-checking.md).

## Enabling the Feature

Navigate to: Healthcare Settings > Patient Duplicate Check tab

1. Check **Enable Patient Duplicate Check** (`enable_patient_duplicate_check`)
2. The `patient_duplicate_check_rules` table below will show linked rules

On a fresh install, `setup_patient_duplicate_check_rules()` automatically creates 24 rules and links them to Healthcare Settings.

## Rule Configuration

### Viewing Rules

Each row in the Healthcare Settings rules table links to a **Patient Duplicate Check Rule Configuration** document showing:
- `rule_name_display` (read-only, fetched from the rule)
- `action_display` (read-only)
- `priority_display` (read-only)

### Creating a Custom Rule

Navigate to: Patient Duplicate Check Rule Configuration > New

| Field | Type | Description |
|-------|------|-------------|
| `rule_name` | Data | Unique descriptive name |
| `action` | Select | **Disallow** (block save), **Warn** (show warning, allow save), **Allow** (no action) |
| `priority` | Int | Lower number = checked first. Default: 10 |
| `message` | Small Text | Custom message shown to user when duplicates are found |
| `duplicate_fields` | Table | Fields to match (see below) |

### Available Match Fields

The `duplicate_fields` child table (Patient Duplicate Check Field) supports:

| Field Name | Maps to Patient Field | Description |
|-----------|----------------------|-------------|
| `first_name` | Patient.first_name | Patient's first name |
| `last_name` | Patient.last_name | Patient's last name |
| `sex` | Patient.sex | Gender |
| `dob` | Patient.dob | Date of birth |
| `mobile` | Patient.mobile | Mobile phone number |
| `email` | Patient.email | Email address |

Each field in the child table has:
- `field_name`: Select (required)
- `field_label`: Data (auto-populated, read-only)
- `is_required`: Check

### How Matching Works

- **Exact match only** — no fuzzy, phonetic, or partial matching
- **All fields must have values** — if a rule checks `first_name + mobile` but the patient hasn't entered `mobile`, the rule is skipped
- Rules are evaluated in **priority order** (ascending) — first non-"Allow" result wins

## Pre-Configured Rules (24 Total)

### Disallow Rules (Priority 1–6)

These block patient creation when high-confidence duplicates are found:

| Priority | Rule | Fields Matched |
|----------|------|---------------|
| 1 | All Fields Exact Match | first_name + last_name + sex + dob + mobile |
| 2 | Name Gender DOB Match | first_name + last_name + sex + dob |
| 3 | Name Gender Mobile Match | first_name + last_name + sex + mobile |
| 4 | Name DOB Mobile Match | first_name + last_name + dob + mobile |
| 5 | First Name Gender DOB Mobile | first_name + sex + dob + mobile |
| 6 | Last Name Gender DOB Mobile | last_name + sex + dob + mobile |

### Warn Rules (Priority 7–23)

These show a warning but allow the user to proceed:

**3-field combinations (Priority 7–14):**
first_name+last_name+sex, first_name+last_name+dob, first_name+last_name+mobile, first_name+sex+dob, first_name+sex+mobile, first_name+dob+mobile, last_name+sex+dob, last_name+sex+mobile

**2-field combinations (Priority 15–23):**
first_name+last_name, first_name+sex, first_name+dob, first_name+mobile, last_name+sex, last_name+dob, last_name+mobile, sex+dob, dob+mobile

## Managing Rules

### Adding a Rule to Healthcare Settings
1. Create the rule configuration document
2. Go to Healthcare Settings > Patient Duplicate Check tab
3. Add a row to `patient_duplicate_check_rules` and link the rule

### Removing a Rule
- Delete the row from Healthcare Settings (doesn't delete the rule config itself)
- Or set the rule's action to "Allow" to effectively disable it

### Adjusting Priority
- Edit the rule configuration's `priority` field
- Lower numbers are checked first
- Ensure Disallow rules have lower priorities than Warn rules

### Disabling Entirely
- Uncheck `enable_patient_duplicate_check` in Healthcare Settings
- All rules are bypassed — no performance overhead

## Setup Patch

The initial 24 rules are created by a migration patch:
- `healthcare/healthcare/patches/v15_0/setup_patient_duplicate_check_rules.py`
- Calls `setup_patient_duplicate_check_rules()` from `healthcare/healthcare/setup/patient_duplicate_check.py`
- Safe to re-run — skips rules that already exist

## Related Code

- Setup: `healthcare/healthcare/setup/patient_duplicate_check.py` — lines 9–263
- Rule DocType: `healthcare/healthcare/doctype/patient_duplicate_check_rule_configuration/`
- Field child table: `healthcare/healthcare/doctype/patient_duplicate_check_field/`
- Link child table: `healthcare/healthcare/doctype/patient_duplicate_check_rule_link/`
- Settings: `healthcare/healthcare/doctype/healthcare_settings/healthcare_settings.json`
- Runtime checker: `healthcare/healthcare/utils.py` — `PatientDuplicateChecker` class, lines 1509–1602
