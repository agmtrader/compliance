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
- Reviewer assignment and comment change history
- Missing-document outreach history and reconciliation
- Scheduled audits and alerts
- Explicit application review decisions
- Server-side submission preflight
- Immutable audit history and retention
- Secure public document-upload requests
- Correct AML risk-assessment inputs
- Sanctions candidate matching and review
- Current FATF jurisdiction reference data
- Persistent transaction-monitoring findings
- Compliance-reference refresh integrity and freshness evidence
- Daily screening population coverage and cache invalidation
- Regulatory-file review data integrity and audit evidence
- Investment-business report reproducibility and approval
- Compliance-manual change notification and acknowledgment
- Complete and current IBKR account-details refreshes
- AGM-to-IBKR account-population reconciliation
- Effective-dated IBKR associated-person and contact-link reconciliation
- Authoritative KYC/AML readiness and account-activation gating
- Resumable onboarding and incomplete-application remediation
- Ongoing and event-driven customer due diligence
- Beneficial-ownership and organization-control structures
- PEP, adverse-media, and regulatory-action screening
- AML risk-model governance and controlled overrides
- Suspicious-activity investigation and SAR/STR decision lifecycles
- Expanded transaction-monitoring coverage
- Source-of-Funds and funding-party verification
- Account closure, holder removal, dormancy, and offboarding
- Compliance exceptions, waivers, and secondary approval
- Compliance control testing, quality assurance, and certification
- Regulatory-obligations and IBKR-reliance mapping

The original gap identifiers are retained so implementation work, process updates, and change records can refer to stable IDs.

## Current-State Summary
The current workflows provide useful application capture, document upload, screening, audit tables, reviewer assignments, and IBKR submission checks. The following weaknesses prevent the lifecycle from being treated as fully controlled:

- Uploaded document metadata is not validated consistently by category at the API boundary.
- Document presence can be treated as completeness without proving validity or review.
- Source-of-Wealth evidence is required by Accounts Audit but not by the IBKR submission gate.
- OCR capability exists but is not automatically triggered by document upload.
- Spanish document language is recorded, but translation is not part of the processing lifecycle.
- Review assignments contain one mutable owner and comment per saved account/contact pair. A later save overwrites the prior assignment and comment, and no actor-specific change history is retained.
- Missing-document emails are sent synchronously through Gmail and now persist a review-linked attempt with primary recipient/source, requested-document set, language, immediate result, send timestamp, failure text, and Gmail message id. The initiating user, selected CC recipients, provider-side delivery state, and explicit resend relationship are not retained.
- Review pages are primarily on-demand and do not create persistent alerts automatically.
- Sending an application to IBKR acts as an implicit approval rather than following a recorded approval decision.
- Critical edits and document deletion do not have a documented immutable application-level audit trail or retention workflow.
- Public compliance upload links identify the target contact but do not document a signed, expiring authorization control.
- AML risk assessment can read a different country field than the Hub stores and can fall back to the wrong joint or organization person.
- Sanctions screening relies on exact normalized-name matches and does not record a reviewer disposition for possible matches.
- FATF-listed jurisdictions are maintained in a hardcoded application array rather than refreshed reference data.
- Transaction-monitoring flags are calculated in Dashboard, while the database retains only freeform comments and can create duplicate rows for the same transaction.
- Compliance-list ETL failures can be returned as HTTP `200` partial results and still produce a GitHub success email; source date, row count, checksum, and exact version are not recorded as control evidence.
- The scheduled screening job skips contact coverage when all sanctions files are unchanged, can reuse stale worker-level caches, includes trusted contacts, and can duplicate same-day rows when coverage is partial.
- The regulatory-file page calls an account-screening route absent from the current API, converts failed sections to empty data, and displays false-positive and true-match dispositions as hardcoded `False`.
- Investment-business reporting is a read-only view without persisted completion, exception disposition, preparer/reviewer approval, or a reproducible run snapshot.
- The compliance-manual notification endpoint sends a generic email to one hardcoded recipient without linking it to a document version or recording acknowledgment.
- The IBKR account-details backup carries prior account details forward and requests fresh details only for account numbers missing from the retained file, so changes to existing accounts and associated persons can remain stale indefinitely.
- No completed reconciliation proves that every in-scope IBKR account exists once in AGM or identifies IBKR-only, AGM-only, duplicate, wrong-master-account, and status-mismatch populations.
- Account-contact links do not retain an authoritative role, active/inactive state, effective dates, source snapshot, or controlled removal history.
- There is no single compliance-readiness decision that proves KYC, AML assessment, current screening, accepted documents, beneficial ownership, and blocking-case resolution before IBKR submission and live-account activation.
- Onboarding creates and changes records through multiple independent operations without a durable checkpoint model that identifies and safely resumes incomplete applications.
- Document-expiration review does not constitute a complete risk-based ongoing CDD program with scheduled and event-driven reassessment.
- Organization review does not maintain a complete versioned ownership and control structure covering indirect owners, beneficial owners, control persons, nominees, and other applicable roles.
- The documented screening program does not define separate PEP, relative/close-associate, adverse-media, or regulatory-action candidate review.
- AML scores do not have a documented model-governance lifecycle covering rule versions, validation, calibration, approval, and controlled overrides.
- Transaction findings do not feed a restricted suspicious-activity investigation, SAR/STR decision, filing, deadline, confidentiality, or continuing-activity workflow.
- Current transaction monitoring is on demand and does not cover aggregated behavior, structuring, rapid movement, third-party funding, dormant reactivation, related-account patterns, or other approved scenarios.
- Source-of-Wealth evidence is not paired with a policy for verifying the source and ownership of funds used for material or unusual funding events.
- Account closure, holder removal, dormancy, reactivation, rejection, and withdrawal are not governed by a complete compliance offboarding lifecycle.
- Compliance exceptions and waivers are not centrally recorded with scope, risk, compensating controls, approval, expiration, and preflight effect.
- The process library defines acceptance criteria but does not define a control-testing and independent quality-assurance program that proves the controls operate effectively.
- The repository does not map each AML/KYC obligation to AGM, IBKR, or shared responsibility by legal entity, jurisdiction, product, account type, deadline, evidence, and retention requirement.

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
| GAP-012 | Correct AML risk-assessment inputs | P0 | Existing application and contact identifiers |
| GAP-013 | Sanctions candidate matching and review | P0 | GAP-012 |
| GAP-014 | Immutable audit trail and retention | P0 | Retention policy and audit-event model |
| GAP-015 | Secure public document-upload requests | P0 | Upload-request token model |
| GAP-016 | Current FATF jurisdiction reference data | P0 | Existing clients ETL pipeline |
| GAP-017 | Persistent transaction-monitoring findings | P0 | Existing deposits and withdrawals report |
| GAP-018 | Compliance-reference refresh integrity and freshness evidence | P0 | Existing clients ETL pipeline |
| GAP-019 | Daily screening population coverage and cache invalidation | P0 | GAP-018 |
| GAP-020 | Regulatory-file review data integrity and audit evidence | P0 | GAP-013, GAP-014 |
| GAP-021 | Retired - former fee-template review gap | N/A | Consolidated into GAP-024 on 2026-07-15 |
| GAP-022 | Investment-business report reproducibility and approval | P1 | Reporting snapshot and approval model |
| GAP-023 | Compliance-manual change notification and acknowledgment | P1 | Documentation governance workflow |
| GAP-024 | Accounts Audit population reconciliation and review evidence | P0 | GAP-005, GAP-009, GAP-013, GAP-014 |
| GAP-025 | Complete IBKR account-details refresh | P0 | Existing IBKR details ETL and master-account inventory |
| GAP-026 | AGM/IBKR account-population reconciliation | P0 | GAP-025 |
| GAP-027 | Associated-person and contact-link lifecycle reconciliation | P0 | GAP-008, GAP-014, GAP-025, GAP-026 |
| GAP-028 | Authoritative KYC/AML readiness and activation gate | P0 | GAP-011, GAP-025 through GAP-027, GAP-030 through GAP-033 |
| GAP-029 | Resumable onboarding and incomplete-application remediation | P0 | GAP-008, GAP-010, GAP-014 |
| GAP-030 | Ongoing and event-driven CDD/KYC refresh | P0 | GAP-008, GAP-009, GAP-014, GAP-025 through GAP-027 |
| GAP-031 | Beneficial ownership and organization-control structure | P0 | GAP-001, GAP-005, GAP-008, GAP-012 through GAP-014, GAP-027 |
| GAP-032 | PEP, adverse-media, and regulatory-action screening | P0 | GAP-008, GAP-013, GAP-014, GAP-018, GAP-019 |
| GAP-033 | AML risk-model governance and overrides | P1 | GAP-012, GAP-014, GAP-016, GAP-030 through GAP-032 |
| GAP-034 | Suspicious-activity investigation and SAR/STR lifecycle | P0 | GAP-008, GAP-014, GAP-017, GAP-035, GAP-040 |
| GAP-035 | Expanded transaction-monitoring coverage | P0 | GAP-008, GAP-014, GAP-017, GAP-025, GAP-026 |
| GAP-036 | Source-of-Funds and funding-party verification | P0 | GAP-002, GAP-005, GAP-008, GAP-014, GAP-017, GAP-035 |
| GAP-037 | Account closure, holder removal, dormancy, and offboarding | P1 | GAP-008, GAP-014, GAP-025 through GAP-030, GAP-034 |
| GAP-038 | Compliance exceptions, waivers, and secondary approval | P0 | GAP-008, GAP-010, GAP-011, GAP-014, GAP-028 |
| GAP-039 | Compliance control testing, QA, and certification | P1 | Implemented controls and retained test evidence |
| GAP-040 | Regulatory obligations and IBKR reliance matrix | P0 | Legal and compliance review of AGM's operating perimeter |

## GAP-001 - Category-Aware Document Validation

### Current State
- Hub and Dashboard upload forms require category, type when configured, and declared language.
- Issue date and expiration date remain optional.
- The contact-document API accepts document metadata without category-specific validation and requires only `contact_id` before creating the raw document and relationship records.
- Applicant identification expiration is already required in the application schema, but the uploaded POI metadata is not required to carry the same value.
- On 2026-07-16, a controlled one-time remediation populated 441 previously blank `contact_document.expiry_date` values from the strict OCR population. The transaction used a fixed 441-row plan and SHA-256 hash, blank-value and document-link guards, a pre-write backup, and post-commit reconciliation of all 441 rows. The retained evidence is `agm-api/id_expiry_update_evidence/20260716T154815Z_manifest.json` and its referenced backup CSV. This remediation does not create an ongoing processing, approval, or exception-management control; database mismatches remain manual review items.

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
- [Accounts Audit Review](../22-accounts-audit-review.md)
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
- [Accounts Audit Review](../22-accounts-audit-review.md)
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
- [Accounts Audit Review](../22-accounts-audit-review.md)
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
- [Accounts Audit Review](../22-accounts-audit-review.md)
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
- [Accounts Audit Review](../22-accounts-audit-review.md)
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
- [Accounts Audit Review](../22-accounts-audit-review.md)
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)

## GAP-007 - Expiration Monitoring in Contact Focus

### Current State
- The current Accounts Audit does not classify missing, expired, expiring-soon, or valid POI expiration dates.
- Contact Focus shows document-category presence but not the contact's current POI validity posture.
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
- [Accounts Audit Review](../22-accounts-audit-review.md)

## GAP-008 - Review Case Lifecycle

### Current State
- Document review supports an assigned user and freeform comment for an account/contact pair.
- The application reads the first matching `document_review_responsible` row and updates that row in place. Changing the assignee or comment replaces the prior values rather than creating a historical record.
- The record does not identify the actor who changed the assignment or comment, and the generic API token identity cannot provide employee-level attribution.
- It does not define a case type, severity, lifecycle status, due date, evidence list, disposition, closure reason, or immutable event history.

### Required Control
- Create a persistent case model for compliance and operational findings.
- Support at minimum `open`, `assigned`, `waiting_for_client`, `under_review`, `resolved`, and `closed` states.
- Record finding type, source, severity, owner, due date, linked account/contact/document, evidence, disposition, closure reason, and timestamps.
- Record every assignment and comment change as an append-only event with actor, timestamp, previous value, new value, and reason where required.
- Prevent duplicate active cases for the same finding identity while retaining recurrence history.
- Require controlled transitions and authorization for resolution and closure.

### Acceptance Criteria
- Document gaps, OCR/translation failures, expiration findings, data mismatches, screening exceptions, and stale required data can create cases.
- Every case has one current owner or an explicit unassigned queue.
- Every transition records actor, timestamp, prior state, new state, and reason.
- Assignment and comment history remains queryable after later edits and distinguishes the current state from prior events.
- Closed cases remain searchable and retain their evidence.
- Reappearing conditions reopen the prior case or create a linked recurrence according to a documented rule.

### Affected Processes
- [Daily Screening Run](../01-daily-screening-run.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)

## GAP-009 - Scheduled Audits and Alerts

### Current State
- Accounts Audit calculates its findings only when a user opens the page.
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
- [Accounts Audit Review](../22-accounts-audit-review.md)
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

## GAP-012 - Correct AML Risk-Assessment Inputs

### Current State
- The Hub stores an applicant's residence country in `residenceAddress.country`, while the AML assessment reads other country paths.
- Joint-account assessment can fall back to the first holder when the linked contact cannot be resolved by entity id.
- Organization assessment can fall back to the first associated individual instead of assessing the organization with its own jurisdiction data.
- IBKR associated-person matching returns the first person when no name or email match is found.

### Required Control
- Read the country fields that are actually stored by the Hub application.
- Resolve joint holders and organization-associated people by stable entity id or external id, with email and normalized name used only as controlled fallbacks.
- Use the organization's own formation and operating jurisdiction when assessing the organization.
- Stop the assessment for manual correction when one contact cannot be resolved to one application or IBKR subject.
- Keep the currently approved AML weighting formula unchanged.

### Acceptance Criteria
- Automated tests cover individual, joint, and organization assessments using the application payload produced by Hub.
- Each assessment uses the intended contact, holder, or organization and the correct country.
- An unresolved or ambiguous subject does not receive a completed automatic score.
- Existing component weights and approved scoring rules are not replaced by this remediation.

### Affected Processes
- [Daily Screening Run](../01-daily-screening-run.md)
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)

## GAP-013 - Sanctions Candidate Matching and Review

### Current State
- OFAC, UK, and UN screening looks up exact normalized names.
- UK and UN data include some alias matching, while OFAC matching uses the primary stored name.
- A possible result has no stored review state, reviewer, review date, or disposition.

### Required Control
- Keep exact normalized-name matching and add conservative fuzzy matching that produces candidates for review.
- Include available aliases from each sanctions source in candidate generation.
- Do not treat a fuzzy candidate as a confirmed match automatically.
- Allow an authorized reviewer to mark a candidate as `pending`, `false_positive`, or `confirmed_match` and retain a short comment, reviewer, and review timestamp.

### Acceptance Criteria
- Exact names, supported aliases, and close candidate names are covered by automated tests.
- Fuzzy candidates remain pending until reviewed.
- The latest disposition, reviewer, timestamp, and comment are retained for each candidate.
- Screening results distinguish no candidate, pending candidate, false positive, and confirmed match.

### Affected Processes
- [Daily Screening Run](../01-daily-screening-run.md)
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)

## GAP-014 - Immutable Audit Trail and Retention

### Current State
- Contact-document metadata can be updated in place.
- Contact documents and orphaned raw documents can be physically deleted.
- Document-review assignments and comments are updated in place without retaining their prior values or the actor responsible for the change.
- Missing-document outreach now creates a review-linked `document_review_email` attempt before sending and records recipient, source selection, requested categories, language, immediate success or failure, send timestamp, and Gmail message id. It does not record the initiating actor, provider-side delivery/open state, or explicit resend relationships.
- The process manuals identify records and exports to retain, but no unified application-level audit-event or retention workflow is documented.
- ITGC controls for regulatory retention and transaction audit trails remain open in the role register.

### Required Control
- Create an append-only audit-event model for critical application and compliance actions.
- Extend the durable review-outreach ledger with initiating-actor attribution, explicit resend relationships, provider-side delivery reconciliation where available, and retention controls.
- Record actor, action, entity, entity id, timestamp, request/correlation id, reason, and appropriate before/after values.
- Replace routine physical deletion of compliance evidence with controlled archival or soft deletion.
- Define retention periods, legal-hold behavior, approved purge, and restoration procedures.
- Restrict destructive evidence actions and require a reason plus secondary approval where appropriate.

### Minimum Audit Coverage
- **Applications and accounts:** account/application creation; material changes to identity, address, tax residence, employment, financial profile, Source of Wealth, ownership, control, and regulatory answers; review decisions; status changes; and IBKR submission attempts and results.
- **Contacts and account relationships:** contact creation, material name/email/type changes, merges, account-contact link or unlink actions, and changes to entity or external identifiers used to resolve a holder for screening.
- **Documents:** raw upload, account/contact link, relink, metadata change, processing result, reviewer acceptance or rejection, replacement, archive, and deletion attempt or completion.
- **Review work:** responsible-user assignment or reassignment, comment creation or revision, review decision, exception or waiver, reopening, and closure.
- **Screening and AML assessment:** screening request, completion, skip, or failure; sanctions candidate creation and disposition; AML score creation or recalculation; and any manual score or risk-level override.
- **Transaction-monitoring findings:** finding creation, status change, comment revision, disposition, escalation, reopening, and closure.
- **Client outreach and public uploads:** missing-document email attempt, success, failure, and resend; upload-request issue, expiration, revocation, and fulfillment; and the resulting document upload.
- **Compliance source refreshes:** OFAC, UK, UN, and FATF refresh success, failure, or skip, including the source date or version used by screening.
- **External compliance operations:** IBKR application, document, and instruction submission attempts and results, including the external request or response reference when available.
- Routine page views, navigation, report filtering, document previews, and unsaved form keystrokes are outside the required v1 audit trail.

### Acceptance Criteria
- Document upload, link, relink, metadata edit, reviewer assignment/comment change, review decision, waiver, deletion/archive, case transition, application decision, and IBKR submission produce audit events.
- Every action listed under Minimum Audit Coverage records the affected record, action, actor or system process, timestamp, result, and appropriate prior/new values or external correlation reference.
- Every missing-document email attempt has a review/account/contact reference, idempotency or correlation key, initiating actor, recipient and source snapshot, requested categories, language/template version, status, timestamps, and Gmail message id when sent; failed and unresolved attempts remain visible for reconciliation.
- Audit events cannot be modified through normal application APIs.
- Archived evidence cannot satisfy active document requirements but remains retrievable by authorized users.
- Retention and purge jobs produce reconciliation reports and approval evidence.
- Process manuals and the ITGC transaction register identify the retained audit evidence.

### Affected Processes
- [Accounts Audit Review](../22-accounts-audit-review.md)
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
- [Accounts Audit Review](../22-accounts-audit-review.md)
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)

## GAP-016 - Current FATF Jurisdiction Reference Data

### Current State
- FATF-listed country codes and names are hardcoded in the AML assessment module.
- The clients ETL already refreshes and backs up other compliance reference data, including OFAC, UK, and UN sanctions files.

### Required Control
- Refresh FATF high-risk and monitored jurisdictions through the existing clients ETL pattern.
- Store the retrieval date and retain the prior snapshot used by earlier assessments.
- Use the latest successful snapshot in new AML assessments.
- Report a failed refresh without silently replacing the last successful snapshot.

### Acceptance Criteria
- New assessments use the latest successfully retrieved FATF snapshot rather than the hardcoded array.
- The snapshot records its source and retrieval date.
- A refresh failure is visible and leaves the last successful snapshot available.
- Automated tests cover listed, unlisted, and unavailable-refresh scenarios.

### Affected Processes
- [Daily Screening Run](../01-daily-screening-run.md)
- [Clients ETL Pipeline](../04-clients-etl-pipeline.md)
- [Compliance Reference Data Refresh](../15b-compliance-reference-data-refresh.md)
- [Contact Screening and AML Risk Assessment](../16-contact-screening-and-aml-risk-assessment.md)

## GAP-017 - Persistent Transaction-Monitoring Findings

### Current State
- Dashboard calculates deposit flags such as amounts over USD 10,000 and amounts above the client's IBKR financial ranges when the report is opened.
- The calculated rule, observed value, threshold, and review outcome are not stored.
- `flagged_deposit` stores only account id, transaction id, and comment, and a comment edit creates another row instead of updating one controlled finding.

### Required Control
- Persist the findings produced by the existing transaction-monitoring rules.
- Store account id, transaction id, rule code, observed amount, threshold, status, reviewer, review timestamp, disposition, and comment.
- Use the minimal statuses `new`, `reviewed_explained`, `escalated`, and `closed`.
- Prevent duplicate findings for the same account, transaction, and rule.
- Keep IBKR as the transaction system of record; AGM stores only the monitoring finding and its review outcome.

### Acceptance Criteria
- A detected finding remains available after the report is closed or its filters change.
- Re-running the report does not create a duplicate for the same account, transaction, and rule.
- A reviewer can record and later retrieve the finding's status, disposition, comment, and review timestamp.
- Existing over-USD-10,000 and financial-profile comparison rules remain explainable from the stored values.

### Affected Processes
- [Clients ETL Pipeline](../04-clients-etl-pipeline.md)
- [Deposits and Withdrawals Monitoring](../17-deposits-and-withdrawals-monitoring.md)

## GAP-018 - Compliance-Reference Refresh Integrity and Freshness Evidence

### Current State
- OFAC, UK, and UN downloads run as steps inside the broad Clients ETL pipeline.
- Individual extraction or transformation failures produce `partial` stage results, but the route returns HTTP `200` and the GitHub workflow sends its success email.
- Later stages continue after an extraction failure and can publish the most recent prior backup as the stable resource.
- OFAC requests do not call `raise_for_status`; none of the three source requests has an explicit timeout.
- Stable resources are replaced without a minimum row-count check, source publication-date check, change tolerance, source version, or retained checksum manifest.
- The screening availability check proves only that date-prefixed backups can be found for today and the previous calendar day.

### Required Control
- Record one explicit refresh result for each source containing source URL, retrieval time, source publication date or version when available, HTTP result, row count, checksum, backup file id, and stable resource id.
- Apply HTTP timeouts and status validation to every source.
- Validate schema and nonempty/minimum-reasonable populations before a new snapshot becomes eligible for screening.
- Replace the stable resource atomically only after validation succeeds; retain the last known good resource on failure.
- Make the workflow fail and notify as a compliance-source failure when any required list is failed, stale, or unverified.
- Keep the generic Clients ETL summary, but produce a separate compliance-source result that can be reconciled before screening.

### Acceptance Criteria
- A green workflow result proves that OFAC, UK, and UN each passed download, schema, population, storage, and publication checks for that run.
- A failed source cannot be reported as a successful compliance refresh and cannot silently republish an older file as if it were current.
- The exact snapshot used by a screening can be identified later by immutable id and checksum.
- Automated tests cover HTTP errors, timeouts, empty data, malformed schema, unreasonable count changes, storage failures, and last-known-good behavior.

### Affected Processes
- [Clients ETL Pipeline](../04-clients-etl-pipeline.md)
- [Compliance Reference Data Refresh](../15b-compliance-reference-data-refresh.md)
- [Daily Screening Run](../01-daily-screening-run.md)

## GAP-019 - Daily Screening Population Coverage and Cache Invalidation

### Current State
- The scheduled job exits when OFAC, UK, and UN are unchanged before checking for new or previously unscreened contacts.
- It reads associated-person roles but does not exclude trusted contacts even though earlier process documentation claimed that behavior.
- If every target is already screened today, it skips. If only some are screened, it inserts rows for the whole planned population and can duplicate existing same-day rows.
- Sanctions lists, sanctions indexes, and IBKR details are cached per API worker without an invalidation or snapshot key.
- Contacts linked to multiple accounts are deduplicated by retaining the first account context encountered.
- Snapshot comparison uses Costa Rica dates while screening coverage uses the server's `date.today()`.

### Required Control
- Evaluate contact coverage on every scheduled run independently of whether source lists changed.
- Define and enforce the approved population policy for holders, beneficial owners, control persons, and trusted contacts.
- Insert only missing required screenings and enforce an idempotency rule for contact, screening date, and source-snapshot version.
- Reload or version-key sanctions and IBKR caches before building a scheduled batch.
- Select account/holder context deterministically for multi-account contacts and retain that context on the screening record.
- Use one explicit business timezone and reconcile planned, created, skipped, and failed contacts before declaring success.

### Acceptance Criteria
- A successful daily job proves that every in-scope contact has required coverage even when no list changed.
- Partial prior coverage produces rows only for missing contacts and cannot create same-day duplicates.
- Every record identifies the source snapshots and account/holder context used.
- Cache state cannot cause a current backup comparison to be followed by screening against an older in-memory list.
- Population counts reconcile exactly or the workflow fails visibly.

### Affected Processes
- [Daily Screening Run](../01-daily-screening-run.md)
- [Contact Screening and AML Risk Assessment](../16-contact-screening-and-aml-risk-assessment.md)
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)

## GAP-020 - Regulatory-File Review Data Integrity and Audit Evidence

### Current State
- The regulatory-file page calls `/accounts/screening`, but no matching route exists in the current API source.
- Screening, financial-range, and document failures are converted to empty collections without a visible section-level failure.
- False Positive and True Match rows are displayed as hardcoded `False` values rather than stored reviewer decisions.
- Organization document coverage considers only the first associated person.
- PDF creation occurs only in the browser and is not recorded.

### Required Control
- Use the supported contact-screening data model or implement and test an authoritative account-level aggregation endpoint.
- Display unavailable, incomplete, and not-reviewed states distinctly from true zero-result states.
- Populate candidate dispositions only from retained review records.
- Apply documented holder/beneficial-owner/control-person coverage rules to organizations.
- Record regulatory-file review and export events with account, source snapshot ids, reviewer, timestamp, result, and generated-file checksum or retained artifact reference.

### Acceptance Criteria
- A failed screening request cannot appear as `0 screenings`.
- False-positive and true-match values reflect a retained disposition or show `Not reviewed`.
- The page and exported file identify the data-as-of time and source snapshots.
- Required organization persons are included under a tested policy.
- Review and export actions are auditable.

### Affected Processes
- [Regulatory File Review and Export](../18-regulatory-file-review-and-export.md)
- [Contact Screening and AML Risk Assessment](../16-contact-screening-and-aml-risk-assessment.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)

## GAP-021 - Retired Identifier

`GAP-021` formerly identified Fee-Template Review Disposition and Reconciliation. The Fee Template Review process was removed on 2026-07-15 as redundant, and the surviving account-population and filter-control requirements were consolidated into `GAP-024 - Accounts Audit Population Reconciliation and Review Evidence`. The identifier is retained for historical traceability and must not be reassigned.

## GAP-022 - Investment-Business Report Reproducibility and Approval

### Current State
- The page is hardcoded to the 2025 trade period and two master account ids.
- Missing NAV is treated as zero and missing country is treated as Costa Rica.
- The page has no formal export, source manifest, run id, preparer, reviewer, reconciliation, certification, or filing record.
- Debug counters are displayed as part of the production-facing report.

### Required Control
- Parameterize and validate the reporting period, as-of date, and included business/master-account population.
- Preserve missing and unknown values instead of assigning factual defaults.
- Generate a reproducible report run with source file ids/checksums, record counts, transformations, reconciliation totals, and exceptions.
- Add preparer/reviewer approval and retain the approved artifact and any external submission reference.
- Separate diagnostic counters from the approved report output.

### Acceptance Criteria
- Re-running an approved report with the retained source manifest produces the same totals.
- Unknown country and missing NAV remain visibly unknown rather than becoming Costa Rica or zero.
- Included and excluded master accounts are explicit and approved.
- Every issued report has a period, run id, preparer, reviewer, approval time, artifact checksum, and filing/submission status where applicable.

### Affected Processes
- [Investment Business Regulatory Reporting](../20-investment-business-regulatory-reporting.md)
- [Clients ETL Pipeline](../04-clients-etl-pipeline.md)
- [Daily IBKR Details Backup and Reporting Feed](../13-daily-ibkr-details-backup.md)

## GAP-023 - Compliance-Manual Change Notification and Acknowledgment

### Current State
- The notification endpoint is invoked manually and is not connected to a committed compliance-document change.
- It sends a generic message to one hardcoded recipient with no manual version, changed sections, reason, owner, due date, or approval request.
- No delivery reconciliation, acknowledgment, review decision, approval, or publication status is stored.

### Required Control
- Create a governed manual-change record containing document/version, change summary, reason, author, approver, effective date, and affected processes or controls.
- Trigger notifications from an approved change event or require the change record id in the manual endpoint.
- Notify the configured review group, track delivery and acknowledgment, and escalate overdue reviews.
- Prevent a changed manual from being marked effective until required approval is retained.

### Acceptance Criteria
- Every notification resolves to a specific immutable manual version and change record.
- Required reviewers, delivery state, acknowledgment, decision, comments, and approval time are retained.
- Rejected or overdue changes remain visibly unresolved.
- The published/effective manual version matches the approved artifact checksum.

### Affected Processes
- [Compliance Manual Update Notification](../21-compliance-manual-update-notification.md)
- [IT Documentation Standard](../../itgc/05-it-documentation-standard.md)

## GAP-024 - Accounts Audit Population Reconciliation and Review Evidence

### Current State
- Account Focus includes only internal accounts with an IBKR account number and matching IBKR details, while Contact Focus considers every internal account but emits rows only for links that resolve to contacts.
- Initial-load failure shows an error toast and then replaces both populations with empty arrays, so an empty screen is not durable evidence of a successful zero-exception review.
- Stage 3 means required category names are present; it does not prove validity, acceptance, expiration, readability, or correct evidence content.
- Contact Focus uses a static bundled IBKR compliance-task snapshot rather than a timestamped live task extract.
- The review stores current assignment/comment values and individual outreach attempts, but no audit-run id, source manifest, selected filters, reviewer, population reconciliation, disposition, or completion certification.
- Account Focus has no dedicated CSV export.
- When no eligible outreach recipient resolves, the current Contact Focus code can fall back to `aa@agmtechnology.com` with recipient source `testing`.
- The Accounts page `Unassigned` advisor option uses the literal value `unassigned`, while blank advisor codes remain blank/null, so it does not return genuinely unassigned accounts.
- On the Accounts page, selecting `Requires Email Change` returns from the filter predicate before `Needs Application` is evaluated, so those two visible filters do not combine.

### Required Control
- Create a durable Accounts Audit run with a source-data manifest, successful-load status, rule version, selected filters, Account Focus and Contact Focus population counts, exclusions, reviewer, start/end times, and completion decision.
- Reconcile the two focus populations explicitly using their documented inclusion rules and identify accounts or links excluded because source records are missing.
- Calculate completeness from accepted and current documents under the approved policy matrix rather than category presence alone.
- Replace the static IBKR task snapshot with a timestamped source and display its as-of time and load status.
- Provide exportable evidence for both focus views and embed the run id, filters, and source-as-of values.
- Block client outreach when no approved recipient resolves; a testing recipient must require an explicit test mode that cannot be confused with production outreach.
- Correct and test Accounts-page population filters against approved definitions, including unassigned advisors, missing applications, and email changes.

### Acceptance Criteria
- An unsuccessful or partial load cannot produce a review marked complete or appear as a successful zero-row population.
- Every completed run reconciles included, excluded, and exception populations for both focus views.
- Every Stage 3 document is accepted, current, correctly linked, and valid under the rule version retained with the run.
- IBKR task indicators identify the source timestamp and cannot silently use stale bundled data.
- Account and Contact Focus exports reproduce the reviewed population and filters from the retained run.
- A production missing-document email cannot be sent to a fallback testing address.
- Accounts-page filter tests prove that combined filters produce the documented intersection and that unassigned advisors are included correctly.

### Affected Processes
- [Accounts Audit Review](../22-accounts-audit-review.md)
- [Accounts Metadata Review and Analysis](../08-accounts-metadata-review.md)
- [Daily IBKR Details Backup and Reporting Feed](../13-daily-ibkr-details-backup.md)
- [Advisor Contact Linking](../15a-advisor-contact-linking.md)

## GAP-025 - Complete IBKR Account-Details Refresh

### Current State
- The daily IBKR details process reads the most recent retained account-details file and requests current details only for internal IBKR account numbers absent from that file.
- Existing account details are carried forward without proving that account status, associated persons, residence, financial information, risk data, and other mutable fields were refreshed.
- Per-account failures and hardcoded exclusions can leave accounts absent without making the resulting dataset unusable for downstream reviews.
- The retained dataset does not provide a run manifest proving the expected population, requested accounts, successes, failures, skips, source time, and checksum.

### Required Control
- Retrieve current details for every in-scope account across every configured IBKR master account on each scheduled refresh.
- Create an immutable run manifest containing run id, business date, master accounts, expected account count, requested count, success count, failure count, skipped count, account-level outcomes, retrieval timestamps, and snapshot checksum.
- Classify each run as `complete`, `partial`, or `failed` under documented reconciliation rules.
- Prevent a partial or failed run from replacing the last complete snapshot used by compliance decisions.
- Retain partial results for investigation without presenting them as a current complete population.
- Require every exclusion to have an approved reason, owner, effective date, and expiration or review date.

### Acceptance Criteria
- A successful run proves that every expected in-scope IBKR account was requested and returned successfully or was covered by a current approved exclusion.
- Every account-level failure is visible and reconciles to the run totals.
- The exact snapshot and manifest used by an audit, screening, reconciliation, or report can be identified later.
- Changes to associated persons and other mutable fields on existing accounts appear in the next complete snapshot.
- Automated tests cover complete runs, per-account failures, unavailable master accounts, exclusions, duplicate responses, and last-known-good behavior.

### Affected Processes
- [Daily IBKR Details Backup and Reporting Feed](../13-daily-ibkr-details-backup.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)
- [Contact Screening and AML Risk Assessment](../16-contact-screening-and-aml-risk-assessment.md)
- [Regulatory File Review and Export](../18-regulatory-file-review-and-export.md)
- [Investment Business Regulatory Reporting](../20-investment-business-regulatory-reporting.md)

## GAP-026 - AGM/IBKR Account-Population Reconciliation

### Current State
- Reviews generally begin with internal AGM accounts and therefore do not prove that every in-scope IBKR account has a corresponding AGM application and compliance record.
- No durable reconciliation identifies IBKR-only accounts, AGM-only live accounts, duplicate mappings, wrong-master-account assignments, status mismatches, or accounts omitted by the IBKR details feed.
- Accounts Audit excludes internal accounts from Account Focus when no matching IBKR detail exists, which can hide rather than resolve population exceptions.

### Required Control
- Reconcile every complete IBKR account snapshot against the complete AGM account population.
- Classify every record into mutually exclusive results: one-to-one match, IBKR-only, AGM-only, duplicate internal mapping, conflicting IBKR identity, unexpected master account, status mismatch, excluded, or unresolved.
- Treat an IBKR-only account as a high-priority compliance exception until AGM obtains the required application, KYC, screenings, ownership information, and approvals or documents that the account is outside AGM's responsibility.
- Create or update a deduplicated case for every unresolved exception with owner, severity, due date, and source snapshot.
- Retain reconciliation counts, matching rules, exclusions, run id, reviewer, and completion decision.

### Acceptance Criteria
- Every completed reconciliation proves that the expected IBKR population equals matched accounts plus explicitly classified exceptions.
- No account can disappear from the population because its details failed to load.
- Duplicate and conflicting mappings cannot be classified as successful matches.
- Dashboard totals reconcile to the retained run rather than recalculating from an unverified browser population.
- A new IBKR-only account creates a case automatically.

### Affected Processes
- [Daily IBKR Details Backup and Reporting Feed](../13-daily-ibkr-details-backup.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)

## GAP-027 - Associated-Person and Contact-Link Lifecycle Reconciliation

### Current State
- `account_contact` records do not define an authoritative holder role, source, active/inactive state, effective dates, last-confirmed IBKR snapshot, or controlled removal reason.
- New identifiers can be populated through name or email matching without a retained identity-resolution decision.
- No scheduled workflow compares current AGM contact links with every associated person in a complete IBKR snapshot.
- A physical link deletion would remove current relationship state without preserving when, why, and by whom the relationship ended.

### Required Control
- Store relationship role, source, `valid_from`, `valid_to`, first-seen snapshot, last-seen snapshot, current state, decision reason, reviewer, and approval timestamp.
- Support at minimum `pending_add`, `active`, `change_pending_review`, `pending_remove`, `inactive`, `ambiguous`, and `rejected` states.
- Match associated persons using stable IBKR entity id, external id, or a previously approved mapping. Use exact verified email only as a controlled candidate and names only for manual review.
- When a new IBKR person appears, create a case and require contact resolution, role classification, required KYC, screening, and approval before activation.
- When an AGM-linked person no longer appears, mark the relationship `pending_remove`; do not delete or deactivate it from one absence.
- Require a complete source snapshot plus consecutive confirmation or live IBKR confirmation, investigation of identifier and role changes, and an approved disposition before setting `valid_to` and `inactive`.
- Preserve the contact, documents, screenings, cases, and historical relationship after deactivation.

### Acceptance Criteria
- Every active relationship identifies its current role and the authoritative source that last confirmed it.
- A name-only candidate cannot update an IBKR entity or external id automatically.
- One incomplete or failed IBKR snapshot cannot unlink or deactivate a person.
- Added, changed, missing, ambiguous, and removed persons create distinct reconciliation findings.
- Historical reports can reproduce which persons and roles were active on a prior date.
- Downstream screening, outreach, and reporting use the approved relationship state and role.

### Affected Processes
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [Daily IBKR Details Backup and Reporting Feed](../13-daily-ibkr-details-backup.md)
- [Contact Deduplication and Dependency Merge](../14-contact-deduplication-workflow.md)
- [Contact Screening and AML Risk Assessment](../16-contact-screening-and-aml-risk-assessment.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)

## GAP-028 - Authoritative KYC/AML Readiness and Activation Gate

### Current State
- Dashboard performs selected document and payload checks before IBKR submission, but there is no single server-owned decision proving that all KYC, AML, screening, ownership, case, and evidence requirements were satisfied.
- Document presence and any historical screening can appear sufficient even when documents are unaccepted or expired and screening is stale or unresolved.
- The manual organization-account path can populate an IBKR account number and submission date without the same compliance decision used for other applications.
- An IBKR account number is treated as the practical boundary between an application and a live account without a separate compliance activation decision.

### Required Control
- Create one authoritative compliance preflight used by readiness displays, all IBKR submission routes, manual organization onboarding, and account activation.
- Before submission, require approved application state; unambiguous required-party resolution; completed identity verification; accepted current documents; applicable Source-of-Wealth and Source-of-Funds evidence; current screening; resolved sanctions and other screening candidates; current AML assessment; required high-risk approval; complete beneficial ownership; no blocking cases; current CDD; current compliance sources; and retained agreements.
- Return structured blockers and warnings tied to the affected party, document, assessment, source, case, or obligation.
- Store the complete preflight result, rule version, source snapshots, acknowledgments, and decision with every submission attempt.
- Before activation, additionally require confirmation in a complete IBKR snapshot, account and master-account reconciliation, associated-person reconciliation, and disposition of required IBKR compliance tasks.
- Separate `approved_for_ibkr`, `submitted_to_ibkr`, `activation_pending`, and `active` states.

### Acceptance Criteria
- Direct API calls, Dashboard, Hub, and manual organization onboarding cannot use different compliance rules.
- An account with a missing or expired document, stale screening, unresolved candidate, incomplete ownership structure, overdue CDD review, or open blocking case cannot be submitted or activated.
- An IBKR account number alone cannot make an internal record active.
- High-risk or exception-based approvals identify the required approver and supporting evidence.
- Submission and activation decisions remain reproducible from the retained preflight result and source versions.

### Affected Processes
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)
- [Contact Screening and AML Risk Assessment](../16-contact-screening-and-aml-risk-assessment.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)

## GAP-029 - Resumable Onboarding and Incomplete-Application Remediation

### Current State
- Onboarding performs account creation, contact resolution, link creation, screening, document upload, agreement storage, IBKR submission, and result persistence as separate operations.
- A later failure can leave earlier records committed without a durable indication of the last completed control step.
- Retrying a partial application does not follow one documented idempotency and reconciliation rule.
- Incomplete drafts can be difficult to distinguish from applications awaiting compliance review.

### Required Control
- Create a durable onboarding workflow with checkpoints such as `draft_created`, `contacts_resolved`, `relationships_created`, `documents_received`, `screenings_completed`, `compliance_review_pending`, `approved_for_ibkr`, `submission_in_progress`, `submitted`, `submission_failed`, `activation_pending`, and `active`.
- Record the last successful step, current blocker, attempts, last error, correlation key, and records created or updated by each step.
- Make each step idempotent so a retry resumes safely without duplicating contacts, relationships, screenings, documents, or external submissions.
- Reconcile any uncertain external submission result before permitting another submission attempt.
- Create a scheduled remediation population for incomplete or stalled workflows and define service levels and escalation.
- Keep draft creation permissible while preventing any draft or partial workflow from satisfying compliance approval or activation.

### Acceptance Criteria
- Every partially completed application has an explicit state, blocker, owner, and recoverable next action.
- A retry after failure does not create duplicate controlled records or duplicate IBKR submissions.
- Stalled applications are detected without a user opening the application page.
- Operators can distinguish client-abandoned drafts, operational failures, compliance holds, rejected applications, and submission failures.
- The onboarding history identifies the result of each step and its source or external correlation reference.

### Affected Processes
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)

## GAP-030 - Ongoing and Event-Driven CDD/KYC Refresh

### Current State
- Document-expiration findings are planned, but the lifecycle does not define a complete risk-based ongoing customer due-diligence review.
- Contacts and accounts do not have a governed last-review date, next-review date, review reason, completed scope, or overdue status.
- Material changes in customer information, ownership, activity, IBKR relationships, and risk do not automatically trigger reassessment.

### Required Control
- Assign each active customer and account a current risk category, last completed CDD review, next review due date, policy-based frequency, and current review status.
- Define periodic review frequency by risk and applicable obligation without assuming that one interval fits every customer.
- Trigger event-driven review for material identity, address, nationality, tax residence, employment, business activity, financial profile, wealth/funds, beneficial ownership, control-person, screening, IBKR relationship, transaction-pattern, document, jurisdiction-risk, dormant-reactivation, and regulatory changes.
- Reassess customer risk and downstream review frequency when a trigger is material.
- Create deduplicated cases for due, overdue, and event-triggered reviews and escalate overdue high-risk reviews.
- Retain the data reviewed, source versions, changes since the prior review, reviewer, decision, and next-review calculation.

### Acceptance Criteria
- No active in-scope customer has an overdue CDD review without an open approved exception.
- High-risk and lower-risk review frequencies follow the approved policy and are visible in the system.
- A material IBKR associated-person or beneficial-ownership change triggers review automatically.
- Completed reviews identify the exact customer information and compliance sources assessed.
- Review completion recalculates risk and the next due date under a retained policy version.

### Affected Processes
- [Daily Screening Run](../01-daily-screening-run.md)
- [Contact Screening and AML Risk Assessment](../16-contact-screening-and-aml-risk-assessment.md)
- [Deposits and Withdrawals Monitoring](../17-deposits-and-withdrawals-monitoring.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)

## GAP-031 - Beneficial Ownership and Organization-Control Structure

### Current State
- Organization workflows do not maintain one complete, versioned ownership and control structure.
- Some reviews use the first organization-associated person rather than applying approved coverage rules to all required beneficial owners, control persons, directors, authorized persons, and other applicable roles.
- Indirect ownership, intermediate entities, ownership percentages, nominee relationships, and changes over time are not reconciled as a governed structure.

### Required Control
- Store the legal entity, formation and operating jurisdictions, direct owners, intermediate entities, indirect owners, calculated ultimate ownership, control persons, directors, officers, partners, authorized traders, nominees, and trust or similar roles where applicable.
- Define the beneficial-ownership threshold and control tests in a versioned policy by applicable obligation and entity type.
- Require ownership percentages to reconcile or create a documented exception.
- Resolve every required natural person to a contact with their own KYC verification, screening, risk assessment, documents, and effective-dated relationship.
- Detect missing intermediate entities, circular ownership, unexplained nominees, unverified controllers, conflicting identifiers, and differences between application, AGM, and IBKR data.
- Preserve each approved ownership version and the evidence supporting it.

### Acceptance Criteria
- AGM can identify who ultimately owns and controls every in-scope organization and how that conclusion was reached.
- Every required owner or controller has current KYC, screening, and relationship evidence.
- Ownership totals and indirect calculations are reproducible under the retained policy version.
- A material ownership or control change creates an event-driven CDD review and blocks readiness where required.
- Organization review cannot be completed from only the first associated person.

### Affected Processes
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)
- [Contact Screening and AML Risk Assessment](../16-contact-screening-and-aml-risk-assessment.md)
- [Regulatory File Review and Export](../18-regulatory-file-review-and-export.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)

## GAP-032 - PEP, Adverse-Media, and Regulatory-Action Screening

### Current State
- The documented screening program focuses on OFAC, UK, UN, and FATF-related results.
- PEP or adverse indicators can influence the current risk formula when already present in source data, but no separate controlled workflow creates, investigates, and dispositions PEP, relative/close-associate, adverse-media, or regulatory-action candidates.
- There is no retained evidence that every required party received current coverage from the approved sources.

### Required Control
- Define approved sources, coverage populations, refresh frequencies, matching rules, and review responsibilities for PEPs, relatives and close associates where required, adverse media, and regulatory or disciplinary actions.
- Generate candidates using available names, aliases, birth dates, nationalities, residences, positions, organizations, locations, identification details, related parties, and timelines.
- Do not automatically treat a candidate as a confirmed result.
- Support `pending`, `false_positive`, `confirmed`, `insufficient_information`, and `escalated` dispositions with reviewer, timestamp, evidence, and rationale.
- Require enhanced due diligence and appropriate approval for confirmed or material unresolved results under the approved policy.
- Include current source versions and dispositions in the compliance-readiness gate and ongoing CDD review.

### Acceptance Criteria
- Every required holder, beneficial owner, control person, and other in-scope party has current coverage under each applicable screening type.
- Candidates remain pending until an authorized reviewer records a disposition.
- Confirmed and unresolved material results affect risk, case severity, approval, and review frequency.
- False positives remain linked to the source candidate and can be reconsidered when identifiers or source data change.
- Screening runs reconcile planned, completed, failed, skipped, and candidate counts by source.

### Affected Processes
- [Daily Screening Run](../01-daily-screening-run.md)
- [Contact Screening and AML Risk Assessment](../16-contact-screening-and-aml-risk-assessment.md)
- [Regulatory File Review and Export](../18-regulatory-file-review-and-export.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)

## GAP-033 - AML Risk-Model Governance and Overrides

### Current State
- The AML assessment uses a defined weighted formula, but the operating workflow does not retain a governed model version, complete source inputs, component-level rationale, approval history, or model-change record.
- Hardcoded jurisdiction and other risk classifications can change without preserving the effective model used by historical assessments.
- Manual risk or score overrides do not have a documented request, approval, evidence, expiration, and review lifecycle.

### Required Control
- Retain model version, rule version, input values, input sources, component scores, weights, final score, assigned risk category, missing inputs, and calculation timestamp for every assessment.
- Define model owner, approver, validation frequency, data-quality requirements, risk thresholds, consequences by risk level, and change-approval process.
- Validate and calibrate the model using actual customer populations, investigations, false positives, missed cases, and material business or regulatory changes.
- Test all model changes before release and preserve the before/after impact on representative populations.
- For every override, retain the calculated result, requested result, reason, evidence, requester, approver, effective date, expiration, and required secondary approval for material risk reductions.
- Never overwrite the original calculated assessment with an override.

### Acceptance Criteria
- Any historical risk assessment can be reproduced from retained inputs and the exact model version.
- Model changes identify their approver, testing evidence, effective date, and population impact.
- Missing or ambiguous required inputs prevent automatic completion rather than receiving silent defaults.
- Overrides are visible separately from calculated results and expire or receive periodic review.
- Risk level drives documented CDD frequency, approval, enhanced-due-diligence, and monitoring consequences.

### Affected Processes
- [Daily Screening Run](../01-daily-screening-run.md)
- [Contact Screening and AML Risk Assessment](../16-contact-screening-and-aml-risk-assessment.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)

## GAP-034 - Suspicious-Activity Investigation and SAR/STR Lifecycle

### Current State
- Transaction flags and screening findings do not feed a formal restricted suspicious-activity investigation and regulatory-reporting decision.
- The process library does not define investigation ownership, decision authority, filing deadlines, filing acknowledgment, continuing-activity review, confidentiality, or a documented no-file rationale.
- Exact filing obligations and information-sharing responsibilities between AGM and IBKR are not mapped.

### Required Control
- Create a restricted investigation model linking originating alerts, customers, related accounts and parties, transaction populations, narrative, evidence, requests for information, investigator, decision maker, and deadlines.
- Support controlled decisions such as `file`, `do_not_file`, `continue_monitoring`, `restrict_activity`, and other outcomes approved in the applicable policy.
- Retain the filing or submission reference, acknowledgment, filing date, decision rationale, continuing-activity review date, and links to related investigations.
- Apply confidentiality and anti-tipping-off handling defined by the applicable jurisdiction without exposing filing existence in ordinary account notes or client communications.
- Escalate overdue investigations and filing deadlines automatically.
- Use GAP-040 to define the applicable SAR/STR authority, threshold, deadline, form, retention, and allocation of responsibility with IBKR.

### Acceptance Criteria
- Every escalated suspicious-activity investigation reaches a documented decision within the applicable deadline.
- A filed report reconciles to a retained external acknowledgment or submission reference.
- A decision not to file retains sufficient analysis, approver, and supporting evidence.
- Related alerts and accounts can be investigated together without losing their individual source evidence.
- Confidential filing information is separated from ordinary operational case information.

### Affected Processes
- [Contact Screening and AML Risk Assessment](../16-contact-screening-and-aml-risk-assessment.md)
- [Deposits and Withdrawals Monitoring](../17-deposits-and-withdrawals-monitoring.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)

## GAP-035 - Expanded Transaction-Monitoring Coverage

### Current State
- Transaction monitoring is initiated when a user opens a report and evaluates a limited set of browser-derived rules.
- The over-USD-10,000 rule uses signed amount and therefore does not detect withdrawals under the same threshold logic.
- The workflow does not define scheduled coverage for aggregated activity, structuring, rapid movement, third-party funding, dormant reactivation, related-account patterns, geographic risk, or other scenarios selected from AGM's risk assessment.
- Alert populations, rule versions, lookback windows, and source reconciliation are not retained.

### Required Control
- Approve a scenario inventory based on AGM's products, customer base, jurisdictions, delivery channels, expected activity, and division of responsibility with IBKR.
- Consider large deposits and withdrawals, aggregation over defined periods, structuring, rapid in/out movement, profile inconsistencies, third-party funding, high-risk geographies, dormant reactivation, sudden volume changes, related-account activity, rejected or reversed instructions, unexplained internal/external transfers, and relevant securities activity.
- Run approved scenarios on a documented schedule against a reconciled source population.
- Persist account, transaction or event identifiers, rule and version, observed values, threshold or comparison, lookback period, customer risk, source manifest, status, owner, disposition, and escalation.
- Deduplicate alerts by a stable finding identity while retaining recurrence and related-pattern history.
- Reconcile source transactions, evaluated transactions, skipped transactions, alerts, failures, and exclusions before declaring a run successful.

### Acceptance Criteria
- Every expected transaction source and period is processed or appears as a visible failure or approved exclusion.
- Re-running the same source and rule version cannot create duplicate active alerts.
- Withdrawals and deposits are evaluated according to the approved amount and aggregation policy.
- Alert evidence explains exactly why the rule fired and which activity was evaluated.
- Escalated alerts can enter GAP-034 without losing their source rule and transaction evidence.

### Affected Processes
- [Clients ETL Pipeline](../04-clients-etl-pipeline.md)
- [Daily IBKR Details Backup and Reporting Feed](../13-daily-ibkr-details-backup.md)
- [Deposits and Withdrawals Monitoring](../17-deposits-and-withdrawals-monitoring.md)

## GAP-036 - Source-of-Funds and Funding-Party Verification

### Current State
- The application captures Source-of-Wealth declarations and Accounts Audit expects supporting Source-of-Wealth evidence.
- The workflow does not separately establish where funds for a material or unusual deposit originated or whether the originating account belongs to the customer.
- Third-party funding, payer-to-customer relationships, funding purpose, and transaction-specific evidence are not governed as a persistent review.

### Required Control
- Define Source-of-Wealth as the origin of overall accumulated wealth and Source-of-Funds as the origin and ownership of funds used for a particular account or transaction.
- Establish risk-based triggers using amount, customer risk, funding method, origin country, payer identity, third-party status, expected account purpose, and consistency with declared financial capacity.
- Retain originating institution, account holder, relationship to the customer, funding purpose, Source-of-Funds category, supporting evidence, verification result, reviewer, and decision.
- Prohibit third-party funding or define the limited permitted scenarios, evidence, approvals, and monitoring required.
- Compare funding behavior with expected activity, Source-of-Wealth, financial profile, and related transaction-monitoring alerts.
- Route material inconsistencies or unexplained funding to a case and, where appropriate, GAP-034.

### Acceptance Criteria
- AGM can identify the origin and ownership of every funding event that meets the approved review criteria.
- A Source-of-Wealth document alone cannot resolve a transaction-specific Source-of-Funds requirement.
- An unapproved or unexplained third-party funding event remains unresolved and cannot be marked reviewed solely through a freeform comment.
- Verification evidence and decisions remain linked to the source transaction and customer profile.
- Repeated or related funding events are evaluated collectively where required by the monitoring policy.

### Affected Processes
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)
- [Account Banking Instructions Handling](../11-account-banking-instructions-handling.md)
- [Deposits and Withdrawals Monitoring](../17-deposits-and-withdrawals-monitoring.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)

## GAP-037 - Account Closure, Holder Removal, Dormancy, and Offboarding

### Current State
- The process library focuses on account opening and active servicing and does not define a complete compliance end-of-relationship lifecycle.
- AGM and IBKR account status can diverge without a case or closure reconciliation.
- Former holders can remain linked without an effective end date and may continue appearing in outreach, screening, document, and review populations.
- Reactivation does not require a documented new compliance-readiness decision.

### Required Control
- Define `closure_requested`, `closure_under_review`, `closure_blocked`, `closing`, `closed`, `dormant`, `reactivation_under_review`, `rejected`, and `withdrawn` states with controlled transitions.
- Before closure, evaluate open investigations, suspicious-activity decisions, filings, pending transactions, required IBKR tasks, remaining assets, legal holds, retention requirements, and active authorized persons.
- Reconcile and retain IBKR closure confirmation, effective date, reason, reviewer, and any remaining exceptions.
- Effective-date associated-person and contact relationships instead of deleting them.
- Remove closed accounts and inactive persons from active outreach and servicing populations while retaining required compliance evidence.
- Define legally required post-closure monitoring, reporting, retention, and purge approval.
- Require a current compliance-readiness review before dormant or closed relationships are reactivated where policy permits reactivation.

### Acceptance Criteria
- AGM and IBKR closure status reconcile or create an owned case.
- No closed account remains in an active outreach or servicing population solely because its internal row still exists.
- Former relationships remain historically reproducible and cannot satisfy current-holder requirements.
- An unresolved investigation or required filing cannot be silently lost through closure.
- Reactivation records a new review, decision, source snapshots, and approval.

### Affected Processes
- [Daily IBKR Details Backup and Reporting Feed](../13-daily-ibkr-details-backup.md)
- [Contact Screening and AML Risk Assessment](../16-contact-screening-and-aml-risk-assessment.md)
- [Deposits and Withdrawals Monitoring](../17-deposits-and-withdrawals-monitoring.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)

## GAP-038 - Compliance Exceptions, Waivers, and Secondary Approval

### Current State
- Individual gaps mention selected waivers or approvals, but there is no central exception lifecycle covering all compliance blockers.
- Operators can otherwise rely on comments or manual actions without recording the exact requirement overridden, risk introduced, compensating control, approval authority, and expiration.
- The submission preflight does not distinguish valid approved exceptions from unresolved blockers.

### Required Control
- Create a controlled exception record identifying the requirement, affected entity, requested exception, reason, evidence, risk, compensating controls, requester, approving role, decision, comments, effective date, expiration, review date, and closure or revocation.
- Define which requirements may be waived, which may never be waived, and which require secondary or senior approval.
- Apply maximum duration, renewal, escalation, and periodic-review rules by exception type and severity.
- Prevent the requester from acting as sole approver where the policy requires separation.
- Make the compliance-readiness gate consume only current approved exceptions and display the exact blocker they affect.
- Do not treat a confirmed sanctions match or legally prohibited activity as an ordinary document or timing waiver.

### Acceptance Criteria
- Every account that proceeds despite a standard blocker has a valid, approved, unexpired exception tied to that blocker.
- An expired, revoked, rejected, or unrelated exception cannot satisfy preflight.
- Compensating controls and follow-up dates create scheduled work and escalation.
- Exception renewal preserves the prior decision and requires a new assessment.
- Reports identify active, expiring, expired, rejected, and overdue exceptions by owner and risk.

### Affected Processes
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)
- [Contact Screening and AML Risk Assessment](../16-contact-screening-and-aml-risk-assessment.md)
- [Deposits and Withdrawals Monitoring](../17-deposits-and-withdrawals-monitoring.md)
- [Regulatory File Review and Export](../18-regulatory-file-review-and-export.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)

## GAP-039 - Compliance Control Testing, QA, and Certification

### Current State
- Gap acceptance criteria describe intended tests, but no documented control-testing program proves that the implemented controls cannot be bypassed and continue to operate over complete populations.
- Operational reviews do not define independent quality-assurance sampling across approved, rejected, high-risk, manually handled, and exception-based accounts.
- Gap closure does not identify a standard design-effectiveness and operating-effectiveness evidence package.

### Required Control
- Define automated negative tests that attempt submission or activation with missing, expired, rejected, incorrectly linked, or unprocessed documents; stale or failed screening; unresolved candidates; incomplete ownership; overdue CDD; and open blocking cases.
- Define operational reconciliation tests for scheduled populations, source manifests, IBKR accounts, associated persons, screenings, alerts, cases, exports, failures, and exclusions.
- Define independent QA sampling using complete populations and documented risk-based or representative sample methods.
- Record test period, scope, control and rule versions, tester, tester independence, population, sample, evidence, exceptions, severity, owner, remediation date, retest, and certification.
- Require failed tests to reopen or prevent closure of the related gap and control certification.
- Report control-failure trends and incorporate them into training, policy, model, and process changes.

### Acceptance Criteria
- Every material KYC, AML, screening, monitoring, reconciliation, approval, and evidence control has documented design and operating-effectiveness testing.
- Negative tests prove the authoritative readiness gate fails closed for each required blocker.
- QA samples can be reproduced from the retained population and selection method.
- Findings remain open until remediation and successful retest are retained.
- A business-owner certification cannot be completed when a material control test failed or the source population was incomplete.

### Affected Processes
- [Daily Screening Run](../01-daily-screening-run.md)
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)
- [Contact Screening and AML Risk Assessment](../16-contact-screening-and-aml-risk-assessment.md)
- [Deposits and Withdrawals Monitoring](../17-deposits-and-withdrawals-monitoring.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)

## GAP-040 - Regulatory Obligations and IBKR Reliance Matrix

### Current State
- The repository does not establish AGM's exact AML/KYC legal and contractual perimeter by legal entity, license, jurisdiction, product, account type, and IBKR relationship.
- The process library does not identify which obligations AGM performs, which IBKR performs, which are shared, and what evidence AGM must retain when relying on IBKR.
- Exact suspicious-activity reporting, CDD, beneficial-ownership, screening, retention, and information-sharing requirements cannot be derived from the application code alone.

### Required Control
- Create an approved matrix organized by AGM legal entity, license or registration, customer jurisdiction, product, account type, master/introducing/clearing relationship, regulator, law or rule, obligation, trigger, deadline, evidence, retention, escalation, and last review.
- Assign each obligation to AGM, IBKR, or shared responsibility and identify the contractual or legal basis for that allocation.
- Cover customer identification, beneficial ownership, sanctions, PEP and enhanced due diligence, ongoing CDD, transaction monitoring, suspicious-activity reporting, recordkeeping, regulatory reporting, account restrictions and closure, information sharing, examinations, and regulatory requests.
- Define how AGM verifies that a control performed by IBKR operated and which evidence AGM receives and retains.
- Link system rules, document requirements, screening populations, deadlines, reports, and process manuals to the obligation and policy version that requires them.
- Review the matrix after regulatory, licensing, product, jurisdiction, contractual, or operating-model changes.

### Acceptance Criteria
- Every material AML/KYC control maps to an approved obligation, policy, or documented risk-based decision.
- AGM can identify its responsibility and IBKR's responsibility for every in-scope account and compliance process.
- Reliance on IBKR identifies the evidence, service level, exception path, and escalation required when the evidence is missing or the control fails.
- Jurisdiction-specific filing and retention deadlines used by the system match the approved matrix.
- Changes to the matrix trigger review of affected process documents, rules, tests, and controls.

### Affected Processes
- [AGM Process Library](../README.md)
- [Account Opening and AGM Account Creation](../09-account-opening-and-agm-account-creation.md)
- [IBKR Account Submission and Onboarding](../10-ibkr-account-submission-and-onboarding.md)
- [Contact Screening and AML Risk Assessment](../16-contact-screening-and-aml-risk-assessment.md)
- [Deposits and Withdrawals Monitoring](../17-deposits-and-withdrawals-monitoring.md)
- [Regulatory File Review and Export](../18-regulatory-file-review-and-export.md)
- [Investment Business Regulatory Reporting](../20-investment-business-regulatory-reporting.md)
- [Accounts Audit Review](../22-accounts-audit-review.md)

## Implementation Sequence

1. Establish AGM's approved regulatory perimeter and allocation of responsibilities with IBKR through GAP-040 so later controls implement the correct obligations, populations, deadlines, evidence, and retention rules.
2. Define the document policy matrix, Source-of-Wealth and Source-of-Funds policies, retention and audit requirements, and exception authority for GAP-001, GAP-002, GAP-014, GAP-036, and GAP-038.
3. Replace the carried-forward IBKR details feed with complete manifested snapshots and reconcile the account population through GAP-025 and GAP-026.
4. Add effective-dated associated-person relationships and complete organization ownership and control structures through GAP-027 and GAP-031.
5. Add document validity, processing, translation, and extracted-data reconciliation through GAP-003 to GAP-006, and replace contact-id public uploads through GAP-015 before expanding outreach.
6. Add the review-case, application-decision, onboarding-checkpoint, and immutable event foundations through GAP-008, GAP-010, GAP-014, and GAP-029.
7. Correct AML inputs and compliance-source assurance, then implement sanctions candidate review, FATF refresh, daily coverage, PEP/adverse-media coverage, and governed model results through GAP-012, GAP-013, GAP-016, GAP-018, GAP-019, GAP-032, and GAP-033.
8. Implement expiration findings, scheduled audits, and ongoing and event-driven CDD through GAP-007, GAP-009, and GAP-030.
9. Make the server-side preflight and activation decision authoritative through GAP-011 and GAP-028, consuming current approved exceptions under GAP-038.
10. Persist and expand transaction-monitoring scenarios and Source-of-Funds review through GAP-017, GAP-035, and GAP-036, then connect escalated cases to the suspicious-activity and SAR/STR lifecycle in GAP-034.
11. Add account closure, holder removal, dormancy, reactivation, and offboarding through GAP-037 using the same IBKR reconciliation, case, audit, and retention foundations.
12. Repair and govern the regulatory file, investment-business report, and manual-change lifecycle through GAP-020, GAP-022, and GAP-023.
13. Add reproducible Accounts Audit population reconciliation, safe outreach, live task freshness, and retained review certification through GAP-024.
14. Execute automated negative tests, population reconciliations, independent QA sampling, remediation, retesting, and business-owner certification through GAP-039 before closing any material lifecycle gap.

Implementation changes that add or alter database records must include SQL migration scripts under the workspace-level `sql/` directory.

## Governance and Closure

- Each gap remains open until its acceptance criteria are implemented, tested, and reflected in the affected process manuals.
- Any implementation that changes a critical transaction flow must update `compliance/itgc/transaction-processing-register.csv` in the same change.
- A gap may be closed only with retained test evidence, an updated process/control description, and business-owner confirmation.
- Partial implementation must be recorded explicitly; the gap must not be marked closed while a required control remains manual or unenforced.

## Last Reviewed
- Status: proposed
- Date: 2026-07-15
- Reviewer: Codex current-state and gap assessment
