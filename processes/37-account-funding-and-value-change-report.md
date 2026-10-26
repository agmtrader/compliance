# Account Funding and Value Change Report

## Business Purpose
Provide an internal account overview of opening value, deposits, withdrawals,
closing value and provisional cash-adjusted change. The default period is the
current calendar year in Costa Rica; an operator can request another period.

## Trigger / Frequency
On demand through Dashboard Reporting → Account Performance, or the funding and
performance link on an individual account's Statements section.

## Systems Involved
AGM Dashboard, AGM API and existing IBKR NAV/cash-flow backups in Google Drive.

## Roles / Owners
Owner: Andres Aguilar. The Dashboard route requires AGM administrator access;
the API independently requires the `all` scope.

## Inputs / Prerequisites
- Daily NAV CSV backups from Flex Query 734782, with account-level values,
  currencies and source report dates.
- Cash transaction CSV backups from query 794867, with signed amounts,
  currencies, FX rates, transaction IDs and detail-level rows.
- A valid period with start on/before end and no future dates.

## Step-by-Step Workflow
1. Open the report. It requests January 1 through today by default. Editing dates
   does not apply them until Load report is submitted.
2. The API selects the latest opening snapshot on/before the day preceding the
   requested start and the latest closing snapshot on/before the requested end.
   The closing snapshot must be within the selected period. Actual dates remain
   visible; snapshots more than seven days from a requested boundary prevent
   calculating change.
3. For a recent opening snapshot, cash-flow coverage begins the following day,
   which can precede the requested start across a non-reporting-day gap. For a
   stale or unavailable opening, cash activity starts at the requested start and
   change remains unavailable. Cash activity always stops at the closing NAV.
4. For each included month, select a single dated backup covering the entire
   needed interval. Do not infer missing days from a partial monthly backup.
   Missing coverage leaves cash totals and change unavailable. Duplicate source
   backups with the same preferred interval are an error.
5. Read source CSV values as strings, preserving transaction IDs and decimal
   amounts. Validate report dates, headers and account-level NAV. Do not add model
   and account-level NAV together or assume missing openings/closings are zero.
6. Filter cash transactions to the actual valuation period. Deduplicate by
   account and transaction ID, reject conflicting duplicate values, and convert
   signed amounts using the source FX rate into the account base currency.
   Same-currency rates must be one when supplied. Missing IDs, invalid amounts,
   unsupported transaction/model/detail categories and invalid FX prevent totals.
7. Calculate provisional cash-adjusted change as closing value minus opening
   value minus deposits plus withdrawals. Withdrawals display as positive outflow
   magnitudes. Base-currency changes prevent comparing amounts directly.
8. Display accounts found at either NAV boundary or in period cash activity.
   Search and currency filters apply to the table, currency-separated totals and
   CSV export. Each total states the number of accounts with known values.
9. Supporting details explain incomplete inputs and actual source dates. A source
   failure clears figures rather than showing old-period values under new dates.

## Outputs / Records Created
- A read-only response and Dashboard overview; no stored balances, broker
  instructions or account records are modified.
- Optional CSV of filtered accounts, with currency-labeled amounts, unavailable
  values, provisional/incomplete status, actual dates and source limitations.
- Source catalog listings have a process-local 30-second cache. Source values
  are read for each report. Responses are private and not cacheable.

## Exception Paths / Failure Handling
- Invalid/reversed/future dates: HTTP 400.
- No closing NAV inside the selected period: HTTP 422.
- Unreadable/ambiguous sources, missing columns, malformed rows or unidentifiable
  accounts/transaction dates: error response, never successful zero activity.
- Missing boundary values, currency changes, invalid or conflicting cash rows,
  or incomplete source intervals: affected totals/change remain unavailable with
  explicit reasons. Empty NAV files cannot certify account values.

## Controls / Verification Points
- API authentication and administrator scope are checked independently of UI.
- Source dates and selected coverage remain visible. An absent row is not proof
  of a zero opening, zero closing, or closed account.
- Different base currencies are never summed together. Partial totals explicitly
  state their included population; CSV values include their own currencies.
- **Securities transfers are unverified:** NAV and cash deposits/withdrawals do
  not establish in-kind transfer coverage. Cash-adjusted change may contain
  transferred securities and must remain provisional. It is not confirmed total
  investment P&L, a standalone realized/unrealized split, or a percentage return.
- Complete investment P&L requires transfer/Change-in-NAV source coverage. Year-
  to-date backup gaps require source recovery or a verified coverage source;
  do not loosen coverage rules to manufacture a number.
- Portfolio Analyst recreation, brokerage sessions, sending client reports,
  deployment and formal business acceptance are separate from this workflow.

## Evidence to Retain
Requested and actual dates; source names and coverage; per-account availability
reasons; selected-filter CSV when exported; local validation recorded in issues.

## Related Code / Pages / Routes
- API: `src/components/tools/public/account_performance.py`,
  `src/app/tools/public/reporting.py`, `GET /reporting/account_performance`.
- Dashboard: `/account-performance`,
  `src/components/dashboard/tools/private/reporting/account-performance/AccountPerformanceReport.tsx`,
  `src/lib/tools/account-performance.ts`, `src/utils/tools/account-performance.ts`.
- Tracking: agmtrader/agm-api#302 and agmtrader/agm-dashboard#49.

## Last Reviewed
- Status: draft; local implementation, not deployed or business accepted.
- Date: 2026-10-23.
- Reviewer: Codex implementation and source-data review.
