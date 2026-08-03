# Source of Wealth OCR, Account Analysis, and Transaction Analysis

## Business Purpose
Maintain one local Source-of-Wealth OCR/extraction history and generate two review-ready analysis datasets:

1. Account-level analysis: whether locally available ID, Source-of-Wealth, and Proof-of-Address OCR evidence supports the account profile, while keeping IBKR-vs-application financial/profile mismatch states as separate context.
2. Transaction-level analysis: whether flagged deposits/withdrawals have enough Source-of-Wealth/profile information for review, with Gemini analysis written to CSV only.

The workflow does not write database records and does not create an AML disposition or reviewer disposition.

## Trigger / Frequency
- Trigger: An authorized operator runs `account_audit.py` or `source_of_wealth_transaction_analysis.py` from `agm-api`.
- Frequency: On demand.

## Systems Involved
- `agm-api`
- Supabase Postgres `contact_document`, `document`, `account_contact`, and `account` tables
- Google Document AI OCR
- Local CSV outputs
- Google Drive deposits/withdrawals report files for transaction analysis
- IBKR details and financial ranges loaded through existing read-only services
- Gemini 3.5 Flash for pending flagged transaction rows by default; local-only mode is available with `SOW_RUN_GEMINI=false`
- Dashboard Accounts Audit page for live account/profile comparison fields and OCR-backed document-support checks sourced from `document_processing`

## Inputs / Prerequisites
- Source-of-Wealth `contact_document` rows
- Document bytes in the `document` table for OCR gaps
- Stored Source-of-Wealth and Proof-of-Identity document-to-account relationships through direct `contact_document.account_id` or `contact_document.contact_id -> account_contact.account_id`
- Local `id_document_ocr_extractions.csv`, `source_of_wealth_ocr_extractions.csv`, and `proof_of_address_ocr_extractions.csv`
- Account rows with application profile data where available
- IBKR account details and financial ranges
- Deposits/withdrawals report files for transaction analysis
- API virtual environment and `.env` loaded according to `agm-api/AGENTS.MD`

## Step-by-Step Workflow
1. `account_audit.py` loads existing `source_of_wealth_ocr_extractions.csv`.
2. It reads Source-of-Wealth `contact_document` rows and selects up to `SOW_TARGET_COUNT` rows not already present in the OCR CSV.
3. It OCRs only missing Source-of-Wealth documents, appending completed or failed attempts to `source_of_wealth_ocr_extractions.csv`.
4. It extracts heuristic Source-of-Wealth financial fields from OCR text, including income, assets, liabilities, equity, balances, source categories, extraction status, confidence, and review reasons.
5. It reads `contact_document`, `account_contact`, and `account`, then links completed OCR rows to accounts only through stored IDs. Names are not used for matching.
6. It loads IBKR details and financial ranges and writes `account_audit.csv`, one row per database account, including Account Focus-style financial/profile audit states, compact application/IBKR JSON, application and IBKR residence addresses, extracted Source-of-Wealth/ID/Proof-of-Address JSON, full completed OCR text, and profile-document support fields.
7. Account-level profile-document support compares local OCR documents to profile data. ID OCR is checked against linked contact and holder names. Source-of-Wealth OCR is checked against application/IBKR liquid-net-worth ranges, and declared liquid net worth is also checked against NAV. Application and IBKR residence addresses are compared with each other and checked against ID and Proof-of-Address OCR. POA is primary residence evidence; an ID address can support residence when no POA is available, while an ID containing no address is inconclusive rather than conflicting. A SOW document is `supported` only when extracted liquid-net-worth evidence supports a declared liquid-net-worth value, NAV does not exceed the declared liquid net worth, and no clear comparable financial conflict exists. CRC and USD amounts are compared after conversion using `SOW_CRC_PER_USD`, defaulting to `500` CRC per USD unless overridden by the operator; when an extracted amount has no field-level currency, the script can infer CRC or USD from document-wide currency context. Source categories are retained as context, but source-only support, unsupported currency pairs, or otherwise unsafe amount comparisons are `cannot_compare`; clear converted/comparable liquid-net-worth mismatches are `not_supported`. Bank or brokerage statement balances can support liquid net worth when they are high enough, but balances below the declared liquid-net-worth range are treated as insufficient evidence rather than contradictions. Missing required local evidence produces `missing_evidence`; unsafe comparisons produce `cannot_compare`; conflicting evidence produces `not_supported`; and all three ID, SOW, and address checks must support the profile for the combined row to be `supported`. IBKR/application financial and profile mismatch states remain separate columns and are not directly copied into this document-support state.
7a. Residence matching normalizes accents and punctuation and requires either a strong postal/street match or a composite of distinctive street/place tokens, locality, and matching house/apartment numbers. Placeholder all-zero postal codes are ignored. A nonmatching document produces `not_supported` only when its type or OCR text indicates address-bearing evidence such as a utility bill, bank statement, lease, tax/insurance record, or service invoice; marriage certificates and unclassified documents remain `cannot_compare` unless their OCR shows an address-evidence signal.
8. `source_of_wealth_transaction_analysis.py` reads the same OCR CSV and stored account/document relationships.
9. It loads deposits/withdrawals from `SOW_MATCH_START_DATE` through `SOW_MATCH_END_DATE`; default start date is `2025-07-01`.
10. It deduplicates transactions by IBKR account and transaction id, falling back to account/date/amount/currency/description where transaction id is missing.
11. It mirrors Dashboard flag logic for `over_10k`, `income_mismatch`, `liquid_mismatch`, and `net_worth_mismatch`.
12. It writes `source_of_wealth_transaction_analysis.csv`, one row per unique flagged transaction, including transaction details, profile financials, compact application/IBKR JSON, extracted Source-of-Wealth JSON, full Source-of-Wealth OCR text, and Gemini output columns.
13. Gemini is on by default and analyzes up to 10 pending flagged rows per run. Rows with completed Gemini decisions are preserved across reruns; rows without completed decisions remain pending. When `SOW_RUN_GEMINI=false`, the transaction script skips Gemini and still writes the same transaction analysis CSV with blank Gemini review columns for pending rows. Gemini output is capped at 500 tokens per review by default and can be adjusted with `SOW_GEMINI_MAX_OUTPUT_TOKENS`; thinking budget defaults to zero and can be adjusted with `SOW_GEMINI_THINKING_BUDGET`; batch size can be adjusted with `SOW_GEMINI_BATCH_SIZE`.

## Outputs / Records Created
- `source_of_wealth_ocr_extractions.csv`: cumulative local OCR and heuristic Source-of-Wealth extraction history
- `account_audit.csv`: account-level review dataset with profile-document support fields from local ID, Source-of-Wealth, and Proof-of-Address OCR
- `source_of_wealth_transaction_analysis.csv`: unique flagged transaction review dataset, with Gemini columns populated in batches for pending flagged rows when Gemini is enabled
- Console summaries
- No database writes

Auxiliary OCR summary/review CSVs are disabled by default and are written only when `SOW_WRITE_AUXILIARY_OCR_CSVS=true`.

## Exception Paths / Failure Handling
- Missing OCR CSV: the OCR script creates it as documents are processed.
- OCR failures are appended to the OCR CSV with `status=failed` and an error message.
- Document AI page-limit errors are retained as failed OCR rows.
- Accounts with Source-of-Wealth documents but no completed OCR remain visible through missing/zero completed-document counts.
- Transaction analysis can run without Gemini by setting `SOW_RUN_GEMINI=false`; Gemini errors populate `gemini_error` and do not write database records.
- Missing billing, depleted Gemini credits, quota errors, or malformed model output remain in the transaction analysis CSV.

## Controls / Verification Points
- Linkage control: OCR joins by `contact_document_id`, with `document_id` only as fallback.
- Matching control: Source-of-Wealth documents are linked to accounts only through `contact_document.account_id` or `account_contact`; names are not used.
- Population control: account analysis writes one row per database account.
- Account analysis control: financial/profile mismatch logic follows the Accounts Audit Account Focus pattern for IBKR-vs-application comparisons but is kept separate from OCR-vs-profile support. Source-of-Wealth document source categories remain metadata; category match alone is not enough to support a profile, and category mismatch alone is not used to fail profile support because statement evidence can support funds/liquidity without restating the declared source category.
- Frontend parity control: Accounts Audit exposes the same profile-document support field names and now derives them from live `document_processing` OCR text attached to linked documents. The Python account-audit output remains the fuller retained dataset, but the dashboard no longer hardcodes those fields to missing evidence.
- Deduplication control: transaction analysis writes one row per unique flagged transaction.
- Gemini execution control: Gemini runs only for pending flagged transaction rows, writes only to CSV, preserves completed decisions across reruns, and can be disabled with `SOW_RUN_GEMINI=false`.
- Gemini cost control: output tokens are capped by `SOW_GEMINI_MAX_OUTPUT_TOKENS`, defaulting to 500 per review, with `SOW_GEMINI_THINKING_BUDGET` defaulting to zero.
- Write control: no script writes to database tables.

## Evidence to Retain
- `source_of_wealth_ocr_extractions.csv`
- `account_audit.csv`
- `source_of_wealth_transaction_analysis.csv`
- Console summary showing counts and date range

## Related Code / Pages / Routes
- Entry surface: `agm-api/account_audit.py`
- Entry surface: `agm-api/source_of_wealth_transaction_analysis.py`
- Frontend surface: `agm-dashboard/src/components/dashboard/tools/private/reporting/accounts-audit/AccountsFocus.tsx`
- Upstream reference: [Source of Wealth OCR Analysis](24-source-of-wealth-ocr-analysis.md)
- Downstream side effects: none

## Last Reviewed
- Status: draft
- Date: 2026-07-20
- Reviewer: Codex current-state implementation review
