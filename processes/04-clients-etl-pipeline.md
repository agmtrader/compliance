# Clients ETL Pipeline

## Business Purpose
Refresh AGM client-related backup and reporting resources by extracting source files, backing them up into managed folders, and running the configured transform steps.

## Trigger / Frequency
- Trigger: GitHub Actions launches the `agm-clients-etl` Cloud Run Job, polls that execution, and retrieves its durable terminal overview. The legacy authenticated `GET /etl/clients` route remains available but is not the scheduled trigger.
- Frequency: Daily at 13:37 UTC / 7:37 AM CST (Costa Rica, UTC-6), plus manual workflow dispatch.

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

## Step-by-Step Workflow
1. The workflow launches the clients Cloud Run Job and polls its specific execution, with no automatic whole-job retry or API fallback.
2. The runner acquires its generation-owned clients lock, copies configured manual inputs into isolated clients staging, and resolves the `clients` ETL configuration from `_get_etl_config_by_name`.
3. The ETL runner executes three stages in order: `extract`, `backup`, and `transform`.
4. During extract, each configured source file with an `extract_func` is fetched. Sources with no extractor are manually supplied batch inputs: if the configured raw batch filename is present at the initial batch-folder snapshot, the step is successful; if absent, the step is skipped.
5. During backup, batch files are renamed, sorted into the resource structure, and the batch folder is cleared.
6. During transform, configured transforms convert the extracted source data into reporting-ready resource files.
7. The runner publishes a durable overview containing raw and operational status, per-stage summary counts, step-level outcomes and warnings. It releases its owned lock and exits according to the operational result.
8. GitHub Actions retrieves that execution's overview and checks both Cloud Run terminal success and the accepted operational statuses. It formats the same detailed summary for Actions and the success/failure notification email.

## Outputs / Records Created
- Refreshed raw files in the batch and resources folders
- Updated transformed client resource files, including the stable OFAC, UK, and UN sanctions resource files when their report steps complete
- Durable ETL overview JSON in private GCS, plus the overview returned by the legacy API route when used
- Actions summary and notification email with workflow/execution identifiers, completion time in UTC, raw/operational status, stage counts, backup sub-step counts, failure details, IBKR freshness/staleness counts and fallback warnings

## Exception Paths / Failure Handling
- Cloud authentication, launch, polling or overview retrieval failure: workflow fails and the summary includes available execution evidence and the workflow-log link. Unavailable counts are not reported as zero.
- Extractor failure: stage status becomes `partial` and the failure is included in the overview, but later stages continue. Manually supplied files are successful or skipped based only on their presence at ETL start; backup/transform report later missing or unusable files.
- Only account/error pairs matching the explicit accepted coverage baseline can yield `completed_with_accepted_coverage_gaps`. Stale retained records remain labeled stale. Unexpected account failures fail the job.
- Configured Flex reports returning IBKR error 1019 / statement generation in progress may use an existing named backup only when its matching transform and the backup/transform stages succeed. The warning identifies the missing new file, backup used and original error. Other source errors remain failures.
- Backup, transform, publication, lock-release or other runner failures produce a failed execution/workflow. The summary shows stage, step and file errors, runner error fields and failed Cloud Run terminal conditions when available.
- Legacy API callers still receive HTTP `200` for a captured `partial` pipeline result; this is not the scheduled workflow's acceptance rule. Do not invoke it concurrently with the clients job.

## Controls / Verification Points
- Preventive control: named pipeline config must exist or the route raises an error.
- Detective control: extract reports total/successful/skipped/failed counts; transform reports total/successful/failed counts; backup reports rename/move counts and clear outcome.
- Detective control: workflow summary and email include stage/step/file failure details and explicit `none` for empty error/warning sections. Missing evidence is labeled unknown/unavailable.
- Detective control: IBKR account failures are grouped by exact error with counts and up to five account examples per group. A private GCS overview URI preserves access to complete account outcomes without expanding every failure into the email.
- Detective control: accepted gaps, raw partial status, retained stale counts and Flex backup-use warnings remain visible independently of operational success.
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

Clients and NAV report reads use an independent process-local cache in `agm-api`, with a default 30-second TTL, configurable through `REPORT_CACHE_TTL_SECONDS` (0 disables, maximum 300 seconds). ETL writes do not broadcast invalidation to Cloud Run instances; an updated report can remain unseen until a consumer cache expires. Expired entries are refreshed from Drive and are not served after refresh errors. Other ETL resources are outside this cache.

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

## Shared scheduled start — October 21, 2026

Owner requested both clients and market ETLs at 07:37 CST / 13:37 UTC daily.
This supersedes earlier schedule times retained in historical cutover evidence.
Each pipeline keeps its own job, staging folder and execution lock. Scheduled
start time is a GitHub Actions target, not a guarantee of exact execution time.

## Automatic source synchronization — October 22, 2026

After successful API deployment and before every Clients workflow dispatch, the workflow refreshes the Clients job from the exact committed source using `dev/sync_clients_job.py`. The immutable overlay is uploaded and byte-verified, the existing job is updated with an etag concurrency guard, and the resulting configuration is read back before dispatch. A synchronization failure stops dispatch. The runtime uses the immutable image of the API revision serving 100% traffic. The deployment workflow checks out the triggering compile commit rather than a newer default-branch head.

The accepted coverage baseline is copied byte-for-byte from the previous verified bundle; new account failures cannot silently become accepted gaps. Existing staging, service identity, networking, limits, timeout, retry policy and unrelated environment settings are preserved. The job source commit is labeled for evidence. Synchronization itself never executes the ETL. Ordinary-run acceptance remains separate.
