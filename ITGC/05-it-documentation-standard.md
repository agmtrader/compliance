# IT Documentation Standard

## Purpose
Ensure IT documentation is complete, current, assigned to owners, and accessible to authorized staff.

## System of Record
GitHub repository document control under `Compliance/ITGC/` in this repository.

## Scope
Applies to documentation for systems, applications, infrastructure, integrations, security controls, and IT operating procedures.

## Ownership and Approval
- Standard Owner: Andres Aguilar
- Executive Approver: Hernan Castro (CEO)
- Approval Method: Pull Request approval and merge record in GitHub.

## Minimum Documentation Requirements
For each critical system/process, the following must exist where applicable:
1. System overview and business purpose
2. Owner and backup owner
3. Access and authorization model
4. Operational runbook (start/stop, routine ops, escalation)
5. Change/deployment procedure reference
6. Backup/recovery and continuity reference
7. Dependencies and key integrations

## Procedure
1. Maintain inventory
- IT Owner maintains `it-documentation-inventory.csv`.
- Each entry must have named owner, review frequency, and current status.

2. Review cadence
- Documentation owners review assigned documents at least annually.
- If major system/process changes occur, documentation must be updated promptly.

3. Change control
- Documentation updates require Pull Request and reviewer approval.

4. Access and availability
- Documentation must remain accessible in GitHub to authorized internal personnel.

5. Gap tracking
- Missing/outdated documentation must be marked `Gap` in the inventory with target remediation date.

## Minimum Evidence to Retain
- `Compliance/ITGC/it-documentation-inventory.csv` with current statuses.
- Pull Request history for document updates.
- Annual review dates recorded in inventory.

## Review Frequency
- Inventory review quarterly (calendar quarters).
- Document-level review at least annually or after major change.

## Control Mapping
- B320 GITC row 41: Adequate IT documentation policies ensuring documentation is comprehensive, current, and available to appropriate staff.
