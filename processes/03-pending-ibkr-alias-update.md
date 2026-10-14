# Pending IBKR Alias Update

## Business Purpose
Populate missing IBKR account aliases for accounts that are still active and should already have an alias assigned.

## Trigger / Frequency
- Trigger: GitHub Actions workflow calls `POST /token` and then `PATCH /actions/update_pending_alias`.
- Frequency: Daily at 20:30 UTC / 2:30 PM Costa Rica time, plus manual workflow dispatch.

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
1. The workflow requests an authenticated AGM user token and calls `PATCH /actions/update_pending_alias`.
2. The API loads the clients report.
3. It filters to accounts whose `Alias` is blank and whose `Status` is not `Rejected`, `Closed`, or `Funded Pending`.
4. For each qualifying account, it reads `Account ID`, `Title`, and `Master Account`.
5. If the required identifiers are present, it generates a new alias as `<Account ID> <Title>`.
6. It calls the IBKR alias update helper to apply the alias change.
   - The supplied master-account identifier is forwarded to IBKR.
   - Dedicated credentials are used for configured master accounts; any other
     nonblank identifier uses the default I6413690 credential set.
7. Successful updates are collected into the response payload with old and new alias values.
8. Failures are logged per account and returned in `failed_accounts` while the remaining accounts continue processing.
9. After every eligible account is attempted, the API returns the complete counts and account lists with HTTP 200, including partial failures.
10. GitHub Actions logs the summary and a warning when account errors occurred, then emails the completed run summary. A batch-level failure still fails the job and sends a failure notice.

## Outputs / Records Created
- Updated aliases in IBKR
- Response payload listing updated, failed, and skipped accounts with counts
- Completed or failure notification email from GitHub Actions

## Exception Paths / Failure Handling
- Nested API functions propagate one ServiceError instance without reporting intermediate failures. Unexpected Python exceptions are normalized once to the generic API failure; final route/Flask or outermost direct-job boundaries handle that failure. The final exception excludes intermediate cause/context chains while retaining correlation IDs, operation/route template, status/code and frame locations without source text. Client-facing controlled errors retain their existing message/details; their final message remains in the diagnostic, while unexpected provider/cause messages are not copied into it. Query values, client IPs and error details remain excluded. This governs shared exception propagation; independent integration/audit logs retain their existing behavior.
- Expected IBKR validation or account-access rejection is retained as a failed-account entry and logged as a warning while processing continues. Unknown alias statuses, upstream server failures, and configuration failures remain reportable in API logs; the same propagated exception is logged once.
- Token failure: workflow exits before calling the route.
- Missing account identifiers: account is silently not updated because the required fields are incomplete.
- IBKR update failure for one account: error is logged and included in the completed run summary while processing continues for other pending accounts.
- Master-account credential selection: unknown master-account identifiers are accepted and use the default I6413690 credentials; the fallback is logged.
- Report loading, authentication, or route failure before the batch completes: workflow fails and sends a failure email.

## Controls / Verification Points
- Preventive control: excluded statuses are not updated.
- Preventive control: alias generation uses a deterministic `Account ID + Title` pattern.
- Detective control: per-account success and failure logs are generated.
- Detective control: workflow completed email includes updated and failed account lists with their counts; Actions logs flag partial failures.

## Evidence to Retain
- GitHub Actions run history for `update_aliases.yml`
- API logs showing account-level update success and failure messages
- Completed email containing updated aliases and any account-level errors
- IBKR-side alias values after update

## Related Code / Pages / Routes
- Entry surfaces: `agm-api/.github/workflows/update_aliases.yml`, `agm-api/src/app/tools/private/actions.py`
- Supporting modules: `agm-api/src/components/tools/private/actions.py`
- Downstream side effects: `agm-api/src/components/clients/accounts.py`

## Last Reviewed
- Status: draft
- Date: 2026-06-16
- Reviewer: Codex initial draft
