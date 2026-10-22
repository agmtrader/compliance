# Market Data ETL Pipeline

## Business Purpose
Refresh AGM market-data resource files used by reporting and analytics by running the configured extract, backup, and transform stages for the market-data pipeline.

## Trigger / Frequency
- Trigger after cutover: GitHub Actions launches the `agm-market-etl` Cloud Run Job and polls its exact execution. The authenticated manual `GET`/`POST /etl/market_data` path dispatches the same job and returns HTTP 202; `GET /etl/market_data?operation=<id>` reads its operation status.
- Frequency: Daily at 13:37 UTC / 7:37 AM CST (Costa Rica, UTC-6), plus manual workflow dispatch. GitHub Actions can still create the run late.

## Systems Involved
- `agm-api`
- GitHub Actions
- Cloud Run Job and private GCS terminal-overview storage
- Google Drive batch and resources folders
- External market-data source functions configured in the ETL pipeline

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- GitHub Google Cloud identity with job invocation, execution read and private overview read permissions; the API runtime identity needs job invocation/operation-read access for manual dispatch
- API auth token with existing `etl/market_data` scope for the manual route only
- Dedicated empty market staging folder, distinct from clients staging and the legacy shared inputs
- Valid `market_data` ETL config
- Access to configured market-data sources and Google Drive
- Functional transform steps for market-data output
- IBKR market-data session can be established, bootstrapped with a successful
  `/iserver/accounts` response, and closed after extraction

## Step-by-Step Workflow
1. The workflow launches one market Cloud Run execution without automatic retry or API fallback, then polls that exact execution. Manual API callers receive an operation reference immediately; acceptance is not ETL completion.
2. The job acquires a generation-owned market lock and verifies its dedicated staging folder is empty. Prior interrupted staging is retained for operator review, never blindly cleared on startup. It initializes the existing database connector and resolves the `market_data` configuration. The clients job and its coverage policy are unchanged.
3. The ETL runner executes `extract`, `backup`, and `transform` stages in order.
4. Extract pulls each configured market-data source into the batch area and summarizes step success, skip, or failure.
   For IBKR sources, each extract opens one SSO/brokerage session, queries
   `/iserver/accounts` before security or snapshot requests, and logs out in a
   `finally` path after the source completes or fails.
5. Backup renames batch files, moves them into the resource structure, and clears the batch folder.
6. Transform converts the resource files into downstream market-data outputs consumed by reporting logic.
   When `DEV_MODE=true`, the extractor also runs the existing IBKR historical daily-bar
   enrichment process. With `DEV_MODE=false` (the production path), it skips those
   requests and preserves the `Current Yield` column as empty so the parent request
   remains bounded.
7. The runner releases only its owned lock and publishes a durable `runs/<execution>/overview.json`, including lock-release failure and raw/operational outcomes. Publication failure exits nonzero. A killed process can leave a lock requiring verified terminal-state review before conditional operator removal.
8. GitHub Actions requires both Cloud Run terminal success and a matching execution/pipeline overview. It validates all four configured market sources, all three stages, nonzero/consistent counts and exact extract-to-transform source IDs. Partial, missing, malformed, inconsistent or provenance-free results fail the workflow; clients accepted-gap/Flex fallback rules do not apply to market. An always-running step publishes a bounded aggregate summary. Unavailable evidence stays unavailable. No whole-operation retry is performed because an interrupted execution may already have published files.

## Outputs / Records Created
- Refreshed raw market-data files in Google Drive
- Updated transformed market-data resources
- ETF and stock resource rows with populated `Current Yield` when `DEV_MODE=true` and
  five years of bars are available
- ETF and stock resource rows with an empty `Current Yield` when `DEV_MODE=false`
- Per-symbol resolution, snapshot, and development-only history failures logged with
  ticker and conid; symbols with missing or invalid snapshot rows are omitted while
  valid symbols continue processing
- Durable private GCS terminal overview; manual API dispatch returns an operation/status URL rather than the completed ETL payload
- Workflow notification email containing the overall result, extract/backup/transform stage statuses and counts, and any reported failures

## Exception Paths / Failure Handling
- Centralized API exception reports retain request/error IDs, HTTP method, route template, endpoint, status/code and exception types with stack file/line/function locations. They omit query values, client IPs, error details, exception messages and source-line text; client-facing controlled errors retain their existing message/details. This redaction covers the shared exception wrapper, not every independent integration/audit log.
- An expired IBKR trading session or the known accounts-bootstrap prerequisite remains a controlled provider error with reconnect guidance. The market job permits one logged recovery for an allowlisted read per extract; persistent failure fails the source and terminal outcome. The dispatch response cannot certify the later brokerage session. Stage/symbol failure outcomes remain visible. IBKR service authentication, unknown upstream failures, and missing or duplicate reporting resources remain reportable.
- Cloud authentication, job launch, polling or overview retrieval failure: workflow fails with available execution references and unavailable metrics; no synchronous fallback. A dispatch timeout is uncertain, not permission to replay.
- Missing/incomplete pipeline config, nonempty staging or lock acquisition failure: job fails before pipeline writes.
- Extract, backup, or transform failure: the overview captures the failing step and the workflow fails. A market transform cannot fall back to a prior backup when its extractor failed.
- Historical-yield enrichment is disabled when `DEV_MODE=false`; failures in the
  development-only enrichment must not change the production behavior.
- A failed stock or ETF security lookup, snapshot response, or historical-yield
  request is logged with `MARKET_DATA_SYMBOL_FAILED` and the symbol/conid when
  available. Missing snapshot rows are excluded from that output; they do not abort
  the remaining symbols. The snapshot summary reports requested, returned, and
  failed counts.
- Bond duration calculation failure for an individual dated bond: the transform
  logs the exception with the row inputs and, when maturity is missing, identifies
  the affected bond by symbol, financial instrument, and company name. It leaves
  that row's duration blank, reports the failure count, and continues processing
  the remaining rows. Perpetual/non-maturing instruments are an expected
  exception: duration is not applicable, so the row remains blank and a warning
  is recorded without raising an error-group traceback.
- No automatic whole-job or dispatch retry is performed; inspect execution, private overview, staging and lock state before any manual replay. API operation completion reports Cloud Run completion only; the private overview is authoritative for pipeline/stage results.

## Controls / Verification Points
- Preventive control: pipeline name must map to a defined config.
- Detective control: each stage reports total, successful, skipped, and failed step counts.
- Detective control: backup-step counts are included in workflow summary output.
- Detective control: workflow logs and email preserve the ETL outcome for review.

## Evidence to Retain
- GitHub Actions run history for `etl_market_data.yaml`
- Terminal ETL overview JSON in private GCS, Cloud Run execution outcome and bounded Actions summary
- Resource folder outputs after the run
- Success or failure notification email

## Related Code / Pages / Routes
- Entry surfaces: `agm-api/.github/workflows/etl_market_data.yaml`, `agm-api/src/jobs/market_etl.py`, `.github/scripts/market_etl_summary.py`, `agm-api/src/app/tools/private/etl.py`
- Supporting modules: `agm-api/src/components/tools/private/etl.py`
- Downstream side effects: reporting resource files and market-data-derived outputs

## Bond Description Parsing

Bond snapshot transformation prioritizes the percentage-marked coupon in the
IBKR financial-instrument description so issue-date components are not
mistaken for coupon values. Descriptions whose maturity is represented only by
an IBKR shorthand code remain dependent on the source snapshot's explicit
maturity field. Descriptions marked ``Perpetual`` (or ``Perp``) intentionally
have no maturity; their duration is left blank because a principal repayment
date does not exist.

## Dependency Initialization

Drive and IBKR trading connectors are constructed on the first ETL operation that needs them, rather than during API startup. Their initialization failures remain within the existing ETL step/error handling. Schema compatibility is checked by the candidate-image deployment preflight before rollout; API startup no longer creates or reflects the entire database schema.

## Last Reviewed
- Status: draft
- Date: 2026-06-16
- Reviewer: Codex initial draft

## Downstream Proposal Rating Eligibility

Proposal generation validates and normalizes bond ratings from transformed
market-data resources before selection. Valid canonical S&P equivalents are
preserved, including the report merge's `_x` column. When a canonical equivalent
is unavailable, valid structured S&P/Moody's fields are preferred to free text;
when both agencies provide ratings, the lower credit rating is retained, matching
the transform's rating-level policy. Free-text parsing matches whole rating tokens
and preserves modifiers rather than matching letters embedded in words.

Only treasury, AAA/AA/A, BBB and BB bond categories are eligible for the configured
proposal buckets. BB+/BB/BB- and Moody's Ba1/Ba2/Ba3 equivalents remain eligible.
Lower-rated, missing, unrated and unrecognized candidates cannot fill a BB bucket.
A requested bucket with no eligible assets returns HTTP 422 before persistence;
existing saved proposals are not rewritten by this consumer control. The ETL
trigger, resource outputs and transformation behavior are unchanged.

## Extractor and publication controls — October 20, 2026

Extractor data and execution metadata are separate: DataFrames contribute row counts, structured outcomes retain their status, and a market source must record the Drive ID returned by its upload. Only that exact source can feed the corresponding transform in the same request. A newer unrelated backup or stale prior-run file cannot substitute. Runtime config copies discard provenance left in templates.

Final resources use verified staging: upload under a noncanonical pending name, check Drive byte length and MD5, then update the existing stable file ID or promote the verified stage when no stable resource exists. Verify the canonical result before cleaning the stage. Never delete the stable file before upload. Duplicate canonical resources fail closed; ambiguous interrupted writes are not automatically replayed. Pending files after an uncertain failure require inspection. This is not a distributed lock or a multi-file transaction.

Regression acceptance covers DataFrame return handling, exact source selection, preservation under upload/checksum/update failures, original exceptions and Actions shell outcomes. Local fixtures do not establish hosted Drive acceptance.

## Async market cutover — October 20, 2026

Implementation uses the working clients architecture with a separate market job, staging folder and generation-owned lock. Job runtime is the current immutable API image plus an exact allowlisted, generation-pinned SHA-256 source overlay; no clients runner/baseline is changed. Job timeout is 60 minutes, one task, one CPU/2 GiB and no automatic retries. The schedule is revised to 12:00 UTC / 06:00 CST (Costa Rica) alongside clients ETL; notification recipients are preserved. Deployment commands, environment and actual acceptance evidence are recorded in `agm-api/docs/market-etl-runner.md` and API #86. API main commit 4d078ff publishes the job-based workflow and dispatch source; normal CI/API deployment remains separately observable. Controlled cloud execution agm-market-etl-vb9g8 completed successfully at 18:26:28 UTC on October 20 (12:26 Costa Rica), with all four extracts and transforms successful, successful backup, durable overview, lock HTTP404 and empty staging. All four canonical outputs were independently checked for unique names, nonzero sizes/MD5 and current-execution modification times. First scheduled invocation and authenticated hosted manual-route acceptance remain separate observations.

## Shared scheduled start — October 21, 2026

Owner requested both clients and market ETLs at 07:37 CST / 13:37 UTC daily.
This supersedes earlier schedule times retained in historical cutover evidence.
Each pipeline keeps its own job, staging folder and execution lock. Scheduled
start time is a GitHub Actions target, not a guarantee of exact execution time.

## Background equity history and session recovery — October 22, 2026

The background runner enables `historical_yields: true` for stock and ETF extracts
with `DEV_MODE=false`. `Current Yield` is the existing arithmetic mean of five
annual price returns in percentage points, using six annual close anchors. It is
not dividend yield or total return. Daily history pages are sequential and paced
at 1.25 seconds per request. Nonfinite/nonpositive prices are rejected; unavailable
or insufficient history leaves a blank yield. The private overview records
requested, populated and unavailable counts; pipeline success does not imply full
historical coverage. Foreground production extracts retain their default behavior.

An IBKR session can be lost after initial SSO, brokerage initialization and account
bootstrap. Each extract permits at most one recovery across its allowlisted
watchlist, security search, snapshot and history reads. Expired-session 401 obtains
a fresh SSO session and repeats initialization/account bootstrap; the exact known
500 accounts prerequisite repeats account bootstrap. Only the failed read is
repeated. Recovery history requests retain pacing. Other authorization/provider
errors do not qualify. Order and other mutation endpoints cannot use this path.
Initial and recovered brokerage initialization waits the documented two seconds
before account bootstrap. Known session errors retain sanitized provider details
and authentication flags in private `session_failures`, including errors after
the recovery limit; unavailable diagnostic reads never replace the source error.
Every recovery is logged and retained in extract `session_recoveries`; a nonzero
count appears in the workflow summary. Persistent session failure aborts symbol
resolution/history extraction rather than silently omitting tickers or yields.
No whole-job retry, stale-source fallback or relaxed four-source gate is added.

Before dispatch, GitHub Actions synchronizes source from its exact committed SHA.
The artifact includes the connector and bootstrap's exact allowlist, is pinned by
GCS generation/SHA-256, and is independently read back. Existing image, staging,
identity, VPC, resources, zero task retries and other settings are preserved; etag
rejects concurrent configuration edits. An active/uncertain execution or retained
market lock blocks the update. Failed synchronization prevents dispatch. The
legacy deployment helper's existing-job path uses this same source synchronizer,
without the unsupported Jobs.patch updateMask parameter. Runtime tests and actual
GitHub/Cloud Run acceptance are recorded separately in API #86 and runner docs.

The requested recovery acceptance passed in GitHub Action
[39843963070](https://github.com/agmtrader/agm-api/actions/runs/39843963070) on
October 22, 2026. All four extracts and transforms succeeded, with successful
backup. Independent downloaded-file checks confirmed current sources, preserved
canonical IDs, row counts and checksums, and stock/ETF values matched the exact
sources. The lock was released and staging was empty. Stock return coverage was
99/99; ETF coverage was 60/61 with one unavailable history left blank. No recovery
was needed in this accepted run. The success-email action passed; recipient
delivery was not separately verified. Earlier candidate failure and read-only
diagnosis are retained separately; this run does not establish the cause of the
earlier IBKR session invalidation. Broader automatic-run/manual dispatch acceptance
remains tracked in API #86. This procedure remains subject to code-owner review.
