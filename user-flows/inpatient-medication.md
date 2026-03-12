# Inpatient Medication Administration

## User Story

> As a **nurse**, I want to administer and track inpatient medications so that patients receive the right drugs on schedule and stock is updated.

## Entry Point

Patient Encounter > Make IP Medication Order (`make_ip_medication_order`)

## Steps

1. **Create medication order**
   - API: `patient_encounter.make_ip_medication_order(source_name)`
   - Data changed: Inpatient Medication Order created with entries from encounter drug prescriptions

2. **Add orders from encounters**
   - API: `inpatient_medication_order.get_from_encounter(encounter)` (instance)
   - API: `inpatient_medication_order.add_order_entries(order)` (instance)
   - Data changed: Order entries added

3. **Create medication entry**
   - Screen: Inpatient Medication Entry > New
   - API: `inpatient_medication_entry.get_medication_orders()` (instance)
   - Data changed: Pending medications loaded into entry form

4. **Administer medications**
   - User action: Mark medications as administered with timestamps
   - Data changed: Inpatient Medication Entry Detail child records

5. **Submit entry**
   - User action: Submit
   - Data changed: Stock Entry created (Material Issue) for administered drugs

6. **Reconcile differences** *(if wastage/returns)*
   - API: `inpatient_medication_entry.make_difference_stock_entry(docname)`
   - Data changed: Additional Stock Entry for differences

## Error States

- Stock insufficient → warning shown
- Medication order cancelled → entry cannot be created

## Permissions

- **Nursing User**: Full access to medication entries
- **Physician**: Can create medication orders

## Related Code

- Order: `healthcare/healthcare/doctype/inpatient_medication_order/inpatient_medication_order.py`
- Entry: `healthcare/healthcare/doctype/inpatient_medication_entry/inpatient_medication_entry.py`
