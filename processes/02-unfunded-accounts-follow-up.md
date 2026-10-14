# Unfunded Accounts Follow-up

## Business Purpose
Identify open accounts that remain unfunded and send a funding reminder email to the client, with the advisor copied when available.

## Trigger / Frequency
- Trigger: GitHub Actions workflow calls `POST /token` and then `GET /actions/send_unfunded_emails`.
- Frequency: Weekly on Mondays at 14:00 UTC / 8:00 AM Costa Rica time, plus manual workflow dispatch.
- GitHub may delay scheduled starts; retain actual run timestamps. Workflow concurrency queues overlapping scheduled/manual runs without cancelling an active send batch.

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
1. The scheduled workflow requests an authenticated AGM user token and calls `/actions/send_unfunded_emails`.
2. The API loads NAV data, clients data, accounts data, contacts data, and advisors data.
3. It builds DataFrames and identifies accounts with either zero NAV or no NAV report row at all.
4. It filters the candidate set down to accounts whose client status is `Open`.
5. It merges each candidate account with the clients report `Date Opened`.
6. It calculates business days since the account opened and derives a `notice_number` using five-business-day intervals.
7. It maps each account’s advisor code to an advisor email address by joining advisors to contacts.
8. It merges the candidate accounts to the client contact record to obtain the destination email.
9. It skips records with missing or invalid client email addresses.
10. It excludes the following opt-out addresses from the outreach batch, matching case-insensitively after trimming whitespace: `ramirezfumero1995@gmail.com`, `andrealeon.aluz@gmail.com`, `paulbernavidesr@gmail.com`, and `esquivelyrodriguez358@gmail.com`.
    Account-level exclusions in `UNFUNDED_ACCOUNT_EXCLUSIONS` are removed before recipient preparation and are not counted in the recipient summary. Email opt-outs are returned as skipped outcomes without sending.
11. For each remaining contact, it sends a funding notification email in Spanish, optionally copying the advisor.
12. Each send exception is retained as a recipient failure and later recipients are still attempted. The batch adds no automatic retry.
13. The route returns an object containing `status`, `summary` (`total`, `sent`, `failed`, `skipped`), and `recipients`. Each recipient has a sent/failed/skipped outcome and context; confirmed sends include the Gmail message ID, failures include error metadata and uncertain-delivery guidance, and skipped recipients include a reason. Missing data is JSON null.
14. Status is `success` when there are no recipient failures, `partial` when sends succeeded and others failed, and `failed` when there are failures but no confirmed sends. Completed batches return HTTP 200 for all three statuses. The workflow reads the result status, completes partial runs with an explicit failure-count notification, and fails all-failed runs after publishing their full summary.

## Outputs / Records Created
- Funding notification emails to qualifying clients
- A response payload listing actual recipient outcomes and reconciled counts
- Workflow logs showing an overview (sent, failed, skipped, and total records) plus expandable per-recipient context/outcome (client, account, email, advisor, aging, notice number)

## Exception Paths / Failure Handling
- Token failure: GitHub workflow exits without calling the route.
- Reporting or database failure before sending: route fails and the workflow returns a non-200 response. Token, transport, invalid-response and other fatal failures publish a failure summary with recipient counts marked unknown; a non-200 response does not establish that zero emails were sent.
- Missing client email: account is logged and skipped instead of failing the full batch.
- Explicit opt-out email: account is logged and returned as skipped without sending.
- Missing advisor email: client email still sends with blank CC.
- Email-send failure: the batch handles the recipient's final exception, reports unexpected 5xx failures once through the shared error logger, retains recipient context as a traceback-free warning, records the failure, and continues later recipients. Nested functions preserve one final exception without intermediate cause/context errors. Controlled 4xx failures retain guidance without exception reports. All failed send attempts are treated conservatively as delivery unknown; a timeout does not prove non-delivery. Verify sent-mail/provider evidence before retrying an uncertain send or replaying a partly completed batch.
- Workflow release compatibility: the summary renderer accepts the old successful-array response during rollout, labels its counts as inferred, and supports the new outcome object. Release the workflow before or together with the API response change. The legacy API still aborts recipient failures until the new API is deployed.
- Execution limit: this remains a synchronous Cloud Run request (currently a 300-second deadline). Recipient exception isolation does not guarantee completion after process termination or repeated long provider timeouts. If the request terminates before returning outcomes, treat counts as unknown and reconcile API/provider evidence before replay. A durable asynchronous outreach job would be a separate follow-up.

## Controls / Verification Points
- Preventive control: only accounts with `Open` status are eligible.
- Preventive control: clients without usable email addresses are not sent malformed messages.
- Detective control: business-day aging and notice-number calculation provide a consistent reminder cadence.
- Detective control: workflow summaries retain the API's recipient outcomes and counts for post-run review.
- Detective control: API outcomes include recipient context and message/error IDs; Actions retains per-recipient results and checks that counts/status match outcomes. Existing squad notifications distinguish partial completion from complete success.

## Evidence to Retain
- GitHub Actions run history for `send_unfunded_emails.yml`
- API logs showing skipped invalid-email records
- API logs showing prepared batch counts and each send attempt, success, exclusion, skip, or failure with client/account/advisor context
- GitHub Actions summary overview showing total sent, failed, skipped, and returned records
- Sent funding-notification emails
- Returned recipient outcomes from the route, including failure IDs and confirmed-send message IDs

## Related Code / Pages / Routes
- Entry surfaces: `agm-api/.github/workflows/send_unfunded_emails.yml`, `agm-api/src/app/tools/private/actions.py`
- Supporting modules: `agm-api/src/components/tools/private/actions.py`, `agm-api/src/components/tools/public/reporting.py`
- Downstream side effects: `agm-api/src/components/clients/accounts.py`, `agm-api/src/utils/connectors/gmail.py`
- Summary renderer: `agm-api/.github/scripts/unfunded_summary.py`

## Last Reviewed
- Status: draft
- Date: 2026-10-14
- Reviewer: Codex recipient failure isolation update, API issue #291
