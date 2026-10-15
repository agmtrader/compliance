# Selected Trade Ticket XLS Export

## Business Purpose
Produce an Improsa Excel workbook from selected broker executions, using the
original Improsa email, XLS template and Vestima PDF as the reference.

## Trigger / Frequency
- Trigger: authorized operator selects executions in an Improsa or ACOBO trade ticket.
- Frequency: on demand; no scheduled export or delivery.

## Systems Involved
- AGM Dashboard, AGM API, IBKR Flex Query.

## Roles / Owners
- Primary owner and recipient-review coordinator: Andres Aguilar.

## Inputs / Prerequisites
- Authenticated user with trade-ticket scope and query ownership, or existing
  administrator access.
- Selected execution IDs from the freshly read authorized query.
- Required approved source values. Normal Improsa XLS requires IBOrderID.
  Review samples may leave it blank; account/instrument IDs are never
  substituted. Any ticket the caller is authorized to read may be used for review.

## Step-by-Step Workflow
1. Open the saved Improsa or ACOBO ticket and select executions.
2. In Improsa request Generate Excel File; in ACOBO use the separate Download
   Improsa Excel (.xls) button. ACOBO retains its existing review/email flow.
3. API checks query ownership, re-reads source executions and resolves selected
   IDs in order. Client-provided replacement trade values are not used.
4. Validate required fields, finite values and approved mappings. ACOBO requests
   review_sample=true; the server enforces query ownership/scopes, with no
   ticket-name or single-assignment restriction. Only review mode permits a blank
   contract cell. Shared queries may have multiple authorized assignment rows.
5. Generate a real XLS using the original 28-column formatting profile, including
   highlights/fills, fonts, widths, number/date formats and contract merge.
6. Download locally. Owner reviews and handles any recipient delivery separately.

## Outputs / Records Created
- Formatted XLS workbook; ACOBO review uses improsa-review-sample.xls so it is
  distinguishable from the normal improsa-trade-ticket.xls. No CSV output.
- No database write, trade submission or email from downloading.

## Exception Paths / Failure Handling
- Missing selection/required fields and unsupported mappings fail visibly.
  Missing contract fails normal XLS but remains blank in ACOBO review XLS.
- CSV requests remain rejected. The blank-contract review behavior is carried
  forward to the replacement ACOBO XLS review path on 2026-10-15.
- Unauthorized query access fails before reading Flex data.
- Failed downloads retain selection and restore the action.

## Controls / Verification Points
- API authentication, route scopes, query ownership and ExecID selection.
- Original email and attachments are authoritative; do not substitute historical
  sample identifiers for a new execution or infer mappings from similar numbers.
- Preserve the independent original-format regression fingerprints.
- USD maps to dolares; GBP remains GBP as approved; no silent FX conversion.
- Current gross-value code uses abs(quantity * price). The mutual-fund reference
  does not establish bond percentage-price scaling or accrued/fee treatment;
  resolve that gap in API #87 before approving a bond sample.
- Recipient acceptance is separate from successful local download/testing.

## Remaining XLS Work
- API #87: authoritative contract/source mapping for the ACOBO sample; explicit
  policy for production contract values. ACOBO review may leave missing values
  blank; production Improsa XLS remains strict.
- API #87: bond trade amount versus settlement amount, accrued interest and fees,
  supported asset-class pricing units and meaningful amount regressions.
- Dashboard #17: ACOBO now has the same styled XLS exporter via a separate
  download button. Backend mapping and representative workbook acceptance remain
  in API #87; clicking the button does not relax validation or send email.
- Verify representative rendered Excel highlights/formats and recipient feedback.

## Evidence to Retain
- Original template/confirmation provenance and selected source execution IDs.
- Workbook/reference comparisons, authorization/amount tests and recipient review.
- Keep private sample records in approved locations, not public issues.

## Related Code / Pages / Routes
- Dashboard TradeTicketModule; utils/tools/trade-tickets.ts.
- POST /trade_tickets/export_improsa; components/tools/public/trade_tickets.py.
- API #87 and Dashboard #17.

## Last Reviewed
- Status: draft; existing XLS retained, remaining mapping/acceptance work open.
- Date: 2026-10-15.
- Reviewer: Codex; owner retains manual delivery/acceptance.
