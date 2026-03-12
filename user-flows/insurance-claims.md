# Insurance Claims

## User Story

> As a **billing administrator**, I want to process insurance claims so that covered services are reimbursed by the insurer.

## Entry Point

Patient Insurance Policy → Coverage determination → Claim

## Steps

1. **Verify active contract**
   - API: `insurance_payor.has_active_contract(insurance_payor, company, on_date)`
   - Returns: Whether the insurer has a valid contract with the facility

2. **Create patient insurance policy**
   - Screen: Patient Insurance Policy form
   - Data changed: Policy linked to patient with payor and plan details

3. **Determine coverage**
   - API: `patient_insurance_coverage.create_insurance_eligibility(doc)`
   - Data changed: Patient Insurance Coverage record specifying covered items and percentages

4. **Deliver healthcare services**
   - Normal clinical workflows (encounters, labs, procedures, etc.)

5. **Create insurance claim**
   - Screen: Insurance Claim form
   - API: `insurance_claim.get_coverages()` (instance) — returns applicable coverages
   - Data changed: Insurance Claim with coverage details

6. **Submit claim to payor**
   - User action: Submit claim
   - Data changed: Claim status → Submitted

7. **Receive payment**
   - User action: Create Payment Entry referencing the claim
   - Doc event: `insurance_claim.validate_payment_entry_and_set_claim_fields` (on PE validate)
   - Doc event: `payment_entry.set_paid_amount_in_healthcare_docs` (on PE submit)
   - Data changed: Claim payment recorded, healthcare docs updated

## Error States

- No active contract → `has_active_contract` returns false
- Coverage not determined → claim creation blocked
- Payment amount mismatch → validation error on Payment Entry

## Permissions

- **Healthcare Administrator**: Full insurance workflow access

## Related Code

- Payor: `healthcare/healthcare/doctype/insurance_payor/insurance_payor.py`
- Coverage: `healthcare/healthcare/doctype/patient_insurance_coverage/patient_insurance_coverage.py`
- Claim: `healthcare/healthcare/doctype/insurance_claim/insurance_claim.py`
- PE hooks: `healthcare/healthcare/custom_doctype/payment_entry.py`
