# IT Strategy and Governance Procedure

## Purpose
Define how management sets, approves, tracks, and reviews the IT plan so IT work supports business priorities.

## System of Record
GitHub repository document control under `Compliance/ITGC/` in this repository.

## Scope
Applies to all IT initiatives, projects, systems, and security-related investments.

## Roles
- Executive Sponsor (Hernan Castro, CEO): final approval of IT plan.
- IT Owner (Andres Aguilar): drafts the plan and tracks execution.
- Process Owners: provide business requirements and priorities.

## Procedure
1. Create annual IT plan.
- IT Owner drafts a 12-month plan including:
  - top business objectives supported by IT
  - key initiatives/projects
  - expected outcomes and target dates
  - required budget/resources
  - key risks and dependencies

2. Management review and approval.
- Executive Sponsor reviews the plan.
- Feedback is incorporated.
- Final approved version is merged through a Pull Request approved by Hernan Castro.

3. Quarterly monitoring.
- IT Owner updates initiative status quarterly: `Not Started`, `In Progress`, `At Risk`, `Done`.
- For `At Risk` items, record cause, impact, and mitigation action.
- Each quarterly update is committed through a Pull Request with reviewer approval.

4. Change control for major plan changes.
- Major scope, budget, or timeline changes require documented approval from Executive Sponsor.
- Approval is captured in the Pull Request review and merge record.

5. Annual refresh.
- Plan is formally reviewed and re-approved at least once every 12 months.

## Minimum Evidence to Retain
- Current approved annual IT plan in `Compliance/ITGC/`.
- Pull Request approval from Hernan Castro for annual plan approval.
- Quarterly update Pull Requests with timestamps and reviewer approval.
- Pull Request history for major changes (diff + approval + merge commit).

## Review Frequency
- Quarterly status review (calendar quarters).
- Annual full refresh and re-approval.

## Control Mapping
- B320 GITC row 26: Management responsibility for IT plan and strategy.
