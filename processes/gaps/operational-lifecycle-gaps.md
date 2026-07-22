Below is the more detailed implementation scope for Phase 1 and Phase 2. Phase 1 is the minimum credible compliance-control layer. Phase 2 makes those controls consistent, reliable and reproducible.

Implementation note: AGM does not need one shared compliance case model to close these gaps. The preferred direction is domain-specific workflow tables with clear escalation paths. For example, transaction monitoring can persist its own alerts, screenings can keep their own operational tables, and suspicious-activity investigations can exist as a separate restricted workflow fed by escalated alerts or other red flags.

The BVI requirements referenced below come primarily from the supplied [BVIFSC Revised Sanctions Guidelines](</Users/aguilarcarboni/Downloads/BVIFSC 31.12.2024_-_revised_sanctions_guidelines-_final.pdf>), particularly pages 27–40 and 48–55.

# Phase 1 — Critical regulatory controls

## 1. Suspicious-activity investigation and SAR/STR filing

### Required outcome

Every material transaction alert, sanctions concern, PEP concern or other red flag must be capable of entering a confidential investigation. The investigation must reach a documented decision and, where required, produce a timely regulatory report.

### What to implement

Create a restricted investigation record containing the fields AGM will actually use in the first investigation workflow. The minimum viable implementation does not need a universal case engine. It does need a dedicated restricted investigation workflow that can be created from escalated alerts and later extended if more timeline or filing fields are required.

The initial investigation record should contain:

- Investigation ID and status.
- Customer or account context.
- Originating alert or other red flag reference.
- Risk level.
- Decision: `file`, `do_not_file`, `continue_monitoring`, `restrict_activity`, `freeze`, or `close/reject relationship`.
- Decision rationale and decision-maker.
- Final closure date and reason.

Additional fields such as AMLCO escalation time, suspicion-formed time, filing deadline, filing references, acknowledgments, or continuing review dates may be added later when AGM is ready to operationalize those exact steps. They should not be invented up front if the workflow does not yet use them.

Access must be limited to the AMLCO and specifically authorized personnel. The existence of a SAR or filing decision must not appear in ordinary account comments, client communications or broadly accessible Dashboard screens.

### Deadline control

When AGM adds filing-deadline tracking, the system should separately record:

1. When the original alert arose.
2. When the information was escalated to the AMLCO.
3. When the AMLCO formed a suspicion.
4. The applicable filing deadline.
5. When the report was actually submitted.

The manuals generally specify filing within five working days. The BVI guidelines use “as soon as reasonably practicable” and identify no more than five working days for certain CTA reporting. AGM should have BVI counsel confirm the deadline and authority for each report type rather than apply one universal deadline to everything.

The system should warn before deadlines and escalate overdue cases to the AMLCO, CEO or other approved authority. This is still a required target state, but it does not need to be part of the first restricted investigation release if AGM is initially implementing only decision and closure tracking.

### Process changes

Add a dedicated suspicious-activity process covering:

- Employee internal reporting.
- Transaction-alert escalation.
- Sanctions-match escalation.
- Investigation steps.
- File/do-not-file authority.
- FIA and Sanctions Unit reporting.
- Confidentiality and anti-tipping-off.
- Account restriction, freeze and closure.
- Retention.
- Continuing-activity review.

Process 17 should end with either a documented non-escalation on the alert record or an escalation into this restricted investigation workflow.

### Completion criteria

This control is complete when:

- Every escalated alert has one investigation or a documented link to an existing related investigation.
- Every investigation has a final decision.
- No-file decisions contain rationale and approval.
- SAR information is inaccessible through ordinary account comments.
- Overdue investigations and filings are visible to the AMLCO once deadline tracking is implemented.
- A test case can reproduce the transaction-alert-to-investigation path and final decision.

---

## 2. Recurring and persistent transaction monitoring

### Required outcome

AGM must regularly evaluate a complete transaction population, retain every material alert and prove that each alert was reviewed and resolved.

Opening a Dashboard report cannot be the event that causes monitoring to occur.

### What to implement

Move the rules from browser-only calculation into a controlled backend process that runs daily over new deposits and withdrawals. AGM does not need a large generic monitoring-run framework for the first release. The first implementation can build on the existing Deposits & Withdrawals workflow by running the current analysis logic on a schedule and persisting alert records for review.

Each alert should contain at least:

- Stable alert ID.
- Account.
- Transaction ID.
- Transaction datetime.
- Amount.
- Currency.
- Deposit/withdrawal direction.
- Alert date.
- Reviewer comment.
- Status and disposition.
- Reviewer and review date.
- Escalation or linked investigation.

### Initial monitoring scenarios

Do not implement dozens of weak rules initially. Begin with the scenarios clearly supported by AGM’s manuals and already closest to the current report logic:

- Large incoming deposits.
- Large outgoing withdrawals.
- Activity inconsistent with declared income, net worth or Source-of-Wealth.

Additional scenarios such as structuring, rapid in-and-out movement, third-party funding, higher-risk jurisdictions, dormancy changes, or related-account patterns may be layered on after the persisted alert workflow is stable.

The current `Amount > 10,000` logic must be corrected so negative withdrawals are not missed merely because they are stored as negative amounts.

### Deduplication

Reprocessing the same source data should not create another active alert for the same transaction when the transaction identifier is already stable in the IBKR source. AGM can treat the IBKR transaction ID as the primary identity for the first implementation rather than introducing an additional deduplication model.

### Process changes

Rewrite Process 17 to describe:

- Scheduled execution.
- Daily processing of new deposits and withdrawals.
- Alert creation.
- Review and disposition.
- Escalation into Priority 1.
- Retention.

### Completion criteria

- Monitoring occurs without a user opening a webpage.
- Deposits and withdrawals are evaluated correctly.
- Alerts survive logout, refresh and later review.
- Every alert reaches a documented disposition.
- Escalated alerts retain their original transaction context.
- New daily deposits and withdrawals produce persisted alerts and reminders for review.

---

## 3. Corporate ownership and required-party coverage

### Required outcome

AGM must be able to identify and reproduce who owns and controls every organization, how that conclusion was reached and whether every required person received KYC, screening and risk review.

### Separate the applicable tests

Do not use one ownership percentage for every purpose.

AGM needs at least two separate rules:

- **CDD/UBO identification threshold:** Must be made consistent across the AGM manuals and confirmed against applicable BVI requirements.
- **Sanctions ownership and control test:** More than 50% ownership or voting rights, ability to appoint/remove a majority of directors, or another ability to control the entity’s affairs.

A person may be relevant because of control even when their ownership is below the numerical threshold.

### What to implement

Create a versioned organization structure containing:

- Legal entity and formation jurisdiction.
- Direct owners and ownership percentages.
- Intermediate entities.
- Calculated indirect ownership.
- Ultimate beneficial owners.
- Voting rights.
- Directors and officers.
- Authorized signatories and traders.
- Powers of attorney.
- Persons exercising control through other arrangements.
- Nominees and the persons behind them.
- Relationship evidence.
- Effective start and end dates.
- Source: AGM application, corporate documents, IBKR or another approved source.
- Reviewer and approval date.

Every required natural person must resolve to an AGM contact with:

- Verified identity.
- Proof of address.
- Current screening.
- Risk assessment.
- PEP/RCA result.
- Applicable EDD.
- Relationship role.

### Validation controls

The system should detect:

- Ownership percentages that do not reconcile.
- Missing intermediate entities.
- Unidentified natural persons.
- Conflicting ownership between AGM documents and IBKR.
- A director or signatory not linked to a contact.
- Circular ownership.
- Unexplained nominee relationships.
- Expired or missing evidence.
- Material ownership changes.

### Submission control

An organization application must not be considered compliance-ready when:

- The ownership structure is incomplete.
- A required person has not been identified.
- A required person lacks KYC or screening.
- Ownership or control is disputed.
- A sanctions ownership/control question is unresolved.

### Completion criteria

AGM can select any organization and reproduce:

- Its complete ownership chain.
- Every applicable natural person.
- Each person’s role and percentage.
- The sanctions control conclusion.
- Supporting documents.
- Current screening and KYC coverage.
- Approval and effective date.

---

## 4. Sanctions-screening coverage and mandatory match response

### Required outcome

AGM must prove that every in-scope party was screened against the applicable sources and that potential or confirmed matches received the required response.

Avoiding duplicate daily rows remains reasonable. The control must prove why screening was safely skipped.

### Population control

Before skipping a scheduled run, calculate:

- Total in-scope contacts.
- Contacts with current complete screening.
- New contacts.
- Contacts with changed names or identity details.
- Contacts with changed ownership or roles.
- Contacts with failed screening.
- Contacts with unresolved candidates.
- Contacts validly skipped.
- Contacts actually screened.

Relevant changes should trigger screening even when the sanctions data files did not change.

### Source control

Create an approved source inventory mapping the AGM manuals to actual sources:

- Source name and authority.
- Official URL or provider.
- Applicable list or regime.
- Retrieval frequency.
- Publication date.
- File checksum.
- Row count.
- Successful/failed status.
- Whether another feed is an approved equivalent.

Do not implement duplicate or retired feeds merely because the draft manuals use overlapping names. Where OFSI, FSC or FIA publish the same consolidated designation information, AGM should document the approved equivalence and update the manuals.

### Candidate handling

A screening result needs more than a list of exact-name hits. It should support:

- `no_candidate`
- `pending_review`
- `false_positive`
- `confirmed_match`
- `insufficient_information`
- `referred_to_FIA`

Candidate review should compare available:

- Names and aliases.
- Date of birth.
- Nationality.
- Passport or identification number.
- Address.
- Entity registration information.
- Ownership and control.
- Relationship to AGM’s customer.

A fuzzy candidate should not automatically be treated as a confirmed designated person. It must remain unresolved until reviewed.

### Mandatory response

Where a potential match is unresolved, AGM’s manuals require account opening and transactions to be halted pending investigation.

Where AGM knows or has reasonable cause to suspect that it is dealing with a designated person or controlled entity, the workflow must support:

- Immediate freeze.
- Cessation of transactions and services.
- Prevention of funds or economic resources being made available.
- Notification to IBKR or other relevant custodian/clearing firm.
- Prompt CRF submission to the Sanctions Unit.
- FIA reporting where applicable.
- Retention of the funds/assets involved.
- CEO or senior-management escalation.
- Complete match and action history.

### Completion criteria

- Every in-scope party has current screening or a valid documented exception.
- A skipped run proves population coverage.
- Candidates cannot disappear without disposition.
- Unresolved candidates block relevant activity.
- Confirmed matches generate mandatory action tasks.
- False positives retain reviewer, rationale, evidence and approval.
- AGM can reproduce the exact source version and data used for any historical screening.

---

## 5. PEP/RCA identification, EDD and senior approval

### Required outcome

AGM must identify politically exposed persons, relatives and close associates and apply the mandatory enhanced controls in the manuals. A PEP boolean affecting a risk score is not sufficient.

### What to implement

Define:

- Which parties must be screened: holders, UBOs, controllers, directors, signatories and other applicable persons.
- Approved PEP/RCA sources.
- Onboarding frequency.
- Ongoing refresh frequency.
- Matching method.
- Review authority.
- Criteria for current and former PEP treatment.
- Required monitoring period after a person ceases to hold the relevant function.

Create PEP/RCA candidates with:

- Source.
- Position and country.
- Relevant dates.
- Related person or entity.
- Identifying information.
- Candidate evidence.
- Reviewer.
- Disposition and rationale.

### Controlled EDD checklist

When PEP/RCA status or another mandatory trigger is confirmed, require:

- Employment and professional-history verification.
- Source-of-Wealth corroboration.
- Source-of-Funds verification where applicable.
- Explanation of expected activity.
- Ownership/control review.
- Higher-risk jurisdiction analysis.
- Adverse information review.
- Senior-management approval.
- Monitoring frequency.
- Next review date.
- Conditions or restrictions.
- Final approval or rejection.

The original calculated risk result should remain preserved even if a reviewer later overrides the category.

### Submission control

IBKR submission must be blocked when:

- PEP/RCA review is pending.
- Mandatory EDD is incomplete.
- Source information is unresolved.
- Required senior approval is missing.
- The relationship was rejected or restricted.

### Completion criteria

- Every required party has current PEP/RCA coverage.
- Every candidate has a disposition.
- Every confirmed PEP/RCA has completed EDD and senior approval.
- Review frequency is automatically increased.
- EDD evidence is retained separately from ordinary application comments.
- Submission cannot bypass the control.

---

## 6. Periodic and event-driven CDD reassessment

### Required outcome

Customer information and risk must remain accurate throughout the relationship, not only at onboarding.

### What to implement

For every active customer and account, store:

- Current risk level.
- Last completed CDD review.
- Next review due date.
- Review frequency.
- Review status.
- Review reason.
- Responsible reviewer.
- Open exceptions.
- Overdue status.

Review frequency should be based on the approved risk policy rather than hardcoded independently in different screens.

### Trigger events

Create review cases after material changes to:

- Legal name or identity.
- Address or country.
- Nationality or tax residence.
- Employment or business activity.
- Income, net worth or Source-of-Wealth.
- Source-of-Funds or funding pattern.
- Ownership or control.
- Director, signatory or power-of-attorney relationships.
- PEP/RCA status.
- Sanctions results.
- Transaction behavior.
- IBKR associated persons or status.
- Account dormancy or reactivation.
- Documents or expiration.
- Jurisdiction risk.
- Relevant law, policy or list information.

### Review content

A completed review should identify:

- Exact customer information reviewed.
- Source records and source dates.
- Changes since the previous review.
- Current documents and screening.
- Ownership and control.
- Transaction activity.
- Open alerts or investigations.
- Updated risk assessment.
- Reviewer conclusion.
- Required remediation.
- Next review date.

### Completion criteria

- Every active relationship has a visible next review date.
- Overdue high-risk reviews escalate automatically.
- Material changes create cases.
- Completing a review recalculates risk and next-review frequency.
- An unresolved material review prevents compliance readiness.
- AGM can reproduce what information was reviewed and what changed.

# Phase 2 — Foundational compliance controls

## 7. Align and govern the AML risk-calculation model

### Required outcome

The application, AML Manual, RBA Procedure and annex must produce the same risk result from the same information.

### Remediation

AGM must approve:

- Risk dimensions.
- Weights.
- Scoring scale.
- Risk thresholds.
- Mandatory overrides.
- EDD triggers.
- Review frequency by category.
- Senior-approval requirements.
- Decline or restriction conditions.

For every assessment, retain:

- Model and rule version.
- Input values.
- Source records.
- Missing inputs.
- Dimension scores.
- Weights.
- Final calculated score.
- Assigned category.
- Explanation.
- Calculation date.

Manual overrides must retain:

- Calculated result.
- Requested result.
- Reason.
- Evidence.
- Requester and approver.
- Effective and expiration dates.

A risk reduction should require stronger approval than a risk increase.

### Completion criteria

A reviewer can take a historical assessment, apply the retained model version and reproduce the exact result.

---

## 8. Institutional Risk Assessment and Board approval

### Required outcome

AGM must maintain a business-level risk assessment distinct from customer risk scores.

### Required scope

The IRA should assess:

- AGM’s introducing-broker and managed-portfolio business model.
- Products and services.
- Customer types.
- Aggregate customer risk distribution.
- Geographic exposure.
- Delivery channels and non-face-to-face onboarding.
- Transaction types and volumes.
- IBKR and other counterparty reliance.
- Sanctions exposure.
- ML, TF and PF risks.
- Control effectiveness.
- Residual risk.
- Required remediation.

### Process

- Prepare the IRA at least annually.
- Use reconciled data with clear as-of dates.
- Identify preparer, reviewer and methodology.
- Present it to the Board.
- Retain Board approval and actions.
- Update it following material business, regulatory, product, customer or jurisdiction changes.
- Connect identified risks to the transaction-monitoring scenarios and CDD frequencies AGM actually uses.

### Completion criteria

AGM can show how its customer-risk model, monitoring rules and remediation priorities derive from its approved institutional risk assessment.

---

## 9. Source-of-Funds and third-party funding verification

### Required outcome

For material, unusual or high-risk funding events, AGM must establish where the specific funds came from and whether the payer is authorized.

### What to capture

- Transaction.
- Amount and currency.
- Originating institution.
- Originating account holder.
- Relationship to AGM’s customer.
- Funding purpose.
- Source-of-Funds category.
- Origin country.
- Supporting evidence.
- Comparison with expected activity.
- Comparison with Source-of-Wealth.
- Reviewer and decision.
- Approval or escalation.

### Policy decision

AGM must either:

- Prohibit third-party funding; or
- Define narrowly permitted situations, evidence, approvals, thresholds and monitoring requirements.

A Source-of-Wealth document cannot automatically clear a transaction-specific Source-of-Funds requirement.

### Completion criteria

Every funding event meeting the approved trigger has a documented payer, origin, evidence and decision. Unexplained third-party funding cannot be closed using only a freeform comment.

---

## 10. Complete and current IBKR compliance data

### Required outcome

Compliance reviews must use a genuine current IBKR snapshot, not a new file containing old carried-forward account data.

### Remediation

For every scheduled refresh:

- Establish the complete expected IBKR account population.
- Request current details for every in-scope account.
- Retain requested, successful, failed, skipped and excluded accounts.
- Record retrieval timestamp per account.
- Record source master account.
- Reconcile IBKR-only and AGM-only accounts.
- Identify duplicate or wrong-master-account records.
- Retain the snapshot checksum.
- Mark the run `complete`, `partial`, or `failed`.
- Prevent a partial run from replacing the last complete compliance snapshot.

Downstream pages should display:

- Snapshot date.
- Run status.
- Missing accounts.
- Stale records.
- Source master account.
- Whether the information is safe for a compliance conclusion.

### Completion criteria

The system can prove that every expected IBKR account was refreshed or clearly identified as a failure or approved exclusion.

---

## 11. Authoritative compliance-readiness decision before IBKR submission

### Required outcome

Before sending an application to IBKR, AGM must produce one server-side decision showing whether all mandatory compliance requirements are satisfied.

### Required checks

The readiness decision should evaluate:

- Application review completed.
- Every required party identified.
- Identity verified.
- Current and accepted documents.
- Complete ownership/control structure.
- Current sanctions screening.
- No unresolved sanctions candidate.
- Current PEP/RCA result.
- Mandatory EDD completed.
- Senior approvals completed.
- Current AML risk assessment.
- Source-of-Wealth evidence.
- Source-of-Funds evidence where triggered.
- No overdue CDD review.
- No open blocking investigation.
- Current reference data.
- Required agreements retained.

Each failure should return a structured blocker containing:

- Blocker code.
- Affected person, document or case.
- Explanation.
- Required remediation.
- Whether an authorized exception is permitted.

### Approval evidence

When the employee submits the application, retain:

- Reviewer.
- Timestamp.
- Application version or checksum.
- Readiness-result version.
- Contacts reviewed.
- Documents reviewed.
- Screening and EDD records reviewed.
- Warnings and overrides.
- Rationale.
- Selected master account.
- External request correlation ID.
- IBKR result.

### Completion criteria

IBKR submission cannot proceed by manipulating the browser or bypassing a client-side check. The API must independently confirm readiness and retain the exact decision used for submission.
