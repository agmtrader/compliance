# Accounts Metadata Review and Analysis

## Business Purpose
Provide an on-demand operational account list for reviewing NAV, status, advisor assignment, SLS devices, application state, temporal-email alignment, fee-template summary, and resolved contact labels. This page is not the Accounts Audit compliance process; that process is documented separately in [Accounts Audit Review](22-accounts-audit-review.md).

## Trigger / Frequency
- Trigger: A user opens the Dashboard `Accounts` page.
- Frequency: On demand. No review schedule or completion record is enforced.

## Systems Involved
- `agm-dashboard`
- `agm-api`
- Accounts-with-metadata API
- Advisors and contacts APIs

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- `ReadAccountsWithMetadata` response
- Advisor list keyed by advisor code
- Contact list and any embedded account-contact/contact data returned with accounts
- Derived metadata such as status, NAV, SLS devices, fee-template summary, client email, temporal email, IBKR account number, application JSON, and IBKR submission date

## Population and Display Derivation

1. The page loads accounts-with-metadata, advisors, and all contacts in parallel.
2. A non-array accounts response fails the load. Valid rows are sorted newest to oldest by the internal `created` string.
3. Missing alias, status, title, master account, SLS devices, client email, and fee-template summary are normalized to `-`; missing NAV is normalized to zero for this page.
4. The contact label uses the first available source in this order:
   - direct first/second contact-name fields, joined with `and` when both exist;
   - account-contact links resolved against embedded and global contacts;
   - direct embedded contact names when links are absent;
   - exact case-insensitive global contact email match to `client_email_address`;
   - exact case-insensitive global contact name match to account title;
   - `-` when no source resolves.
5. The grid displays account number, account title, contact name, IBKR username, fee-template summary, master-account classification, status, advisor, and rounded NAV.
6. Master account is displayed as `Broker` only when `master_account == I6413690`; every other value is displayed as `Advisor`.

## Filter Definitions

NAV, advisor, SLS device, and status combine with logical AND. The email/application interaction is an exception: when `Requires Email Change` is selected, the row predicate returns the email result immediately and never evaluates `Needs Application`. The DataTable also provides its own general table filtering and CSV export.

| Filter | Exact current rule |
|---|---|
| NAV | `All`; exactly 0; greater than 0 through 5,000; greater than 5,000 through 10,000; greater than 10,000 through 50,000; or greater than 50,000. Text NAV removes commas before parsing. Unparseable NAV is excluded from specific bands. |
| Advisor | `All` or exact string equality between selected advisor code and `account.advisor_code`. The UI includes `Unassigned`, but its value is the literal string `unassigned`; blank/null advisor codes are not normalized to that value, so the option currently does not return genuinely unassigned accounts. |
| SLS device | Dynamic values are created by splitting `sls_devices` on commas and trimming each value. Blank/null/`-` becomes `None`. A row matches when any normalized device equals the selected value. |
| Status | Dynamic exact value after blank/null/`-` normalization to `None`. Values are case-sensitive after trimming. |
| Application state | `Needs Application` includes only rows with no `ibkr_account_number`, no `date_sent_to_ibkr`, and a truthy `application_json`. |
| Email state | `Requires Email Change` includes only rows where both temporal email and client email are present after trim/lowercase normalization and the values are unequal. Rows missing either email are excluded. Selecting it bypasses the application-state filter because the predicate returns immediately. |

## Step-by-Step Workflow

1. Open `Accounts` and confirm the account, advisor, and contact requests completed; a load error leaves the page unavailable.
2. Select the intended population filters using the exact rules above.
3. Use the grid search/filter where a further text-based subset is needed.
4. Review unusual account status, zero/low NAV, missing devices, fee-template summary, contact label, application state, or email mismatch.
5. Use `View` to open the account/application in a new tab for follow-up.
6. If retaining review evidence, export `accounts.csv` and separately record the selected filters and review date because the export does not embed them.

## Outputs / Records Created
- Filtered browser view
- Optional `accounts.csv` export
- Navigation to account/application follow-up
- No persisted review assignment, exception, disposition, filter set, reviewer, or completion status

## Exception Paths / Failure Handling
- Any initial load failure shows `Failed to fetch accounts`; because the render requires both accounts and advisors, the list remains unavailable.
- Missing contact metadata triggers fallback matching; email/title fallback can associate a display label without an account-contact link and must not be treated as relationship evidence.
- The `Unassigned` advisor filter is currently defective and may show an empty population instead of unassigned accounts.
- `Requires Email Change` overrides/bypasses `Needs Application` instead of combining with it.
- Missing NAV is normalized to zero, so the zero-NAV filter cannot distinguish a confirmed zero from absent metadata.
- The master-account label treats every account other than `I6413690` as `Advisor`, including unknown or unexpected master-account values.

## Controls / Verification Points
- Detective control: explicit NAV, device, status, application, and email filters help identify operational exceptions.
- Detective control: CSV export can retain the resulting population when paired with a manual record of the filters used.
- Control limitation: the advisor-unassigned filter does not implement its label.
- Control limitation: display fallbacks and zero defaults can hide missing source data.
- Control limitation: this page is analysis only and does not prove that an exception was reviewed or corrected.

## Evidence to Retain
- `accounts.csv` plus selected filters and review timestamp
- Source account metadata response where needed for reconciliation
- Downstream account/application changes and their separate evidence

## Related Code / Pages / Routes
- Entry: `agm-dashboard/src/app/(dashboard)/(services)/(clients)/accounts/page.tsx`
- Page logic: `agm-dashboard/src/components/dashboard/clients/accounts/AccountsPage.tsx`
- Data client: `agm-dashboard/src/utils/clients/account.ts`
- Current compliance audit: [Accounts Audit Review](22-accounts-audit-review.md)

## Last Reviewed
- Status: draft
- Date: 2026-07-15
- Reviewer: Codex current-state code review
