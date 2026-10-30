# Clients ETL Pipeline

## Business Purpose
Refresh AGM client-related backup and reporting resources by extracting source files, backing them up into managed folders, and running the configured transform steps.

## Trigger / Frequency
- Trigger: GitHub Actions launches the `agm-clients-etl` Cloud Run Job, polls that execution, and retrieves its durable terminal overview. The legacy authenticated `GET /etl/clients` route remains available but is not the scheduled trigger.
- Frequency: Daily at 06:00 UTC / 12:00 AM CST (midnight Costa Rica, UTC-6), plus manual workflow dispatch.

## Systems Involved
- `agm-api`
- GitHub Actions
- Cloud Run Job and private GCS terminal-overview storage
- Google Drive batch and resources folders
- Flex Query sources and other configured client-source extracts
- ETL transform logic in `private/etl.py`

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- GitHub Google Cloud service-account authentication and job invocation/storage permissions
- Valid ETL config for the `clients` pipeline
- Access to Flex Query, Google Drive, and any other configured source systems
- Working extract, backup, and transform step functions

## Daily task CSV resources — October 30, 2026

The Clients ETL publishes three separate stable task CSV resources:

| Resource | Collection |
| --- | --- |
| `ibkr_compliance_tasks.csv` | Latest manually provided Portal `compliance_tasks.csv`; validated, then republished without pretending the export was captured today. |
| `ibkr_pending_tasks.csv` | Daily per-account Account Management `tasks?type=pending` GETs. |
| `ibkr_registration_tasks.csv` | Daily per-account Account Management `tasks?type=registration` GETs, including completed history. |

The two API feeds run as independent extract steps alongside account details. Each refreshes the internal IBKR account population plus prior-only details/snapshot accounts. A missing master may use the exact prior details account's master, matching the details refresh policy; stored master conflicts are not overwritten. Unsupported/missing/conflicting masters remain explicit failed coverage. Four workers and four request starts per second are the defaults; each GET has a 10-second connection/45-second read timeout. Clients remain bound to their credential context. As with the current details refresher, HTTP 403 permits one recorded read using the other configured credential context; there are no transport retries or broker writes.

Each API CSV has `record_type=account` coverage rows and `record_type=task` requirement rows. Coverage distinguishes confirmed zero tasks from unreadable accounts, captures attempts/context, last successful read, refresh time and retained-stale status. Original task JSON preserves person/task IDs, flags, warnings and questionnaires; secret-bearing URL/token values are redacted. Explicit `pendingTaskPresent=false` with an omitted inventory is a valid empty result. Missing arrays without that evidence, account mismatches, contradictory flags and provider errors fail. Registration forms are never merged with newer pending instances by form number.

On a failed account/source refresh, retain its previous tasks and mark them stale; first-read failure has unknown task count, not zero. A successful empty inventory replaces previous requirements with confirmed zero coverage. Partial account coverage publishes a truthful partial snapshot and remains partial in the extract overview. Fatal population/serialization/backup errors preserve the prior stable resource. Each dated CSV backup is byte/MD5 verified before transform, and transform accepts only the exact newly extracted source from that run. Each stable resource is independently staged/verified and updated in place; the three publications are not an atomic multi-file transaction.

Staff reporting reads use `GET /reporting/pending_tasks` and `/reporting/registration_tasks`. JSON supports `account_id`, `status=all|pending|completed`, `offset` and `limit` (default 500, maximum 2000), returns coverage/freshness and paginated task rows, and performs no IBKR reads. `format=csv` downloads the full selected-account CSV, including coverage; it cannot be combined with status/pagination filters. Report reads use separate bounded version-aware cache entries and private/no-store responses. The existing Compliance JSON endpoint remains compatible for AccountPage and Accounts Audit; its CSV is the new primary file resource.

The Clients source overlay/allowlist includes the task refresher and accepts the two known predecessor layouts while preserving the existing accepted-coverage baseline. A release/source synchronization is required for scheduled execution of new code; local or scoped resource publication alone does not update the scheduled job. AccountPage retains its selected-account live reads. Login-message snapshots and callbacks remain outside collection scope. The owner-approved [Tasks Review](38-tasks-review.md) reads these saved sources through a staff-only paginated projection, including coverage gaps and exact-instance links without discarding observations. The historical subaccount CSV remains a separate baseline.

## Step-by-Step Workflow
1. The workflow launches the clients Cloud Run Job and polls its specific execution, with no automatic whole-job retry or API fallback.
2. The runner acquires its generation-owned clients lock, copies configured manual inputs into isolated clients staging, and resolves the `clients` ETL configuration from `_get_etl_config_by_name`. Individual copy failures are recorded and do not prevent later copies or pipeline execution.
3. The ETL runner executes three stages in order: `extract`, `backup`, and `transform`.
4. During extract, each configured source file with an `extract_func` is fetched. Sources with no extractor are manually supplied batch inputs: if the configured raw batch filename is present at the initial batch-folder snapshot, the step is successful; if absent, the step is skipped.
5. During backup, batch files are renamed, sorted into the resource structure, and the batch folder is cleared.
6. During transform, configured transforms convert the extracted source data into reporting-ready resource files.
7. After all three stages have been attempted, the runner publishes a durable overview containing raw and operational status, per-stage summary counts, step-level outcomes and warnings. Captured processing errors produce `completed_with_errors` and exit zero; a clean run produces `success`. The runner releases its owned lock. Fatal runner, overview-publication or lock-release errors still produce a failed execution.
8. GitHub Actions retrieves that execution's overview and checks both Cloud Run terminal success and the completed operational statuses. It formats the same detailed summary for Actions and email, with a `COMPLETED WITH ERRORS` subject when processing errors occurred. The earlier `completed_with_accepted_coverage_gaps` status remains readable for compatibility.

## Outputs / Records Created
- Refreshed raw files in the batch and resources folders
- Updated transformed client resource files, including the stable OFAC, UK, and UN sanctions resource files when their report steps complete
- Durable ETL overview JSON in private GCS, plus the overview returned by the legacy API route when used
- Actions summary and notification email with workflow/execution identifiers, completion time in UTC, raw/operational status, stage counts, backup sub-step counts, failure details, IBKR freshness/staleness counts and fallback warnings

## Exception Paths / Failure Handling
- Cloud authentication, launch, polling or overview retrieval failure: workflow fails and the summary includes available execution evidence and the workflow-log link. Unavailable counts are not reported as zero.
- Extractor failure: stage status becomes `partial` and the failure is included in the overview, but later stages continue. Manually supplied files are successful or skipped based only on their presence at ETL start; backup/transform report later missing or unusable files.
- Account and file errors remain failed outcomes in the raw overview but do not fail a completed batch. The accepted account/error baseline is no longer a completion gate and is not expanded. Stale retained records remain labeled stale; account identity/master validation is preserved.
- Configured Flex reports returning IBKR error 1019 / statement generation in progress retain a warning when the matching transform successfully publishes a named existing backup. The warning identifies the missing new file, backup used and original error. Other source errors also remain visible without preventing batch completion.
- Captured extract, backup or transform errors do not stop later stages and yield `completed_with_errors` after all stages are attempted. Startup, lock acquisition, uncaught runner, overview-publication, lock-release, missing terminal evidence or Cloud Run execution failures still fail the execution/workflow. The summary shows stage, step and file errors and available runner/Cloud Run conditions.
- Legacy API callers still receive HTTP `200` for a captured `partial` pipeline result. Do not invoke the route concurrently with the clients job.

## Controls / Verification Points
- Preventive control: named pipeline config must exist or the route raises an error.
- Detective control: extract reports total/successful/skipped/failed counts; transform reports total/successful/failed counts; backup reports rename/move counts and clear outcome.
- Detective control: workflow summary and email include stage/step/file failure details and explicit `none` for empty error/warning sections. Missing evidence is labeled unknown/unavailable.
- Detective control: IBKR account failures are grouped by safe reason/status category with counts; account identifiers stay in the private overview. A private GCS overview URI preserves access to complete account outcomes without expanding every failure into the email.
- Detective control: completed-with-errors status, raw partial status, retained stale counts, manual-copy errors and Flex backup-use warnings remain visible independently of operational completion. Completion does not certify that every source refreshed or every report published successfully.
- Current limitation: the notification summary is aggregate and does not state the status, source date, row count, or version of each compliance reference list.

## Evidence to Retain
- GitHub Actions run history for `etl_clients.yaml`
- Terminal ETL overview JSON in private GCS and its formatted Actions summary
- Google Drive resource folder contents after the run
- Notification email generated by the workflow

## Related Code / Pages / Routes
- Entry surfaces: `agm-api/.github/workflows/etl_clients.yaml`, `agm-api/src/jobs/clients_etl.py`, legacy `agm-api/src/app/tools/private/etl.py`
- Supporting modules: `agm-api/src/components/tools/private/etl.py`, `agm-api/.github/scripts/clients_etl_summary.py`
- Downstream side effects: Google Drive connectors, Flex Query extraction helpers, transformed reporting resource files, and [Compliance Reference Data Refresh](15b-compliance-reference-data-refresh.md)

## Reporting Data-Quality Note

The accrued-interest consumer may fall back from `OpenDateTime` to
`HoldingPeriodDateTime` when calculating holding days. Bond descriptions with
an IBKR issue-date prefix (`DMM/DD/YY`) must not have that date treated as the
maturity date.

## Reporting Consumer Freshness

Clients, NAV, client-fee, open-position and daily IBKR-details reporting snapshot reads use independent entries in a process-local cache in `agm-api`, with a default 30-second TTL, configurable through `REPORT_CACHE_TTL_SECONDS` (0 disables, maximum 300 seconds). The IBKR-details entry is the file in Drive, not the live `/accounts/ibkr/details` provider response. ETL writes do not broadcast invalidation to Cloud Run instances; an updated report can remain unseen until a consumer cache expires. Expired entries are refreshed from Drive and are not served after refresh errors. Market bond, Treasury, stock and ETF snapshots share the same bounded policy in separate entries. Live provider, banking and transaction-write responses are outside this cache. Ownership/account filters and optional quote enrichment run after retrieving a copied base snapshot, so cached rows are not filtered or enriched in place.

## Dependency Initialization

Drive and IBKR trading connectors are constructed on the first ETL operation that needs them, rather than during API startup. Their initialization failures remain within the existing ETL step/error handling. Schema compatibility is checked by the candidate-image deployment preflight before rollout; API startup no longer creates or reflects the entire database schema.

## Last Reviewed
- Status: draft
- Date: 2026-07-15
- Reviewer: Codex current-state code review

## Clients Cloud Run cutover (2026-10-14)

The clients-only `agm-clients-etl` Cloud Run Job is provisioned in agm-datalake/us-central1 with the existing service identity/VPC and immutable API runtime image, 1 CPU/2 GiB, one task, a 60-minute timeout and no automatic retries. Its Python bootstrap loads only an immutable generation-pinned SHA-256-verified source bundle from private `agm-datalake-etl-runs`; the main API service is not rebuilt or updated for this cutover.

The job uses isolated clients staging (`1EIshGq22oFg-WJ9aHdS37Heh5KaqN2ct`). It copies only configured manual client inputs from the original shared batch folder; no original manual/market input is moved or deleted. Copy source IDs/modified times are retained in the overview. A GCS generation-conditional clients lock prevents overlapping job executions; a crash can leave a lock requiring operator verification of terminal execution state before conditional removal. The market ETL stays on the existing API path and schedule. Do not manually invoke the legacy API clients endpoint alongside the job because that endpoint is not covered by the job lock.

The prepared clients GitHub schedule remains 10:00 UTC (04:00 Costa Rica), but invokes the job rather than HTTP ETL after cutover. It polls its specific execution and retrieves `runs/<execution>/overview.json` before deciding outcome; no whole-job retries or API fallback. The owner explicitly accepts the known incomplete account population. Only the 208 account/error pairs from the tested baseline may yield `completed_with_accepted_coverage_gaps`; raw partial status, freshness/stale counts and outcomes remain visible. New gaps or failures in other stages, publication or execution produce failure. First scheduled-run acceptance remains pending until the next actual 04:00 execution is reviewed.

## Safe report publication and notification summaries — October 20, 2026

NAV and other ETL final resources stage and verify uploaded byte length/MD5 before updating the stable resource in place. The existing file is not deleted; upload or staging-verification failure leaves it intact. The original publication exception/traceback is retained in runtime evidence instead of being replaced by a string containing the entire configuration. Verified staging cleanup is separate from publication; uncertain writes require inspection rather than automatic replay.

Actions outputs and email carry bounded aggregate counts, report names and approved reason/status categories. Raw exception/provider/configuration strings and account examples are omitted to prevent secret-like values from suppressing the cross-job summary. The private overview and runtime logs retain investigation detail. Account-refresh mappings, permissions and accepted-gap policy are unchanged by this maintenance.

## Shared scheduled start — October 21, 2026

Owner requested both clients and market ETLs at 07:37 CST / 13:37 UTC daily.
This supersedes earlier schedule times retained in historical cutover evidence.
Each pipeline keeps its own job, staging folder and execution lock. Scheduled
start time is a GitHub Actions target, not a guarantee of exact execution time.

## Automatic source synchronization — October 22, 2026

After successful API deployment and before every Clients workflow dispatch, the workflow refreshes the Clients job from the exact committed source using `dev/sync_clients_job.py`. The immutable overlay is uploaded and byte-verified, the existing job is updated with an etag concurrency guard, and the resulting configuration is read back before dispatch. A synchronization failure stops dispatch. The runtime uses the immutable image of the API revision serving 100% traffic. The deployment workflow checks out the triggering compile commit rather than a newer default-branch head.

The accepted coverage baseline is copied byte-for-byte from the previous verified bundle; new account failures cannot silently become accepted gaps. Existing staging, service identity, networking, limits, timeout, retry policy and unrelated environment settings are preserved. The job source commit is labeled for evidence. Synchronization itself never executes the ETL. Ordinary-run acceptance remains separate.

## Open-position publication and delinked accounts — October 22, 2026

The full-position companion `ibkr_open_positions_all.csv` uses verified staging
and an in-place update, preserving the stable resource's ID and permissions.
There is no pre-delete. The full report and the bond/LOT template are individually
verified; failure of either keeps the transform failed. A successful full report
can coexist with the previous template if the later template update fails.
Uncertain publication is not automatically retried.

IBKR can return a delinking tombstone: HTTP 200, the exact requested account ID,
no master account, and a valid nonfuture `dateDelinked`. The refresh records this
as `skipped / account_delinked`, separately from fresh details and failed coverage.
Previous details remain historical/stale, with the previous successful-fetch time
and explicit delinking reason/date; no previous record means no fresh substitute
is created. The private overview retains account-level evidence; Actions/email
show the aggregate delinked count. This does not alter database account/master
records or expand the accepted failure baseline. Wrong identities, explicit
master conflicts, unexplained absent masters, invalid/future dates and arbitrary
skips remain failures. Release and actual-run verification remain separate gates.

## Shared scheduled start revised — October 23, 2026

Owner requested both clients and market ETLs daily at 03:07 CST (Costa Rica,
UTC-6) / 09:07 UTC (`7 9 * * *`). This supersedes earlier schedule times retained
in historical evidence. Each pipeline keeps its separate job, staging folder and
execution lock. GitHub Actions may start scheduled workflows late.

## Revenue screen loading — October 23, 2026

The Dashboard Revenue MTD view requests client fees without waiting for or downloading historical statement balances. The first Monthly/Yearly visit or explicit debug export requests those balances; the page reuses that request/result for later tab switches. Export still includes the full fee and balance payloads after both are available. Individual source failures remain visible with retry controls; no partial historical financial totals are presented as complete.

## Shared scheduled start revised — October 26, 2026

Owner requested both clients and market ETLs daily at 06:00 UTC / 00:00 CST
(midnight Costa Rica, UTC-6), using `0 6 * * *`. This supersedes earlier schedule
times retained in historical evidence. GitHub Actions may start scheduled workflows
late; the configured time is a target.
