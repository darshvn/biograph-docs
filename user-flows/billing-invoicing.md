# Billing & Invoicing

## User Story

> As a **billing administrator**, I want to aggregate all healthcare services into invoices so that patients or insurers are billed accurately.

## Entry Point

Desk > Sales Invoice > New > Get Healthcare Services to Invoice

## Steps

1. **Open Sales Invoice**
   - Screen: Sales Invoice form (ERPNext)
   - User action: Click "Get Healthcare Services to Invoice" button (added by Biograph's JS override)

2. **Select patient**
   - User action: Choose patient (must have a linked Customer in ERPNext)
   - API: `utils.get_healthcare_services_to_invoice(patient, customer, company, link_customer)`

3. **Review billable items**
   - System aggregates from 10 sources:
     - Appointments, Encounters, Lab Tests, Clinical Procedures
     - Inpatient Services, Therapy Plans, Therapy Sessions
     - Service Requests, Observations, Package Subscriptions
   - User action: Select which items to invoice

4. **Add items to invoice**
   - API: `sales_invoice.set_healthcare_services(checked_values)` (instance)
   - Data changed: Sales Invoice items populated

5. **Submit invoice**
   - User action: Submit
   - Doc event: `on_submit` → `utils.manage_invoice_submit_cancel`
   - Data changed: Linked healthcare docs marked as "Invoiced"

6. **Record payment**
   - User action: Create Payment Entry against the invoice
   - Doc events:
     - `payment_entry.manage_payment_entry_submit_cancel` → handles insurance claims
     - `payment_entry.set_paid_amount_in_healthcare_docs` → updates Treatment Counselling / Package Subscription paid amounts

## Error States

- Patient not linked to Customer → warning with option to auto-link
- Service already invoiced → excluded from billable items list
- Invoice cancelled → `on_cancel` reverts all billing status flags

## Permissions

- **Healthcare Administrator**: Can generate healthcare invoices
- **Accounts User**: Standard ERPNext invoice permissions

## Related Code

- Utilities: `healthcare/healthcare/utils.py` → `get_healthcare_services_to_invoice()` (entry point that calls 10 sub-functions)
- SI override: `healthcare/healthcare/custom_doctype/sales_invoice.py`
- PE hooks: `healthcare/healthcare/custom_doctype/payment_entry.py`
- JS: `healthcare/public/js/sales_invoice.js`
