# Tasks Review

## Purpose and trigger
Staff open Dashboard `/tasks-review` to review saved IBKR requirements and account collection coverage. This is an on-demand review; it does not call IBKR or refresh accounts when a page opens. Owner: Andres Aguilar. Source collection is governed by [Clients ETL](04-clients-etl-pipeline.md).

## Sources and scope
- Manual Portal `compliance_tasks.csv`, published as `ibkr_compliance_tasks.csv` and compatible JSON. Original source time is preserved; Portal capture time is unknown. Missing rows do not prove account-wide completeness.
- Daily `ibkr_pending_tasks.csv` and `ibkr_registration_tasks.csv`, including explicit coverage rows and completed registration history.
- Login messages remain selected-account reads in AccountPage and are not part of this bulk snapshot.

## Review workflow
1. Open Tasks Review from Compliance navigation. Pending is selected initially. Source panels show full-inventory record counts, source time, freshness and API account read coverage.
2. Select Tasks or Account coverage. Filter by source, master account, freshness/coverage, Portal restriction and text search; Tasks also offers Pending, Completed and All. The account-coverage view includes confirmed empty reads and unavailable accounts. Unknown task counts stay unknown.
3. The staff-only `GET /reporting/tasks_review` reads cached saved resources, projects and filters on the API, and returns a bounded page (50 records by default; maximum 200). Counts describe source observations, not deduplicated requirements. Source panels retain full-inventory counts; task summaries describe filters across all pages, before the status tab. Tables are ordered by account/task/source. No per-account broker fan-out occurs.
4. Inspect supporting record data through accessible hover/focus/tap Details cards. Details fetch only the selected saved observation. Exact unique pending/Compliance account + task-instance + form identities link observations; source records stay separate and state disagreements/ambiguity remain visible. Registration completion never closes another task by form/name alone.
5. View account opens the unique internal account link when account context is available. Duplicate or missing internal mappings do not select an arbitrary account. This report has no signing/completion actions.

## Controls and exceptions
- Navigation is admin-only; the API requires `all` scope. Reads use authenticated no-store transport and private/no-store responses. Existing per-source bounded caches remain version/freshness controlled.
- Missing or invalid sources appear unavailable and successful sources remain visible as a partial inventory. All-source failure returns 503. Source failure is never a successful empty inventory; contextual Retry rereads saved resources, not the broker.
- A failed request hides prior page rows. Late requests from earlier filters cannot overwrite current results. Search is debounced and filters reset pagination. Summary counts and empty states refer to the current successful request.
- Restrictions reflect only the manual Portal export and its displayed age. API-only accounts have unknown Portal restriction. API account coverage uses both API source outcomes; Compliance absence remains Not listed.
- Source rows are never silently removed or merged by task/form name. Full original task records remain in the source CSVs and selected-record Details.
- Supporting details use AGM shadcn HoverCard with labeled 40px triggers, keyboard focus/Escape, tap and bounded portal content. Tables use shared DataTable with contained horizontal scrolling.

## Outputs and evidence
Read-only paginated review and saved-record details; no broker submissions, messages or persisted review decisions. Retain release/serving revision evidence, authenticated application results, snapshot counts/coverage and source freshness in API #133 and Dashboard #27. Normal scheduled ETL acceptance is distinct from a scoped refresh.
