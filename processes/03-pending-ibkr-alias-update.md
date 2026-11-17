# Pending IBKR Alias Update

## Business Purpose
Populate missing IBKR account aliases for accounts that are still active and should already have an alias assigned.

## Trigger / Frequency
- Trigger: GitHub Actions workflow calls `POST /token` and then `PATCH /actions/update_pending_alias`.
- Frequency: Daily at 20:30 UTC / 2:30 PM Costa Rica time, plus manual workflow dispatch.

## Systems Involved
- `agm-api`
- GitHub Actions
- Internal accounts table and daily IBKR account-details snapshot
- IBKR API update helper used by `update_account_alias`

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Internal accounts and the latest IBKR account-details snapshot
- API auth token
- Working IBKR alias-update connectivity

## Step-by-Step Workflow
1. The workflow requests an authenticated AGM user token and calls `PATCH /actions/update_pending_alias`.
2. The API loads internal accounts and the Clients ETL account-details snapshot, matching unique records by IBKR account number. No Clients CSV is read.
3. A fresh detail with an omitted, blank, null or `-` alias is a candidate. Existing aliases are excluded. Missing, duplicate or stale details, invalid alias values and missing status are skipped. Rejected, Closed and Funded Pending statuses and account `U27090911` remain excluded.
4. The title comes from account details and the master from the internal account table. Missing titles, unsupported or missing masters, duplicate internal account numbers and aliases longer than 80 characters are skipped; no default master is used.
5. For each eligible account, generate `<Account ID> <Title>`.
6. It calls the IBKR alias update helper to apply the alias change.
   - The supplied master-account identifier is forwarded to IBKR.
   - The batch requires a supported stored master and uses its dedicated credentials.
   - The API logs the HTTP status and a bounded, sanitized representation of the IBKR response body before interpreting success or failure. Response structure, statuses, request IDs and error messages are retained; credentials and unknown scalar fields are redacted. No request headers or signed request body are recorded.
7. Successful updates are collected into the response payload with old and new alias values.
8. Failures are logged per account and returned in `failed_accounts` while the remaining accounts continue processing.
9. After every eligible account is attempted, the API returns the complete counts and account lists with HTTP 200, including partial failures.
10. GitHub Actions logs each returned account's sanitized IBKR response separately, followed by the summary and a warning when account errors occurred, then emails the completed run summary including skipped-reason counts. Diagnostic response bodies are not added to the email. A batch-level failure still fails the job and sends a failure notice.

## Individual account changes

The Dashboard account overview can submit an individual alias through `POST /accounts/ibkr/alias`, restricted to the all scope. The API resolves the IBKR account number and supported master from the stored internal account and rejects missing or unsupported identifiers. This is separate from the scheduled batch. The response indicates submission only; independently verify the alias in IBKR before retrying because a successful transport response or an interrupted request does not prove persistence. The Clients report is a separate snapshot and may lag.

## Outputs / Records Created
- Updated aliases in IBKR
- Response payload listing updated, failed, and skipped accounts with counts
- Completed or failure notification email from GitHub Actions

## Exception Paths / Failure Handling
- Centralized API exception reports retain request/error IDs, HTTP method, route template, endpoint, status/code, controlled exception messages, sanitized allowlisted upstream diagnostics and stack file/line/function locations. They omit query values, client IPs, arbitrary error payloads and source-line text. Alias-specific response diagnostics are recorded separately in API and Actions logs.
- Expected IBKR validation or account-access rejection is retained as a failed-account entry and logged as a warning while processing continues. Unknown alias statuses, upstream server failures, and configuration failures remain reportable in API logs; the same propagated exception is logged once.
- Token failure: workflow exits before calling the route.
- Accounts without an IBKR number are excluded from candidate matching; accounts with missing title or an unsupported master are returned in `skipped_accounts` with a reason.
- IBKR update failure for one account: error is logged and included in the completed run summary while processing continues for other pending accounts.
- Master-account credential selection: missing or unsupported identifiers are skipped without a broker request.
- Diagnostic limitation: the existing connector can still count an HTTP 200 response with no parsed alias status as successful. Response capture does not establish that the alias persisted in IBKR or refresh the separate Clients report. Inspect the captured response and independently verify broker state before declaring remediation complete; do not automatically replay uncertain writes.
- Report loading, authentication, or route failure before the batch completes: workflow fails and sends a failure email.

## Controls / Verification Points
- Preventive control: excluded statuses are not updated. The snapshot can omit aliases present in other sources; this automated selection can overwrite a current alias if the snapshot is incomplete or out of date. Review broker state and source coverage before enabling the changed batch.
- Preventive control: alias generation uses a deterministic `Account ID + Title` pattern.
- Detective control: per-account success and failure logs are generated.
- Detective control: workflow completed email includes updated and failed account lists with their counts; Actions logs flag partial failures.

## Evidence to Retain
- GitHub Actions run history for `update_aliases.yml`
- API and Actions logs showing account-level update messages, HTTP status and sanitized IBKR response bodies
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
