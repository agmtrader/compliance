# ACOBO Trade Confirmation Review

## Business Purpose
Generate an ACOBO confirmation message from selected executions for operator
review before a separate, explicit email action.

## Trigger / Frequency
On demand: the operator selects executions and clicks Generate Trade Ticket.

## Systems / Owner
AGM Dashboard and API; operator and technical owner: Andres Aguilar.

## Workflow and Controls
1. The authenticated Dashboard submits selected source rows and indices to
   POST /trade_tickets/generate. This existing generation path uses submitted
   rows; it does not use the separate ownership-checked export re-read workflow.
2. The API extracts coupon and maturity and preserves the existing aggregation
   of broker interest, quantity and cash. Generation itself sends no email.
3. For ISIN XS1046593908, the verified issuer terms specify annual March 19
   coupons and Actual/Actual. Count calendar days from the preceding scheduled
   March 19 through settlement, including leap days. Settlement on March 19
   starts the new period with zero accrued days. Do not infer days by dividing
   rounded interest or by assuming a fixed 360/365-day year.
4. Known-bond consolidation requires consistent ISIN identifiers and settlement
   dates. Missing/invalid dates, conflicting identities or settlement outside
   the verified issue-to-maturity schedule return a visible HTTP 400 error.
5. Securities without a verified schedule retain the pre-existing inferred
   360-basis calculation. Their conventions are not newly certified by this fix;
   add independently verified security terms before extending schedule support.
6. The Dashboard displays the generated message in its review dialog. Sending
   remains a separate operator action; viewing or testing must not send it.

## Outputs / Evidence
Confirmation preview; selected executions, settlement, issuer reference and
local generator/HTTP plus Dashboard review evidence. Preserve broker monetary
values; successful generation does not establish email delivery or settlement.

## Source / Related Code
- [Issuer final terms](https://www.sec.gov/Archives/edgar/data/101368/000119312514095596/d692806dfwp.htm).
- API components/tools/public/bond_accrual.py and trade_tickets.py.
- Dashboard AcoboEmailAction.tsx; API issue #295.
- PROC-035 documents the separate Improsa XLS/CSV download workflow.

## Last Reviewed
2026-10-15; draft. Formal ITGC transaction register is absent from this workspace;
PROC-036 is recorded in the existing working process register.
