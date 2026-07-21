# OCR CSV Database Import

## Business Purpose
Persist previously completed identity and Source-of-Wealth OCR text from the retained analysis CSVs into `document_processing` so API and frontend consumers can use the same extracted text without rerunning OCR.

## Trigger / Frequency
- Trigger: An authorized operator runs the importer after reviewing the current OCR CSV outputs.
- Frequency: On demand.

## Systems Involved
- `agm-api`
- Local `id_document_ocr_extractions.csv` and `source_of_wealth_ocr_extractions.csv`
- Supabase Postgres tables `document` and `document_processing`

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Completed CSV rows must contain `document_id`, `status`, `output_text`, provider, and document language.
- Database access must be available to the API environment.
- API virtual environment and `.env` must be loaded according to `agm-api/AGENTS.MD`.

## Step-by-Step Workflow
1. The operator runs `import_ocr_csv_to_document_processing.py`. The script uses the two fixed OCR CSV paths at the `agm-api` repository root and accepts no command-line parameters.
2. The importer reads both OCR extraction CSVs and retains only rows whose status is `completed` and whose OCR text is non-empty.
3. It validates document UUIDs, rejects conflicting duplicate CSV rows, and resolves eligible identifiers against `document`. Rows for source documents that have since been deleted are reported and skipped because the foreign key target no longer exists.
4. It locks existing `text_extraction` processing rows and skips them; the importer never replaces existing extracted text.
5. It inserts new `document_processing` rows with status `completed`, process type `text_extraction`, source language, OCR text, provider, and no error.
6. It reads the written values back in the same transaction. Any count or field mismatch rolls back all writes.

## Outputs / Records Created
- One `document_processing` `text_extraction` row per newly imported document.
- Console source, eligibility, insert, skip, and reconciliation counts.
- No changes to `document`, `contact_document`, financial profiles, or expiry dates.

## Exception Paths / Failure Handling
- Missing CSV, missing required columns, invalid document UUID, conflicting duplicate OCR, or duplicate live processing rows stop the run before writes.
- CSV rows whose source document has been deleted are counted and skipped.
- Existing processing rows are never changed by this importer.
- Reconciliation failure rolls back the transaction.

## Controls / Verification Points
- Preventive control: only successful non-empty OCR rows are eligible.
- Preventive control: fixed root CSV inputs and no command-line parameters constrain the source population.
- Preventive control: existing extracted text is always preserved.
- Integrity control: source conflicts and database duplicate processing rows fail closed.
- Referential control: only CSV document identifiers still present in `document` are eligible for insertion; stale identifiers are reported and skipped.
- Reconciliation control: every inserted or replaced field is read back before commit.

## Evidence to Retain
- The two source OCR CSVs used for the run.
- Applied-run console summary.
- Resulting `document_processing` rows and timestamps.

## Related Code / Pages / Routes
- Importer: `agm-api/import_ocr_csv_to_document_processing.py`
- Source analyses: `agm-api/account_audit.py` and `agm-api/id_expiration_analysis.py`
- Runtime processing implementation: `agm-api/src/components/clients/document_processing.py`

## Last Reviewed
- Status: draft
- Date: 2026-07-21
- Reviewer: Codex implementation review
