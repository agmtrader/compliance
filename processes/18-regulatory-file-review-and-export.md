# Regulatory File Review and Export

## Business Purpose
Assemble an account-level regulatory view containing IBKR account risk data, holder screening information, financial information, document coverage, and document links, and allow an operator to export the rendered view as a PDF.

## Trigger / Frequency
- Trigger: An authorized user opens `/accounts/{accountId}/regulatory` and may select Refresh or Download PDF.
- Frequency: On demand.

## Systems Involved
- `agm-dashboard`
- `agm-api`
- IBKR account-details endpoint
- Internal account, account-contact, contact-document, and raw-document records
- IBKR financial-range endpoint

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Internal account with both `master_account` and `ibkr_account_number`
- Live or connector-backed IBKR account details
- Linked account contacts and their documents
- Financial-range definitions
- Account-screening API response, when available

## Step-by-Step Workflow
1. The page loads the internal account and requires its master account and IBKR account number.
2. It requests IBKR account details and reads all account-contact links for the internal account.
3. In parallel, it attempts to load account-level screenings, IBKR financial ranges, and all linked contacts' document data and metadata.
4. Screening, financial-range, and document requests use `Promise.allSettled`; a failed request is replaced by an empty collection while the rest of the page continues.
5. The page groups returned account-level screenings by exact case-insensitive holder name and calculates Low, Medium, or High using score thresholds below `3.5`, below `7`, or `7` and above.
6. If no screening-derived account score is available, the page displays the IBKR account risk score.
7. The screening log expands each returned screening into FATF, OFAC, UK, UN, False Positive, and True Match display rows. False Positive and True Match are currently displayed as `False` constants rather than stored dispositions.
8. The page displays account financial ranges, objectives, Sources of Wealth, holder screening coverage, document-category counts, unassigned documents, and missing Proof of Identity, Proof of Address, and Source of Wealth by relevant holder.
9. For organizations, document-gap review uses only the first associated person. For other account types, trusted contacts are excluded where possible.
10. Stored documents can be previewed. PDF documents with data can also be downloaded directly.
11. Download PDF clones the rendered page, removes interactive controls, renders the clone to a canvas, creates a multipage PDF, and adds clickable document links based on their rendered position.

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
- Detective control: the page consolidates account risk, financial, screening, and document information for review.
- Detective control: missing required document categories and unassigned documents are displayed.
- Detective control: the PDF can retain the rendered review context and document links when an operator saves it.
- Current limitation: there is no recorded review decision, reviewer, export event, source snapshot, or approval.
- Current limitation: screening failures can appear as zero screenings without a visible error.

## Evidence to Retain
- Exported regulatory PDF, when an operator creates and retains it
- Underlying account, IBKR detail, document, and screening records
- Page error toast or browser logs for failed sections
- Source document files linked from the view

## Related Code / Pages / Routes
- Entry surfaces: `agm-dashboard/src/app/(dashboard)/(services)/(clients)/accounts/[accountId]/regulatory/page.tsx`
- Supporting modules: `agm-dashboard/src/components/dashboard/clients/accounts/AccountRegulatoryPage.tsx`, `agm-dashboard/src/utils/clients/account.ts`
- Downstream side effects: browser PDF generation and direct document download

## Last Reviewed
- Status: draft
- Date: 2026-07-15
- Reviewer: Codex current-state code review
