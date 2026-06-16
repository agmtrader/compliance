# Daily IBKR Details Backup and Reporting Feed

## Business Purpose
Provide a backed-up dataset of IBKR account details, including account metadata, financial information, and account-holder associations, for downstream review and analysis workflows.

## Trigger / Frequency
- Trigger: Reporting layer reads the latest available IBKR details backup.
- Frequency: Daily backup consumption, plus on-demand access by Dashboard review pages.

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
1. The reporting helper `get_ibkr_details` reads the latest available IBKR details backup from the configured storage location.
2. The returned dataset is exposed through the reporting layer for API consumers.
3. Dashboard pages such as Documents Review use the backup to obtain account open dates, applicant type, associated-person roles, and other account context not stored directly on the local account record.
4. The dataset acts as a stable daily snapshot used for review logic rather than a real-time IBKR read during every page action.

## Outputs / Records Created
- Daily backup dataset of IBKR details
- API responses consumed by Dashboard workflows
- Derived review rows on downstream pages

## Exception Paths / Failure Handling
- Missing or stale backup file: downstream review pages may fail to load or may produce incomplete classifications.
- Google Drive access failure: the reporting helper cannot provide the dataset.

## Controls / Verification Points
- Detective control: Dashboard review workflows rely on the same backed-up source rather than ad hoc manual extraction.
- Detective control: downstream pages can be cross-checked against the latest backup date when investigating review anomalies.

## Evidence to Retain
- Latest IBKR details backup file
- Reporting API logs for `get_ibkr_details`
- Dashboard reviews that consume the backup

## Related Code / Pages / Routes
- Entry surfaces: `agm-api/src/components/tools/public/reporting.py`
- Supporting modules: `agm-dashboard/src/components/dashboard/tools/private/reporting/documents-review/DocumentsReviewPage.tsx`
- Downstream side effects: documents review and other account-analysis workflows

## Last Reviewed
- Status: draft
- Date: 2026-06-16
- Reviewer: Codex initial draft
