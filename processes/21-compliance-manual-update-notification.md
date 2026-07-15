# Compliance Manual Update Notification

## Business Purpose
Send a simple email notifying the configured recipient that the compliance manual requires review after an update.

## Trigger / Frequency
- Trigger: An authenticated caller sends `POST /actions/send_compliance_manual_update_email`.
- Frequency: On demand. No scheduled workflow or automatic repository-change trigger is present.

## Systems Involved
- `agm-api`
- Gmail connector
- Compliance manual update HTML email template

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Valid API authorization
- Working Gmail credentials and connector
- `compliance_manual_update.html` template

## Step-by-Step Workflow
1. An authenticated caller invokes the action endpoint.
2. The service constructs an empty content payload.
3. The Gmail helper sends an email with subject `Compliance Manual Update Requires Review` using the compliance-manual-update template.
4. The recipient is hardcoded as `aa@agmtechnology.com`; no CC or BCC is used.
5. The route returns `sent`, the recipient, and the Gmail helper result.

## Outputs / Records Created
- Notification email to the hardcoded recipient
- API response containing send status and connector result
- No compliance-manual version, changed-section list, reviewer assignment, due date, or acknowledgment record

## Exception Paths / Failure Handling
- Gmail or template failure returns a service error through the standard API response handling.
- There is no retry workflow, alternate recipient, escalation, or follow-up reminder.
- The endpoint can be called independently of an actual compliance-document change.

## Controls / Verification Points
- Detective control: when called successfully, the endpoint produces an email notification.
- Current limitation: the notification is not automatically connected to a committed manual change.
- Current limitation: completion of review and approval is not recorded.

## Evidence to Retain
- API request and response logs
- Sent Gmail message
- Separately retained manual change and approval evidence, if any

## Related Code / Pages / Routes
- Entry surfaces: `agm-api/src/app/tools/private/actions.py`
- Supporting modules: `agm-api/src/components/tools/private/actions.py`, `agm-api/src/components/tools/public/email.py`
- Downstream side effects: `agm-api/src/lib/email_templates/compliance_manual_update.html`

## Last Reviewed
- Status: draft
- Date: 2026-07-15
- Reviewer: Codex current-state code review
