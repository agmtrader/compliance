# Account Comments History

## Business Purpose
Provide a centralized internal-notes history for an AGM account so authorized staff can record operational context, follow-up decisions, and account-level review notes without mixing those notes with client communications or restricted AML investigation records.

## Trigger / Frequency
- Trigger: An authorized Dashboard user opens an account and selects the `Comments` tab, then reads, creates, edits, or deletes an internal comment.
- Frequency: On demand.

## Systems Involved
- `agm-dashboard`
- `agm-api`
- Supabase/Postgres `account`, `user`, and `account_comments` tables

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Existing AGM account id.
- Signed-in Dashboard user id, display name, and email.
- Comment body for a new or edited comment.
- The `account_comments` table migration must be applied before the API is started with strict schema validation.

## Step-by-Step Workflow
1. The user opens an account page and selects `Comments`.
2. Dashboard reads `/accounts/comments?account_id=...` and displays comments ordered by their stored creation timestamp.
3. For a new note, Dashboard submits the account id, user id, author snapshot, and trimmed comment body to `POST /accounts/comments`.
4. API validates the account and required fields, then writes an `account_comments` row with creation and update timestamps.
5. The author may edit or soft-delete their own comment through the update/delete routes. Editing records `edited_at`; deletion records `deleted_at` and replaces visible body text with `[deleted]`.
6. The process closes when the updated comment list is reloaded and displayed.

## Outputs / Records Created
- `account_comments` rows containing account, author, body, timestamps, and edit/delete metadata.
- Account-page chronological comment history.
- API request and error logs.

## Exception Paths / Failure Handling
- Missing account, author, or body: API returns a validation error and no row is created.
- Unknown account: API returns not found.
- Edit/delete by a different user: API returns not found and leaves the comment unchanged.
- Database or API failure: Dashboard shows an error toast; the user must retry after the failure is investigated.
- Deleted comments remain in the database as soft-deleted history and are displayed as `[deleted]`.

## Controls / Verification Points
- Preventive control: account existence and nonblank body are validated before insert.
- Preventive control: edit and delete operations require the stored `user_id` to match the acting user id supplied by the Dashboard.
- Detective control: review `account_comments` rows and API logs for account, author, timestamp, and operation outcome.
- Limitation: the current API receives the acting user id from the authenticated Dashboard session but does not independently derive it from a server-side identity claim; this should be strengthened if the API authentication model changes.
- Limitation: edits update the current row rather than creating immutable revision rows. The history is chronological comment history, not a field-level audit trail.
- Account comments must not be used to record SAR details, confidential AML investigations, sanctions dispositions, passwords, or other restricted information.

## Evidence to Retain
- `account_comments` database rows, including `created`, `updated`, `edited_at`, and `deleted_at`.
- API request/error logs for comment operations.
- Screenshots or exports only when the comment is used as evidence in a separate review.

## Related Code / Pages / Routes
- Entry surface: `agm-dashboard/src/components/dashboard/clients/accounts/AccountPage.tsx`
- Dashboard client: `agm-dashboard/src/utils/clients/account.ts`
- API routes: `agm-api/src/app/clients/accounts.py`
- Service: `agm-api/src/components/clients/accounts.py`
- Data model: `agm-api/src/utils/connectors/supabase.py`
- Migration: `sql/20260805_account_comments.sql`

## Last Reviewed
- Status: Draft
- Date: 2026-08-05
- Reviewer: Andres Aguilar
