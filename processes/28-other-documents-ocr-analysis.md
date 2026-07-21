# Other Documents OCR Analysis

## Business Purpose
Create resumable OCR text for contact documents not categorized as `Proof of Address`, `Proof of Identity`, or `Source of Wealth`. This includes blank categories. The workflow extracts text only and does not classify, interpret, approve, or update documents.

## Trigger / Frequency
- Trigger: An authorized operator runs the root API script.
- Frequency: On demand, in batches of up to 100 previously unprocessed documents.

## Systems Involved
- `agm-api`
- Supabase Postgres tables `contact_document` and `document` (read only)
- Google Document AI Enterprise OCR
- Local `other_documents_ocr_extractions.csv`

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro

## Step-by-Step Workflow
1. Run `other_documents_ocr_analysis.py` from the `agm-api` root. It accepts no command-line parameters.
2. Read the existing root CSV and exclude previously processed contact-document and document identifiers.
3. Normalize category comparison by trimming whitespace and ignoring case, then count and print the total eligible contact-document rows, total unique documents, remaining unique documents, and rows missing a document identifier.
4. Select up to 100 unique raw documents whose category is null or is not one of the three excluded categories.
5. Decode each document and use the shared Google Document AI OCR pipeline and cache.
6. Append each completed or failed row immediately so subsequent runs resume with the next batch.

## Outputs / Records Created
- `other_documents_ocr_extractions.csv` with source metadata, provider and quality metadata, full OCR text, status, timing, and exact errors.
- Console total-population, remaining-population, and batch counts.
- No database writes.

## Exception Paths / Failure Handling
- Configuration failures stop initialization.
- Missing documents, invalid data, and OCR errors are retained as failed CSV rows.
- An incompatible existing CSV header stops processing.

## Controls / Verification Points
- Scope control: POA, POI, and SOW categories are excluded case-insensitively after trimming; null categories are included.
- Batch control: no more than 100 unique raw documents are selected per run.
- Resume control: previously recorded identifiers and duplicate raw documents are skipped.
- Side-effect control: the script performs no database writes or semantic extraction.

## Related Code / Pages / Routes
- Entry point: `agm-api/other_documents_ocr_analysis.py`
- Shared batch implementation: `agm-api/proof_of_address_ocr_analysis.py`
- Shared OCR implementation: `agm-api/src/components/clients/document_processing.py`

## Last Reviewed
- Status: draft
- Date: 2026-07-21
- Reviewer: Codex implementation review
