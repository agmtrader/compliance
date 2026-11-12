# SLS Review

## Purpose and frequency
On demand, an authorized administrator reviews the SLS devices recorded for AGM accounts. This is a review aid, not live IBKR device telemetry or an outreach record.

## Inputs and controls
Dashboard `/sls-review` reads the account list and Clients report through `/api/reporting/sls-review`, which requires an authenticated administrator with `all` scope and returns private, uncached results. Rows are matched by normalized IBKR account number. A missing or duplicate Clients match, or a non-string SLS field, is Unknown. A blank or dash string is No device recorded; another string is Device recorded. Counts respect the open-account filter. The table supports search, export, and opening the account.

The Clients report is a snapshot and the source capture time is not provided here. Confirm the current device state and recipient before outreach. Unknown is not a finding that a device is absent. The page does not send messages, change IBKR, or retain review dispositions.

## Status
Dashboard implementation was confirmed complete by the owner and published in commit 539833f. The owner confirmation does not independently establish source freshness or broker state. See Dashboard issue #44 for the separate outreach and its acceptance.
