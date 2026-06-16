# Documents Review Workflow

## Business Purpose
Provide an operational and compliance review surface for recently opened accounts to identify missing holder documents, assign responsible reviewers, upload missing documents, and send missing-document email reminders.

## Trigger / Frequency
- Trigger: User opens the Dashboard `Documents Review` page.
- Frequency: On demand.

## Systems Involved
- `agm-dashboard`
- `agm-api`
- Accounts, contacts, account-contact links, contact documents, and review-responsible records
- Daily IBKR details backup
- Email service for missing-document reminders

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Accounts and contacts data
- Account-contact link records with optional `entity_id`
- Contact-document metadata
- IBKR details backup with account open date, applicant type, and associated-person roles
- User list for reviewer assignment

## Step-by-Step Workflow
1. A user opens the Dashboard `Documents Review` page.
2. The page loads contacts, account-contact links, accounts, IBKR details backup, contact documents, reviewer assignments, and users in parallel.
3. The page filters to accounts opened within the last five years using the IBKR backup `dateOpened`.
4. For each account-contact relationship, it matches the linked contact to the relevant account and excludes trusted contacts identified in IBKR associated-person roles.
5. It determines whether each contact has the required primary document:
   - natural person: `Proof of Identity`
   - company contact: `Proof of Existence`
6. It also checks for `Proof of Address` and `Source of Wealth`.
7. It builds one review row per account/contact combination showing missing-document status and any assigned responsible user.
8. Reviewers can assign or update the responsible user and review comment through the `document_review_responsibles` upsert flow.
9. Reviewers can upload a missing document, including metadata such as category, type, issue date, and expiration date.
10. Reviewers can send a missing-documents email using the page-level email dialog.

## Outputs / Records Created
- Review rows derived from current source records
- Upserted `document_review_responsible` records
- Uploaded contact documents and related metadata
- Missing-documents emails to clients

## Exception Paths / Failure Handling
- Data-load failure: the page shows an error toast and the review grid is not populated.
- Upload failure: upload dialog remains in the user flow and the user receives a failure toast.
- Missing-document email failure: email dialog surfaces failure and no success notification is shown.
- Incorrect or incomplete linkages between contacts and accounts can produce false positives and must be manually reviewed.

## Controls / Verification Points
- Preventive control: trusted contacts are explicitly excluded from required-document review.
- Preventive control: only accounts opened within the last five years are reviewed by this page.
- Detective control: missing-document logic is visible row by row and can be exported or reviewed manually.
- Detective control: reviewer assignment and comment history are stored in dedicated records instead of ad hoc notes.

## Evidence to Retain
- `document_review_responsible` records
- Uploaded document metadata and file records
- Missing-documents emails
- Dashboard exports such as `documents-review.csv`

## Related Code / Pages / Routes
- Entry surfaces: `agm-dashboard/src/app/(dashboard)/(services)/(tools)/(private)/(compliance)/documents-review/page.tsx`
- Supporting modules: `agm-dashboard/src/components/dashboard/tools/private/reporting/documents-review/DocumentsReviewPage.tsx`
- Downstream side effects: accounts, contacts, contact documents, document-review-responsibles, `reporting/ibkr_details`, missing-documents email route

## Last Reviewed
- Status: draft
- Date: 2026-06-16
- Reviewer: Codex initial draft
