# Advisor Contact Linking

## Business Purpose
Maintain the advisor-to-contact relationship used to resolve advisor names, email addresses, and copied email recipients without relying on automatic name matching.

## Trigger / Frequency
- Trigger: An authorized operator opens the Dashboard Advisors page and selects `Link contact` for an advisor.
- Frequency: On demand when an advisor has no contact, has the wrong contact, or needs a newly created contact.

## Systems Involved
- `agm-dashboard`
- `agm-api`
- Supabase PostgreSQL `advisor` and `contact` tables

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Existing advisor record
- Existing contact record, or the advisor's name and email for a new contact
- Valid authenticated Dashboard/API session

## Step-by-Step Workflow
1. The Advisors page loads current advisor and contact records and displays each advisor's linked contact or `Not linked`.
2. The operator opens `Link contact` for the intended advisor. The dialog shows the current linked contact before any change is made.
3. For an existing contact, the operator searches by name, email, phone, or contact ID and selects the intended record.
4. Save remains disabled until the selected contact differs from the current link.
5. The API verifies that both the advisor and selected contact exist, updates only `advisor.contact_id`, reads the advisor back, and rejects the request if the saved link cannot be verified.
6. If no suitable contact exists, the operator selects `Create new`. The form prefills the advisor name and requires a valid email; phone is optional.
7. The Dashboard blocks a new contact when the loaded contact list already contains the same email. The API repeats required-field, email-format, advisor-existence, and exact-email duplicate checks.
8. The API locks the advisor row, creates the contact, and assigns its ID to `advisor.contact_id` in one transaction. Any failure rolls back both operations.
9. The API returns the saved advisor and contact. The Dashboard updates the table and contact search list and shows a success toast.

## Outputs / Records Created
- Updated `advisor.contact_id` for an existing-contact link
- New `contact` row plus updated `advisor.contact_id` for a create-and-link action
- Updated advisor `updated` timestamp

## Exception Paths / Failure Handling
- Missing or invalid advisor/contact IDs: request is rejected and no link changes.
- Unknown advisor or contact: request is rejected and no link changes.
- Invalid or duplicate new-contact email: creation is rejected and no contact is created.
- Create-and-link database failure: the transaction rolls back both the new contact and advisor update.
- Existing-contact readback mismatch: the API rejects the result and the Dashboard shows an error toast.

## Controls / Verification Points
- Preventive control: advisor contact selection is manual; no fuzzy or automatic name match can update production data.
- Preventive control: the dialog displays the current link before replacement and disables unchanged saves.
- Preventive control: the existing-link API permits only `advisor.contact_id`; it cannot modify other advisor fields.
- Preventive control: new contacts require a valid email and exact duplicate emails are rejected by both the loaded-page check and the API.
- Preventive control: new-contact creation and advisor assignment commit in one transaction while holding an advisor-row lock.
- Detective control: existing-link updates are read back and compared with the requested contact ID before success is returned.
- Detective control: the Advisors table immediately displays the linked contact name and email after a successful response.

## Evidence to Retain
- Current `advisor.contact_id` and referenced `contact` row
- Advisor `updated` timestamp
- API logs for the link or create-and-link request
- Operator-visible success or error toast during the remediation session

## Known Limitation
- The workflow does not create a separate immutable change-history record containing the prior contact link or initiating user. The current advisor row and mutable `updated` timestamp are the retained database state.

## Related Code / Pages / Routes
- Entry surface: `agm-dashboard/src/components/dashboard/clients/advisors/AdvisorsPage.tsx`
- Dashboard client: `agm-dashboard/src/utils/clients/advisor.ts`
- API routes: `POST /advisors/contact`, `POST /advisors/contact/create`
- API service: `agm-api/src/components/clients/advisors.py`
- Downstream consumers: unfunded-account advisor CC and Accounts Audit missing-document advisor CC

## Last Reviewed
- Status: draft
- Date: 2026-07-15
- Reviewer: Codex implementation update
