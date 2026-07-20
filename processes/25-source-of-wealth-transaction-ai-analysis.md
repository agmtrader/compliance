# Source of Wealth Document OCR Matching

## Business Purpose
Create a read-only reconciliation between database `contact_document` rows categorized as Source of Wealth and the local `source_of_wealth_ocr_extractions.csv` file. The result shows which database Source-of-Wealth records already have retained local OCR text, which are missing local OCR, and which local OCR rows no longer match a current database Source-of-Wealth row.

This process does not analyze deposits, withdrawals, account financial ranges, employment, Source of Funds, or AML disposition.

## Trigger / Frequency
- Trigger: An authorized operator runs `source_of_wealth_transaction_analysis.py` from `agm-api`.
- Frequency: On demand.

## Systems Involved
- `agm-api`
- Supabase Postgres `contact_document` table (read-only)
- Local filesystem OCR extraction CSV and reconciliation outputs

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Completed or attempted local rows in `source_of_wealth_ocr_extractions.csv`
- Database `contact_document` rows where `category` equals Source of Wealth
- API virtual environment and `.env` loaded according to `agm-api/AGENTS.MD`

## Step-by-Step Workflow
1. The script reads every database `contact_document` categorized as Source of Wealth. This is one read-only database operation; the script does not read raw document bytes or write any table.
2. It reads local OCR rows from `source_of_wealth_ocr_extractions.csv`.
3. It matches each database Source-of-Wealth row to local OCR by `contact_document_id`.
4. If no `contact_document_id` match exists, it falls back to matching by `document_id`.
5. It writes one detail row for every database Source-of-Wealth row, including whether local OCR was found, the link method, completion status, extracted financial fields, preview, and retained OCR text.
6. It writes summary analytics for linked versus unlinked database rows, completed usable text, link-method counts, unmatched local OCR rows, and read/write counts.

## Outputs / Records Created
- `source_of_wealth_document_ocr_matches.csv`: one row per database Source-of-Wealth `contact_document`, enriched with local OCR fields when linked
- `source_of_wealth_document_ocr_match_summary.csv`: linked/unlinked population counts
- Console progress and final counts
- One read-only database table load and no database writes

## Exception Paths / Failure Handling
- Missing local OCR file or empty file: all database Source-of-Wealth rows are reported as unlinked to local OCR.
- Database connection or schema-reflection failure: the run fails visibly.
- Local OCR rows with no current database Source-of-Wealth match are counted in the summary.
- Local OCR rows with failed or incomplete status are linked when IDs match, but are not counted as completed usable text.

## Controls / Verification Points
- Preventive control: database access is limited to a read of `contact_document`; there is no database write path.
- Population control: every database Source-of-Wealth row appears in the detail output, including unlinked rows.
- Linkage control: local OCR joins by `contact_document_id`, with `document_id` only as fallback; names, account numbers, and transaction data are not used.
- Completeness control: the summary separately reports database rows with any local OCR row, database rows with completed text, database rows without local OCR, and local OCR rows not matched back to the database.

## Evidence to Retain
- Local OCR extraction CSV used by the run
- Detail CSV and summary CSV from the same run
- Console summary or retained terminal output showing the run counts

## Related Code / Pages / Routes
- Entry surface: `agm-api/source_of_wealth_transaction_analysis.py`
- Tests: `agm-api/tests/test_source_of_wealth_transaction_analysis.py`
- Upstream extraction: [Source of Wealth OCR Analysis](24-source-of-wealth-ocr-analysis.md)
- Downstream side effects: none

## Last Reviewed
- Status: draft
- Date: 2026-07-20
- Reviewer: Codex current-state implementation review
