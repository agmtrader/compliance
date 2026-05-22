# Information Security Policy

## Purpose
Establish minimum security requirements to protect company data, systems, and operations.

## System of Record
GitHub repository document control under `Compliance/ITGC/` in this repository.

## Scope
Applies to all AGM employees, contractors, systems, applications, infrastructure, and data.

## Ownership and Approval
- Policy Owner: Andres Aguilar
- Executive Approver: Hernan Castro (CEO)
- Approval Method: Pull Request approval and merge record in GitHub.

## Policy Statements
1. Access Control
- Access to systems and data must be based on least privilege.
- Shared accounts are not allowed unless explicitly approved and documented.
- Access must be removed promptly when no longer required.

2. Authentication
- Company systems must require authenticated access.
- Administrative or privileged access must use stronger authentication controls where technically available.

3. Change and Release Security
- Changes to production systems must be documented, reviewed, and approved before implementation.
- Emergency changes must be documented after implementation with reason and approval.

4. Logging and Monitoring
- Security-relevant activities must be logged where technically feasible.
- Logs must be retained and available for review/investigation.

5. Data Protection
- Sensitive data must be protected in storage and transmission using industry-standard controls where supported.
- Data retention and deletion must follow documented retention requirements.

6. Incident Reporting
- Security incidents or suspected incidents must be reported to Andres Aguilar immediately.
- Incidents must be documented with actions taken and closure status.

7. Vendor and Third-Party Access
- Third-party access to AGM systems/data requires explicit approval and defined access scope.
- Third-party access must be reviewed and revoked when no longer needed.

8. Security Awareness
- Personnel with system access must receive security guidance appropriate to their role.

## Communication Requirement
- This policy must be communicated to personnel with system access.
- Communication and acknowledgements are tracked in `Compliance/ITGC/security-policy-acknowledgments.csv`.
- Acknowledgement is required:
  - at initial issuance
  - after any material policy update
  - at least annually
- If policy changes are merged, a GitHub workflow notification must be sent to request review and acknowledgement.
- Communication evidence must remain in GitHub (workflow run link, PR link, or equivalent).

## Exceptions
- Any exception must be documented with:
  - business justification
  - risk assessment
  - compensating controls
  - expiration date
  - approver (Hernan Castro)

## Review Frequency
- Reviewed at least annually or after material business/technology changes.

## Minimum Evidence to Retain
- Approved policy version in `Compliance/ITGC/`.
- Pull Request approval and merge history.
- Policy communication record.
- Exception records (if any).
- Annual review record with date and reviewer.

## Control Mapping
- B320 GITC row 36: Formal IT security policy approved by management and communicated to employees.
