# New Documents OCR Backfill

## Business Purpose
Backfill missing `document_processing` `text_extraction` rows for linked contact documents while preserving a resumable CSV trail of every OCR attempt. The workflow exists to improve shared OCR coverage for Dashboard and API consumers that read live `document_processing` text.

## Trigger / Frequency
- Trigger: New contact uploads attempt OCR in the shared API; an authorized operator runs `new_documents_ocr.py` from `agm-api` for missing/failed extraction recovery.
- Frequency: Event driven for new uploads; on demand for backfill.

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

## Contact Upload OCR and Content Reuse — October 14, 2026
Dashboard Contact Documents, Accounts Audit uploads, Hub Contact Documents and Hub application uploads use the shared `POST /contacts/documents` API. The API verifies SHA-1 against decoded content and atomically reuses/creates the canonical raw `document`, reuses/creates one `contact_document` link per contact/file regardless of account context and claims one `document_processing` `text_extraction` attempt with status `pending`. A PostgreSQL transaction advisory lock by content serializes identical uploads across workers; raw bytes are compared before reuse so a matching checksum alone is insufficient. Existing review metadata and original account context are preserved on repeated upload; the response uses `link_reused=true` when no new contact link is created.

After commit, PDFs and supported raster images run the existing Google Document AI Enterprise OCR provider explicitly; supported plain text uses the shared native text parser. OCR completion stores nonempty text and `completed`; provider/configuration/unsupported-file failures store `failed`. If processing-status persistence fails or the worker stops, the durable pending row remains for later recovery. Upload returns successful `id`/`document_id` once persistence succeeded, regardless of OCR outcome; no OCR warning/error is exposed to the client. Existing completed, failed or pending extraction attempts are reused rather than automatically repeated by upload. Recovery scheduling is outside this change; the existing backfill still selects documents without completed nonempty extraction.

Duplicate responses include `duplicate=true`; interactive upload surfaces show a successful existing-file reuse message. Route authentication and stored caller scopes still authorize writes. Clients cannot select an existing raw document by arbitrary ID/hash: uploaded bytes must match canonical content. Edit/delete sends `contact_document_id` and changes only that selected link. Legacy document-only writes are permitted only when exactly one link exists; ambiguous shared documents return 409. Raw content and its processing rows are deleted only when the last link is removed. Existing broader document-write scope remains unchanged; this is not a new per-client capability system.

Evidence: synthetic Flask/JWT upload and mutation tests, retained completed/failed/pending states, repeated and concurrent identical uploads, hash conflicts, atomic rollback and shared-document edit/delete isolation. Local tests use SQLite transaction serialization and mocked OCR; CI is configured with disposable local PostgreSQL for advisory-lock concurrency. No client documents/provider calls/production writes are used by tests. No schema changes or DDL migration are required.
