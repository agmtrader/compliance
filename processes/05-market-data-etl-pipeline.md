# Market Data ETL Pipeline

## Business Purpose
Refresh AGM market-data resource files used by reporting and analytics by running the configured extract, backup, and transform stages for the market-data pipeline.

## Trigger / Frequency
- Trigger: GitHub Actions workflow calls `POST /token` and then `GET /etl/market_data`.
- Frequency: Daily at 10:00 UTC / 4:00 AM Costa Rica time, plus manual workflow dispatch.

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

## Step-by-Step Workflow
1. The workflow requests an API token and calls `/etl/market_data`.
2. The API resolves the `market_data` ETL configuration.
3. The ETL runner executes `extract`, `backup`, and `transform` stages in order.
4. Extract pulls each configured market-data source into the batch area and summarizes step success, skip, or failure.
5. Backup renames batch files, moves them into the resource structure, and clears the batch folder.
6. Transform converts the resource files into downstream market-data outputs consumed by reporting logic.
7. The route returns a stage overview with summary counts and step details.
8. GitHub Actions formats the overview and sends a success or failure email.

## Outputs / Records Created
- Refreshed raw market-data files in Google Drive
- Updated transformed market-data resources
- ETL overview payload returned by the API
- Workflow notification email

## Exception Paths / Failure Handling
- Token failure: workflow exits before the route call.
- Missing pipeline config: route fails immediately.
- Extract, backup, or transform failure: overview captures the failing step and workflow can terminate with failure.
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

## Last Reviewed
- Status: draft
- Date: 2026-06-16
- Reviewer: Codex initial draft
