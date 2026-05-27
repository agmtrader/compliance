# Transaction Processing Procedure Standard

## Purpose
Define the minimum documented method for correct, complete, and traceable processing of business transactions in each critical application.

## System of Record
GitHub repository document control under `Compliance/ITGC/` in this repository.

## Scope
Applies to critical transaction flows executed by AGM applications and supporting operational processes.

## Ownership and Approval
- Procedure Owner: Andres Aguilar
- Executive Oversight: Hernan Castro (CEO)
- Approval Method: Pull Request approval and merge record in GitHub.

## Minimum Required Definition Per Transaction Flow
Each critical transaction flow must have a documented entry in `transaction-processing-register.csv` including:
1. Application/system name
2. Transaction name and business purpose
3. Trigger/input source
4. Validation checks performed
5. Processing steps (high-level)
6. Output/result and storage location
7. Error/exception handling path
8. Reconciliation or verification control
9. Owner and backup owner

## Procedure
1. Identify critical transaction flows
- Andres Aguilar maintains the in-scope transaction list.

2. Document processing method
- For each flow, complete required fields in `transaction-processing-register.csv`.

3. Define control points
- For each flow, define at least one validation control and one post-processing verification/reconciliation control.

4. Test and review
- Review each documented flow at least annually or after material process/system changes.

5. Change control
- Changes to transaction methods must be updated in the register via Pull Request before or with deployment.

## Minimum Evidence to Retain
- Current `transaction-processing-register.csv`.
- Pull Request history showing updates for transaction/process changes.
- Annual review dates and statuses in the register.

## Review Frequency
- Flow inventory review quarterly (calendar quarters).
- Detailed flow review at least annually or after material change.

## Control Mapping
- B320 GITC row 50: Documented procedures explaining methods for proper processing of transactions in each application.
