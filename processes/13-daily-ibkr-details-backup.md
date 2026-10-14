# Daily IBKR Details Backup and Reporting Feed

## Business Purpose
Provide a backed-up dataset of IBKR account details, including account metadata, financial information, and account-holder associations, for downstream review and analysis workflows.

## Trigger / Frequency
- Trigger: The daily Clients ETL refreshes the backup and resource copy; reporting routes and Dashboard pages read the stable resource on demand.
- Frequency: Daily ETL at 4:00 AM Costa Rica time, plus on-demand consumption.

## Systems Involved
- `agm-api`
- Google Drive reporting resources
- `agm-dashboard` review pages that consume the dataset

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Latest IBKR details backup file stored in the reporting resources location
- Working Google Drive access

## Step-by-Step Workflow
1. During Clients ETL extract, `extract_account_details_backup` loads the most recent valid account-details JSON backup; companion refresh reports are excluded. An empty folder starts with an empty population; malformed backups fail explicitly.
2. The refresh population is the union of normalized internal account numbers and prior-backup account IDs. Every eligible existing account is re-fetched, including prior-only accounts. Blank internal masters can use the master already recorded in that account's prior IBKR response. Conflicting or unsupported masters and invalid IDs produce explicit failure outcomes; no default credentials or silent skip list is used.
3. Four worker requests use immutable bearer tokens, paced at five request starts per second globally, with 10-second connect and 45-second read timeouts. Tokens are obtained/refreshed outside worker requests. Requests are not automatically retried.
4. Successful, identity/master-validated details replace the prior record and receive `_agm_refresh` freshness metadata. Failed accounts retain prior records as stale, with last-success time retained when known (legacy unknown times remain null). Failure without prior data is recorded in the companion report.
5. Upload the refreshed backup before the companion `.refresh.json` report. Do not delete the prior backup before upload. The date-stamped name is evaluated for the current ETL run. Per-account failures yield `partial`, with account-level outcomes and counts in the extract overview.
6. The existing transform publishes the details list to `ibkr_account_details.json`, excluding companion refresh reports. Existing downstream report consumers continue to read the details list.
7. The standalone `python -m src.jobs.clients_etl` runner calls the existing clients pipeline directly and returns a nonzero exit code for partial/failed results. Local-details-only mode invokes the same extractor, reads live sources and saves private local JSON without remote writes. Cloud Job provisioning and trigger cutover remain pending.

## Outputs / Records Created
- Daily backup dataset of IBKR details
- API responses consumed by Dashboard workflows
- Derived review rows on downstream pages

## Exception Paths / Failure Handling
- Per-account refresh failure: retain prior details as explicitly stale; report an absent record when no prior data exists. Extract and overall pipeline become partial. Upload/report failure also prevents success.
- Missing or stale backup file: downstream review pages may fail to load or may produce incomplete classifications.
- Google Drive access failure: the reporting helper cannot provide the dataset.
- A Clients ETL `partial` result can still produce a successful GitHub workflow notification.

## Controls / Verification Points
- Detective control: Dashboard review workflows rely on the same backed-up source rather than ad hoc manual extraction.
- Detective control: downstream pages can be cross-checked against the latest backup date when investigating review anomalies.

## Evidence to Retain
- Latest IBKR details backup file
- Reporting API logs for `get_ibkr_details`
- Dashboard reviews that consume the backup

## Related Code / Pages / Routes
- Entry surfaces: `agm-api/.github/workflows/etl_clients.yaml`, `agm-api/src/components/tools/private/etl.py`, `agm-api/src/components/tools/public/reporting.py`
- Supporting modules: `agm-dashboard/src/components/dashboard/tools/private/reporting/accounts-audit/AccountsAuditReport.tsx`
- Downstream consumers: Accounts Audit and other account-analysis workflows

## Last Reviewed
- Status: draft
- Date: 2026-07-15
- Reviewer: Codex current-state code review

## Runner acceptance and rollout gates

Local tests do not establish production publication or scheduling. Before enabling the separate Cloud Run Job, verify durable overview publication, service permissions, shared batch-folder single-execution protection, full backup/transform writes, and the first scheduled run. Existing API/GitHub schedules are unchanged by local implementation. Brand-new IBKR account discovery, automatic account/contact persistence, and removal of clients.xls are outside this change. Existing screening caches are independent of this refresh and require separate freshness handling.
