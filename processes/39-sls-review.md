# SLS Review

## Purpose and frequency

On demand, an authorized administrator reviews saved IBKR task observations and recorded SLS devices, and may send an account holder a follow-up.

## Inputs and controls

Dashboard `/sls-review` requires an authenticated AGM administrator with `all` scope. It matches internal accounts to the Clients report by normalized IBKR account number. Missing or duplicate report matches and invalid SLS values are Unknown. Blank or dash strings indicate No device recorded. Other strings indicate Device recorded. Staff can combine device, pending-task and open-account filters.

The pending-task count includes observations from the saved Tasks Review feeds. An observation can appear in more than one source. Rows show incomplete coverage or unknown conditions where applicable. The outreach dialog shows source and freshness for each pending observation. See [Tasks Review](38-tasks-review.md) for source coverage.

## Email review and evidence

1. Open Email & history for the account. Choose a linked holder, notice and language, then generate the preview. The API matches a linked contact to a unique holder in the saved IBKR account details and lists fresh, actionable tasks for that holder. Task conflicts and ambiguous task identities are excluded from the notice.
2. Verify the recipient and review the outstanding actions in IBKR. For SLS activation, confirm the live device status. The Clients report is a snapshot and the SLS notice also requires a uniquely matched open account with no recorded device.
3. Send the preview. The API rebuilds it and rejects changed details. It records the attempt before calling Gmail. The same request ID cannot send twice, and another send of the same notice is blocked while the earlier attempt remains unresolved.
4. Check Email history for the notice type, recipient and Gmail acceptance time. An unconfirmed result requires checking Gmail before attempting further outreach. Provider acceptance is evidence of a send, while inbox delivery requires separate confirmation.

The dedicated outreach table retains the staff actor, account, contact, recipient, notice type, task snapshot, message and outcome. The document-review email workflow retains its own history.

## Status

Dashboard issue #72 tracks task and SLS outreach. Apply the API migration before release and verify the authenticated page, report coverage, email preview and send history in the intended environment.
