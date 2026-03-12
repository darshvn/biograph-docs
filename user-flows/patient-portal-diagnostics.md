# Patient Portal: Diagnostics

## User Story

> As a **patient**, I want to view my lab test orders and results online so that I can track my health without visiting the clinic.

## Entry Point

`/patient-portal` > Diagnostics tab

## Steps

1. **View orders**
   - Screen: DiagnosticModel component
   - API: `patient_portal.get_orders()`
   - Shows: All test orders (from Service Requests + Sales Invoices), sorted by date

2. **Review test details**
   - Each order card shows:
     - Order name, date, referring practitioner
     - Billing status (Paid / Partly Paid / Unpaid)
     - Diagnostic report status
     - Individual tests with results, units, approval status

3. **View component results** *(for panel tests)*
   - Nested child observations displayed under parent test
   - Shows: template name, result value, unit, reference ranges

4. **Check sample collection status**
   - Shows: Collection status, date/time, collection point

5. **Print diagnostic report**
   - User action: Click print icon
   - API: `patient_portal.get_print_format("Diagnostic Report", name)`
   - Returns: Print format and letter head for PDF generation

## Error States

- No orders found → empty state displayed
- Observation pending approval → shown as "Pending" status

## Permissions

- **Patient role** required
- Data filtered to logged-in patient + patient relations

## Related Code

- Frontend: `patient_portal/src/components/DiagnosticModel.vue`
- Backend: `healthcare/healthcare/api/patient_portal.py` → `get_orders()`, `get_print_format()`
