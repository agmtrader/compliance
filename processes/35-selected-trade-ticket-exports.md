# Selected Trade Ticket XLS Export

## Business Purpose
Produce an Improsa Excel workbook from selected broker executions, using the
original Improsa email, XLS template and Vestima PDF as the reference.

## Trigger / Frequency
- Trigger: authorized operator selects executions in an Improsa trade ticket.
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
1. Open Trade Tickets. The labeled shadcn Select automatically selects Improsa
   when available, otherwise the first saved ticket, and loads its executions. The selector remains above
   the table for switching between authorized tickets; a direct ticket link
   selects that ticket. Switching tickets clears the prior execution selection.
   Use the Asset type and Order type (All orders / Buy / Sell) shadcn Select
   filters above the table, then select the executions to export. Asset type
   choices come from the loaded executions. Both filters apply together and
   changing either clears the execution selection.
2. In Improsa, request Generate Excel File, the only action below its execution
   table. The button requires a selection and is disabled while generating.
   ACOBO instead exposes only Generate Trade Ticket for text preview and a
   separate email send within the review dialog (PROC-036); it has no Excel action.
3. API checks query ownership, re-reads source executions and resolves selected
   IDs in order. Client-provided replacement trade values are not used.
4. Validate required fields, finite values and approved mappings. Improsa uses
   normal XLS mode with strict contract validation. The API's existing
   review_sample=true capability remains available separately, with ownership/
   scope checks and no ticket-name or single-assignment restriction. Only review
   mode permits a blank contract cell; ACOBO no longer invokes it from the UI.
5. Generate a real XLS using the original 28-column formatting profile, including
   highlights/fills, fonts, widths, date formats and contract merge. Display the
   original quoted price in column K with four decimal places.
6. Download locally. Owner reviews and handles any recipient delivery separately.

## Outputs / Records Created
- Formatted improsa-trade-ticket.xls workbook. The separate API review mode
  retains improsa-review-sample.xls; it is not exposed by the ACOBO UI. No CSV output.
- No database write, trade submission or email from downloading.

## Exception Paths / Failure Handling
- Missing selection/required fields and unsupported mappings fail visibly.
  Missing contract fails normal XLS but remains blank in API review mode.
- CSV requests remain rejected. The blank-contract review behavior is carried
  forward to the replacement ACOBO XLS review path on 2026-10-15.
- Unauthorized query access fails before reading Flex data.
- Failed downloads retain selection and restore the action.

## Controls / Verification Points
- API authentication, route scopes, query ownership and ExecID selection.
- Original email and attachments are authoritative; do not substitute historical
  sample identifiers for a new execution or infer mappings from similar numbers.
- Preserve the independent original-format regression fingerprints except the
  owner-approved four-decimal quoted-price format in column K.
- USD maps to dolares; GBP remains GBP as approved; no silent FX conversion.
- For IBKR AssetClass BOND/BILL, column M uses abs(AccruedInterest) and column N
  uses ACOBO's established TotalAmount calculation:
  round(abs(NetCash) + abs(AccruedInterest), 2). Both paths share the same total
  helper. NetCash and AccruedInterest must be present and finite; explicit zero
  is valid. Never replace missing cash with a price-derived principal.
- Commission fields remain informational in column AB and are not added again
  to column N. NetCash supplies the existing ACOBO cash/fee treatment. Matching
  source execution reconciliation and recipient acceptance remain separate gates.
- Other asset classes retain abs(quantity * price) in column N and optional
  accrued-interest handling. AssetClass is required. Preserve the original
  quoted price value in column K while displaying four decimals.
- Recipient acceptance is separate from successful local download/testing.

## Remaining XLS Work
- API #87: authoritative contract/source mapping for the ACOBO sample; explicit
  policy for production contract values. ACOBO review may leave missing values
  blank; production Improsa XLS remains strict.
- API #87: the owner's October 22 bond workbook confirmed the earlier percentage
  scaling. The owner then authorized four price decimals and alignment of bond
  cash/accrual values with ACOBO, superseding clean principal in column N. Local
  XLS/Flask acceptance covers that implementation; check a newly generated actual
  application download against matching source executions before final acceptance.
- Dashboard #17: one action per ticket restored: ACOBO Generate Trade Ticket
  opens text preview/email review; Improsa Generate Excel File downloads selected
  executions. Both retain the table, filters and centered loader. Backend mapping and representative workbook acceptance remain
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
- Status: draft; four-decimal quote and ACOBO bond cash convention implemented locally;
  actual application download and contract/recipient acceptance remain pending.
- Date: 2026-10-22.
- Reviewer: Codex; owner retains manual delivery/acceptance.
