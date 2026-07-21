# Document Completion Analysis

## Business Purpose
Produce a row-level readiness view for the OCR-reviewed other-document population so operators can separate records that are ready for metadata remediation from records still blocked by OCR failure, low OCR quality, conflicting OCR/category signals, duplicate document links, or unresolved `Other` classification.

## Trigger / Frequency
- Trigger: An authorized operator runs the analysis after refreshing `document_recategorization_proposal.csv`.
- Frequency: On demand.

## Systems Involved
- `agm-api`
- Local `other_documents_inventory.csv`
- Local `document_recategorization_proposal.csv`
- Local `document_completion_analysis.csv`

## Step-by-Step Workflow
1. Refresh `other_documents_inventory.csv` and `document_recategorization_proposal.csv`.
2. Run `document_completion_analysis.py` from the `agm-api` root with no parameters.
3. Join each proposal row back to the inventory metadata to recover duplicate-link and raw-document availability flags.
4. Classify each row into `ready`, `review`, or `blocked`.
5. Mark the completion basis and blocker according to OCR status, OCR quality, OCR classification assessment, review flag, duplicate-link flag, and whether the row still lands in `Other`.
6. Write one complete completion-analysis CSV with the row-level readiness decision and operator-review reason.

## Outputs / Records Created
- `document_completion_analysis.csv`
- Console reconciliation counts
- No database writes

## Controls / Verification Points
- Every proposal row must appear exactly once in the completion-analysis CSV.
- Rows with failed or missing OCR must not be marked ready.
- Rows that still resolve to `Other` after OCR-aware recategorization must require review.
- Duplicate raw-document links must require review before remediation.
- Any later metadata write remains a separate approved remediation step.

## Related Code / Pages / Routes
- Analysis: `agm-api/document_completion_analysis.py`
- Input proposal: `agm-api/document_recategorization_proposal.csv`
- Input inventory: `agm-api/other_documents_inventory.csv`

## Last Reviewed
- Status: draft
- Date: 2026-07-21
- Reviewer: Codex implementation review
