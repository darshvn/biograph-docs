# Clinical Procedure

## User Story

> As a **physician**, I want to perform and document clinical procedures so that consumable usage is tracked and the procedure is billable.

## Entry Point

Service Request from Patient Encounter

## Steps

1. **Create procedure**
   - API: `service_request.make_clinical_procedure(service_request)`
   - Data changed: Clinical Procedure record created

2. **Verify stock availability**
   - User action: Check consumables are in stock
   - API: `clinical_procedure.get_procedure_consumables(procedure_template)`

3. **Start procedure**
   - User action: Click "Start"
   - API: `clinical_procedure.start_procedure()` (instance method)
   - Data changed: Procedure status → In Progress

4. **Complete procedure**
   - User action: Click "Complete"
   - API: `clinical_procedure.complete_procedure()` (instance method)
   - Data changed: Procedure status → Completed

5. **Consume stocks**
   - Automatic: Stock entry created for consumed items
   - API: `clinical_procedure.make_stock_entry(doc)`, `set_stock_items(doc, ...)`
   - Data changed: Stock Entry (Material Issue) created in ERPNext

## Error States

- Insufficient stock → `verify_stock` shows warning
- Missing billing item → validation error on procedure template

## Permissions

- **Physician**: Can create and perform procedures
- **Nursing User**: Can assist and update stock consumption

## Related Code

- Backend: `healthcare/healthcare/doctype/clinical_procedure/clinical_procedure.py`
- Template: `healthcare/healthcare/doctype/clinical_procedure_template/`
