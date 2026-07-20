# Source of Wealth Transaction AI Analysis

## Business Purpose
Create a read-only manual-review dataset that compares locally retained Source-of-Wealth OCR text with IBKR financial ranges, declared wealth sources, holder employment, and deposit/withdrawal activity. The result highlights consistency, gaps, and contradictions; it does not approve evidence, make an AML disposition, or establish the Source of Funds for a specific transaction.

## Trigger / Frequency
- Trigger: An authorized operator runs `source_of_wealth_transaction_analysis.py` from `agm-api`.
- Frequency: On demand. The default is 10 eligible accounts; `SOW_MATCH_TARGET_ACCOUNTS` may set a positive larger batch.

## Systems Involved
- `agm-api`
- Supabase Postgres `contact_document`, `account_contact`, and `account` tables (read-only)
- Local filesystem OCR and analysis CSV/JSONL files
- Google Drive deposits/withdrawals and IBKR-account-details resources
- IBKR financial-range endpoint
- Configured Gemini model for structured evidence comparison

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Completed local rows in `source_of_wealth_ocr_extractions.csv`
- Source-of-Wealth `contact_document` rows and their account/contact relationships
- Deposits/withdrawals report files and IBKR account details available through the existing reporting service
- IBKR financial-range definitions
- Google GenAI secret available to the API environment
- API virtual environment and `.env` loaded according to `agm-api/AGENTS.MD`

## Step-by-Step Workflow
1. The operator selects an inclusive date range through `SOW_MATCH_START_DATE` and `SOW_MATCH_END_DATE`; the default is current year to current date.
2. The script reads every database `contact_document` categorized as Source of Wealth, plus `account_contact` and `account` rows. These are three read-only database operations; the script does not read raw document data or write any table.
3. It reads completed OCR text only from the local extraction CSV and joins it to Source-of-Wealth rows by `contact_document_id`, falling back to `document_id`. It does not run OCR again.
4. It resolves each Source-of-Wealth row to an internal account through its direct `account_id`, or through `contact_id → account_contact.account_id`, then resolves the internal account to `ibkr_account_number`.
5. It loads the deposits/withdrawals report, IBKR account details, and financial ranges through existing read-only services.
6. It maps report rows to IBKR accounts using available account identifiers.
7. It deduplicates mapped report rows by IBKR account and transaction id. Rows without a transaction id use date, amount, currency, and description as a fallback identity. Raw-row and duplicate counts remain visible in the summary.
8. It counts unique mapped transactions with and without ID-linked local Source-of-Wealth OCR text.
9. Eligible accounts are ranked by largest absolute transaction and then latest transaction. The requested number is selected.
10. For each selected account, the script sends the IBKR profile, up to the 20 largest unique transactions, and locally retained OCR text to the configured structured-output AI model.
11. Before a model call, the script reuses a completed checkpoint only when its account, model, and SHA-256 of the full input payload match the current input. Failed, changed, or missing inputs are analyzed again.
12. The model compares identity, financial ranges, employment, declared wealth sources, and transaction plausibility. OCR is marked as untrusted documentary content, and every result is forced to manual review.
13. Detail CSV and JSONL files are checkpointed after each account. A final summary CSV records database linkage, population, duplicate, completion, and result-status counts.

## Outputs / Records Created
- `source_of_wealth_transaction_matches.csv`: account-level result and compact findings
- `source_of_wealth_transaction_matches.jsonl`: full model inputs and structured outputs, including OCR text
- `source_of_wealth_transaction_match_summary.csv`: population, deduplication, completion, and status counts
- Console progress and final counts
- Three read-only database table loads and no database writes

## Exception Paths / Failure Handling
- Missing local OCR file or no completed text: no accounts qualify for analysis.
- Reporting, IBKR-range, Drive, secret, or model initialization failure: the run fails visibly.
- Individual model failure: the error is checkpointed for that account and remaining accounts continue.
- Missing direct/account-contact relationship or missing IBKR account number: the document is not associated with an IBKR account and remains outside the transaction comparison population.
- Long OCR evidence is truncated at configured per-document and per-account limits and the truncation flag is retained.

## Controls / Verification Points
- Preventive control: database access is limited to reads of `contact_document`, `account_contact`, and `account`; there is no database write path.
- Population control: raw report rows, mapped rows, duplicate rows ignored, and unique transactions with and without local Source-of-Wealth evidence are separately reconciled.
- Linkage control: Source-of-Wealth-to-account association uses stored foreign-key relationships, and local OCR joins by contact-document or document identifier; names are not used for linkage.
- Evidence control: model inputs and structured outputs are retained together in JSONL.
- Semantic control: Source-of-Wealth evidence is not represented as proof of Source of Funds for an individual transaction.
- Review control: all outputs require manual review, including results labeled consistent.
- Resilience control: detail outputs are atomically checkpointed after every account and completed results resume only when the account, model, and complete input hash match.

## Evidence to Retain
- Local OCR extraction CSV used by the run
- Detail CSV, JSONL, and summary CSV from the same run
- Date range, environment overrides, and source report snapshots
- Any separately documented reviewer conclusion or follow-up

## Related Code / Pages / Routes
- Entry surface: `agm-api/source_of_wealth_transaction_analysis.py`
- Tests: `agm-api/tests/test_source_of_wealth_transaction_analysis.py`
- Upstream extraction: [Source of Wealth OCR Analysis](24-source-of-wealth-ocr-analysis.md)
- Related interactive workflow: [Deposits and Withdrawals Monitoring](17-deposits-and-withdrawals-monitoring.md)
- Downstream side effects: none

## Last Reviewed
- Status: draft
- Date: 2026-07-20
- Reviewer: Codex current-state implementation review
