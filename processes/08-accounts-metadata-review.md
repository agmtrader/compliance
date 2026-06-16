# Accounts Metadata Review and Analysis

## Business Purpose
Provide a consolidated operational view of account metadata, including NAV, status, advisor, contact labeling, fee-template summary, devices, and email alignment, so users can review account populations and identify anomalies.

## Trigger / Frequency
- Trigger: User opens the Dashboard `Accounts` page.
- Frequency: On demand.

## Systems Involved
- `agm-dashboard`
- `agm-api`
- Accounts-with-metadata API
- Advisors and contacts data

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- `ReadAccountsWithMetadata`
- Advisors list
- Contacts list
- Derived fields returned by the metadata endpoint, such as alias, status, NAV, devices, fee template summary, and client email

## Step-by-Step Workflow
1. A user opens the Dashboard `Accounts` page.
2. The page loads accounts-with-metadata, advisors, and contacts in parallel.
3. The client normalizes core account fields for display consistency, including alias, status, NAV, and client email address.
4. It derives display labels for linked contacts using direct account fields, linked contact records, and fallback matching logic.
5. It exposes filters across NAV ranges, advisor, SLS devices, status, temporal email matching, and application state.
6. Users review the filtered account population and navigate into specific account or application detail pages as needed.

## Outputs / Records Created
- No direct record creation by the listing page itself
- Operationally filtered views of accounts
- Navigation into account- or application-level follow-up screens

## Exception Paths / Failure Handling
- Data-load failure: page shows an error toast and remains unavailable.
- Missing contact metadata: page falls back through multiple naming and email-matching heuristics before showing `-`.
- Inconsistent metadata returned by the API can affect analysis quality but does not block the grid from rendering.

## Controls / Verification Points
- Detective control: cross-account filtering surfaces unusual states such as zero NAV, blank metadata, and mismatched advisor/device values.
- Detective control: contact-name and email comparison logic helps reveal inconsistent account metadata.
- Preventive control: metadata normalization reduces display-time ambiguity caused by null or nonstandard source values.

## Evidence to Retain
- Review exports or screenshots retained by operations, if used
- API responses from `ReadAccountsWithMetadata`
- Follow-up changes made in downstream account screens

## Related Code / Pages / Routes
- Entry surfaces: `agm-dashboard/src/app/(dashboard)/(services)/(clients)/accounts/page.tsx`
- Supporting modules: `agm-dashboard/src/components/dashboard/clients/accounts/AccountsPage.tsx`, `agm-dashboard/src/utils/clients/account.ts`
- Downstream side effects: navigation into `AccountOrApplicationPage`

## Last Reviewed
- Status: draft
- Date: 2026-06-16
- Reviewer: Codex initial draft
