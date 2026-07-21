# Document Recategorization Analysis

## Business Purpose
Produce a complete row-level proposal that reduces the generic `Other` population by assigning specific evidence-backed document categories after the retained OCR pass is available. Existing `Proof of Existence` records are locked to that category because they support organization-document requirements.

## Trigger / Frequency
- Trigger: An authorized operator runs the analysis after refreshing `other_documents_inventory.csv`.
- Frequency: On demand.

## Systems Involved
- `agm-api`
- Local `other_documents_inventory.csv`
- Local `other_documents_ocr_extractions.csv`
- Local `document_recategorization_proposal.csv`
- Dashboard and Hub document taxonomies as downstream dependencies

## Step-by-Step Workflow
1. Refresh `other_documents_inventory.csv` and `other_documents_ocr_extractions.csv`, then run `document_recategorization_analysis.py` from the `agm-api` root with no parameters.
2. Preserve all existing `Proof of Existence`, `Manifest`, `New Application Form`, and `Deposits & Withdrawals` assignments.
3. Apply deterministic normalized type-and-filename rules to propose advisor authorization, organization existence, account-opening resolution, ownership and beneficial ownership, compliance questionnaire, financial statements, corporate resolution, security questions, and transaction categories.
4. Compare the proposed category with OCR phrase detection and record whether OCR supports, conflicts with, or is inconclusive for the proposal.
5. Route insufficient evidence, explicit transaction ambiguity, and OCR-only category changes to review rather than forcing a no-review decision.
6. Write the complete proposal CSV with current/proposed category, OCR status and quality, OCR classification assessment, confidence, review flag, rule, and rationale.

## Outputs / Records Created
- `document_recategorization_proposal.csv`
- Console reconciliation counts
- No database writes

## Controls / Verification Points
- Every inventory row must appear exactly once in the proposal.
- Existing Proof-of-Existence rows cannot be proposed to another category.
- Low-confidence, OCR-conflict, and transaction-ambiguous rows require review.
- New proposed category names must be added consistently to Dashboard and Hub before any approved database remediation.
- Any later database write requires a separate approved remediation step and post-write reconciliation.

## Related Code / Pages / Routes
- Analysis: `agm-api/document_recategorization_analysis.py`
- Input inventory: `agm-api/other_documents_inventory.csv`
- Input OCR: `agm-api/other_documents_ocr_extractions.csv`
- Taxonomy: `agm-dashboard/src/lib/clients/documents.ts` and `agm-hub/src/lib/clients/documents.ts`

## Last Reviewed
- Status: draft
- Date: 2026-07-21
- Reviewer: Codex implementation review
