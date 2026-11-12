# Bank and IBKR Accounting Statement Controls

## Purpose and frequency
On demand, authorized staff review original monthly statements, backup coverage and reconciliation for a selected year and account. Dashboard calls this page **Statements** (`/statements`); the older `/accounting-statements` route is retained for existing links.

## Sources and access
The API's `/reporting/accounting_statements` routes require `all` scope. The service reads the account register and controlled Google Drive originals. The register is revisioned; concurrent or stale edits are rejected. Staff can update account settings and upload originals through authorized routes. These are real Drive writes and are not performed merely by opening or reviewing the report.

## Review
1. Select the year and accounts; inventory loading and failures are shown per account. Completed months in an account's lifecycle are evaluated, while current and future months are not yet due.
2. Review pending and completed months. Backup completeness and financial reconciliation are separate: a month requires both to be audit-ready. Missing, ambiguous, partial, unreadable or unsupported originals remain exceptions.
3. Open a month to review original sources, transactions, statement totals and reconciliation, including continuity to the prior month where applicable. The API uses deterministic institution-specific parsers and may use local OCR for scanned bank summaries. It does not infer missing financial values.
4. An authorized operator can upload a missing original, select a source where applicable, refresh and repeat the review. Uploads retain original bytes and integrity receipts. Download individual originals or an annual ZIP with a manifest; an incomplete bundle is labeled accordingly.
5. Record the review and unresolved items in the existing operational case record. An application status is not an independent auditor's approval, and this process does not automatically send originals to auditors.

## Failure and evidence
Report/source failures and unresolved account periods remain visible; missing inputs are not treated as zero or successful reconciliation. Preserve the source originals, selected account and period, review outcome, exceptions and owner. The annual manifest identifies included files and incomplete coverage; it is not a certification of historical immutability.

## Source and verification status
Reviewed against the API accounting-statement routes and service, and Dashboard monthly-review component, on November 12, 2026. This describes source behavior, not an independently verified deployment or an authenticated financial reconciliation. Historical account-specific decisions and approvals must be checked in their original records before use.
