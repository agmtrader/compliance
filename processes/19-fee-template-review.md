# Fee Template Review

## Business Purpose
Provide an on-demand account population view for inspecting IBKR fee-template assignments alongside account status, NAV, advisor, devices, email alignment, and application availability.

## Trigger / Frequency
- Trigger: An authorized Dashboard user opens `Fee Template Review`.
- Frequency: On demand.

## Systems Involved
- `agm-dashboard`
- `agm-api`
- Accounts-with-metadata API
- Daily IBKR details reporting resource

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Internal accounts-with-metadata response
- Latest available IBKR details resource
- Fee-template data in the IBKR account detail or the internal fallback summary

## Step-by-Step Workflow
1. The page loads internal accounts with metadata and the IBKR details resource in parallel.
2. IBKR details are indexed by IBKR account id and joined to internal accounts by `ibkr_account_number`.
3. For each account, the page prefers the IBKR fee-template object and parses it into a summary. If no object is available, it displays the internal `fee_template_summary` fallback.
4. Rows are sorted from newest to oldest by the internal account creation timestamp.
5. The reviewer can filter by NAV range, advisor, SLS device, account status, application state, and temporal-email/client-email comparison.
6. The `Requires Email Change` option includes rows where both emails exist and do not match.
7. The `Needs Application` option uses the current client-side condition `!ibkr_account_number && has_application_json`.
8. Opening a row's fee-template action displays the parsed IBKR fee-template details in a dialog.
9. The filtered population can be exported as `fee-template-review.csv`.

## Outputs / Records Created
- Filtered, read-only fee-template review grid
- Fee-template detail dialog
- Operator-generated CSV export
- No persisted review result, exception, assignment, or approval

## Exception Paths / Failure Handling
- The page has no explicit try/catch around its initial parallel load; a rejected request can leave the page in its loading state and surface through client error handling.
- Missing fee-template data falls back to the internal summary or `-`.
- Join failures caused by missing or inconsistent IBKR account numbers leave IBKR fee-template details unavailable.
- The `Needs Application` condition is counterintuitive and may not represent the intended review population.

## Controls / Verification Points
- Detective control: filters can surface missing metadata, email differences, unusual NAV/account states, and fee-template assignments.
- Detective control: the CSV export can preserve the filtered population reviewed by an operator.
- Current limitation: the application does not require review coverage, record a disposition, or reconcile the population against an approved fee schedule.

## Evidence to Retain
- CSV export or screenshot retained by the reviewer
- Accounts-with-metadata and IBKR details snapshots supporting the review
- Follow-up evidence from any separate account change process

## Related Code / Pages / Routes
- Entry surfaces: `agm-dashboard/src/app/(dashboard)/(services)/(tools)/(private)/(reporting)/fee-template-review/page.tsx`
- Supporting modules: `agm-dashboard/src/components/dashboard/tools/private/reporting/fee-template-review/FeeTemplateReviewReport.tsx`, `agm-dashboard/src/utils/clients/fee-templates.ts`
- Downstream side effects: none; the current page is read-only

## Last Reviewed
- Status: draft
- Date: 2026-07-15
- Reviewer: Codex current-state code review
