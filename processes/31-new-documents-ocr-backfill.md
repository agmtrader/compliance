# New Documents OCR Backfill

## Business Purpose
Backfill missing `document_processing` `text_extraction` rows for linked contact documents while preserving a resumable CSV trail of every OCR attempt. The workflow exists to improve shared OCR coverage for Dashboard and API consumers that read live `document_processing` text.

## Trigger / Frequency
- Trigger: An authorized operator runs `new_documents_ocr.py` from `agm-api`.
- Frequency: On demand.

## Systems Involved
- `agm-api`
- Supabase Postgres `contact_document`, `document`, and `document_processing`
- Google Document AI Enterprise OCR
- Local filesystem for `new_documents_ocr_extractions.csv`

## Inputs / Prerequisites
- Database and Google Document AI access available to the API environment
- Existing `contact_document` rows linked to live `document` rows
- Stored base64 document payloads and MIME types in `document`
- API virtual environment and `.env` loaded according to `agm-api/AGENTS.MD`

## Step-by-Step Workflow
1. The operator runs `new_documents_ocr.py` from `agm-api`.
2. The script performs a database-only preflight and prints how many unique linked raw documents would be OCR'd.
3. It selects every unique linked raw document that does not already have a completed non-empty `document_processing` `text_extraction` row. The CSV is not read and does not affect eligibility.
4. It decodes the raw stored document bytes and runs the shared Google OCR provider.
5. For PDFs above the provider's 30-page limit, it reruns the same OCR request in ordered page chunks and combines the per-chunk OCR pages into one document result.
6. If OCR succeeds and produces non-empty text, it upserts a completed `document_processing` `text_extraction` row for that `document_id`.
7. It appends one CSV row per attempt as an output log, including OCR status, quality metadata, preview text, full text, and any exact failure message.

## Outputs / Records Created
- `new_documents_ocr_extractions.csv`
- Completed `document_processing` `text_extraction` rows for successful OCR attempts
- Console population, progress, and reconciliation summaries

## Exception Paths / Failure Handling
- Missing secret, database, or Google Document AI configuration: initialization fails and the batch does not start.
- Missing `document_id`, missing raw document, empty data, or invalid base64: the individual row is retained as failed and processing continues.
- OCR/provider failure: the failed row and exact error are retained; the script does not hide the failure with another provider.
- PDF page-limit constraint: oversized PDFs are retried automatically in page chunks instead of failing immediately for page count alone.
- Database write failure after OCR success: the row is retained as failed and no completed `document_processing` row is persisted for that attempt.

## Controls / Verification Points
- Population control: only documents without a completed non-empty `document_processing` `text_extraction` row are selected.
- Database-only eligibility control: CSV history never blocks or admits a document; only a completed non-empty database extraction row does.
- Provider-limit control: oversized PDFs are chunked into ordered page groups of at most 30 pages.
- Write control: only successful OCR attempts write to `document_processing`.
- Reconciliation control: console output reports eligible rows, selected rows, completed rows, failed rows, and successful database writes.

## Evidence to Retain
- `new_documents_ocr_extractions.csv`
- Console output for the run
- Any follow-up remediation notes for rows that continue to fail

## Related Code / Pages / Routes
- Entry surface: `agm-api/new_documents_ocr.py`
- Shared OCR: `agm-api/src/components/clients/document_processing.py`
- Operational consumer: `agm-dashboard/src/components/dashboard/tools/private/reporting/deposits-withdrawals/DepositsWithdrawalsReport.tsx`
- Operational consumer: `agm-dashboard/src/components/dashboard/tools/private/reporting/accounts-audit/AccountsAuditReport.tsx`

## Last Reviewed
- Status: draft
- Date: 2026-07-21
- Reviewer: Codex current-state implementation review
