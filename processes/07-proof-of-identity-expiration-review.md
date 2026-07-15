# Proof of Identity Expiration Review (Not a Current Standalone Process)

## Business Purpose
Record that POI-expiration review existed in the legacy Documents Review implementation but is not currently an operating standalone process in Accounts Audit.

## Trigger / Frequency
- Trigger: None in the current Accounts Audit workflow.
- Frequency: Not operating. The old `/proofs-of-identity-review` route redirects to the legacy `/documents-review` route.

## Systems Involved
- Legacy `agm-dashboard` Documents Review component
- Contact-document metadata

## Roles / Owners
- Process owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- No current standalone-process prerequisites.
- Expiration dates may exist in `contact_document`, but current Accounts Audit completeness checks only category presence and do not classify expiration.

## Step-by-Step Workflow
1. Current Accounts Audit checks whether a required document category exists.
2. It does not determine whether a POI expiration date is missing, expired, or expiring soon.
3. It does not remove an expired POI from Stage 3 completeness.
4. Treat POI-expiration monitoring as an open control gap until a current workflow is implemented and approved.

## Outputs / Records Created
- No current standalone review output or retained expiration finding.

## Exception Paths / Failure Handling
- An expired or undated POI can still be counted as present by Accounts Audit.
- The legacy page must not be cited as evidence that periodic POI-expiration review occurs.

## Controls / Verification Points
- Current limitation: category presence is not document validity.
- Required future control: scheduled expiration evaluation, retained findings, assignment, remediation, and closure evidence.

## Evidence to Retain
- Current contact-document expiration metadata
- Any manual review evidence retained outside the application
- Future case and remediation records once implemented

## Related Code / Pages / Routes
- Redirecting route: `agm-dashboard/src/app/(dashboard)/(services)/(tools)/(private)/(compliance)/proofs-of-identity-review/page.tsx`
- Legacy implementation: `agm-dashboard/src/components/dashboard/tools/private/reporting/documents-review/DocumentsReviewPage.tsx`
- Current completeness implementation: `agm-dashboard/src/components/dashboard/tools/private/reporting/accounts-audit/AccountsAuditReport.tsx`
- Gap record: [Operational Lifecycle Control Gaps](gaps/operational-lifecycle-gaps.md)

## Last Reviewed
- Status: retired
- Date: 2026-07-15
- Reviewer: Codex, based on process-owner clarification
