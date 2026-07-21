# Proof of Address OCR Analysis

## Business Purpose
Create a resumable text-extraction dataset for every document categorized as `Proof of Address`. The workflow records OCR text and quality metadata only; it does not interpret, validate, approve, or persist address data.

## Trigger / Frequency
- Trigger: An authorized operator runs the root API script when Proof-of-Address OCR is required.
- Frequency: On demand, in batches of up to 100 previously unprocessed documents.

## Systems Involved
- `agm-api`
- Supabase Postgres tables `contact_document` and `document` (read only)
- Google Document AI Enterprise OCR
- Local `proof_of_address_ocr_extractions.csv`

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro

## Inputs / Prerequisites
- Database and Google Document AI access.
- `contact_document` rows categorized as `Proof of Address`, compared case-insensitively after trimming whitespace, with linked document data.
- API `.env` and virtual environment configured according to `agm-api/AGENTS.MD`.

## Step-by-Step Workflow
1. The operator runs `proof_of_address_ocr_analysis.py` from the `agm-api` root. The script accepts no command-line parameters.
2. The script reads the existing root CSV and excludes previously processed contact-document and raw-document identifiers.
3. It reads up to 100 remaining `Proof of Address` rows per run, selecting each raw document only once. Repeated runs continue with the next unprocessed batch.
4. It decodes each stored document and calls the same Google Document AI OCR pipeline, render settings, and cache used by the identity analysis.
5. It appends each result immediately so an interrupted run can resume without repeating completed provider calls.

## Outputs / Records Created
- `proof_of_address_ocr_extractions.csv` containing identifiers, source metadata, provider/model/pipeline metadata, OCR quality data, preview, full OCR text, status, timing, and errors.
- Console progress and final completed/failed counts.
- No database writes and no address extraction or validation output.

## Exception Paths / Failure Handling
- Missing provider or database configuration stops initialization.
- Missing document identifiers, missing raw documents, invalid base64, or OCR failures are appended as failed rows with the exact error.
- An incompatible existing CSV header stops the run instead of corrupting its history.

## Controls / Verification Points
- Preventive control: normalized category filtering, a fixed batch limit of 100, and a fixed root output path.
- Cost/resume control: existing identifiers and duplicate raw documents are not OCRed again.
- Evidence control: provider, version, quality, timing, full text, and error details are retained per row.
- Scope control: the workflow performs no semantic address extraction and no database writes.

## Related Code / Pages / Routes
- Entry point: `agm-api/proof_of_address_ocr_analysis.py`
- Reference workflow: `agm-api/id_expiration_analysis.py`
- Shared OCR implementation: `agm-api/src/components/clients/document_processing.py`

## Last Reviewed
- Status: draft
- Date: 2026-07-21
- Reviewer: Codex implementation review
