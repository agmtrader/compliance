# Daily Trade Confirmations Report

## Business Purpose
Show daily executions and commissions across IBKR accounts without opening each account separately in TWS.

## Trigger / Frequency
- Trigger: An authorized Dashboard user opens or refreshes Trade Confirmations in Reporting.
- Frequency: On demand. The available period is controlled by the IBKR Flex Query configuration.

## Systems Involved
- `agm-api`
- `agm-dashboard`
- IBKR Flex Query `1215183`

## Roles / Owners
- Primary owner: Andres Aguilar
- Requester and operational reviewer: Cristian Ramírez
- Backup owner: To be confirmed

## Inputs / Prerequisites
- An authenticated AGM user with access to the Reporting route.
- A working IBKR Flex Query token and query `1215183` configured to include all intended accounts.
- Execution rows with order, account, trade date, commission, amount, and currency fields.

## Step-by-Step Workflow
1. The user opens or refreshes the Dashboard Trade Confirmations report.
2. The API requests fixed Flex Query `1215183` from IBKR on demand. The caller cannot select another query ID.
3. The API validates required columns and numeric values, then retains `EXECUTION` rows.
4. It groups rows by trade date, order ID, API-order flag, asset class, symbol, client account ID, account alias, advisor, order type, amount currency, and commission currency.
5. Each group's commission is summed with its original sign. Each execution's `Amount` is converted to its absolute value before summing.
6. The Dashboard displays source trade dates, execution count, currency-separated totals, grouped rows, and an export of those grouped rows.

## Outputs / Records Created
- Read-only report and optional CSV downloaded by the user.
- No trade, commission, database, or source-file records are written by this workflow.

## Exception Paths / Failure Handling
- Missing required columns or invalid numeric values cause an explicit API error; the page displays it.
- An empty query result displays an empty state.
- IBKR access or generation failures prevent report loading and are not replaced with zero totals.
- `Advisor` is unavailable in the supplied 2026-10-01 query sample and is displayed as unavailable. No account-alias inference is made.

## Controls / Verification Points
- Fixed query ID prevents a caller from retrieving arbitrary Flex Queries through this route.
- Only execution rows enter the calculation.
- Amount and commission totals remain separate by their respective currencies.
- Operational review should compare the displayed execution count and currency totals with the IBKR source for the same trade date before using the result as a commission control.
- This report shows IBKR's `Commission`; it does not calculate AGM revenue, advisor compensation, or billing allocations.

## Evidence to Retain
- The IBKR Flex Query source CSV for the reviewed date.
- The exported grouped report and reviewer notes or screenshot when used for an operational decision.
- API error logs if a query or validation fails.

## Related Code / Pages / Routes
- Entry surfaces: Dashboard `/trade-confirmations`; API `GET /reporting/trade_confirmations`.
- Supporting modules: `agm-api/src/components/tools/public/trade_confirmations.py`; `agm-dashboard/src/components/dashboard/tools/private/reporting/trade-confirmations/TradeConfirmationsReport.tsx`.
- Downstream side effects: none.

## Last Reviewed
- Status: draft
- Date: 2026-10-01
- Reviewer: Codex implementation review; business owner review pending
