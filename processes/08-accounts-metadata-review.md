# Accounts Metadata Review and Analysis

## Business Purpose
Provide an on-demand operational account list for reviewing NAV, status, advisor assignment, SLS devices, application state, temporal-email alignment, and resolved contact labels. This page is not the Accounts Audit compliance process; that process is documented separately in [Accounts Audit Review](22-accounts-audit-review.md).

## Trigger / Frequency
- Trigger: A user opens the Dashboard `Accounts` page.
- Frequency: On demand. No review schedule or completion record is enforced.

## Systems Involved
- `agm-dashboard`
- `agm-api`
- Full accounts-read API and separate reporting APIs
- Advisors and contacts APIs

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- One full `/accounts/read` request alongside account-contact links, contacts and advisors
- Advisor list keyed by advisor code
- Contact list and any embedded account-contact/contact data returned with accounts
- Derived metadata such as status, NAV, SLS devices, fee-template summary, client email, temporal email, IBKR account number, application presence/customer type, and IBKR submission date

## Population and Display Derivation

1. The page shows a spinner while authenticated GET section requests fetch the full account population, account-contact links, contacts and advisors concurrently. The spinner disappears as soon as full accounts arrive, and the table appears even while links, contacts or advisors remain pending. Each completed dataset recomputes rows without wiping prior results. Clients and NAV reports start concurrently when accounts arrive, filling their columns independently. No IBKR-details report is requested by this page. GET handlers avoid client Server Action queuing; the legacy combined loader remains for other reports. Filters, general table search and CSV export remain unavailable until all reference datasets and both reports succeed. A report failure preserves the core table with an explicit warning. An accounts failure immediately shows a load error; reference-data failure preserves the table with a warning.
2. A non-array accounts response fails the load. Valid rows are sorted newest to oldest by the internal `created` string.
3. Missing alias, status, title, master account, SLS devices, client email, and fee-template summary are normalized to `-`; missing NAV remains null with no IBKR-equity fallback.
4. The contact label uses the first available source in this order:
   - direct first/second contact-name fields, joined with `and` when both exist;
   - account-contact links resolved against embedded and global contacts;
   - direct embedded contact names when links are absent;
   - exact case-insensitive global contact email match to `client_email_address`;
   - exact case-insensitive global contact name match to account title;
   - `-` when no source resolves.
5. The grid displays Account Number, Contacts, IBKR Username, Master Account, Status, Advisor, and NAV, in that order. Status uses only the Clients report Status field; NAV uses only the NAV report Total field. Missing report values display `-`; pending report columns display Loading.
6. Master account is displayed as `Broker` only when `master_account == I6413690`; every other value is displayed as `Advisor`.

## Filter Definitions

Account type, status, application presence, contact state, NAV, advisor, SLS device and email conditions combine with logical AND. The DataTable also provides its own general table filtering and CSV export.

| Filter | Exact current rule |
|---|---|
| NAV | `All`; exactly 0; greater than 0 through 5,000; greater than 5,000 through 10,000; greater than 10,000 through 50,000; or greater than 50,000. Text NAV removes commas before parsing. Unparseable NAV is excluded from specific bands. |
| Advisor | All or normalized advisor code; blank codes and codes absent from the advisor map match Unassigned. |
| SLS device | Dynamic values are created by splitting `sls_devices` on commas and trimming each value. Blank/null/`-` becomes `None`. A row matches when any normalized device equals the selected value. |
| Status | Uses the Clients report Status field for every row, with `-` for a missing value. Status keys are normalized to lowercase underscore-separated values. Multiple selections combine with OR. |
| Application state | With application or Without application checks whether application_json is truthy. |
| Account type | Individual, Joint or Organization from stored application customer type. |
| Contact state | With/without linked contacts. |
| Email state | `Requires Email Change` includes only rows where both temporal email and client email are present after trim/lowercase normalization and the values are unequal. Rows missing either email are excluded. Other filter conditions are evaluated before this condition. |

## Step-by-Step Workflow

1. Open `Accounts`; the spinner stays visible only until the full account list arrives; reference data and Clients/NAV then fill the visible table. Wait for loading to complete before filtering or exporting. A failed supplementary load preserves the available rows but leaves filters/export unavailable; refresh to retry.
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
- Full accounts failure displays a load error; reference-data and Clients/NAV failures preserve the table with an explicit warning and disabled filters/export. Pending NAV and status display Loading rather than a confirmed zero or status.
- Missing contact metadata triggers fallback matching; email/title fallback can associate a display label without an account-contact link and must not be treated as relationship evidence.
- Missing report NAV remains unknown and do not match specific NAV bands.
- The master-account label treats every account other than `I6413690` as `Advisor`, including unknown or unexpected master-account values.

## Controls / Verification Points
- Detective control: explicit NAV, device, status, application, and email filters help identify operational exceptions.
- Detective control: CSV export can retain the resulting population when paired with a manual record of the filters used.
- Control limitation: display fallbacks can hide missing source data; pending NAV/status are explicitly marked during loading.
- Control limitation: this page is analysis only and does not prove that an exception was reviewed or corrected.

## Evidence to Retain
- `accounts.csv` plus selected filters and review timestamp
- Source account metadata response where needed for reconciliation
- Downstream account/application changes and their separate evidence

## Related Code / Pages / Routes
- Entry: `agm-dashboard/src/app/(dashboard)/(services)/(clients)/accounts/page.tsx`
- Page logic: `agm-dashboard/src/components/dashboard/clients/accounts/AccountsPage.tsx`
- Server loader: `agm-dashboard/src/utils/tools/account_pages.ts`
- Data client: `agm-dashboard/src/utils/clients/account.ts`
- Current compliance audit: [Accounts Audit Review](22-accounts-audit-review.md)

## Individual Account Loading

The individual account/application resolver uses one authenticated private/no-store GET to read the selected internal account and, for submitted accounts, current IBKR details. Unsubmitted applications resolve without an IBKR request. The Account page reuses that response instead of rereading the account during initial rendering; explicit refresh obtains fresh core data. After core rendering, private/no-store GET requests independently load client report data, advisors, financial ranges, primary-contact lookup and linked holder contacts/screenings, avoiding browser Server Action queuing. Contact IDs are deduplicated and contacts/screenings load concurrently on the server. Existing missing entity/external ID link reconciliation remains; links are reread only after an actual update, and contact/screening reads are reused. Account-navigation data is requested only when the search dropdown is opened, using accounts, links and contacts without a client report; responses omit application payloads and unrelated contact fields. A failed search shows a toast and can be retried by reopening the dropdown. Supplementary failure preserves the core page with an explicit warning. Request generations and resolver cleanup prevent late responses from replacing data after navigation or refresh.

## Implementation Boundary

The loader fetches the entire population in one request without an initial 50-row preview to preserve existing browser filters and complete CSV exports. Report-derived filtering remains client-side; it has not been moved into SQL. The concurrent datasets do not form a transactional snapshot across inserts/deletes. The list GET response retains only application presence/customer type rather than full application bodies; the individual account/application response remains complete. Clients and NAV list responses retain the fields used by joins, display and filters. The API caches the parsed Clients and NAV reports independently in each process for 30 seconds by default (`REPORT_CACHE_TTL_SECONDS`, 0 disables, maximum 300 seconds). Concurrent misses for the same report share one read. Every request still authenticates and checks scopes. Errors are not cached, and expired values are not served after a failed refresh. A source update can take up to the configured cache TTL to appear; this is additional to the daily report source schedule. New instances/workers start empty. No schema changes are included.

## API Startup and Deployment Gate

API Cloud Build imports and exports Docker BuildKit layers using the dedicated `agm-api:buildcache` Artifact Registry tag. Dependency installation is reusable when its inputs are unchanged; application source changes invalidate subsequent layers. A missing cache on the first run allows a full build. Each candidate retains its own image tag, and cache reuse does not bypass the exact-image schema preflight described below. Build logs retain cache-hit and timing evidence.

API workers construct local table metadata from the declared models without live schema inspection, table creation, or full reflection. Cloud Build and the daily refresh workflow both call `agm-api/dev/deploy_api.sh`. It resolves an exact image tag through Artifact Registry tag metadata, without querying Container Analysis or requiring vulnerability-metadata permissions. Already pinned SHA256 digests bypass lookup; missing, ambiguous or malformed digest results block deployment. The daily refresh reads the serving revision’s resolved image digest rather than re-resolving its mutable tag. It runs that exact image as a temporary Cloud Run schema-preflight job, waits for successful completion, and only then updates the API service with the same digest. The job uses the existing API service account, region and VPC route and has no automatic retries. A missing permission, unavailable database or table/column/constraint/type mismatch stops deployment and preserves the current service. Job cleanup is attempted while execution/deployment logs retain the outcome. Direct manual deployments must use this gate; a direct gcloud service update can bypass it. Strict schema validation runs even when DEV_MODE is true. Schema changes require explicit migrations before preflight; the application no longer calls create_all. This does not prevent schema drift after a successful deployment. Drive and IBKR connector construction is deferred until the first operation that uses each connector; first-use latency and failure can therefore appear on a request rather than at startup. Authentication/scope enforcement remains per request.

## Last Reviewed
- Status: draft
- Date: 2026-10-08
- Reviewer: Codex current-state code review
