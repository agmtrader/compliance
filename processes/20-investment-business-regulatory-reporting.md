# Investment Business Regulatory Reporting

## Business Purpose
Display a point-in-time set of investment-business metrics for open advisory and brokerage accounts, including NAV, account counts, geographic distribution, and 2025 managed-account trading activity.

## Trigger / Frequency
- Trigger: An authorized Dashboard user opens `Investment Business Returns`.
- Frequency: On demand. The current page is fixed to 2025 trade data and labels the output as of December 31, 2025.

## Systems Involved
- `agm-dashboard`
- `agm-api`
- Internal accounts, contacts, and account-contact data
- Clients, NAV, IBKR details, and trades reporting resources

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Internal account, contact, and account-contact records
- Clients report used to identify accounts whose status is `Open`
- NAV report
- Latest IBKR details backup
- Trades reports for every month of 2025

## Step-by-Step Workflow
1. The page loads accounts, NAV, contacts, account-contact links, clients report, IBKR details, and all twelve 2025 trade months in parallel.
2. It identifies open IBKR account numbers from the Clients report and filters internal accounts and trades to that population.
3. It classifies open accounts as advisory when `master_account` is `F10740574` and brokerage when it is `I6413690`. Other master accounts are not included in either category.
4. Country is taken from the first IBKR associated person when available, otherwise from the first linked contact, otherwise it defaults to `Costa Rica`.
5. NAV is joined by IBKR account number from the NAV report and defaults to zero when no row is found.
6. The page calculates advisory and brokerage NAV, account counts, total NAV, and country distribution.
7. Trade metrics keep rows whose `LevelOfDetail` is `EXECUTION`. Total trade money sums all execution rows.
8. Unique orders use `IBOrderID` when nonzero and fall back to `IBExecID`; rows sharing an order key are combined.
9. Average trades per client is based on unique orders for accounts represented in those orders. Average trade amount is based on combined unique orders.
10. The page displays debug counts for source rows, execution rows, unique orders, and unique accounts.

## Outputs / Records Created
- Read-only investment-business metrics and country table
- Visible list intended to identify accounts with unknown country
- No database record, certified report file, review decision, or submission record

## Exception Paths / Failure Handling
- The parallel data load has no explicit error handling; a rejected request can leave the page loading or surface through client error handling.
- Missing NAV values become zero.
- Missing country values become `Costa Rica`, so the separate `Unknown Country` population cannot identify those missing values.
- Accounts outside the two hardcoded master accounts are excluded from advisory/brokerage totals.
- The reporting period and page date are hardcoded to 2025.

## Controls / Verification Points
- Detective control: the page restricts account metrics to accounts marked open in the Clients report.
- Detective control: it shows account and NAV totals by business category and country.
- Detective control: it shows debug record counts for manual reasonableness review.
- Current limitation: no source snapshot ids, run timestamp, preparer, reviewer, certification, reconciliation, or submission evidence is retained.

## Evidence to Retain
- Source Clients, NAV, IBKR details, and 2025 trades files
- Screenshot or separately created report retained by the operator
- Any external filing or review evidence maintained outside the application

## Related Code / Pages / Routes
- Entry surfaces: `agm-dashboard/src/app/(dashboard)/(services)/(tools)/(private)/(compliance)/investment-business/page.tsx`
- Supporting modules: `agm-dashboard/src/components/dashboard/tools/private/reporting/investment-business/InvestmentBusiness.tsx`, reporting API utilities
- Downstream side effects: none; the current page is read-only

## Last Reviewed
- Status: draft
- Date: 2026-07-15
- Reviewer: Codex current-state code review
