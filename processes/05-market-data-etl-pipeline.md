# Market Data ETL Pipeline

## Business Purpose
Refresh AGM market-data resource files used by reporting and analytics by running the configured extract, backup, and transform stages for the market-data pipeline.

## Trigger / Frequency
- Trigger after cutover: GitHub Actions launches the `agm-market-etl` Cloud Run Job and polls its exact execution. The authenticated manual `GET`/`POST /etl/market_data` path dispatches the same job and returns HTTP 202; `GET /etl/market_data?operation=<id>` reads its operation status.
- Frequency: Daily at 06:00 UTC / 12:00 AM CST (midnight Costa Rica, UTC-6), plus manual workflow dispatch. GitHub Actions can still create the run late.

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
   The background market runner enables historical daily-bar enrichment through
   per-run `historical_yields: true` for stocks and ETFs while retaining
   `DEV_MODE=false`. Direct development extracts also retain enrichment through
   `DEV_MODE=true`; foreground production extracts skip it by default.
   `Current Yield` is the existing arithmetic mean of five annual price returns
   in percentage points, using six annual close anchors. It is not dividend,
   distribution or total return. Requests, including the older daily-bar page,
   are sequential and wait 1.25 seconds before each history call.
7. The runner releases only its owned lock and publishes a durable `runs/<execution>/overview.json`, including lock-release failure and raw/operational outcomes. Publication failure exits nonzero. A killed process can leave a lock requiring verified terminal-state review before conditional operator removal.
8. GitHub Actions requires both Cloud Run terminal success and a matching execution/pipeline overview. It validates all four configured market sources, all three stages, nonzero/consistent counts and exact extract-to-transform source IDs. Partial, missing, malformed, inconsistent or provenance-free results fail the workflow; clients accepted-gap/Flex fallback rules do not apply to market. An always-running step publishes a bounded aggregate summary. Unavailable evidence stays unavailable. No whole-operation retry is performed because an interrupted execution may already have published files.

## Outputs / Records Created
- Refreshed raw market-data files in Google Drive
- Updated transformed market-data resources
- ETF and stock resource rows with populated `Current Yield` in the background
  job when five years of usable price history are available
- An empty `Current Yield` for unavailable/invalid/insufficient history, with
  per-extract requested, populated and unavailable counts in the terminal overview;
  zero and negative calculated returns are preserved
- Foreground production snapshots retain an empty `Current Yield` by default
- Per-symbol resolution, snapshot, and history failures logged with
  ticker and conid; symbols with missing or invalid snapshot rows are omitted while
  valid symbols continue processing
- Durable private GCS terminal overview; manual API dispatch returns an operation/status URL rather than the completed ETL payload
- Workflow notification email containing the overall result, extract/backup/transform stage statuses and counts, and any reported failures

## Exception Paths / Failure Handling
- Centralized API exception reports retain request/error IDs, HTTP method, route template, endpoint, status/code and exception types with stack file/line/function locations. They omit query values, client IPs, error details, exception messages and source-line text; client-facing controlled errors retain their existing message/details. This redaction covers the shared exception wrapper, not every independent integration/audit log.
- An expired IBKR trading session or the known accounts-bootstrap prerequisite remains a controlled provider error with reconnect guidance, without an exception report. In the job it becomes a failed stage/terminal outcome; the dispatch response cannot certify the later brokerage session. Stage/symbol failure outcomes remain visible. IBKR service authentication, unknown upstream failures, and missing or duplicate reporting resources remain reportable.
- Cloud authentication, job launch, polling or overview retrieval failure: workflow fails with available execution references and unavailable metrics; no synchronous fallback. A dispatch timeout is uncertain, not permission to replay.
- Missing/incomplete pipeline config, nonempty staging or lock acquisition failure: job fails before pipeline writes.
- Extract, backup, or transform failure: the overview captures the failing step and the workflow fails. A market transform cannot fall back to a prior backup when its extractor failed.
- Historical-yield enrichment is enabled independently of `DEV_MODE` in the
  background market runner. Missing/invalid history leaves the symbol's yield
  blank and does not abort the other symbols. Pipeline success alone does not
  certify complete historical-yield coverage; inspect terminal overview counts.
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


## Proposal selection and count controls

Risk-questionnaire proposal generation uses the October 26 archetype counts: 25, 29,
32, 33, 35, 36, 37 and 33 positions from Conservative A through Aggressive C,
with a maximum 4.5% per position. The
portfolio planner passes a user-selected total to the same selection function for
both preview and save. Money allocation percentages remain independent of security
counts: each active sleeve retains its assigned capital weight, split equally among
its selected securities. Within screened candidate pools, selection maximizes the
allocation-weighted mean of stored return estimates across feasible sleeve counts.
Corporate bond rating, yield validity, outlier and issuer-exclusion controls apply.
Treasury uniqueness is by instrument. Equity ETF and GLD historical returns retain
source percentage-point values, including zero or negative returns. This
objective is a model based on source estimates, not a guarantee of future return.
Missing eligible sleeves or an infeasible exact count fail before persistence; no
silent underfilled proposal is created. Custom proposals retain the user's positive
asset weights and omit zero-weight holdings. Custom preview and save build from
the same explicitly selected holdings and source-yield interpretation; the custom
return card does not use an automatically selected portfolio estimate. Presentation and return calculations
use money weights and exclude zero-weight classes in views and exports. Existing
saved proposals are not regenerated by these code changes; historical missing
weights remain a separate reconciliation requirement.

## Shared scheduled start revised — October 23, 2026

Owner requested both clients and market ETLs daily at 03:07 CST (Costa Rica,
UTC-6) / 09:07 UTC (`7 9 * * *`). This supersedes earlier schedule times retained
in historical evidence. Each pipeline keeps its separate job, staging folder and
execution lock. GitHub Actions may start scheduled workflows late.

## Dashboard snapshot reads — October 23, 2026

Parsed bond, Treasury, stock and ETF reporting snapshots use separate entries in the API's process-local report cache. The default TTL is 30 seconds (`REPORT_CACHE_TTL_SECONDS`, 0 disables, maximum 300 seconds). Concurrent misses share one load; callers receive independent copies. Source errors are not cached, and expired values are never served as fallback. ETL publication does not broadcast invalidation, so consumers can observe the prior snapshot until the configured TTL expires. This applies to stored reporting snapshots, not live IBKR responses or orders.

## Shared scheduled start revised — October 26, 2026

Owner requested both clients and market ETLs daily at 06:00 UTC / 00:00 CST
(midnight Costa Rica, UTC-6), using `0 6 * * *`. This supersedes earlier schedule
times retained in historical evidence. GitHub Actions may start scheduled workflows
late; the configured time is a target.

## Proposal allocation categories — October 26, 2026

Risk generation, the Dashboard portfolio planner and custom proposal editor use
five separate classes: Treasuries, Investment grade (AAA–BBB), High yield (BB),
Equity (classified equity ETFs) and Gold (GLD). Individual stocks, bond ETFs and
other commodity ETFs cannot enter new proposals. Gold never consumes equity
weight. Planner payloads carry the five direct percentages totaling 100%; no
separate credit suballocation or cash sleeve is accepted. Matching archetype
presets retain the archetype position count and 4.5% cap. Edited plans retain
their requested weights and count and fail explicitly when infeasible.

Custom preview and save validate each security against its selected category and
market feed; duplicate holdings and misclassified bonds or ETFs fail before save.
Custom capital weights are editable independently of archetype counts and caps.
The editor splits each class across its selected holdings. Saved proposals,
Dashboard/Hub charts, summaries and exports keep investment grade and Gold
separate for every source type. Newly saved records store those buckets directly.
Existing holdings and metadata are preserved on read; this follow-up performs no
additional data rewrite. The 13 risk-profile proposals were regenerated on October
26 under the separate authorized operation tracked in API issue #304.

Manual UI and export acceptance belongs to the owner, who requested stopping
automated application tests. Type checking and source review do not establish
that acceptance.
