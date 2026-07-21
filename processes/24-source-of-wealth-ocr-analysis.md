# Source of Wealth OCR Analysis

## Business Purpose
Create a manual-review dataset from documents categorized as `Source of Wealth`. The analysis records document-declared financial amounts and their stated basis, including net worth, liquid net worth, annual or monthly income, gross or net income, salary, revenue, profit, assets, liabilities, equity, account balances, and supported wealth-source categories. It does not approve the evidence, change customer financial profiles, or treat Source-of-Wealth evidence as transaction-specific Source-of-Funds evidence.

## Trigger / Frequency
- Trigger: An authorized operator runs the API analysis script when Source-of-Wealth evidence needs extraction or review.
- Frequency: On demand. The default is 10 previously unprocessed documents; an operator may set a positive `SOW_TARGET_COUNT` for an explicitly approved larger batch.

## Systems Involved
- `agm-api`
- Supabase Postgres tables `contact_document` and `document`
- Google Document AI Enterprise OCR
- Local filesystem for resumable CSV extraction, summary, and review output

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Database and Google Document AI access available to the API environment
- Existing `contact_document` rows whose category is `Source of Wealth`
- Linked `document` rows containing the stored base64 file payload and MIME type
- API virtual environment and `.env` loaded according to `agm-api/AGENTS.MD`

## Step-by-Step Workflow
1. The operator runs `account_audit.py` from `agm-api` after loading `.env`.
2. The script reads its existing extraction CSV and excludes already processed contact-document and document identifiers. Existing rows are not text-only reanalyzed because that would discard the positioned OCR evidence used for table extraction.
3. It selects the requested number of newest remaining documents categorized as Source of Wealth, defaulting to 10 and without selecting the same raw document more than once in a batch.
4. It reads and decodes each linked raw document, then calls the shared Google Document AI OCR pipeline with the same render settings and cache behavior used by the identity-expiration analysis.
5. It assesses OCR quality and classifies common evidence kinds such as income certifications, bank or brokerage statements, tax returns, payslips, financial statements, property-sale agreements, inheritance documents, and dividend resolutions.
6. It extracts labeled monetary evidence in English and Spanish, retaining the raw value, normalized amount, explicit currency or unresolved `$` symbol, period, field basis, line number, score, and compact evidence context.
7. It separately records wealth-source categories and any percentage shown on the same source line.
8. It selects primary values per field. `declared_wealth_amount` is populated only from an explicitly labeled net-worth or equity value. Total assets, a combined liabilities-plus-equity total, a bank-account balance, and low-value numbered disclosure notes weakly adjacent to terms such as Net Asset Value are not promoted to declared wealth. Retained or prior-period earnings, forecast amounts, date tokens, and narrative policy codes are not promoted to current income. Spanish `mil` and `millón` amount scales are normalized. A derived assets-minus-liabilities result is identified as derived, not declared.
9. It routes missing, conflicting, ambiguous-currency, statement-only, or low-quality results for higher-priority review. Every pilot result remains subject to manual validation.
10. It appends the detailed extraction CSV and rewrites the cumulative summary and review CSV. The script performs no database writes.

## Outputs / Records Created
- `source_of_wealth_ocr_extractions.csv`: resumable detailed OCR and extraction history, including OCR text
- `source_of_wealth_ocr_summary.csv`: cumulative counts and the current batch size
- `source_of_wealth_ocr_review.csv`: prioritized manual-review queue with recommended action
- Console progress and final counts
- No `contact_document`, `document`, financial-profile, or `document_processing` updates

## Exception Paths / Failure Handling
- Missing secret, database, or Google Document AI configuration: initialization fails and the batch does not start.
- Missing `document_id`, missing raw document, empty data, or invalid base64: the individual row is retained as a failed high-priority review item and processing continues.
- OCR/provider failure: the failed row and exact error are retained; the script does not conceal the failure with another provider.
- No supported labeled amount: the result is `not_found` and requires inspection; currency-bearing unlabeled amounts remain visible as low-confidence evidence.
- Multiple materially different high-confidence values for the same field, currency, and period: the result is `conflict` and no value is approved automatically.
- A bare `$` remains currency `$` and receives an ambiguity reason; the script does not silently convert it to USD.

## Controls / Verification Points
- Preventive control: batch size defaults to 10, a positive explicit override is required for a larger run, and already processed identifiers are excluded using the retained detail CSV.
- Preventive control: there is no database write path.
- Semantic control: Source of Wealth is accumulated-wealth evidence; the output is not a Source-of-Funds decision for a deposit or withdrawal.
- Semantic control: bank or brokerage balances are not promoted to income; total assets, combined liabilities-plus-equity totals, and low-value numbered disclosure-note proximity matches are not represented as net worth; retained or prior-period earnings, forecast values, dates, and narrative policy codes are not represented as current income.
- Evidence control: every normalized monetary value retains its raw OCR value, currency, basis, period, score, line number, and compact context.
- Detective control: missing values, competing field candidates, ambiguous dollar symbols, statement-only evidence, derived values, and OCR-quality limitations are explicit review reasons.
- Reconciliation control: the summary records selected, completed, failed, OCR-quality, extraction-status, declared-wealth, income, and assets-plus-liabilities counts; database-write counts remain zero.
- Review control: all pilot rows require manual validation before their values are used in a customer profile or compliance decision.

## Evidence to Retain
- Detailed extraction CSV for the reviewed batch history
- Matching summary and prioritized review CSV
- Operator review decisions outside this script
- Any separately approved customer-profile correction or compliance decision and its supporting evidence

## Related Code / Pages / Routes
- Entry surface: `agm-api/account_audit.py`
- Shared OCR: `agm-api/src/components/clients/document_processing.py`
- Source-of-Wealth declarations: `agm-hub/src/lib/clients/schemas/application.ts` and `agm-hub/src/lib/clients/application.ts`
- Operational consumer: [Deposits and Withdrawals Monitoring](17-deposits-and-withdrawals-monitoring.md)
- Related control gap: `GAP-036 - Source-of-Funds and Funding-Party Verification`
- Downstream side effects: none

## Last Reviewed
- Status: draft
- Date: 2026-07-17
- Reviewer: Codex current-state implementation review
