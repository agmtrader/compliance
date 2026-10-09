# Market Data ETL Pipeline

## Business Purpose
Refresh AGM market-data resource files used by reporting and analytics by running the configured extract, backup, and transform stages for the market-data pipeline.

## Trigger / Frequency
- Trigger: GitHub Actions workflow calls `POST /token` and then `GET /etl/market_data`.
- Frequency: Daily at 01:00 UTC / 7:00 PM on the previous calendar day in Costa Rica, plus manual workflow dispatch. This is a schedule-timing test; GitHub Actions can still create the run late.

## Systems Involved
- `agm-api`
- GitHub Actions
- Google Drive batch and resources folders
- External market-data source functions configured in the ETL pipeline

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- API auth token
- Valid `market_data` ETL config
- Access to configured market-data sources and Google Drive
- Functional transform steps for market-data output
- IBKR market-data session can be established, bootstrapped with a successful
  `/iserver/accounts` response, and closed after extraction

## Step-by-Step Workflow
1. The workflow requests an authenticated AGM user token and calls `/etl/market_data`.
2. The API resolves the `market_data` ETL configuration.
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
7. The route returns a stage overview with summary counts and step details.
8. GitHub Actions persists the workflow output, formats the overview (including stage counts, backup sub-step counts, and failure details), and sends a success or failure email. If the API call fails before an overview is available, the email includes the captured request error or directs the reviewer to the workflow logs.

## Outputs / Records Created
- Refreshed raw market-data files in Google Drive
- Updated transformed market-data resources
- ETF and stock resource rows with populated `Current Yield` when `DEV_MODE=true` and
  five years of bars are available
- ETF and stock resource rows with an empty `Current Yield` when `DEV_MODE=false`
- Per-symbol resolution, snapshot, and development-only history failures logged with
  ticker and conid; symbols with missing or invalid snapshot rows are omitted while
  valid symbols continue processing
- ETL overview payload returned by the API
- Workflow notification email containing the overall result, extract/backup/transform stage statuses and counts, and any reported failures

## Exception Paths / Failure Handling
- Centralized API exception reports retain request/error IDs, HTTP method, route template, endpoint, status/code and exception types with stack file/line/function locations. They omit query values, client IPs, error details, exception messages and source-line text; client-facing controlled errors retain their existing message/details. This redaction covers the shared exception wrapper, not every independent integration/audit log.
- An expired IBKR trading session or the known accounts-bootstrap prerequisite returns a structured 409 with reconnect guidance, without an exception report. Stage/symbol failure outcomes remain visible. IBKR service authentication, unknown upstream failures, and missing or duplicate reporting resources remain reportable.
- Token failure: workflow exits before the route call.
- Missing pipeline config: route fails immediately.
- Extract, backup, or transform failure: overview captures the failing step and workflow can terminate with failure.
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
- Workflow retry behavior is short compared with the clients pipeline and should be reviewed if repeated source instability occurs.

## Controls / Verification Points
- Preventive control: pipeline name must map to a defined config.
- Detective control: each stage reports total, successful, skipped, and failed step counts.
- Detective control: backup-step counts are included in workflow summary output.
- Detective control: workflow logs and email preserve the ETL outcome for review.

## Evidence to Retain
- GitHub Actions run history for `etl_market_data.yaml`
- ETL overview JSON in workflow logs
- Resource folder outputs after the run
- Success or failure notification email

## Related Code / Pages / Routes
- Entry surfaces: `agm-api/.github/workflows/etl_market_data.yaml`, `agm-api/src/app/tools/private/etl.py`
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
