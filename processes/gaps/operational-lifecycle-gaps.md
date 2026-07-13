# Operational Lifecycle Control Gaps

## Purpose
Define the operational controls required for AGM to manage account applications, compliance documents, review findings, and IBKR submission through a trustworthy and auditable lifecycle.

This document records confirmed gaps in the current documented and implemented workflows. It is a remediation backlog, not a description of controls that already operate.

## Scope
This first remediation scope covers:

- Category-aware document validation
- Source-of-Wealth requirement alignment
- Automatic OCR and document processing
- Spanish-to-English translation
- Validity-based document completeness
- Extracted-data reconciliation
- Contact-level expiration monitoring
- Review case management
- Scheduled audits and alerts
- Explicit application review decisions
- Server-side submission preflight
- Immutable audit history and retention
- Secure public document-upload requests

The original gap identifiers are retained so implementation work, process updates, and change records can refer to stable IDs.

## Current-State Summary
The current workflows provide useful application capture, document upload, screening, audit tables, reviewer assignments, and IBKR submission checks. The following weaknesses prevent the lifecycle from being treated as fully controlled:

- Uploaded document metadata is not validated consistently by category at the API boundary.
- Document presence can be treated as completeness without proving validity or review.
- Source-of-Wealth evidence is required by Accounts Audit but not by the IBKR submission gate.
- OCR capability exists but is not automatically triggered by document upload.
- Spanish document language is recorded, but translation is not part of the processing lifecycle.
- Review assignments contain an owner and comment but no case status, due date, disposition, or closure history.
- Review pages are primarily on-demand and do not create persistent alerts automatically.
- Sending an application to IBKR acts as an implicit approval rather than following a recorded approval decision.
- Critical edits and document deletion do not have a documented immutable application-level audit trail or retention workflow.
- Public compliance upload links identify the target contact but do not document a signed, expiring authorization control.

## Priority Summary

| Gap | Control | Priority | Primary dependency |
| --- | --- | --- | --- |
| GAP-001 | Category-aware document validation | P0 | Document policy matrix |
| GAP-002 | Source-of-Wealth requirement alignment | P0 | GAP-001 |
| GAP-003 | Automatic document-processing pipeline | P0 | Processing storage and worker execution |
| GAP-004 | Spanish translation with original preservation | P1 | GAP-003 |
| GAP-005 | Validity-based document completeness | P0 | GAP-001, GAP-003 |
| GAP-006 | Extracted-data reconciliation | P1 | GAP-003, GAP-004 |
| GAP-007 | Expiration monitoring in Contact Focus | P0 | GAP-001, GAP-005 |
| GAP-008 | Review case lifecycle | P0 | Case data model and audit events |
| GAP-009 | Scheduled audits and alerts | P0 | GAP-005, GAP-007, GAP-008 |
| GAP-010 | Explicit application-review lifecycle | P0 | Application state model and audit events |
| GAP-011 | Server-side submission preflight | P0 | GAP-002, GAP-005, GAP-008, GAP-010 |
| GAP-014 | Immutable audit trail and retention | P0 | Retention policy and audit-event model |
| GAP-015 | Secure public document-upload requests | P0 | Upload-request token model |

## GAP-001 - Category-Aware Document Validation

### Current State
- Hub and Dashboard upload forms require category, type when configured, and declared language.
- Issue date and expiration date remain optional.
- The contact-document API accepts document metadata without category-specific validation and requires only `contact_id` before creating the raw document and relationship records.
- Applicant identification expiration is already required in the application schema, but the uploaded POI metadata is not required to carry the same value.

### Required Control
- Define one server-owned document policy matrix by account type, holder role, category, and document type.
- Require expiration date for Proof of Identity unless an explicitly permitted non-expiring document rule applies.
- Require an approved waiver reason for any permitted expiration exception.
- Validate issue date is not after expiration date and that expiration values are parseable.
- Prevent expired or otherwise invalid documents from satisfying application readiness, even if the file is retained as evidence.
- Apply the same rules to Hub uploads, Dashboard uploads, public compliance uploads, metadata edits, and orphan-document relinking.

### Acceptance Criteria
- The API rejects POI metadata with no expiration date unless a valid policy exception and reason are supplied.
- UI validation mirrors the API but is not the authoritative control.
- Existing records with missing or invalid required metadata appear in a remediation queue.
- A document cannot become `valid` solely because its category and type are populated.
- Automated tests cover valid, expired, missing, malformed, and non-expiring-exception cases.

### Affected Processes
- [Documents Review Workflow](../06-documents-review-workflow.md)
- [Proof of Identity Expiration Review](../07-proof-of-identity-expiration-review.md)
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)

## GAP-002 - Source-of-Wealth Requirement Alignment

### Current State
- The account application schema requires at least one Source-of-Wealth declaration in financial information.
- Accounts Audit expects Source-of-Wealth supporting evidence for individual, joint, and organization account types.
- The Dashboard IBKR submission gate requires W8, Proof of Identity, and Proof of Address, but not a Source-of-Wealth document.

### Required Control
- Approve and document one Source-of-Wealth evidence policy by account type, holder role, risk level, and declared source type.
- Require supporting evidence for every in-scope applicant or holder defined by that policy.
- Block approval and IBKR submission when required SOW evidence is absent, invalid, unreadable, or unresolved.
- Treat declarations and supporting evidence as separate requirements.

### Acceptance Criteria
- The application review and Accounts Audit use the same policy matrix.
- The server-side preflight identifies the specific holder and missing SOW requirement.
- SOW evidence cannot satisfy the requirement until it reaches an accepted review state.
- Any waiver requires an authorized reviewer, reason, timestamp, and retained evidence.

### Affected Processes
- [Documents Review Workflow](../06-documents-review-workflow.md)
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)

## GAP-003 - Automatic Document-Processing Pipeline

### Current State
- The API contains PDF text extraction and EasyOCR fallback capability.
- Upload routes store the raw file and metadata without triggering text extraction.
- Processing status can be read for existing records, but no documented upload-to-processing lifecycle guarantees that every required document is processed.
- The current extraction path supports PDF and plain-text inputs; common image uploads require equivalent OCR support.

### Required Control
- Create a durable asynchronous processing job after the raw document is stored successfully.
- Support at minimum PDF, JPEG, and PNG input for compliance documents.
- Track `pending`, `processing`, `completed`, `failed`, and `manual_review_required` states.
- Make processing idempotent by document id and processing version.
- Add controlled retries, maximum attempts, error classification, and operator reprocessing.
- Create or update a review case when processing fails or remains incomplete beyond its service level.

### Acceptance Criteria
- Every new in-scope upload receives a processing record automatically.
- Upload success is distinguishable from processing success.
- Failed processing never silently counts as document completeness.
- Operators can inspect status, error, attempt count, provider, version, and timestamps.
- Reprocessing does not create conflicting active results.

### Affected Processes
- [Documents Review Workflow](../06-documents-review-workflow.md)
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)

## GAP-004 - Spanish Translation With Original Preservation

### Current State
- Upload flows require a declared English or Spanish document language.
- Spanish documents are not automatically translated as part of the documented upload or review workflow.
- The original extracted text and a reviewer-facing English translation are not defined as separate controlled outputs.

### Required Control
- Detect source language and compare it with the declared language.
- Translate completed Spanish extracted text into English automatically.
- Preserve the original file and original extracted text unchanged.
- Store translated text separately with provider, model/version, source language, target language, processing status, and timestamps.
- Allow a reviewer to accept, reject, or correct the translation without overwriting the original result.

### Acceptance Criteria
- Spanish documents automatically enter translation after successful extraction.
- A language mismatch creates a review finding.
- Reviewers can view original and translated text side by side.
- Translation failure produces a visible case and cannot be presented as completed review.
- All translation corrections and approvals are auditable.

### Affected Processes
- [Documents Review Workflow](../06-documents-review-workflow.md)
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)

## GAP-005 - Validity-Based Document Completeness

### Current State
- Accounts Audit primarily determines completeness from the presence of a `contact_document` category.
- A document can therefore appear present even when its file is unreadable, metadata is incomplete, it is expired, processing failed, or review has not occurred.

### Required Control
- Define document states such as `received`, `processing`, `review_required`, `accepted`, `rejected`, `expired`, and `superseded`.
- Calculate account/contact completeness only from accepted, current, correctly linked documents.
- Preserve rejected, expired, and superseded evidence without allowing it to satisfy readiness.
- Show the reason a document does not satisfy a requirement.

### Acceptance Criteria
- Accounts Audit Stage 3 means every required document has a current accepted document.
- Missing, failed, rejected, expired, and superseded documents produce distinct findings.
- Completeness calculation is server-owned or uses a shared authoritative rules implementation.
- Manual acceptance records reviewer, timestamp, disposition, and any exception reason.

### Affected Processes
- [Documents Review Workflow](../06-documents-review-workflow.md)
- [Proof of Identity Expiration Review](../07-proof-of-identity-expiration-review.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)

## GAP-006 - Extracted-Data Reconciliation

### Current State
- Extracted Source-of-Wealth text can be displayed when processing results exist.
- The workflow does not define automated comparison between document content, application data, contact data, and IBKR data.

### Required Control
- Extract structured candidate fields appropriate to each document category.
- Compare POI name, identification number, issuing country, and expiration against the application/contact record.
- Compare POA holder and address against the application address.
- Compare SOW holder, source type, and relevant financial context against declarations and financial profile.
- Compare organization evidence against legal name, registration number, and jurisdiction.
- Route material mismatches to a reviewer without automatically overwriting source data.

### Acceptance Criteria
- Each comparison records source value, extracted value, match result, confidence, and rule version.
- Material mismatches create one deduplicated review case.
- Reviewers can confirm match, record a justified exception, request new evidence, or correct source data through an auditable action.
- Low-confidence extraction cannot automatically resolve a requirement.

### Affected Processes
- [Documents Review Workflow](../06-documents-review-workflow.md)
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)

## GAP-007 - Expiration Monitoring in Contact Focus

### Current State
- A standalone POI Expiration Review classifies missing, expired, expiring-soon, and valid dates when a user opens the page.
- Contact Focus shows document presence but not the contact's current POI validity posture.
- Expiration review is not connected to case status, owner, due date, outreach, replacement, or closure.

### Required Control
- Add POI validity and next-expiration information to Contact Focus.
- Identify missing expiration, expired POI, expiring thresholds, application/document date mismatches, and contacts with no current accepted POI.
- Apply the review to all active in-scope contacts, not only accounts opened within an arbitrary recent period unless a documented policy explicitly permits that limit.
- Link each actionable finding to its case and replacement-document lifecycle.

### Acceptance Criteria
- Contact Focus supports filters for missing, expired, expiring, mismatched, and valid POI states.
- Thresholds are configurable and documented.
- New expiration findings create cases automatically without duplicating an open case.
- A case closes only after an accepted replacement or approved exception exists.
- Scheduled processing detects expiration changes without requiring a user to open the page.

### Affected Processes
- [Documents Review Workflow](../06-documents-review-workflow.md)
- [Proof of Identity Expiration Review](../07-proof-of-identity-expiration-review.md)

## GAP-008 - Review Case Lifecycle

### Current State
- Document review supports an assigned user and freeform comment for an account/contact pair.
- It does not define a case type, severity, lifecycle status, due date, evidence list, disposition, closure reason, or immutable event history.

### Required Control
- Create a persistent case model for compliance and operational findings.
- Support at minimum `open`, `assigned`, `waiting_for_client`, `under_review`, `resolved`, and `closed` states.
- Record finding type, source, severity, owner, due date, linked account/contact/document, evidence, disposition, closure reason, and timestamps.
- Prevent duplicate active cases for the same finding identity while retaining recurrence history.
- Require controlled transitions and authorization for resolution and closure.

### Acceptance Criteria
- Document gaps, OCR/translation failures, expiration findings, data mismatches, screening exceptions, and stale required data can create cases.
- Every case has one current owner or an explicit unassigned queue.
- Every transition records actor, timestamp, prior state, new state, and reason.
- Closed cases remain searchable and retain their evidence.
- Reappearing conditions reopen the prior case or create a linked recurrence according to a documented rule.

### Affected Processes
- [Daily Screening Run](../01-daily-screening-run.md)
- [Documents Review Workflow](../06-documents-review-workflow.md)
- [Proof of Identity Expiration Review](../07-proof-of-identity-expiration-review.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)

## GAP-009 - Scheduled Audits and Alerts

### Current State
- Accounts Audit, Documents Review, and POI Expiration Review primarily calculate findings when a user opens the relevant page.
- Assignments and comments do not provide a scheduled alert, acknowledgment, escalation, or overdue workflow.

### Required Control
- Run a scheduled audit that evaluates document validity, missing evidence, processing failures, expiration, source-data freshness, and unresolved IBKR tasks.
- Create or update deduplicated cases from findings.
- Notify owners of new, due-soon, overdue, and escalated cases.
- Require acknowledgment and escalate unowned or overdue high-priority findings.
- Record each scheduled run, rules version, counts, failures, and reconciliation outcome.

### Acceptance Criteria
- Findings are detected without a reviewer opening a page.
- Repeated runs do not create duplicate active cases.
- Failed or partial runs generate an operational alert and never report false success.
- Dashboard totals reconcile to the latest successful audit run.
- Notification and escalation events are retained as evidence.

### Affected Processes
- [Documents Review Workflow](../06-documents-review-workflow.md)
- [Proof of Identity Expiration Review](../07-proof-of-identity-expiration-review.md)
- [Daily IBKR Details Backup and Reporting Feed](../13-daily-ibkr-details-backup.md)

## GAP-010 - Explicit Application-Review Lifecycle

### Current State
- Applications are stored internally and reviewed in Dashboard before an operator sends them to IBKR.
- Sending to IBKR acts as the practical approval action.
- The documented workflow does not define a persistent review decision, rejection/hold outcome, decision reason, reviewer, or approval timestamp.

### Required Control
- Introduce application states including `received`, `under_review`, `information_requested`, `ready_for_approval`, `approved_for_ibkr`, `submitted_to_ibkr`, `rejected`, `withdrawn`, and `submission_failed`.
- Record each transition with actor, timestamp, reason, and supporting case/evidence references.
- Separate approval from the external submission attempt.
- Require secondary approval for defined high-risk or exception scenarios.

### Acceptance Criteria
- Every application has one current state and complete transition history.
- Rejection, withdrawal, information request, approval, submission, and submission failure are distinguishable.
- Only an approved application can enter IBKR submission.
- Submission failure does not erase or reverse the recorded approval decision.
- High-risk approval rules identify the required approving role.

### Affected Processes
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)

## GAP-011 - Server-Side Submission Preflight

### Current State
- Dashboard performs document payload, file-size, document-type, and required-form checks before sending an application to IBKR.
- The documented control is primarily UI-driven and does not require a recorded approval decision or resolution of every operational finding.

### Required Control
- Create one authoritative server-side preflight used by both the readiness display and the IBKR submission route.
- Evaluate application approval state, required accepted documents, SOW policy, document processing, translation where required, expiration, reconciliation findings, screening freshness/results, open blocking cases, and required source-data freshness.
- Return structured blocker and warning codes tied to the affected holder, document, case, or source.
- Re-run preflight inside the submission transaction immediately before the external request.

### Acceptance Criteria
- Direct API calls cannot bypass the same controls enforced in Dashboard.
- The readiness UI and submission route consume the same preflight result.
- Blocking conditions prevent the IBKR request and create auditable evidence.
- Warnings require explicit acknowledgment where policy permits continuation.
- The stored submission record includes the preflight result and rules version used.

### Affected Processes
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [Daily Screening Run](../01-daily-screening-run.md)

## GAP-014 - Immutable Audit Trail and Retention

### Current State
- Contact-document metadata can be updated in place.
- Contact documents and orphaned raw documents can be physically deleted.
- The process manuals identify records and exports to retain, but no unified application-level audit-event or retention workflow is documented.
- ITGC controls for regulatory retention and transaction audit trails remain open in the role register.

### Required Control
- Create an append-only audit-event model for critical application and compliance actions.
- Record actor, action, entity, entity id, timestamp, request/correlation id, reason, and appropriate before/after values.
- Replace routine physical deletion of compliance evidence with controlled archival or soft deletion.
- Define retention periods, legal-hold behavior, approved purge, and restoration procedures.
- Restrict destructive evidence actions and require a reason plus secondary approval where appropriate.

### Acceptance Criteria
- Document upload, link, relink, metadata edit, review decision, waiver, deletion/archive, case transition, application decision, and IBKR submission produce audit events.
- Audit events cannot be modified through normal application APIs.
- Archived evidence cannot satisfy active document requirements but remains retrievable by authorized users.
- Retention and purge jobs produce reconciliation reports and approval evidence.
- Process manuals and the ITGC transaction register identify the retained audit evidence.

### Affected Processes
- [Documents Review Workflow](../06-documents-review-workflow.md)
- [Proof of Identity Expiration Review](../07-proof-of-identity-expiration-review.md)
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)
- [Contact Deduplication and Dependency Merge](../14-contact-deduplication-workflow.md)

## GAP-015 - Secure Public Document-Upload Requests

### Current State
- Missing-document outreach can include a public AGM Hub upload link keyed to a target `contact_id`.
- The process manual does not define a signed, expiring, revocable authorization token or a request-specific category scope.

### Required Control
- Generate a cryptographically signed, random, expiring upload-request token instead of relying on the contact identifier as authorization.
- Scope the request to the intended contact, account, requested categories, expiration, and permitted upload count.
- Store request status such as `issued`, `opened`, `partially_fulfilled`, `fulfilled`, `expired`, and `revoked`.
- Apply rate limits, file-security checks, and audit logging.
- Prevent the public client from selecting a different contact or unauthorized document category.

### Acceptance Criteria
- A contact id alone cannot authorize a public upload.
- Expired, revoked, altered, or overused tokens are rejected.
- Successful uploads are linked only to the token's permitted contact/account and category scope.
- Opening, upload attempts, successful uploads, failures, expiration, and revocation are auditable.
- Completing the requested evidence updates the related case and upload request without automatically accepting the document.

### Affected Processes
- [Documents Review Workflow](../06-documents-review-workflow.md)
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)

## Implementation Sequence

1. Define the document policy matrix and retention/audit requirements for GAP-001, GAP-002, and GAP-014.
2. Add the document validity, processing, translation, and reconciliation foundations for GAP-003 through GAP-006.
3. Add the case and application state models for GAP-008 and GAP-010.
4. Integrate expiration findings and scheduled audits through GAP-007 and GAP-009.
5. Make GAP-011 preflight authoritative before changing the IBKR submission route.
6. Replace contact-id public uploads with GAP-015 request tokens before expanding public outreach.

Implementation changes that add or alter database records must include SQL migration scripts under the workspace-level `sql/` directory.

## Governance and Closure

- Each gap remains open until its acceptance criteria are implemented, tested, and reflected in the affected process manuals.
- Any implementation that changes a critical transaction flow must update `compliance/itgc/transaction-processing-register.csv` in the same change.
- A gap may be closed only with retained test evidence, an updated process/control description, and business-owner confirmation.
- Partial implementation must be recorded explicitly; the gap must not be marked closed while a required control remains manual or unenforced.

## Last Reviewed
- Status: proposed
- Date: 2026-07-13
- Reviewer: Codex gap assessment
