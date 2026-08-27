# Regulatory File Review and Export

## Business Purpose
Assemble an account-level compliance review for one account that combines IBKR account risk data, screening history, application-versus-IBKR financial/profile comparison, linked-contact document completeness, and document links, and allow an operator to export the rendered view as a PDF.

## Trigger / Frequency
- Trigger: An authorized user opens `/accounts/{accountId}/regulatory` and may select Refresh or Download PDF.
- Frequency: On demand.

## Systems Involved
- `agm-dashboard`
- `agm-api`
- IBKR account-details endpoint
- Internal account, contact, account-contact, contact-document, raw-document, and linked `document_processing` records
- IBKR financial-range endpoint

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Internal account with both `master_account` and `ibkr_account_number`
- Live or connector-backed IBKR account details
- Stored `application_json`, when available
- Linked account contacts and their documents
- Financial-range definitions
- Account-screening API response, when available
- Linked document-processing OCR rows, when available

## Step-by-Step Workflow
1. The page loads the internal account and requires its master account and IBKR account number.
2. It requests IBKR account details and reads all account-contact links for the internal account.
3. In parallel, it attempts to load account-level screenings, IBKR financial ranges, all Dashboard contacts, and all linked contacts' document data, metadata, and `document_processing` rows.
4. Screening, financial-range, contact, and document requests use `Promise.allSettled`; a failed request is replaced by an empty collection while the rest of the page continues.
5. The page groups returned account-level screenings by exact case-insensitive holder name and calculates Low, Medium, or High using the approved AML/RBA thresholds: Low when less than or equal to `3.5`, Medium when greater than `3.5` and less than or equal to `5.0`, and High when greater than `5.0`.
6. If no screening-derived account score is available, the page displays the IBKR account risk score.
7. The page derives a single-account document stage using the same required-category rules as Account Focus: individual/joint accounts require Proof of Identity, Proof of Address, and Source of Wealth; organization accounts additionally require Proof of Existence.
8. When `application_json` exists, the page compares application-versus-IBKR financial fields for net worth, liquid net worth, and annual income using IBKR financial-range bounds when needed, and it separately compares account type, base currency, investment objectives, and normalized Source-of-Wealth families.
9. For each linked contact, the page shows contact role, linked IBKR person, entity-linked IBKR phone/employment enrichment, contact-level document stage, missing required documents, and expired document categories using the same Contact Focus document rules: person contacts always require Proof of Identity and Proof of Address, add Source of Wealth only when the linked IBKR employment type is `Employed` or `Self-Employed`, and company contacts require Proof of Existence, Proof of Address, and Source of Wealth.
10. When linked `document_processing` rows exist with completed output text, the page derives the same OCR-backed account-level evidence analysis used by Accounts Audit for Proof of Identity, residence, Proof of Address, and Source of Wealth. For SOW, it compares OCR evidence only against declared liquid net worth and checks liquid net worth against NAV. It shows overall support plus detailed ID-vs-name, residence-profile, POA-vs-residence, and SOW-vs-liquid-net-worth findings, but it still does not include outreach or assignment workflow.
11. The screening log expands each returned screening into FATF, OFAC, UK, UN, False Positive, and True Match display rows. False Positive and True Match are currently displayed as `False` constants rather than stored dispositions.
12. Stored documents can be previewed. PDF documents with data can also be downloaded directly.
13. Download PDF clones the rendered page, removes interactive controls, renders the clone to a canvas, creates a multipage PDF, and adds clickable document links based on their rendered position.

## Outputs / Records Created
- Read-only regulatory account view
- Browser-generated `regulatory-file-{account}.pdf` when requested
- No database record that the file was reviewed, refreshed, or exported

## Exception Paths / Failure Handling
- Missing account, master account, IBKR account number, or IBKR details blocks the page and displays an error toast.
- Account-screening request failure is silently converted to an empty screening history.
- Financial-range or document failure is silently converted to empty data for that section.
- PDF generation failure displays an error toast.
- The Dashboard calls `/accounts/screening`, but no matching `agm-api` route exists in the current API code; the screening portion therefore depends on an unavailable or externally supplied endpoint.

## Controls / Verification Points
- Detective control: the page consolidates account risk, application-versus-IBKR comparison, linked-contact completeness, screening, and document information for one account on one screen.
- Detective control: missing required document categories and expired linked-contact documents are displayed, and documentation is grouped by assigned contact only.
- Detective control: the PDF can retain the rendered review context and document links when an operator saves it.
- Current limitation: there is no recorded review decision, reviewer, export event, source snapshot, or approval.
- Current limitation: screening failures can appear as zero screenings without a visible error.
- Current limitation: the OCR support analysis depends on stored `document_processing` text quality and linked document coverage; missing or low-quality OCR can still force `missing_evidence` or `cannot_compare`.

## Evidence to Retain
- Exported regulatory PDF, when an operator creates and retains it
- Underlying account, IBKR detail, document, and screening records
- Page error toast or browser logs for failed sections
- Source document files linked from the view

## Related Code / Pages / Routes
- Entry surfaces: `agm-dashboard/src/app/(dashboard)/(services)/(clients)/accounts/[accountId]/regulatory/page.tsx`
- Supporting modules: `agm-dashboard/src/components/dashboard/clients/accounts/AccountRegulatoryPage.tsx`, `agm-dashboard/src/utils/clients/account.ts`
- Related review process: `compliance/processes/22-accounts-audit-review.md`
- Downstream side effects: browser PDF generation and direct document download

## Last Reviewed
- Status: draft
- Date: 2026-07-22
- Reviewer: Codex current-state code review
