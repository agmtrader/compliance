# IBKR Account Closure

## Business Purpose
Submit an operator-approved request to close an eligible IBKR account when the client no longer wants an AGM investment account.

## Trigger / Frequency
- Trigger: Authorized operator submits the account-closure route or explicitly runs the closure script.
- Frequency: On demand.

## Systems Involved
- `agm-api`
- IBKR Account Management Web API

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro

## Inputs / Prerequisites
- IBKR account identifier and closure reason.
- Correct IBKR master-account credential set.
- Operator confirmation that IBKR reports status `O` or `Q` and a cleared balance.

## Step-by-Step Workflow
1. The operator confirms the account is eligible for closure and records the business reason.
2. The API validates `account_id` and `close_reason`.
3. AGM signs the `accountManagementRequests.accountClose` payload and submits `PATCH /gw/api/v1/accounts` to IBKR.
4. The response is retained in the operator's execution output for follow-up and status verification in IBKR.

## Outputs / Records Created
- IBKR account-close request and response.
- Local execution output when the closure script is used.

## Exception Paths / Failure Handling
- Centralized API exception reports retain request/error IDs, HTTP method, route template, endpoint, status/code and exception types with stack file/line/function locations. They omit query values, client IPs, error details, exception messages and source-line text; client-facing controlled errors retain their existing message/details. This redaction covers the shared exception wrapper, not every independent integration/audit log.
- Missing required input returns structured HTTP 400. Known IBKR validation or account-access rejection returns 400/422 or 403 without an exception report. Unknown upstream and credential/configuration failures remain reportable. Operators must still verify the business outcome before retrying.
- Missing inputs are rejected before the IBKR request.
- IBKR rejects accounts that are funded or not in an eligible status; the operator must resolve the condition and retry only after confirmation.
- Network errors are retried by the connector's transient-connection retry wrapper.

## Controls / Verification Points
- Closure script is dry-run by default and requires `--execute` for an external state change.
- Closure reason and target account are printed before execution.
- Operator must verify the IBKR response and resulting account status; HTTP success alone is not evidence that closure completed.

## Evidence to Retain
- Submitted payload fields (excluding credentials).
- IBKR response, request timestamp, operator, and eligibility confirmation.

## Related Code / Pages / Routes
- `agm-api/src/utils/connectors/ibkr_web_api.py:IBKRWebAPI.close_account`
- `agm-api/src/components/clients/accounts.py:close_account`
- `agm-api/src/app/clients/accounts.py:POST /accounts/ibkr/close_account`
- `agm-api/close_natalia_account.py`

## Last Reviewed
- Status: draft
- Date: 2026-08-25
- Reviewer: Codex current-state code review
