# Bank and IBKR Accounting Statement Controls

## Business Purpose
Meet Cristian Ramírez's July 31, 2026 accounting request: preserve monthly original PDF and raw CSV statements, establish the required account/month population, compare transaction totals with independent statement totals, and download original evidence for an annual audit period.

## Trigger / Frequency
- Trigger: An authorized AGM administrator opens Accounting Statements in Dashboard Reporting, refreshes sources, uploads an original, updates the account register, reviews a duplicate, or requests a download.
- Frequency: On demand. Completed calendar months are due; the current and future months are not yet due.
- Reporting period: The selected calendar year. The request does not specify a single historical retention start date. Opening and closing months are included, even when partial.

## Systems Involved
- `agm-api`: Python/Flask, Google Drive API, deterministic CSV/XLS/XLSX adapters, Decimal arithmetic, PDFium text extraction, bounded local Tesseract OCR for scanned summaries, isolated PDF worker processes, and ZIP packaging.
- `agm-dashboard`: Next.js, authenticated server-side API access, source inventory, progressive month analysis, exception review, uploads and downloads.
- Google Drive originals under the two roots identified in the email. Original files are not rewritten or deleted by this workflow.
- Google Drive is the authorized accounting data store, as confirmed by the requester on October 14, 2026. Analysis is generated from source originals; separate SQL summary/transaction tables are not required. Account-register revisions and upload integrity receipts are preserved as separate JSON records in Drive.

## Roles / Owners
- Primary technical owner: Andrés Aguilar.
- Operational account-register owner and reviewer: Cristian Ramírez or an authorized AGM operator.
- Backup owner: To be confirmed by AGM.
- Account settings, source selection, uploads and downloads require verified API `all` scope. Dashboard navigation alone is not an authorization control.

## Inputs / Prerequisites
- The seed register transcribes all 16 accounts from the supplied email screenshot, including open and closed accounts. Each account specifies institution, owner, opening/closing dates, account identifier, source identifier, currency, suffix and its controlled Drive folder.
- Seven closed accounts in the email have no closing date. Mercury’s stored bank confirmation explicitly closes account 9801451222 effective June 21, 2025; that date and its Drive evidence ID are recorded. On October 14, 2026 the requester instructed that the unavailable closing-date cutoff be ignored for the remaining six accounts. Their dates stay null; completed months from opening through the selected year remain in coverage without an unresolved-date blocker. A subsequently supplied actual closing date provides the normal cutoff.
- The requester confirmed Truist account 1100026505060 on October 14, 2026. That number is used for account identity; the email's 110026505060 is retained as provenance and no longer blocks identity checks.
- Original monthly PDF and CSV files. BAC XLS/XLSX exports are readable for transaction analysis, but do not satisfy the email's CSV-backup requirement. Conversions must not replace the bank's originals.
- Institution formats must match a supported deterministic adapter. Unsupported or unreadable originals remain exceptions; no model infers balances or transactions.

## Step-by-Step Workflow
1. Authenticate the administrator and enforce API scope before any Drive access.
2. Read the seed register or the unique latest immutable register revision. Reject stale saves. Concurrent sibling revisions cause a visible conflict; no edit is silently discarded.
3. Inventory controlled account folders, year folders and original ZIP members. Bound traversal, file size and archive expansion. Do not treat editable Google documents with `.CSV` names as preserved original CSV files.
4. For each account, produce every month in the selected year. A completed month intersecting the opening/closing dates is required. Months outside the account lifecycle or not yet completed are not due. Unknown lifecycle dates produce unresolved requirements unless an explicit approved policy omits an unavailable closing-date cutoff. The six accounts covered by the October 14 instruction have that policy recorded; no closing date is inferred.
5. Inventory PDF, CSV and alternative raw sources separately. A wholly absent month remains in the required denominator. A bank file with CSV MIME type and a misleading PDF filename is classified as CSV only after its transaction contents validate; its original name and bytes remain unchanged. A multi-month export can supply analysis but does not satisfy monthly CSV backup. Duplicate originals need an explicit retained source selection; they are not silently replaced or removed.
6. Load independent month detail progressively. Validate statement account, closing period and currency where supplied. IBKR uses the Base Currency Summary in `Statement of Funds`, with opening balance and closing-row reported debit/credit totals. Separate Paxos sections cannot override the primary account ledger or NAV; consolidated layouts retain the source account and use their own declared header. NAV is kept separate from cash.
7. For banks, extract account summary totals from the original PDF and transactions from the original raw export. BAC uses the first-page account summary: its detail closing-credit column includes opening cash and must not be treated as monthly credits. Scanned summary pages use local OCR; independent raw totals must still agree. OCR status is visible to the reviewer. The legacy Terrabank service-charge field can contain a doubled decimal point; the parser canonicalizes only that punctuation, retains the original and parsed fragments, and still requires all independent financial checks to pass. Transactions retain source row, activity/posted dates, signed amount, reference and code where supplied. A Terrabank PDF whose printed beginning date is later than the required month start (or actual account opening date in its opening month) is explicitly partial-period evidence. It cannot certify full-month backup or reconciliation, even if its own partial totals agree.
8. Independently calculate transaction debit and credit totals. Calculate four differences: transaction debits minus reported debits, transaction credits minus reported credits, statement opening plus reported credits minus reported debits minus closing, and statement opening plus transaction credits minus transaction debits minus closing. Each absolute difference must be strictly below 0.01 in the configured account currency. Do not combine currencies.
9. Compare current opening cash with the preceding month's closing cash, including the prior year's December for January. A missing or ambiguous predecessor is unavailable, not a successful continuity check. The account's opening month does not require a predecessor.
10. Keep backup completeness and data reconciliation independent, as the email requests. A month’s data reconciles when its independent totals, continuity, account configuration and source-selection controls pass. A BAC XLS can pass financial checks while its required CSV backup remains missing. The coverage grid turns green only when both backups and reconciliation pass. Show pending checks and failures explicitly.
11. Operators resolve missing originals through Upload Originals, correct account metadata through Account Settings, or choose a duplicate source explicitly. Upload validation occurs before creation; existing originals are never overwritten. Preserve the original bytes and record account/month, source name, Drive ID, actor, timestamp, size and SHA-256 in a separate receipt.
12. Download an original only after resolving it within the authorized account/month inventory. Annual downloads include observed originals and a JSON manifest with source IDs, names, modification timestamps, sizes, hashes, required-month inventory and coverage exceptions. A partial bundle remains visibly incomplete in its manifest and receives an `-incomplete.zip` filename. Originals that cannot be downloaded are itemized in the manifest instead of being silently omitted.

## Outputs / Records Created
- Required-month inventory and exception detail.
- Two business conclusions: whether all required statements are backed up, and whether all required monthly data has reconciled. Both remain false while lifecycle requirements or checks are unresolved.
- Immutable account-register revisions and original-upload integrity receipts in Drive.
- Original-file downloads and annual ZIP/manifest downloaded by the administrator.
- Normalized transaction rows with nullable reference/code fields. `Not supplied` means the original lacked that field; it is not an inferred or synthesized value.

## Exception Paths / Failure Handling
- Missing lifecycle dates: retain unresolved coverage unless the account has the explicit approved unknown-closing-date policy. For the six approved accounts, retain the null date and omit its cutoff; do not infer closure from the last source file.
- Incorrect account identity, currency or closing period: reject analysis/upload; do not use filename evidence alone to override source content.
- Missing monthly PDF or CSV, editable-document substitutes, multi-month exports or duplicate originals: do not count a valid monthly backup until resolved.
- XLS-only BAC month: permit analysis and original download, show missing required CSV.
- Malformed values, non-finite amounts, unsupported headers, unreadable/scanned source or local OCR unavailable: explicit month exception; never convert missing values to zero.
- Source/provider failure: show failed account/month reads without suppressing other accounts. No automatic overwrite, duplicate resolution, or source deletion.
- Upload receipt failure after file creation: report that the original exists and its receipt failed. Do not replay the upload; inspect the existing source and retain the integrity receipt through the technical owner.
- Conflicting account-register revisions: preserve both revisions, stop register use/saves and have the technical owner reconcile their changes. No arbitrary latest-timestamp selection is permitted.
- File/download limits: original limit 25 MB; annual bundle limit 128 MB and 500 files. Select one account when needed. Archive expansion is bounded.

## Controls / Verification Points
- Preventive: server-side authenticated `all` scope, fixed source roots, managed folder checks on configuration changes, historical account retention, source-account and closing-period validation, bounded uploads/downloads and no source-overwrite/delete operations.
- Detective: expected account/month denominator, explicit duplicate and missing evidence states, separate native/OCR extraction provenance, four independent financial checks, prior-month continuity, and explicit currency.
- Integrity: preserve original bytes; upload receipts and annual manifests retain SHA-256 and source metadata. These prove the bytes recorded at upload/download; they are not a certification that historical Drive files have never changed.
- Human review: compare significant exceptions and OCR summaries with the original PDF. Truist identity is confirmed and the six unavailable closing-date cutoffs are omitted by the requester's explicit policy. Coverage conclusions apply to that documented policy.
- Caching: bounded in-process caches keyed by file identity/modified time and statement identity. Refresh clears them. Errors are not cached as successful values.

## Evidence to Retain
- Source PDF and raw CSV; retain original XLS/XLSX and archive containers where supplied by a bank.
- Immutable register revisions, source selections and upload receipts.
- Annual bundle manifest and originals used for review, plus reviewer dispositions in the established business record.
- Validation evidence on coordinating API issue #98; authenticated deployment/business acceptance separately from local checks.
- The application does not currently certify a monthly reviewer disposition. Using the report is not proof an operational reviewer accepted every exception.

## Related Code / Pages / Routes
- Dashboard: `/accounting-statements`.
- API: `GET /reporting/accounting_statements`, `GET /reporting/accounting_statements/detail`, `GET/POST /reporting/accounting_statements/accounts`, `POST /reporting/accounting_statements/upload`, `POST /reporting/accounting_statements/download`, `POST /reporting/accounting_statements/annual_download`.
- Supporting modules: `agm-api/src/components/tools/public/accounting_statements.py`, `accounting_parsers.py`, `agm-api/src/config/accounting_accounts.json`, `agm-dashboard/src/components/dashboard/tools/private/reporting/accounting-statements/AccountingStatementsReport.tsx`.
- Legacy IBKR summary route also now requires independent reported debit/credit totals before returning `cash_reconciled=true`.
- Downstream side effects: original uploads and separate account/receipt records in the existing controlled Drive roots. No email is sent and no historical original is modified.

## Last Reviewed
- Status: draft.
- Date: 2026-10-14.
- Reviewer: Codex implementation and source review; AGM operational metadata and target-environment acceptance remain pending.
