# IBKR Account Holder Profile Maintenance

## Purpose and access
On demand, an authorized operator can update an eligible IBKR account holder's legal name, birth details, residence or mailing address, and phone numbers through the Dashboard account page. The API enforces the authenticated route scope; application navigation alone is not an authorization boundary.

## Preconditions
The API resolves the internal account, stored IBKR account and supported master, and reads fresh IBKR details. It checks that the returned account and master match. Only Individual and Joint accounts in open status, or pending accounts with IBKR task 9974, are eligible. The selected person must resolve uniquely to an account holder. Dual-language changes are directed to IBKR Portal.

## Workflow and controls
1. Open the account and choose the relevant holder. Review the fresh broker-sourced context and requested changes.
2. The API accepts only supported name, birth, address and phone fields and validates changed values. Login email and other unsupported fields are outside this operation. Phone updates require an explicit country and include a mobile number.
3. On submission the service rechecks identity and eligibility and sends an account-scoped IBKR request. It inspects the nested business result, not only the HTTP status.
4. The operator checks the receipt and independently verifies applied broker details and follow-up tasks. An accepted or pending request is not proof of completion. Check a failed or uncertain response in IBKR before resubmission; there is no automatic write retry.

The operation does not rewrite AGM contact records or stored application JSON from proposed changes. Record the business reason, operator, account/holder, outcome and subsequent verification in the authorized case record; the browser receipt is not a durable audit ledger.

## Source and verification status
Reviewed against `account_holder_service.py`, `holder_updates.py` and account-holder API routes on November 12, 2026. This describes source behavior, not evidence of a new broker submission, production deployment or applied profile change.
