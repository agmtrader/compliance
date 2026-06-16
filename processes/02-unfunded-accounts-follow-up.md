# Unfunded Accounts Follow-up

## Business Purpose
Identify open accounts that remain unfunded and send a funding reminder email to the client, with the advisor copied when available.

## Trigger / Frequency
- Trigger: GitHub Actions workflow calls `POST /token` and then `GET /actions/send_unfunded_emails`.
- Frequency: Weekly on Mondays at 14:00 UTC / 8:00 AM Costa Rica time, plus manual workflow dispatch.

## Systems Involved
- `agm-api`
- GitHub Actions
- NAV reporting dataset
- Clients reporting dataset
- AGM accounts, contacts, and advisors data
- Email sending through the Gmail helper

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Latest NAV report
- Latest clients report
- Current accounts table
- Contacts table with client email addresses
- Advisors table with advisor-contact linkage

## Step-by-Step Workflow
1. The scheduled workflow requests an API token and calls `/actions/send_unfunded_emails`.
2. The API loads NAV data, clients data, accounts data, contacts data, and advisors data.
3. It builds DataFrames and identifies accounts with either zero NAV or no NAV report row at all.
4. It filters the candidate set down to accounts whose client status is `Open`.
5. It merges each candidate account with the clients report `Date Opened`.
6. It calculates business days since the account opened and derives a `notice_number` using five-business-day intervals.
7. It maps each account’s advisor code to an advisor email address by joining advisors to contacts.
8. It merges the candidate accounts to the client contact record to obtain the destination email.
9. It skips records with missing or invalid client email addresses.
10. For each remaining contact, it sends a funding notification email in Spanish, optionally copying the advisor.
11. The route returns the merged contact list used for the outreach batch.

## Outputs / Records Created
- Funding notification emails to qualifying clients
- A response payload listing candidate accounts and email targets
- Workflow logs showing the batch result

## Exception Paths / Failure Handling
- Token failure: GitHub workflow exits without calling the route.
- Reporting or database failure: route fails and the workflow returns a non-200 response.
- Missing client email: account is logged and skipped instead of failing the full batch.
- Missing advisor email: client email still sends with blank CC.

## Controls / Verification Points
- Preventive control: only accounts with `Open` status are eligible.
- Preventive control: clients without usable email addresses are not sent malformed messages.
- Detective control: business-day aging and notice-number calculation provide a consistent reminder cadence.
- Detective control: workflow logs retain the API response body for post-run review.

## Evidence to Retain
- GitHub Actions run history for `send_unfunded_emails.yml`
- API logs showing skipped invalid-email records
- Sent funding-notification emails
- Returned candidate contact list from the route

## Related Code / Pages / Routes
- Entry surfaces: `agm-api/.github/workflows/send_unfunded_emails.yml`, `agm-api/src/app/tools/private/actions.py`
- Supporting modules: `agm-api/src/components/tools/private/actions.py`, `agm-api/src/components/tools/public/reporting.py`
- Downstream side effects: `agm-api/src/components/tools/public/email.py`

## Last Reviewed
- Status: draft
- Date: 2026-06-16
- Reviewer: Codex initial draft
