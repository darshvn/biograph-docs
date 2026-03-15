# Medical Code Masters & Medication/Procedure Setup

## Overview

Biograph uses a FHIR-aligned coding system with two core DocTypes: **Code System** (the standard, e.g., ICD-10, SNOMED-CT) and **Code Value** (individual codes within a system). These are linked to clinical records via the **Codification Table** child table.

Additionally, **Medication** and **Clinical Procedure Template** masters define the clinical services available, their billing items, and consumable requirements.

---

## Part 1: Medical Code Masters

### Code System

Represents a medical coding standard (e.g., ICD-10, SNOMED-CT, LOINC, CPT).

Navigate to: Healthcare workspace > Code System > New

| Field | Type | Description |
|-------|------|-------------|
| `code_system` | Data | Unique name (e.g., "ICD-10-CM") |
| `uri` | Data | Unique URI identifier (FHIR-style, e.g., "http://hl7.org/fhir/sid/icd-10-cm") |
| `description` | Small Text | Description of the coding system |
| `status` | Link → Code Value | Active/Retired/Draft |
| `version` | Link → Code Value | Version identifier |
| `is_fhir_defined` | Check | Whether this is a standard FHIR code system (default: ✓) |
| `oid` | Data | OID identifier (unique, optional) |
| `experimental` | Check | Mark as experimental (default: ✓) |
| `immutable` | Check | Codes cannot be modified once created |
| `complete` | Check | All codes in the system are present |
| `custom` | Check | User-created (read-only, auto-set) |
| `disabled` | Check | Deactivate the code system |

### Code Value

Individual codes within a Code System.

| Key Fields | Type | Description |
|-----------|------|-------------|
| `code_system` | Link → Code System | Which system this code belongs to |
| `code_value` | Data | The code itself (e.g., "J45.0") |
| `display` | Data | Human-readable name (e.g., "Predominantly allergic asthma") |
| `definition` | Small Text | Detailed description |

### Codification Table (Child Table)

This reusable child table links codes to clinical masters. Used in:
- **Appointment Type** → `codification_table`
- **Medication** → `codification_table`
- **Clinical Procedure Template** → `codification_table`
- **Lab Test Template** → `codification_table`

| Field | Type | Description |
|-------|------|-------------|
| `code_system` | Link → Code System | The coding standard |
| `system` | Data | Fetched URI (read-only) |
| `is_fhir_defined` | Check | Fetched from code system (read-only) |
| `oid` | Data | Fetched OID (read-only) |
| `code_value` | Link → Code Value | The specific code |
| `code` | Data | Fetched code value (read-only) |
| `display` | Data | Fetched display name (read-only) |
| `definition` | Data | Fetched definition (read-only) |

---

## Part 2: Medication Master {#medication-master}

### Overview

A Medication defines a drug or compound with its strength, dosage form, linked ERPNext items for billing/stock, and default prescription settings.

Navigate to: Healthcare workspace > Medication > New

### Fields

**Basic Details**

| Field | Type | Description |
|-------|------|-------------|
| `generic_name` | Data | Required, unique. The drug name (e.g., "Paracetamol 500mg") |
| `medication_class` | Link → Medication Class | Drug category (e.g., "Analgesic", "Antibiotic") |
| `national_drug_code` | Data | National drug code identifier |
| `abbr` | Data | Short abbreviation |
| `disabled` | Check | Deactivate this medication |

**Strength & Dosage**

| Field | Type | Description |
|-------|------|-------------|
| `strength` | Float | Numeric strength value (e.g., 500) |
| `strength_uom` | Link → UOM | Unit of measurement (e.g., "mg", "ml") |
| `dosage_form` | Link → Dosage Form | Form factor (e.g., "Tablet", "Syrup", "Injection") |

**Combination Medications**

| Field | Type | Description |
|-------|------|-------------|
| `is_combination` | Check | Mark as combination drug |
| `combinations` | Table (Medication Ingredient) | Component medications with their strengths (visible when is_combination=1) |

Medication Ingredient child table fields:
- `medication`: Link → Medication (the component drug)
- `strength`: fetched from the component medication
- `strength_uom`: fetched from the component medication

**Item Linking (Billing & Stock)**

| Field | Type | Description |
|-------|------|-------------|
| `linked_items` | Table (Medication Linked Item) | ERPNext Items associated with this medication |
| `price_list` | Link → Price List | Price list for auto-created Item Prices |

Medication Linked Item fields:
| Field | Type | Description |
|-------|------|-------------|
| `item_code` | Data | Auto-generated, set-only-once |
| `item` | Link → Item | The created ERPNext Item (read-only) |
| `item_group` | Link → Item Group | Required |
| `stock_uom` | Link → UOM | Stock unit of measure |
| `brand` | Link → Brand | Optional brand |
| `manufacturer` | Link → Manufacturer | Optional manufacturer |
| `is_billable` | Check | Whether to bill for this item (default: ✓) |
| `rate` | Float | Price (visible when is_billable=1) |
| `description` | Small Text | Item description |

**Prescription Defaults**

| Field | Type | Description |
|-------|------|-------------|
| `default_prescription_dosage` | Link → Prescription Dosage | e.g., "1-0-1" (morning-afternoon-night) |
| `default_prescription_duration` | Link → Prescription Duration | e.g., "7 Days", "1 Month" |
| `default_interval` | Int | Time between doses |
| `default_interval_uom` | Select | Hour / Day |

**Medical Coding**

| Field | Type | Description |
|-------|------|-------------|
| `codification_table` | Table (Codification Table) | Medical codes (e.g., ATC codes, RxNorm) |

### Auto Item Creation (medication.py)

- `after_insert()` (line 15): Calls `create_item_from_medication()` — creates ERPNext Items from `linked_items`
- `on_update()` (line 18): If linked items exist, calls `update_item_and_item_price()` to sync changes
- `insert_item()` (line 92): Creates an Item with flags `is_stock_item=1, is_sales_item=1`
- `make_item_price()` (line 126): Creates Item Price in the configured price list
- `change_item_code_from_medication()` (line 138): Renames item code if medication name changes

---

## Part 3: Clinical Procedure Template

### Overview

Defines a clinical procedure (surgery, biopsy, dressing change, etc.) with billing configuration, consumable items for stock consumption, sample collection, and nursing checklists.

Navigate to: Healthcare workspace > Clinical Procedure Template > New

### Fields

**Basic Details**

| Field | Type | Description |
|-------|------|-------------|
| `template` | Data | Required, unique name |
| `medical_department` | Link → Medical Department | Department that performs this procedure |
| `description` | Small Text | Procedure description (required) |
| `disabled` | Check | Deactivate |

**Billing**

| Field | Type | Description |
|-------|------|-------------|
| `link_existing_item` | Check | Link to an existing ERPNext Item instead of auto-creating one |
| `item` | Link → Item | The linked/created Item (read-only if auto-created) |
| `item_code` | Data | Item code |
| `item_group` | Link → Item Group | Required |
| `is_billable` | Check | Whether this procedure is chargeable |
| `rate` | Float | Price (required if billable and not linking existing item) |

**Consumable Items (Stock Consumption)**

| Field | Type | Description |
|-------|------|-------------|
| `consume_stock` | Check | Enable stock consumption tracking |
| `items` | Table (Clinical Procedure Item) | Consumable items used during the procedure |

Clinical Procedure Item fields:
| Field | Type | Description |
|-------|------|-------------|
| `item_code` | Link → Item | The consumable item (required) |
| `item_name` | Data | Auto-fetched (read-only) |
| `qty` | Float | Quantity consumed (required) |
| `uom` | Link → UOM | Unit of measure (required) |
| `batch_no` | Link → Batch | Batch tracking (optional) |
| `invoice_separately_as_consumables` | Check | Bill consumables separately from the procedure fee |

**Sample Collection**

| Field | Type | Description |
|-------|------|-------------|
| `sample` | Link → Lab Test Sample | Sample type to collect (e.g., Blood, Urine) |
| `sample_qty` | Float | Quantity |
| `sample_details` | Small Text | Collection instructions |

**Nursing Checklists**

| Field | Type | Description |
|-------|------|-------------|
| `pre_op_nursing_checklist_template` | Link → Nursing Checklist Template | Pre-operation checklist |
| `post_op_nursing_checklist_template` | Link → Nursing Checklist Template | Post-operation checklist |

**Service Request Defaults**

| Field | Type | Description |
|-------|------|-------------|
| `patient_care_type` | Link → Patient Care Type | Classification |
| `staff_role` | Link → Role | Required staff role to perform |

**Medical Coding**

| Field | Type | Description |
|-------|------|-------------|
| `codification_table` | Table (Codification Table) | Procedure codes (e.g., CPT, SNOMED) |

### Auto Item Creation (clinical_procedure_template.py)

- `after_insert()` (line 26): Creates ERPNext Item if `link_existing_item` is unchecked
- `on_update()` (line 30): Detects changes and updates Item + Item Price
- `before_insert()` (line 15): If linking existing item, fetches rate from existing Item Price
- `enable_disable_item()` (line 45): Enables/disables the linked Item based on `is_billable`
- `get_item_details()` (line 54, whitelisted): Returns item details for the procedure template

---

## Part 4: Lab Test Template (Reference)

Similar to Clinical Procedure Template but for laboratory tests. Supports 6 test types:

| Type | Description |
|------|-------------|
| **Single** | One numeric result with UOM and normal range |
| **Compound** | Multiple numeric results (child table: Normal Test Template) |
| **Descriptive** | Free-text results with optional sensitivity testing |
| **Grouped** | Composite of multiple other lab tests |
| **Imaging** | Radiology/imaging results |
| **No Result** | Procedure that doesn't produce a result |

Key unique fields: `lab_test_name`, `lab_test_template_type`, `lab_test_code`, `lab_test_group`, `lab_test_rate`, `lab_test_uom`, `lab_test_normal_range`, `sensitivity` (for culture tests).

---

## Related Code

- Code System: `healthcare/healthcare/doctype/code_system/code_system.json`
- Codification Table: `healthcare/healthcare/doctype/codification_table/codification_table.json`
- Medication: `healthcare/healthcare/doctype/medication/medication.py`
- Medication JSON: `healthcare/healthcare/doctype/medication/medication.json`
- Medication Ingredient: `healthcare/healthcare/doctype/medication_ingredient/medication_ingredient.json`
- Medication Linked Item: `healthcare/healthcare/doctype/medication_linked_item/medication_linked_item.json`
- Dosage Form: `healthcare/healthcare/doctype/dosage_form/dosage_form.json`
- Clinical Procedure Template: `healthcare/healthcare/doctype/clinical_procedure_template/clinical_procedure_template.py`
- Clinical Procedure Item: `healthcare/healthcare/doctype/clinical_procedure_item/clinical_procedure_item.json`
- Lab Test Template: `healthcare/healthcare/doctype/lab_test_template/lab_test_template.json`
