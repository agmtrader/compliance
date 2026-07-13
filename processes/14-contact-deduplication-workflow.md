# Contact Deduplication and Dependency Merge

## Business Purpose
Identify duplicate contact records conservatively and merge only high-confidence duplicates while preserving account links, documents, screenings, review assignments, user/advisor references, and IBKR holder identity.

## Trigger / Frequency
- Trigger: Authorized operator runs the read-only planner and explicitly approves the resulting plan hash for application.
- Frequency: On demand when duplicate-contact records are detected.

## Systems Involved
- `agm-api`
- Supabase PostgreSQL production database
- Latest IBKR details backup exposed by `get_ibkr_details`

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Current contact and dependency tables
- Latest available IBKR account details and associated-person records
- Database credentials with read access for planning and explicit write access for application
- An operator-reviewed plan hash generated from current production data

## Step-by-Step Workflow
1. Run `dev/contact_dedup_dry_run.py` to read the database in a read-only transaction and produce JSON and CSV evidence.
2. Normalize names and evaluate exact and fuzzy similarities across all contacts. Similar names alone are never sufficient for an automatic merge.
3. Corroborate candidates using account links, documents, screenings, populated identity fields, and holder-specific IBKR details. Treat `mail.com`, `mailfence.com`, `pm.me`, `proton.me`, `protonmail.ch`, and `protonmail.com` as weak temporary-email evidence.
4. Exclude candidates with conflicting populated identity data, including different non-empty `entity_id` or `external_id` values for the same account. Leave excluded pairs for manual review.
5. Run `dev/contact_dedup_apply.py` without `--apply` to regenerate the production plan and record its SHA-256 hash.
6. Apply only that exact plan using `--apply --confirm-plan-hash`. The script regenerates the plan and refuses execution if the hash changed.
7. Inside one serializable transaction, acquire an advisory lock, save a pre-apply backup, consolidate or repoint account links, repoint document links and other dependencies, verify no references remain, and then delete only duplicate contact rows. Raw `document` rows are not deleted.
8. Run the read-only planner again after commit. Confirm that the contact count changed by the committed deletion count and that no high-confidence automatic merge remains.
9. If an authorized operator approves a reviewed second pass, use `--reviewed-second-pass`. This mode selects only pairs with the same IBKR holder and name similarity of at least 0.80, or the same account plus phone plus document evidence. It still excludes any connected cluster with conflicting account `entity_id` or `external_id` values.
10. For the reviewed second pass, retain a holder-specific non-temporary IBKR email when it corroborates a current contact email. Otherwise retain a non-temporary current email ahead of a configured temporary-domain address. Apply and verify the second-pass plan using the same exact-hash, backup, transaction, and post-run controls.

## Outputs / Records Created
- Prepared plan JSON and candidate-review CSV
- Pre-apply JSON backup of every affected row
- Apply-result JSON containing the plan hash, commit timestamp, and mutation counts
- Consolidated canonical contacts and repointed dependency records
- Post-apply read-only audit report

## Exception Paths / Failure Handling
- Plan hash mismatch: refuse all writes and require a new review.
- Conflicting account `entity_id` or `external_id`: downgrade the candidate to manual review.
- Database connection, lock, validation, or dependency failure: roll back the entire transaction.
- Missing or ambiguous holder-specific IBKR details: do not treat an account-level email as authoritative; retain the pair for manual review unless other evidence is sufficient.
- Recovery after an unexpected committed result uses the retained pre-apply backup and a separately reviewed restoration procedure.

## Controls / Verification Points
- Preventive control: the planner runs in a read-only transaction and has no apply mode.
- Preventive control: application requires an exact SHA-256 plan hash and a serializable transaction protected by an advisory lock.
- Preventive control: populated identity conflicts block automatic merging.
- Preventive control: reviewed second-pass clusters are evaluated again after transitive grouping; a conflict anywhere in the connected cluster excludes the entire cluster.
- Preventive control: dependency references are repointed or consolidated before contact deletion; raw documents are never deleted.
- Detective control: the apply script verifies the deleted-contact count and writes a committed result only after transaction completion.
- Detective control: a post-commit dry run must report zero remaining automatic merge clusters for the evaluated rules.

## Evidence to Retain
- Prepared plan and its SHA-256 hash
- Candidate-review CSV
- Pre-apply backup JSON
- Apply-result JSON
- Post-apply dry-run JSON/CSV and operator summary

## Related Code / Pages / Routes
- Planner: `agm-api/dev/contact_dedup_dry_run.py`
- Transactional apply script: `agm-api/dev/contact_dedup_apply.py`
- IBKR details source: `agm-api/src/components/tools/public/reporting.py`
- Database tables: `contact`, `account_contact`, `contact_document`, `contact_screening`, `document_review_responsible`, `user`, and `advisor`

## Last Reviewed
- Status: draft
- Date: 2026-07-13
- Reviewer: Codex implementation review
