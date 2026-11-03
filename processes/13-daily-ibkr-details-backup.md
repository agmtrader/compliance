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
1. During Clients ETL extract, `extract_account_details_backup` reads the most recent file from the account-details backup folder.
2. It compares the detail population with internal accounts that have IBKR account numbers and calls the IBKR details endpoint for missing accounts when a master account is available and the account is not in the hardcoded skip set.
3. Per-account failures and missing master accounts are logged and skipped while other accounts continue.
4. The enriched list is uploaded under a backup name based on the previous business-day date calculated when the API worker initialized.
5. The transform stage selects the most recent backup and replaces the stable `ibkr_account_details.json` resource.
6. The reporting helper `get_ibkr_details` reads the stable resource and exposes it through the reporting layer.
7. Dashboard pages use the resource for account dates, financial information, applicant type, associated-person roles, fee templates, and other account context.

## Outputs / Records Created
- Daily backup dataset of IBKR details
- API responses consumed by Dashboard workflows
- Derived review rows on downstream pages

## Exception Paths / Failure Handling
- Per-account enrichment failure: the remaining details are still saved and the failed account is absent from the refreshed result.
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

## Accepted coverage and cloud execution

Owner accepted current incomplete account coverage on 2026-10-14. The cloud runner compares failed account/error pairs with the 208-pair baseline from the full local validation. Accepted pairs preserve raw partial status and stale/absent evidence but may complete operationally when all other ETL stages and publication succeed. New gaps are failures, not silently accepted. The job retains final overviews in private GCS and companion details reports in Drive. Scheduled clients cutover and first scheduled-run acceptance are tracked in API #86/#94; account discovery and contact/user reconciliation are deferred to API #294. Market migration stays in #86.

## Owner-directed refresh exclusions — November 3, 2026

Supersedes the earlier missing/unsupported-master failure classification. Accounts without a usable master (including no master in prior broker details) and unsupported parent mappings, including the two known nested-STL cases, are skipped before broker requests. Reasons and counts remain visible separately from failures/freshness; they are not classified as closed or successfully refreshed. Unsupported mappings remain deferred pending IBKR guidance and no parent mapping is rewritten. Conflicting mappings and malformed identities remain failures.

Previously broker-confirmed delinked accounts are skipped before repeat reads only when the prior refresh metadata retains the account_delinked reason and a valid nonfuture delinking date. New exact-identity delinking tombstones retain the existing validation. Historical details remain stale with last-success and delinking evidence; no deletion occurs. HTTP403 accounts remain eligible each ordinary run, trying the mapped credential then one alternate credential. Root-master response validation is unchanged and remains a separate unresolved case. This policy change is local pending release and ordinary-run verification.

### Root master exclusion — November 3, 2026

Owner additionally excludes the FD master account itself, I6413690. Skip its details request as root_master_account, report the count separately and retain any historical record as stale. This supersedes the earlier unresolved-root-validation gate for this account; it does not accept a parentless response as fresh or relax exact identity/parent validation for subaccounts. The other38HTTP403 accounts remain eligible with one alternate credential attempt. For the observed824-account population, expected classification is622fresh,38failed and164skipped; release and ordinary-run verification remain pending.
