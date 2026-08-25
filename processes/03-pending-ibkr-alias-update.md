# Pending IBKR Alias Update

## Business Purpose
Populate missing IBKR account aliases for accounts that are still active and should already have an alias assigned.

## Trigger / Frequency
- Trigger: GitHub Actions workflow calls `POST /token` and then `PATCH /actions/update_pending_alias`.
- Frequency: Daily at 21:00 UTC / 3:00 PM Costa Rica time, plus manual workflow dispatch.

## Systems Involved
- `agm-api`
- GitHub Actions
- Clients reporting dataset
- IBKR API update helper used by `update_account_alias`

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Latest clients report
- API auth token
- Working IBKR alias-update connectivity

## Step-by-Step Workflow
1. The workflow requests an API token and calls `PATCH /actions/update_pending_alias`.
2. The API loads the clients report.
3. It filters to accounts whose `Alias` is blank and whose `Status` is not `Rejected`, `Closed`, or `Funded Pending`.
4. For each qualifying account, it reads `Account ID`, `Title`, and `Master Account`.
5. If the required identifiers are present, it generates a new alias as `<Account ID> <Title>`.
6. It calls the IBKR alias update helper to apply the alias change.
   - The supplied master-account identifier is forwarded to IBKR.
   - Dedicated credentials are used for configured master accounts; any other
     nonblank identifier uses the default I6413690 credential set.
7. Successful updates are collected into the response payload with old and new alias values.
8. Failures are logged per account while the remaining accounts continue processing.
9. GitHub Actions emails the list of updated aliases on success or a failure notice if the job fails.

## Outputs / Records Created
- Updated aliases in IBKR
- Response payload listing updated accounts and total count
- Success or failure notification email from GitHub Actions

## Exception Paths / Failure Handling
- Token failure: workflow exits before calling the route.
- Missing account identifiers: account is silently not updated because the required fields are incomplete.
- IBKR update failure for one account: error is logged and processing continues for other pending accounts.
- Master-account credential selection: unknown master-account identifiers are accepted and use the default I6413690 credentials; the fallback is logged.
- Route failure: workflow sends failure email.

## Controls / Verification Points
- Preventive control: excluded statuses are not updated.
- Preventive control: alias generation uses a deterministic `Account ID + Title` pattern.
- Detective control: per-account success and failure logs are generated.
- Detective control: workflow success email includes the alias update list and total count.

## Evidence to Retain
- GitHub Actions run history for `update_aliases.yml`
- API logs showing account-level update success and failure messages
- Success email containing updated aliases
- IBKR-side alias values after update

## Related Code / Pages / Routes
- Entry surfaces: `agm-api/.github/workflows/update_aliases.yml`, `agm-api/src/app/tools/private/actions.py`
- Supporting modules: `agm-api/src/components/tools/private/actions.py`
- Downstream side effects: `agm-api/src/components/clients/accounts.py`

## Last Reviewed
- Status: draft
- Date: 2026-06-16
- Reviewer: Codex initial draft
