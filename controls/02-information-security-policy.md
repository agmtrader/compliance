# Information Security Policy

## Purpose

Establish minimum security requirements to protect AGM data, systems, operations, and third-party integrations.

## Scope

Applies to AGM employees, contractors, systems, applications, infrastructure, data, and external services supporting AGM operations, including IBKR and Interclear integrations.

## Ownership and Approval

- Policy Owner: Chief Compliance Officer.
- Executive Approver: Junta Directiva.
- Approval Method: approval and merge record in the AGM Compliance GitHub repository, supported by the board communication record.

## Policy Statements

1. Access is based on least privilege, authenticated identities, defined roles, and segregation of functions. Shared accounts are prohibited unless explicitly approved.
2. Administrative and privileged access uses stronger authentication controls where technically available.
3. Production changes are documented, reviewed, and approved before implementation. Emergency changes are documented after implementation.
4. Security-relevant activities are logged where technically feasible and retained for review or investigation.
5. Sensitive data is protected in storage and transmission using appropriate industry-standard controls where supported.
6. Security incidents or suspected incidents are reported promptly, documented, escalated, and tracked through closure.
7. Third-party access requires explicit approval, defined scope, appropriate security requirements, and review or revocation when no longer needed.
8. Personnel with system access receive security guidance appropriate to their role.

## Critical Asset Inventory

AGM maintains an inventory of critical hardware, software, and services in `compliance/controls/logs/critical-asset-inventory.csv`. The inventory uses the B310 Understanding the IT Environment - Listing 2025 for the equipment and application baseline and the official third-party provider register for external services. It identifies the asset or service, purpose, type, criticality, dependency scope, primary provider, review date, and status. It is reviewed at least annually and after material changes.

## Communication and Acknowledgement

- The policy is communicated to personnel with system access at issuance and after material updates. The communication record is retained with the policy evidence.

## Adequacy Review

The policy is reviewed at least annually, after material business or technology changes, or after a significant security incident. The reviewer evaluates company size and staffing, system criticality, data sensitivity, regulatory obligations, vendor access exposure, and current threats or incidents. The decision, rationale, gaps, and actions are recorded in `compliance/controls/logs/security-policy-review-log.csv`. The Junta Directiva reviews the outcome and approves material updates.

## Exceptions

Exceptions require documented business justification, risk assessment, compensating controls, expiration date, and approval by the Junta Directiva.

## Minimum Evidence to Retain

- Current approved policy version.
- GitHub approval and merge history.
- Board approval and policy communication record.
- Policy communication and completed adequacy reviews.
- Exception records, if any.

## Control Mapping

- B320 GITC row 36: formal IT security policy approved by management and communicated to employees.
- B320 GITC row 38: IT security policy appropriate for the size and complexity of the entity.
