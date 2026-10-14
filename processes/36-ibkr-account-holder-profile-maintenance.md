# IBKR Account Holder Profile Maintenance

## Business Purpose
Correct a brokerage account holder's legal name, date/country of birth, home or mailing address, and phone numbers from the Dashboard's Associated Individuals section.

## Trigger / Frequency
- Trigger: An authorized operator opens Edit profile and submits changed fields.
- Frequency: On demand.

## Systems Involved
- `agm-dashboard` Account page
- `agm-api` account-holder service and IBKR connector
- IBKR Account Management Web API

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Operator: authenticated AGM user with `all`, `accounts`, or the exact update-route scope. Reading accounts alone does not grant update permission.

## Inputs / Prerequisites
- Selected internal AGM account UUID with a submitted IBKR account number and supported stored master account.
- Fresh IBKR account details, unique holder entity ID or external code, and holder association.
- Individual or Joint account with status O; status N/P also requires pending registration task 9974.
- Single-language profile. Dual-language changes require both versions and are directed to IBKR Portal.
- Correct phone country selected explicitly when editing phones; the details response does not provide these countries, so AGM does not infer them from residence.

## Step-by-Step Workflow
1. Open the account and select Edit profile for the intended individual. Organization/Trust accounts, non-holders such as trusted contacts, missing identities and ineligible statuses show an explanatory restriction.
2. The API resolves the stored account/master and reads fresh IBKR details. Pending accounts also load current pending tasks. The editor prefills from these broker details rather than the local application/contact snapshot.
3. Edit profile, addresses or phones. Only changed groups are submitted; a changed name/address group preserves its other displayed fields. There is no phone-delete operation. Login email, employment, identification and tax changes are outside this editor's scope.
4. Review the changed-group summary and the warning that a new update replaces the account's existing pending profile request. Name/address changes may require new tax forms or supporting documents.
5. Submit to IBKR. The API rechecks the current account/master, status/task and unique holder identity, validates the allowed fields, and signs one `changeAccountHolderDetail` PATCH to `/gw/api/v1/accounts`. It includes accountId and a verified holder id/externalId. It never sends referenceUserName, which would target linked accounts.
6. The API inspects the nested business result, including HTTP-200 ERROR responses. The Dashboard shows IBKR's status and request ID and refreshes current broker details. A PENDING/accepted request does not replace displayed fields with proposed values.
7. The operator verifies the resulting profile in refreshed details/IBKR Portal and reviews pending registration tasks and login messages for tax/document follow-up before treating the correction as complete.

## Outputs / Records Created
- One broker profile-change request; normalized business status and broker request ID displayed in the browser.
- Updated broker profile only after IBKR applies the request.
- AGM contact records, stored application JSON, and account-contact identity links are not rewritten from proposed edits. Existing ETL/reconciliation remains a separate process.
- No new database schema or persistent local submission ledger.

## Exception Paths / Failure Handling
- Missing/invalid fields, arbitrary payload keys, empty/no-op changes and non-holder identities are rejected before a broker write.
- Eligibility, account/master mismatch and dual-language requirements block submission with a controlled explanation.
- IBKR transport validation/access failures and nested rejected business results retain structured errors. The form keeps edits available for correction.
- Unknown/malformed results, connection failures and timeouts are uncertain outcomes. The write has no automatic retry; verify in IBKR Portal before resubmitting.
- A failed display refresh preserves the submission receipt and previous broker details with an explicit refresh-failure message.
- Submission controls and modal dismissal are disabled while a request is running to prevent accidental repeated writes.

## Controls / Verification Points
- Authentication and default-deny route scopes apply to both context reads and submissions.
- The API derives account/master and person identity from stored records and fresh broker details, independently of client eligibility checks.
- Strict field allowlist prevents email/credentials, account reassignment and linked-account username targeting.
- Broker business outcome is checked separately from HTTP success.
- Operator verifies applied changes and required follow-up tasks; accepted-for-processing alone is not completion evidence.

## Evidence to Retain
- Business reason, selected account/holder, edited fields and operator/time in the existing operational case record.
- Broker status/request ID shown after submission and subsequent broker profile/task verification.
- Retain evidence in the authorized case record: the browser receipt is transient and does not constitute a durable audit ledger.

## Related Code / Pages / Routes
- `agm-dashboard/src/components/dashboard/clients/accounts/AccountPage.tsx`
- `agm-dashboard/src/components/dashboard/clients/accounts/AccountHolderProfileEditor.tsx`
- `agm-dashboard/src/lib/clients/holder-update.ts`
- `GET /accounts/ibkr/account_holder_edit_context`
- `POST /accounts/ibkr/account_holder_detail`
- `agm-api/src/components/clients/account_holder_service.py`
- `agm-api/src/components/clients/holder_updates.py`
- `IBKRWebAPI.change_account_holder_detail`
- [IBKR profile-update documentation](https://www.interactivebrokers.com/campus/ibkr-api-page/web-api-account-management/#update-information)
- [Client ETL and reconciliation](04-clients-etl-pipeline.md)

## Last Reviewed
- Status: draft
- Date: 2026-10-14
- Reviewer: Codex implementation and local synthetic validation; no live customer mutation used for testing.
