# IBKR Reference Data Downloads

## Business Purpose
Load IBKR forms and enumeration datasets used to drive account-opening choices and account-servicing options in AGM interfaces.

## Trigger / Frequency
- Trigger: Hub or Dashboard needs reference data for account-opening or servicing flows.
- Frequency: On demand for reference reads; daily at scheduled 11:00 UTC / 05:00 Costa Rica for the agreement PDF preload. GitHub schedule dispatch can be delayed.

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
- IBKR OAuth token requested with `enumerations.read`, including for the complex-asset-transfer broker/custodian enumeration used by Dashboard. Deploy the API scope change and obtain a fresh token before checking hosted behavior.
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
7. The separate Daily IBKR Agreement Submission workflow calls `/actions/submit_daily_agreements` for its configured master. Its shared `agm-api/src/lib/ibkr_daily_agreements.json` manifest contains 29 standard agreements, including forms 3230, 3044, 4615, 4155 and 4402. Existing exclusions of tax form 5001, form 9902 and KYC upload forms remain unchanged. This preloads generic PDFs; it does not collect client signatures or establish completion of account tasks.
8. The API validates each PDF's identity, length and SHA-1 and submits only a complete prepared set. It partitions that set in manifest order using a conservative 4,000,000-byte ceiling on each final signed JWT, with allowance for the second base64 layer, claims, headers and signature. All batches are signed and checked before the first POST. This is an application ceiling below the [documented IBKR 20 MB request maximum](https://www.interactivebrokers.com/campus/ibkr-api-page/web-api-account-management/), not a claim about the gateway's exact limit. A real combined 29-form request returned HTTP 413 on October 21. Preparation-only mode returns the batch plan without signing or submitting documents. Submission proceeds sequentially, once per batch; a rejected, malformed, timed-out or unconfirmed outcome stops subsequent batches. Successful earlier batches must be reconciled before any manually authorized replay. The response/receipt retains each completed batch's form numbers, request size and available request ID/submission date; error details distinguish prior accepted, failed/uncertain and not-yet-submitted forms. Full success still requires explicit successful outcomes for all 29 forms. The raw provider envelope, signed JWT and PDF payloads are omitted.
9. Check GitHub Actions → Daily IBKR Agreement Submission → the selected run's Summary for the outcome. Updated runs retain a `daily-agreements-receipt` artifact for 30 days containing a text summary and a JSON receipt with the master, required forms, outcomes and available request metadata. Authentication failures before an API response can leave only the failure summary. Inspecting these saved results does not replay the submission; do not call the submit route merely to check a previous run.
10. The workflow verifies the exact manifest and per-form results rather than a hard-coded count or the API's reported expected list alone. A server still returning only the previous 24 forms fails verification. Deploy the expanded API manifest before enabling the updated workflow. The artifacts record that run's submission outcome; they are not a read-only latest-preload endpoint or proof that an existing account's tasks are complete.

## Outputs / Records Created
- In-memory reference data used by Hub and Dashboard
- Agreement form content and metadata used during account opening
- Daily per-master preload run summary and downloadable receipt, with explicit per-form outcomes

## Exception Paths / Failure Handling
- Centralized API exception reports retain request/error IDs, HTTP method, route template, endpoint, status/code and the propagated failure with stack file/line/function locations. Request context omits query values, client IPs and source-line text. An explicit diagnostic allowlist retains bounded IBKR status, error code/message and request/trace ID, plus agreement batch sizes and accepted/uncertain/unsubmitted form identifiers. Credential-looking values, emails, account identifiers and long encoded data are redacted from these diagnostic fields; arbitrary provider envelopes and detail dictionaries are excluded. This rule covers the shared diagnostic formatter, not every independent integration/audit log.
- Actions prints the same sanitized failure diagnostic to its console, Summary and 30-day receipt. Authentication errors retain API correlation/error information and explicitly state that no broker submission was attempted. Submission errors preserve API and available IBKR correlation separately. Non-JSON gateway failures retain a bounded sanitized explanation. Neither failed validation nor a successful HTTP status alone establishes broker acceptance. No automatic POST retry, failure-triggered batch shrinking or replay is performed.
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
