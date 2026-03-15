# Patient Duplicate Checking

## User Story

> As a **healthcare administrator**, I want the system to automatically detect duplicate patients on registration so that we maintain clean patient records.

> As a **system administrator**, I want to configure which field combinations trigger duplicate warnings or blocks so that the rules match our hospital's data quality policy.

## Entry Point

- **Automatic**: Triggered on every new Patient save (validate event)
- **Configuration**: Healthcare Settings > Patient Duplicate Check tab

## How It Works

When a new Patient record is saved, the system runs all configured duplicate check rules against existing patients. Rules are ordered by priority (lower number = higher priority). Each rule specifies a combination of fields to match and an action to take:

- **Disallow**: Block the save entirely — user cannot proceed
- **Warn**: Show a warning with matching patients — user can override and continue
- **Allow**: No action even if matches are found

The first non-"allow" result stops the check — so a priority-1 "Disallow" rule always takes precedence.

## Steps — What Happens on Patient Save

1. **Trigger duplicate check**
   - Event: `Patient.validate()` (patient.py line 33–56)
   - Condition: Patient is new (`self.is_new()`) AND `enable_patient_duplicate_check = 1` in Healthcare Settings
   - Creates: `PatientDuplicateChecker(self)` instance

2. **Run rules in priority order**
   - Method: `check_duplicates()` (utils.py line 1515–1545)
   - Fetches all rules from `Healthcare Settings.patient_duplicate_check_rules`
   - Sorts by priority ascending (lower = higher priority, line 1537)
   - Iterates through rules, calling `_check_rule(rule)` for each

3. **Evaluate each rule**
   - Method: `_check_rule(rule)` (utils.py line 1547–1592)
   - For each field in `rule.duplicate_fields`:
     - Collects the field value from the patient document (line 1558–1561)
   - **Important**: Only proceeds if ALL fields in the rule have values (line 1563–1568). If the patient hasn't filled in `mobile` and the rule requires it, that rule is skipped.
   - Builds filter dict from field values
   - Queries Patient table excluding the current patient (line 1575–1576)
   - Returns matching patients with: name, patient_name, sex, dob, mobile, email

4. **Handle the result**
   - If `action = "Disallow"` and matches found:
     - Throws error with `wide=True, is_minimizable=True` (line 45–51)
     - Patient save is **blocked**
     - Error shows the rule's custom message + list of matching patients
   - If `action = "Warn"` and matches found:
     - Stores warning in `self.flags.duplicate_check_warning` (line 53–56)
     - Patient is saved, but a **warning banner** appears
   - If `action = "Allow"` or no matches:
     - Continues to next rule

## The 24 Pre-Configured Rules

Created by `setup_patient_duplicate_check_rules()` in `healthcare/healthcare/setup/patient_duplicate_check.py`.

### Disallow Rules (Priority 1–6)

| # | Rule Name | Fields | Priority |
|---|-----------|--------|----------|
| 1 | All Fields Exact Match | first_name, last_name, sex, dob, mobile | 1 |
| 2 | Name Gender DOB Match | first_name, last_name, sex, dob | 2 |
| 3 | Name Gender Mobile Match | first_name, last_name, sex, mobile | 3 |
| 4 | Name DOB Mobile Match | first_name, last_name, dob, mobile | 4 |
| 5 | First Name Gender DOB Mobile | first_name, sex, dob, mobile | 5 |
| 6 | Last Name Gender DOB Mobile | last_name, sex, dob, mobile | 6 |

### Warn Rules (Priority 7–23)

Various 2-field and 3-field combinations:
- 3-field: first_name+last_name+sex, first_name+last_name+dob, first_name+last_name+mobile, first_name+sex+dob, first_name+sex+mobile, first_name+dob+mobile, last_name+sex+dob, last_name+sex+mobile
- 2-field: first_name+last_name, first_name+sex, first_name+dob, first_name+mobile, last_name+sex, last_name+dob, last_name+mobile, sex+dob, sex+mobile, dob+mobile

## Field Matching Strategy

- **Exact match only** — no fuzzy/phonetic matching
- **All fields required** — if a rule checks 4 fields but the patient only has 3 filled, that rule is skipped entirely
- **Fields available**: first_name, last_name, sex, dob, mobile, email
- **Priority ordering** ensures the most restrictive rules (4-5 field matches → Disallow) are checked before lenient ones (2-field matches → Warn)

## Configuration

### Enabling the Feature
Healthcare Settings > Patient Duplicate Check tab:
- `enable_patient_duplicate_check`: Check to enable (default: off)

### Managing Rules
Healthcare Settings > `patient_duplicate_check_rules` table:
- Each row links to a `Patient Duplicate Check Rule Configuration` document
- Displays: rule name, action, priority (all read-only in the table)
- Add/remove rows to control which rules are active

### Creating Custom Rules
Navigate to Patient Duplicate Check Rule Configuration > New:
- `rule_name`: Descriptive name (unique)
- `action`: Allow / Warn / Disallow
- `priority`: Lower number = higher priority
- `message`: Custom message shown to the user on match
- `duplicate_fields`: Table of fields to match (select from: first_name, last_name, sex, dob, mobile, email)

## Error States

- All rule fields missing from patient → rule silently skipped
- No rules configured → check returns "allow" (no blocking)
- Feature disabled → check bypassed entirely
- Duplicate detected (Disallow) → save blocked with error dialog showing matches
- Duplicate detected (Warn) → save succeeds, warning banner shown

## Permissions

- **System Manager**: Can enable/disable feature, manage rules
- **Healthcare Administrator**: Can manage rules
- All roles that can create Patients: affected by duplicate checking on save

## Related Code

- Backend: `healthcare/healthcare/doctype/patient/patient.py` — `validate()` lines 33–56
- Checker: `healthcare/healthcare/utils.py` — `PatientDuplicateChecker` class, lines 1509–1602
- Rule setup: `healthcare/healthcare/setup/patient_duplicate_check.py` — lines 9–263
- Rule config: `healthcare/healthcare/doctype/patient_duplicate_check_rule_configuration/patient_duplicate_check_rule_configuration.py`
- Settings: `healthcare/healthcare/doctype/healthcare_settings/healthcare_settings.json` — Patient Duplicate Check tab
