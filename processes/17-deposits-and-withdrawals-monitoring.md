# Deposits and Withdrawals Monitoring

## Business Purpose
Provide an on-demand compliance review of IBKR deposits and withdrawals, flag high-value transactions and transactions above stored financial-profile ranges, expose Source-of-Wealth evidence, and allow reviewers to save freeform transaction comments.

## Trigger / Frequency
- Trigger: An authorized Dashboard user opens `Deposits & Withdrawals` or changes and submits the date range.
- Frequency: On demand. There is no scheduled monitoring or alert-generation job.

## Systems Involved
- `agm-dashboard`
- `agm-api`
- Google Drive deposits/withdrawals backups and IBKR details resource
- IBKR financial-range reference endpoint
- Supabase-backed accounts, contacts, documents, and `flagged_deposit` table

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Deposits/withdrawals Flex Query backups produced by the Clients ETL
- User-selected inclusive start and end datetimes; the page defaults to the current calendar month
- Internal account-to-IBKR account mapping
- Latest IBKR details and financial-range definitions
- Contact, account-contact, contact-document, and document-processing data for Source-of-Wealth evidence

## Step-by-Step Workflow
1. The page loads previously saved `flagged_deposit` rows, internal accounts, IBKR details, IBKR financial ranges, contacts, account-contact links, and contact-document metadata.
2. Saved comments are placed in a client-side map keyed only by transaction id. When multiple database rows share a transaction id, the last row returned wins in the displayed map.
3. The page requests deposits and withdrawals for the selected date range. The API selects the latest available backup file for each included year and month and then filters rows using the report's `Date/Time` value.
4. Each row is matched to an internal account and an IBKR account using the available account identifiers and aliases.
5. The displayed transaction direction is derived from the sign of `Amount`, while the displayed amount is absolute.
6. A transaction receives an `over_10k` flag when its signed amount is greater than USD 10,000. Negative withdrawals do not satisfy this rule because the comparison is not made on absolute value.
7. When IBKR financial information and range definitions are available, the signed amount is compared with the upper bounds for annual income, liquid net worth, and net worth. Exceeding a bound produces a separate mismatch flag.
8. The page compares stored application financial information with current IBKR financial ranges and displays mismatched fields in account details.
9. The page locates linked Source-of-Wealth documents, shows whether evidence is on file, displays extracted processing text when available, and permits loading the original stored document.
10. The reviewer can filter to flagged transactions, select a flag type, search by account identifier, and view summary counts.
11. Editing a comment calls `POST /flagged_deposits/create`, creating a new row with account id, transaction id, and comment. Clearing a comment also creates a new row.

## Date and Filter Definitions

- The initial queried range starts at 00:00:00 on the first day of the user's current local month and ends at 23:59:59.999 on the current local day.
- Editing the date inputs does not change the report until the user submits the query. Both dates are required, and start after end is rejected.
- `Account Search` is a case-insensitive substring match against `AccountAlias`, `ClientAccountID`, `AccountId`, `AccountID`, `account_id`, or `accountId` from the transaction row. It does not search contact name, transaction id, amount, or comment.
- `Flag View = Flagged only` requires at least one browser-derived flag.
- `Specific Flag` requires the selected code: `over_10k`, `income_mismatch`, `liquid_mismatch`, or `net_worth_mismatch`. The dropdown contains only flag types present somewhere in the currently loaded report before the other filters are applied.
- Search, flag view, and specific flag combine with logical AND.
- Summary totals are recalculated from the filtered population: total transactions, signed amounts above USD 10,000, any financial-range mismatch, any flag, and flagged rows with a nonblank saved comment.
- The table uses infinite scroll and has no built-in CSV export in the current component.

## Outputs / Records Created
- On-screen transaction flags and summary counts calculated in the browser
- Financial-profile comparison and Source-of-Wealth review context
- New `flagged_deposit` rows containing account id, transaction id, and freeform comment
- No persistent record of the calculated rule, threshold, amount, reviewer, status, or disposition

## Exception Paths / Failure Handling
- Invalid or reversed date ranges are rejected in the page; the API also rejects a reversed range.
- Missing Google Drive folders or files can return an empty report after logging the retrieval failure.
- Missing account mapping prevents a comment from being stored and produces only a browser console warning.
- Comment-save failures are written to the browser console without a failure toast.
- Source-of-Wealth original-document failures leave the preview unavailable.
- Flag calculations disappear when the page closes because only freeform comments are persisted.
- A user can select a date range and filters without leaving a durable record of the reviewed population or criteria.

## Controls / Verification Points
- Detective control: visible rules flag signed deposits above USD 10,000 and above the upper bounds of three financial categories.
- Detective control: reviewers can compare application and IBKR financial data and inspect Source-of-Wealth evidence.
- Detective control: saved comments can be reloaded by transaction id.
- Current limitation: the process is user-initiated and does not guarantee that every period or transaction population is reviewed.
- Current limitation: comments are append-only duplicate rows rather than a controlled finding lifecycle.

## Evidence to Retain
- Selected date range and latest monthly source files used by the report
- Screenshot or separately produced evidence of the reviewed population and exact date/filter criteria; this page has no built-in CSV export
- `flagged_deposit` comment rows
- Relevant Source-of-Wealth documents and processing output
- API and browser logs for load or save failures

## Related Code / Pages / Routes
- Entry surfaces: `agm-dashboard/src/app/(dashboard)/(services)/(tools)/(private)/(compliance)/deposits-withdrawals/page.tsx`
- Supporting modules: `agm-dashboard/src/components/dashboard/tools/private/reporting/deposits-withdrawals/DepositsWithdrawalsReport.tsx`, `agm-api/src/components/tools/public/reporting.py`
- Downstream side effects: `agm-api/src/app/clients/flagged_deposits.py`, `agm-api/src/components/clients/flagged_deposits.py`

## Last Reviewed
- Status: draft
- Date: 2026-07-15
- Reviewer: Codex current-state code review
