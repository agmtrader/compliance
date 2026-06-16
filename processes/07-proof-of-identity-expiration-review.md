# Proof of Identity Expiration Review

## Business Purpose
Identify proof-of-identity documents that are missing expiration data, already expired, or approaching expiration so the document metadata can be corrected or remediated.

## Trigger / Frequency
- Trigger: User opens the Dashboard `POI Expiration Review` page.
- Frequency: On demand.

## Systems Involved
- `agm-dashboard`
- `agm-api`
- Contacts, account-contact links, accounts, and contact-document records

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Contacts table
- Account-contact links
- Accounts table
- Contact-document metadata with category, issue date, and expiration date

## Step-by-Step Workflow
1. A user opens the `POI Expiration Review` page.
2. The page loads contacts, account-contact links, accounts, and contact-document data.
3. It filters the document list down to records categorized as `Proof of Identity`.
4. For each POI document, it resolves the associated account and contact context.
5. It parses the expiration date and classifies the record as:
   - missing
   - expired
   - expiring soon
   - valid
6. The page calculates `days_to_expiration` for records with parseable expiration dates.
7. Users can filter the table by expiration state.
8. Users can open the edit dialog and update document metadata such as category, type, issue date, expiration date, and comment.

## Outputs / Records Created
- Operational review rows for POI documents
- Updated contact-document metadata when the edit flow is used
- Dashboard export `proofs-of-identity-review.csv`

## Exception Paths / Failure Handling
- Data-load failure: page shows an error toast and no rows are displayed.
- Invalid or missing expiration dates: row is classified as `missing` instead of causing the page to fail.
- Update failure: the edit dialog reports failure and no row refresh occurs until the user retries.

## Controls / Verification Points
- Detective control: explicit classification of missing, expired, and expiring-soon POI documents.
- Detective control: exportable review grid allows follow-up outside the page.
- Preventive control: document metadata edits flow through the API instead of spreadsheet-based manual edits.

## Evidence to Retain
- Updated contact-document records
- Dashboard review export
- Page audit screenshots or review notes, if retained operationally

## Related Code / Pages / Routes
- Entry surfaces: `agm-dashboard/src/app/(dashboard)/(services)/(tools)/(private)/(compliance)/proofs-of-identity-review/page.tsx`
- Supporting modules: `agm-dashboard/src/components/dashboard/tools/private/reporting/proofs-of-identity-review/ProofsOfIdentityReviewPage.tsx`
- Downstream side effects: contacts/documents update API

## Last Reviewed
- Status: draft
- Date: 2026-06-16
- Reviewer: Codex initial draft
