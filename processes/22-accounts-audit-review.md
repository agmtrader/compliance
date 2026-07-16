# Accounts Audit Review

## Business Purpose
Provide the current on-demand compliance review of account completeness, application-versus-IBKR financial information, account/contact linkage, required documents, IBKR task indicators, reviewer ownership, missing-document outreach, and supporting-document uploads.

## Trigger / Frequency
- Trigger: An authorized user opens Dashboard `Accounts Audit`.
- Frequency: On demand. There is no scheduled audit, required review period, or automatic completion certification.
- Default view: `Account Focus`.

## Systems Involved
- `agm-dashboard`
- `agm-api`
- Internal accounts, contacts, account-contact links, advisors, contact documents, users, review assignments, and email-attempt records
- Latest available IBKR details reporting resource and IBKR financial ranges
- Static Dashboard IBKR compliance-task snapshot
- Gmail send path and public AGM Hub compliance-document upload page

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Internal accounts and stored `application_json`
- IBKR details with account id, title, applicant type, status, equity, risk score, open/close dates, financial information, and associated-person phone and employment data
- Financial-range definitions for net worth, liquid net worth, and annual net income
- Contacts and account-contact links, including optional linked `entity_id` and `external_id`
- Contact-document metadata; file bodies are excluded from the initial load
- Advisors and advisor-linked contacts
- `document_review_responsible` and `document_review_email` records
- Dashboard users whose email ends in `@agmtechnology.com`

## Population and Join Rules

### Shared source loading

1. The page loads accounts, IBKR details, financial ranges, contacts, account-contact links, contact-document metadata, and advisors in parallel.
2. Nested `ibkrdetails` arrays are flattened. IBKR details are indexed by `account.accountId`; a later duplicate account id replaces an earlier map entry.
3. Contacts are indexed by contact id. Advisors are indexed by normalized advisor code.
4. Account-contact links are grouped by account id and also used to identify all accounts associated with a contact.
5. A contact document with its own `account_id` is assigned to that account. If it has no `account_id`, it is assigned to every account linked to its `contact_id`.
6. Account-level document deduplication uses account id, document id or fallback file identifier, category, and contact id. Contact-level deduplication uses contact id, document identifier, and lowercased category.

### Account Focus population

1. An account is included only when it has an internal account id, a nonblank `ibkr_account_number`, and a matching IBKR-details record.
2. Account type comes from IBKR `applicantType`; stored application customer type is the fallback. Values containing `org` become `org`, values containing `joint` become `joint`, and all other values become `individual`.
3. Account title, status, NAV, risk score, open date, and close date come from IBKR details, with internal title used only as the title fallback.
4. NAV removes nonnumeric characters before numeric conversion. Missing IBKR equity remains unknown (`null`) rather than zero.
5. Linked-contact display names come only from account-contact links that resolve to existing contacts, using contact name and then contact email.
6. An IBKR associated person is considered linked only when a linked database contact matches by linked entity id, linked external id, or exact case-insensitive email. Name alone is not a match.
7. The page separately derives `any holder missing contact` and `any non-trusted holder missing contact`; the latter ignores IBKR associated persons whose association contains `Trusted Contact`.

### Contact Focus population

1. Every internal account with an id is considered, even if it has no IBKR number or matching IBKR details.
2. One row is created for each account-contact link that resolves to an existing contact. Accounts with no valid contact link create no Contact Focus row.
3. Missing IBKR enrichment is displayed as blank, `-`, or unknown values; it does not exclude the row.
4. A contact is a company when `contact_type` or `type` equals `company`, or when `company_name` is nonblank. Otherwise it is a person.
5. Phone and employment enrichment comes only from the IBKR associated person whose normalized `entityId` exactly matches the account-contact link's normalized `entity_id`. A missing link entity id or missing exact IBKR person leaves those fields blank; this enrichment does not fall back to external id, email, or name.
6. The displayed phone prefers `phones.mobile`; when mobile is blank, the first nonblank value in the IBKR `phones` object is used. Employment displays the available employment type, occupation, employer, employer business, employer address, and description.
7. A linked contact is a trusted contact only when it matches an IBKR associated person by entity id, external id, or email and that associated person has the `Trusted Contact` association.
8. Advisor status is `available` only when `account.advisor_code` resolves to an advisor and that advisor's linked contact has an email; otherwise it is `no_advisor` or `no_email`.

## Document Completeness and Stage Rules

### Account Focus

- Individual and joint accounts require `Proof of Identity`, `Proof of Address`, and `Source of Wealth`.
- Organization accounts require `Proof of Identity`, `Proof of Existence`, `Proof of Address`, and `Source of Wealth`.
- Presence is based on a case-insensitive exact category name found across all documents assigned to the account.

### Contact Focus

- Person contacts require `Proof of Identity`, `Proof of Address`, and `Source of Wealth`.
- Company contacts require `Proof of Existence`, `Proof of Address`, and `Source of Wealth`.
- Presence is based on the contact's documents, so one contact's document can count on every linked account row for that contact.

### Stage derivation

- Stage 1 — No Documents: zero required categories are present.
- Stage 2 — Some Documents: at least one but not all required categories are present.
- Stage 3 — Complete: every required category is present.
- The stage does not evaluate approval, rejection, issue date, expiration, OCR result, translation, file readability, or whether the evidence actually supports the account.
- Rows sort by Stage 1, Stage 2, Stage 3, then by account title in Account Focus or contact name in Contact Focus.

## Financial Comparison Rules

1. The IBKR side is `detail.financialInformation`.
2. The application side is the first financial-information object under `jointHolders` for `JOINT`, `organization` for `ORG`, and `accountHolder` otherwise.
3. The compared fields are net worth, liquid net worth, and annual net income.
4. Text such as `Level N` is normalized to range id `N`.
5. Both values missing is not a mismatch. One side missing is a mismatch for that field.
6. Equal normalized range ids match.
7. When the IBKR value resolves to known range bounds and the application value is numeric, the numeric value matches if it falls inside the lower and upper bounds. An upper bound of zero/nonpositive is treated as open-ended.
8. When bounds/numeric comparison is unavailable, unequal normalized values are a mismatch.
9. Audit state is `mismatch` if any field mismatches, `match` only when both financial snapshots exist and no field mismatches, and `cannot_compare` when a complete two-sided comparison is unavailable without a derived mismatch.

## Account Focus Filters

All Account Focus filters combine with logical AND. `Reset Filters` returns every filter to `All` and clears search.

| Filter | Exact inclusion rule |
|---|---|
| Account type | All, individual, joint, or org using the normalized account-type rule above. |
| Status | Dynamic exact match on a normalized IBKR status key; punctuation/spaces become underscores and blank becomes `missing`. |
| Application | All; with application when `application_json` is truthy; without application when it is falsy. |
| Financial audit | All, match, mismatch, or cannot compare using the financial rules above. |
| Contacts | All; with contacts when at least one link resolves; without contacts when none resolve; any holder missing contact; or any non-trusted holder missing contact. |
| Account age | All; opened within 1 through 10 years; or missing open date. Age is elapsed days divided by 365.25, the boundary is inclusive, invalid/missing dates are unknown, and future dates are age zero. |
| NAV | All; exactly 0; greater than 0 through 5,000; greater than 5,000 through 10,000; greater than 10,000 through 50,000; or greater than 50,000. Unknown NAV is excluded from every specific band. |
| Search | Case-insensitive substring across account title, IBKR account number, status, and joined linked-contact display names. |

The three Account Focus summary cards are recalculated after filtering: financial mismatches, missing application JSON, and no linked contacts. Account Focus uses 25-row pagination and sorting but has no dedicated CSV export.

## Contact Focus Filters

Different Contact Focus filters combine with logical AND. Within the Status, Missing documents, Responsible, and NAV filters, reviewers may select multiple values and those selected values combine with logical OR. An empty selection means `All`. The initial and reset contact-role filter is `Account holders only`; all other filters reset to `All`, and search resets blank.

| Filter | Exact inclusion rule |
|---|---|
| Stage | All, Stage 1, Stage 2, or Stage 3. |
| Account type | All, individual, joint, or org. |
| Contact type | All, people only, or companies only using the company-contact rule above. |
| Status | Multi-select dynamic normalized IBKR statuses. A row is included when its status matches any selected value. Rows without IBKR status appear under the normalized missing value. |
| Contact role | All contacts; account holders only excludes trusted contacts; trusted contacts only includes only trusted contacts. |
| Date closed | All; closed within 1 through 10 years; or without a closed date. It uses the same elapsed-days/365.25 calculation as account age. A future close date is age zero. |
| IBKR tasks | All or `IBKR flagged only`. A static task maps to address when form number is 8002 or its name contains `proof of address`; to the primary document when form number is 8001/8137 or its name contains `proof of identity`, `identity verification`, or `proof of existence`; and to Source of Wealth when form number is 2150/8553 or its name contains `source of wealth` or `proof of sow`. Any mapped task on the account includes the row. Duplicate indicators with the same form number and name are suppressed. |
| Missing documents | Multi-select missing any; missing POI only for person rows; missing POE only for company rows; missing POA; or missing SOW. A row is included when it matches any selected state. |
| Responsible | Multi-select unassigned when no current user id is stored and Dashboard user ids. A row is included when it matches any selected responsible value. The selectable user list is limited to `@agmtechnology.com`. |
| Account age | All; opened within 1 through 10 years; or missing open date. |
| NAV | Multi-select using the same exact bands as Account Focus. A row is included when its NAV matches any selected band. |
| Search | Case-insensitive substring across contact name/email, IBKR phone and employment fields, account title/number/status, and joined missing-document names. |

The stage chart and counts are recalculated from the filtered rows. The table uses 25-row pagination and sorting.

## Step-by-Step Operating Workflow

1. Open `Accounts Audit`; confirm the page loads without the `Failed to load accounts audit data` error.
2. In Account Focus, review population exceptions: missing application JSON, financial mismatches, no linked contacts, and IBKR holders missing a matched database contact.
3. Apply the exact filters above and retain the filter combination used for any review evidence.
4. Switch to Contact Focus; remember the default population excludes trusted contacts.
5. Review document stages and specific missing categories. IBKR task warning icons are supporting indicators from a static snapshot, not live task confirmation.
6. Assign a responsible user or edit the review comment. Saving upserts the current account/contact record; it does not create immutable assignment/comment history.
7. Upload a document when needed. The action requires a selected row, file, category, and language; only the first selected file is sent. The browser calculates SHA-1 and sends filename, size, MIME type, base64 file data, category, type, language, dates, and comment.
8. Send a missing-documents email only after verifying the contact, missing categories, greeting, primary recipient, CC list, and company name where applicable.
9. Review the email-attempt history and use `View Account` for account-level follow-up.
10. Export the filtered Contact Focus population and retain it with the filter criteria and review date.

## Outreach Rules

- Trusted contacts cannot be assigned as responsible and cannot use upload or missing-document-email actions. Review comments remain editable for trusted-contact rows in the current code.
- Email language defaults to Spanish when the action opens.
- Greeting and recipient candidates combine linked database contacts and IBKR associated persons for the same account, excluding trusted contacts.
- A person row requests POI/POA/SOW; a company row requests POE/POA/SOW. The public upload link includes `contact_id` and required form numbers.
- A corporate email is blocked when the selected company contact name is blank or `-`.
- Related CC candidates are deduplicated case-insensitively and the primary recipient is removed.
- The advisor CC checkbox is selected by default only when an advisor and advisor-contact email resolve; reviewers may uncheck it.
- If no eligible recipient resolves, the current code falls back to `aa@agmtechnology.com` with recipient source `testing`. This is a control limitation and must not be mistaken for client outreach.
- A send creates/updates review email evidence with primary recipient, source, requested keys, language, status, send time, provider id, or error. Selected CC recipients and the initiating user are not retained in that ledger.

## CSV Export

Contact Focus exports the current filtered rows as `contacts-audit-YYYY-MM-DD.csv`. The export includes contact identity/type/role; entity-id-matched IBKR phone and employment fields; account identity/type/status; NAV; open and close dates; account age; risk score; stage; present/required/missing documents; POI/POE/POA/SOW flags; IBKR-task flag; responsible; review comment; sent-email count; last sent time and recipient; and latest attempt status. Values beginning with `=`, `+`, `-`, or `@` are prefixed to reduce spreadsheet-formula execution risk.

## Outputs / Records Created
- Browser-derived Account Focus and Contact Focus review populations
- Current `document_review_responsible` assignment/comment per account/contact pair
- Uploaded raw document and contact-document metadata
- Missing-document email attempt and provider result
- Filtered Contact Focus CSV export
- No persisted overall audit run, filters used, population count, reviewer certification, exception disposition, or completion record

## Exception Paths / Failure Handling
- Initial load failure: an error toast appears and both populations become empty arrays; empty results therefore require confirmation that the load succeeded.
- Assignment metadata failure: a separate error toast appears and responsible/comment/email-history context may be unavailable.
- Upload or email failure: the action shows an error toast; email attempts can remain available as failed evidence when attempt creation succeeded.
- Missing/incorrect account-contact links can omit Contact Focus rows or incorrectly share contact-level documents across linked account rows.
- Missing IBKR details exclude accounts from Account Focus but not Contact Focus, so the two focus populations are not directly reconcilable without applying their documented population rules.
- A static IBKR task snapshot can become stale and is not proof of the live IBKR task state.

## Controls / Verification Points
- Preventive control: contact document requirements are chosen by contact type, preventing person rows from requesting POE and company rows from requesting POI.
- Preventive control: trusted contacts are excluded by default and blocked from assignment, upload, and outreach actions.
- Detective control: missing-holder filters use entity id, external id, or email rather than assuming a name match.
- Detective control: Contact Focus phone and employment fields use only the account-contact entity-id join, preventing one associated person's IBKR data from being attributed to another contact through a loose name or email match.
- Detective control: application and IBKR financial values are displayed side by side with explicit mismatch labels.
- Detective control: Contact Focus exports the exact filtered row set visible to the reviewer.
- Control limitation: Stage 3 proves category presence only, not valid or approved evidence.
- Control limitation: the process is on demand and stores no audit-run completion or population reconciliation.
- Control limitation: the `testing` email fallback can redirect intended outreach to an internal address when no recipient resolves.

## Evidence to Retain
- `contacts-audit-YYYY-MM-DD.csv` with a separate record of filters used
- Current `document_review_responsible` rows
- Uploaded document and contact-document records
- `document_review_email` attempts and Gmail message evidence
- Supporting IBKR-details and financial-range snapshots used for the review
- Follow-up account changes and remediation evidence

## Related Code / Pages / Routes
- Entry: `agm-dashboard/src/app/(dashboard)/(services)/(tools)/(private)/(compliance)/accounts-audit/page.tsx`
- Population and derivation: `agm-dashboard/src/components/dashboard/tools/private/reporting/accounts-audit/AccountsAuditReport.tsx`
- Account filters: `agm-dashboard/src/components/dashboard/tools/private/reporting/accounts-audit/AccountsFocus.tsx`
- Contact filters/actions/export: `agm-dashboard/src/components/dashboard/tools/private/reporting/accounts-audit/ContactsFocus.tsx`
- Static task source: `agm-dashboard/src/components/dashboard/tools/private/reporting/documents-review/ibkrComplianceTasks.ts`
- Public upload: `agm-hub/src/components/hub/apply/ContactDocuments.tsx`
- Downstream records: contact documents, document-review-responsibles, document-review-email attempts, Gmail messages

## Last Reviewed
- Status: draft
- Date: 2026-07-16
- Reviewer: Codex current-state code review and process-owner clarification
