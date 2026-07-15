# Legacy Documents Review Workflow (Retired)

## Business Purpose
Preserve traceability for the former Dashboard `Documents Review` workflow. This is not the current compliance-review process. The current operating workflow is [Accounts Audit Review](22-accounts-audit-review.md), implemented by `AccountsAuditReport.tsx` and its Account Focus and Contact Focus components.

## Trigger / Frequency
- Trigger: None as an approved operating process.
- Frequency: Retired. The `/documents-review` route and `DocumentsReviewPage.tsx` remain in the repository but are not used for the current review.

## Systems Involved
- Legacy `agm-dashboard` route and component only
- Superseding `agm-dashboard` Accounts Audit page

## Roles / Owners
- Process owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- No current operating prerequisites.
- The retained code previously read accounts, contacts, account-contact links, IBKR details, documents, reviewer assignments, and users.

## Step-by-Step Workflow
1. Do not use the legacy Documents Review page as evidence of the current compliance review.
2. Open `Accounts Audit` for the current account- and contact-level audit.
3. Apply the population and filter rules documented in [Accounts Audit Review](22-accounts-audit-review.md).
4. Retain this record until the legacy route and component are removed or formally reactivated.

## Outputs / Records Created
- No approved current-process output.
- Historical code may still be technically reachable, but an output from that surface is not evidence that the current Accounts Audit process was performed.

## Exception Paths / Failure Handling
- If a user reaches `/documents-review`, stop and use `/accounts-audit` for the current process.
- Any intended continued use of legacy functions such as POI expiration, unlinked-document review, or no-type-document review must first be confirmed by the process owner and documented as a current process.

## Controls / Verification Points
- Preventive governance control: current process documentation references `AccountsAuditReport.tsx`, not the retired component.
- Detective governance control: code reachability and operational use are recorded separately; a file remaining in the repository does not make it a current control.
- Current limitation: the application still exposes a route importing the retired component, which can confuse users and documentation reviews.

## Evidence to Retain
- This retirement record
- The current Accounts Audit process manual and exports
- Change evidence if the legacy route/component is removed

## Related Code / Pages / Routes
- Retired route: `agm-dashboard/src/app/(dashboard)/(services)/(tools)/(private)/(compliance)/documents-review/page.tsx`
- Retired component: `agm-dashboard/src/components/dashboard/tools/private/reporting/documents-review/DocumentsReviewPage.tsx`
- Current route: `agm-dashboard/src/app/(dashboard)/(services)/(tools)/(private)/(compliance)/accounts-audit/page.tsx`
- Current component: `agm-dashboard/src/components/dashboard/tools/private/reporting/accounts-audit/AccountsAuditReport.tsx`

## Last Reviewed
- Status: retired
- Date: 2026-07-15
- Reviewer: Codex, based on process-owner clarification
