# IBKR Reference Data Downloads

## Business Purpose
Load IBKR forms and enumeration datasets used to drive account-opening choices and account-servicing options in AGM interfaces.

## Trigger / Frequency
- Trigger: Hub or Dashboard needs reference data for account-opening or servicing flows.
- Frequency: On demand.

## Systems Involved
- `agm-hub`
- `agm-dashboard`
- `agm-api`
- IBKR reference-data endpoints

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- API token
- Requested form numbers for the forms route
- IBKR availability for:
  - forms and disclosures
  - product-country bundles
  - financial ranges
  - business and occupation types

## Step-by-Step Workflow
1. The onboarding or servicing UI requests reference data through shared account utility functions.
2. `/accounts/ibkr/forms` downloads agreements and disclosure forms used during account opening for the requested form numbers.
3. `/accounts/ibkr/product_country_bundles` loads the available product-country bundle enums used for products by market.
4. `/accounts/ibkr/financial_ranges` loads the IBKR financial-range types used in financial-information capture.
5. `/accounts/ibkr/business_and_occupation` loads the IBKR business and occupation types used in applicant profiling.
6. The UI stores the responses locally and uses them to populate form selectors and downstream request payloads.

## Outputs / Records Created
- In-memory reference data used by Hub and Dashboard
- Agreement form content and metadata used during account opening

## Exception Paths / Failure Handling
- Centralized API exception reports retain request/error IDs, HTTP method, route template, endpoint, status/code and exception types with stack file/line/function locations. They omit query values, client IPs, error details, exception messages and source-line text; client-facing controlled errors retain their existing message/details. This redaction covers the shared exception wrapper, not every independent integration/audit log.
- Dashboard preserves controlled API error messages across server-action boundaries for toast display. Product-country-bundle failures propagate to the caller instead of returning null. Reference-data permission/configuration failures remain reportable 502 responses; known request validation remains a structured 400/422.
- IBKR reference-data failure: onboarding or servicing forms cannot render the affected selector set correctly.
- Missing or malformed form numbers: forms response may be incomplete.
- UI fetch failure: pages surface error toasts and the workflow remains blocked until retried.

## Controls / Verification Points
- Preventive control: shared utility functions centralize the reference-data requests instead of duplicating endpoint logic.
- Detective control: users cannot proceed normally when critical reference data fails to load because the page surfaces an error condition.

## Evidence to Retain
- API request and response logs for reference-data calls
- Agreement form metadata returned to the UI
- Screenshots or exports of the loaded reference values where needed for review

## Related Code / Pages / Routes
- Entry surfaces: `agm-hub/src/components/hub/apply/IBKRApplicationForm.tsx`, `agm-dashboard/src/utils/clients/account.ts`, `agm-hub/src/utils/clients/account.ts`
- Supporting modules: `agm-api/src/app/clients/accounts.py`
- Downstream side effects: `/accounts/ibkr/forms`, `/accounts/ibkr/product_country_bundles`, `/accounts/ibkr/financial_ranges`, `/accounts/ibkr/business_and_occupation`

## Last Reviewed
- Status: draft
- Date: 2026-06-16
- Reviewer: Codex initial draft

## Daily Agreement Preload — Local Validation, October 14, 2026
The connector's `submit_all_agreements` can prepare the configured 24 English agreement/disclosure PDFs for the IBKR daily `processDocuments` endpoint. This is distinct from per-account pending tasks and customer signatures. It requires an explicit master account. Tax form5001 remains per application. Preparation verifies returned form number, exact filename, PDF header, decoded length/SHA and file/request limits. Incomplete preparation aborts the whole batch before submission. `dry_run=True` performs real form reads and validation but never signs/posts the daily submission and never reports acceptance.

Only one explicit per-form provider outcome can establish acceptance; absent, missing or conflicting outcomes remain unconfirmed. The daily agreement POST has a bounded timeout and is excluded from automatic retry because a timeout can leave acceptance unknown. Logs do not retain PDF payload prefixes. The owner authorized the actual daily PDF upload on October 14. POST /actions/submit_daily_agreements invokes the connector with dry_run=False; the separate preparation route retains dry_run=True. The daily_agreements.yaml workflow calls the submission route at 11:00 UTC (05:00 Costa Rica) for I6413690; manual dispatch can select F10740574. It authenticates through the existing AGM API secrets and enforces API user scope. Success requires all 24 configured forms to have explicit successful IBKR outcomes; HTTP 200 alone is insufficient. Concurrency serializes runs and no workflow submission retries are configured. GitHub Actions retains the final per-form summary and run URL and sends standard success/failure emails from aa@agmtechnology.com to cr@agmtechnology.com and as@agmtechnology.com. Interrupted or timed-out submissions have unknown acceptance and require inspection before manual replay. The preparation workflow was already published; this live upgrade must pass CI and deploy its API route before execution. Hosted broker acceptance and any application-level per-master/day acceptance gate remain unverified. Existing application submission continues including its own agreement PDFs.

Live read-only primary-master check returned24 valid PDFs and no usable PDF for configured9902. All valid returned files matched filenames, lengths and SHA; aggregate PDF bytes6702357. This is download/preparation evidence, not upload acceptance. The owner explicitly excluded9902 from the daily preload list on October14; the remaining24 forms are unchanged. This exclusion does not establish whether9902 is mandatory for an individual client. Keep coordinated Hub72/API197 open until a runner with per-master/day evidence exists and explicit broker acceptance is verified. No daily agreement submission was performed by this validation.
