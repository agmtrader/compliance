# Contact Document Categorization Quality Analysis

## Business Purpose
Identify `contact_document` records whose stored category or type may be inconsistent with the document filename, existing extracted text, canonical English labels, or the document taxonomy used by Dashboard and Hub. The output is a review queue for correcting regulatory-file metadata; it does not change database records.

## Trigger / Frequency
- Trigger: An authorized operator runs the API development analysis script when document metadata quality needs review.
- Frequency: On demand.

## Systems Involved
- `agm-api`
- Supabase Postgres tables `contact_document`, `document`, and `document_processing`
- Local filesystem for CSV and JSON report output

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Database and Google Secret Manager access available to the API environment
- Existing `contact_document` metadata and linked document filenames
- Optional completed `text_extraction` rows in `document_processing`
- API virtual environment and `.env` loaded according to `agm-api/AGENTS.MD`

## Step-by-Step Workflow
1. The operator runs `dev/contact_document_category_analysis.py` from `agm-api` after loading `.env`.
2. The script reads contact-document metadata and linked filenames without reading the raw base64 document payload.
3. It reads existing text-extraction results where available. It does not start OCR or create processing records.
4. It normalizes known English and Spanish category/type aliases, including `Prueba de Identidad`, `Prueba de Dirección`, and `Source of Wealth` used as a type.
5. It scores category and document-kind evidence from stored type, filename, and existing extracted text. Ambiguous multi-purpose documents such as bank statements are retained for review unless independent evidence identifies the intended category.
6. It distinguishes supported stored types from more specific detected kinds. For example, an income certification is retained as the detected kind but suggested as Source-of-Wealth type `Other` because that is the current frontend enum.
7. It writes a timestamped review CSV and JSON summary. Rows are ranked as `likely_misclassified`, `review`, or `consistent`, with confidence and non-sensitive evidence labels.
8. An operator reviews the report before making any separate metadata correction. The script has no update path.

## Outputs / Records Created
- Timestamped `*_contact_document_review.csv`
- Timestamped `*_summary.json`
- Console counts and a limited sample of flagged metadata
- No database writes or raw-document changes

## Exception Paths / Failure Handling
- Missing secret or database access: initialization fails and no report is produced.
- Missing linked `document` row: the inner join excludes the link; this should be investigated separately as referential-integrity damage.
- Missing or failed text extraction: analysis continues using stored metadata and filename evidence.
- Conflicting or insufficient evidence: the row remains a review item with low or medium confidence rather than receiving an automatic correction.

## Controls / Verification Points
- Preventive control: there is no database update code in the analyzer.
- Preventive control: raw document payloads and extracted-text snippets are not written to the reports.
- Detective control: canonical-label, unsupported-type, category-conflict, type-conflict, and ambiguous-category issues are counted separately.
- Reconciliation control: the JSON summary records total rows analyzed, total rows reported, status counts, confidence counts, extraction coverage, and that database writes were disabled.
- Review control: every suggested correction requires human review before a separate change is made.

## Evidence to Retain
- Review CSV used for the operator's decision
- Matching JSON summary for the run
- Any separately approved correction plan and change evidence
- Database records after a separately authorized remediation, if one occurs

## Related Code / Pages / Routes
- Entry surface: `agm-api/dev/contact_document_category_analysis.py`
- Tests: `agm-api/tests/test_contact_document_category_analysis.py`
- Source taxonomy: `agm-dashboard/src/lib/clients/documents.ts`, `agm-dashboard/src/lib/clients/schemas/documents.ts`, and matching Hub modules
- Related review process: [Regulatory File Review and Export](18-regulatory-file-review-and-export.md)
- Downstream side effects: none

## Last Reviewed
- Status: draft
- Date: 2026-07-16
- Reviewer: Codex current-state code review
