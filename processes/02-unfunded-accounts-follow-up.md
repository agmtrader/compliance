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
3. It excludes accounts without a real IBKR account number (blank or `-`) and identifies accounts whose NAV report `Total` is below `0.5` or whose NAV report row is missing. The threshold matches the Accounts page's whole-unit NAV display.
4. Before contact lookup, it removes the account IDs in the explicit exclusion set: `U24289762`, `U26306704`, `U2421560`, `U27640959`, `U8458872`, `U20351921`, `U20405882`, `U20613112`, `U22335324`, `U7115856`, `U11438607`, `U12443091`, `U19565575`, `U2580655`, `U6170355`, `U26450466`, and `U26540841`. These accounts receive no funding reminder even if their NAV is zero or missing.
5. It filters the remaining candidates to accounts whose client status is `Open`, after trimming whitespace and ignoring case.
6. It merges each candidate account with the clients report `Date Opened`.
7. It calculates business days since the account opened and derives a `notice_number` using five-business-day intervals.
8. It maps each account’s advisor code to an advisor email address by joining advisors to contacts.
9. It merges the candidate accounts to the client contact record to obtain the destination email.
10. It skips records whose client email is missing or blank.
11. It excludes the following opt-out addresses from the outreach batch, matching case-insensitively after trimming whitespace: `ramirezfumero1995@gmail.com`, `andrealeon.aluz@gmail.com`, `paulbernavidesr@gmail.com`, and `esquivelyrodriguez358@gmail.com`.
12. For each remaining contact, it sends a funding notification email in Spanish, optionally copying the advisor.
13. The route returns the merged contact list used for the outreach batch, excluding account IDs and opt-out addresses.

## Outputs / Records Created
- Funding notification emails to qualifying clients
- A response payload listing candidate accounts and email targets
- Workflow logs showing an overview (sent, failed, skipped, and total records) plus expandable per-recipient context/outcome (client, account, email, advisor, aging, notice number)

## Exception Paths / Failure Handling
- Token failure: GitHub workflow exits without calling the route.
- Reporting or database failure: route fails and the workflow returns a non-200 response.
- Missing or blank client email: account is logged and skipped instead of failing the full batch.
- Explicit opt-out email: account is logged and excluded from the batch without sending.
- Missing advisor email: client email still sends with blank CC.
- Email-send failure: the API logs the full recipient context and exception, then fails the route so the workflow cannot report a false success.

## Controls / Verification Points
- Preventive control: only accounts with `Open` status are eligible.
- Preventive control: explicit account and email exclusions prevent reminders to those recipients.
- Preventive control: clients without a nonblank email value are skipped before the send helper runs.
- Detective control: business-day aging and notice-number calculation provide a consistent reminder cadence.
- Detective control: workflow logs retain the API response body for post-run review.

## Evidence to Retain
- GitHub Actions run history for `send_unfunded_emails.yml`
- API logs showing skipped invalid-email records
- API logs showing prepared batch counts and each send attempt, success, exclusion, skip, or failure with client/account/advisor context
- GitHub Actions summary overview showing total sent, failed, skipped, and returned records
- Sent funding-notification emails
- Returned candidate contact list from the route

## Related Code / Pages / Routes
- Entry surfaces: `agm-api/.github/workflows/send_unfunded_emails.yml`, `agm-api/src/app/tools/private/actions.py`
- Supporting modules: `agm-api/src/components/tools/private/actions.py`, `agm-api/src/components/tools/public/reporting.py`
- Downstream side effects: `agm-api/src/components/tools/public/email.py`

## Last Reviewed
- Status: draft
- Date: 2026-10-01
- Reviewer: Codex, aligned with the current code and requested account exclusions
