# Security Policy Adequacy Review Procedure

## Purpose
Ensure the Information Security Policy remains appropriate for AGM's size, operational complexity, systems, and risk profile.

## System of Record
GitHub repository document control under `Compliance/ITGC/` in this repository.

## Scope
Applies to the Information Security Policy and related security control requirements.

## Ownership and Approval
- Procedure Owner: Andres Aguilar
- Executive Approver: Hernan Castro (CEO)
- Approval Method: Pull Request approval and merge record in GitHub.

## Review Triggers
A policy adequacy review is required:
- at least annually
- after material business or technology changes
- after significant security incidents

## Adequacy Criteria
The reviewer must confirm whether policy coverage is still adequate for:
1. Company size and staffing
2. Number and criticality of systems/applications
3. Data sensitivity and regulatory obligations
4. External/vendor access exposure
5. Current threat/risk conditions and known incidents

## Procedure
1. Perform adequacy assessment
- Andres Aguilar reviews the current security policy against the adequacy criteria.

2. Record decision
- Mark decision as `Adequate` or `Needs Update`.
- Record rationale and identified gaps in `security-policy-adequacy-reviews.csv`.

3. Remediate if needed
- If `Needs Update`, open and complete a policy update Pull Request with required approvals.

4. Management oversight
- Hernan Castro reviews adequacy outcome annually and approves major updates.

## Minimum Evidence to Retain
- Completed entries in `Compliance/ITGC/security-policy-adequacy-reviews.csv`.
- Pull Request links for resulting policy updates (if any).
- Annual management review/approval evidence in GitHub.

## Review Frequency
- At least annually, and event-driven as listed above.

## Control Mapping
- B320 GITC row 38: IT security policy is appropriate for the size and complexity of the entity.
