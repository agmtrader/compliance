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
