# Compliance Reference Data Refresh

## Business Purpose
Refresh and retain the sanctions reference files used by AGM contact screening so the screening process can compare daily snapshots and read a current resource copy. This document describes the control as it operates now; FATF jurisdiction data is not part of this automated refresh.

## Trigger / Frequency
- Trigger: The scheduled Clients ETL GitHub Actions workflow requests an API token and calls `GET /etl/clients`; the route can also be dispatched manually.
- Frequency: Daily at 10:00 UTC / 4:00 AM Costa Rica time.

## Systems Involved
- `agm-api`
- GitHub Actions
- Google Drive batch, backup, and resource folders
- U.S. Treasury OFAC Sanctions List Service
- UK Foreign, Commonwealth & Development Office sanctions list
- United Nations Security Council consolidated sanctions list

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Working API authentication and access to `/etl/clients`
- Network access to the three official source URLs configured in `private/etl.py`
- Google Drive access to the shared batch folder, the three list-specific backup folders, and the reporting resources folder
- Sanctions file configurations for `ofac_sdn_list`, `uk_sanctions_list`, and `un_sanctions_list`

## Step-by-Step Workflow
1. GitHub Actions calls the Clients ETL route. Sanctions refreshes run as three extract steps inside the broader clients pipeline rather than as an independent job.
2. The OFAC extractor downloads the SDN and consolidated primary CSV exports, concatenates them under a common header, parses the rows, and rewrites names marked as individuals from `last, first` order to `first last` where possible.
3. The UK extractor downloads the UK Sanctions List CSV, requires a header beginning with `Last Updated,Unique ID,`, parses all rows as strings, and uploads the result to the batch folder.
4. The UN extractor downloads and parses the consolidated XML list, creates rows for individuals and entities, joins name components, retains aliases in a pipe-separated field, and uploads a timestamped CSV to the batch folder.
5. The ETL backup stage renames matching batch files to list-specific names containing the Costa Rica run timestamp.
6. Before moving a new file, the sorter deletes files in the target backup folder whose Google Drive creation date is the current date. It then moves the new file into that list's backup folder.
7. The transform stage selects the most recent matching backup for each list and publishes a stable resource filename: `ofac_sdn_list.csv`, `uk_sanctions_list.csv`, or `un_sanctions_list.csv`.
8. Existing resource files with the same stable name are deleted before the replacement resource is uploaded.
9. The API returns an ETL overview with extract, backup, and transform step results. The workflow formats aggregate stage counts and sends a generic Clients ETL success or failure email.
10. The later Daily Screening Run locates backup files by date-prefixed filename and compares today with the previous calendar day.

## Outputs / Records Created
- Timestamped OFAC, UK, and UN backup files in separate Google Drive folders
- Stable OFAC, UK, and UN resource files used by screening
- Clients ETL response containing step-level outcomes
- GitHub Actions run logs and a generic Clients ETL notification email

## Exception Paths / Failure Handling
- OFAC download or parse failure is recorded as a failed extract step; the OFAC requests currently have no explicit timeout and do not call `raise_for_status`.
- UK or UN HTTP, format, XML, or parse failures are recorded as failed extract steps.
- An individual extract failure does not stop the backup and transform stages from running. Those stages can therefore process the most recent previously retained backup.
- The API returns HTTP `200` with overall status `partial` when stages or reports are not fully successful.
- The GitHub workflow treats any HTTP `200` response as a successful job and does not fail when the returned overall status is `partial`.
- Extract steps with no configured extractor are marked `skipped`; the current extract summary also counts those skipped steps in its `failed` total and makes the extract stage `partial`.
- FATF high-risk and increased-monitoring jurisdictions are not downloaded by this process. Screening uses hardcoded country sets in the contacts service.

## Controls / Verification Points
- Detective control: each extracted list has a timestamped backup and a stable resource copy.
- Detective control: the UK extractor validates the expected CSV header and the UN extractor requires parseable XML.
- Detective control: the ETL response contains per-list extract and transform entries that can be inspected in workflow logs.
- Current limitation: the success email does not identify each compliance list's status, source publication date, row count, checksum, or resource version.
- Current limitation: the process does not validate minimum row counts, unexpected count changes, source publication dates, or source-provided versions before replacing the stable resource.

## Evidence to Retain
- GitHub Actions run for `etl_clients.yaml`
- Full ETL overview JSON from the API response, including the three compliance-list steps
- Timestamped list backups in Google Drive
- Stable resource files and their Google Drive metadata
- Success or failure email from the Clients ETL workflow

## Related Code / Pages / Routes
- Entry surfaces: `agm-api/.github/workflows/etl_clients.yaml`, `agm-api/src/app/tools/private/etl.py`
- Supporting modules: `agm-api/src/components/tools/private/etl.py`, `agm-api/src/utils/connectors/drive.py`
- Downstream side effects: `agm-api/src/components/tools/public/reporting.py`, `agm-api/src/components/clients/contacts.py`, `agm-api/src/components/tools/private/actions.py`

## Last Reviewed
- Status: draft
- Date: 2026-07-15
- Reviewer: Codex current-state code review
