# Contact Screening and AML Risk Assessment

## Business Purpose
Create contact-level sanctions results and a weighted AML risk score during onboarding, manual screening, or batch screening, and expose the latest results in account and application views.

## Trigger / Frequency
- Trigger: Finalization of an AGM Hub application with a linked contact that has no screening history; an authenticated `POST /contacts/screening`; or the Daily Screening Run.
- Frequency: Event-driven during onboarding or manual use, and conditionally during the scheduled screening workflow.

## Systems Involved
- `agm-hub`
- `agm-dashboard`
- `agm-api`
- Supabase-backed `account`, `account_contact`, `contact`, and `contact_screening` tables
- Google Drive sanctions resource files and IBKR details backup

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- Contact with a nonblank name
- A linked account, or an account whose stored `application_json` can be matched to the contact by email or normalized name
- Latest available account/application and IBKR details data
- OFAC, UK, and UN resource files available through the reporting layer
- Hardcoded jurisdiction-risk and FATF-listed country sets in the contacts service

## Step-by-Step Workflow
1. During final Hub application persistence, the Hub reads each linked contact's screening history. If a contact has no records, it calls `POST /contacts/screening` for that contact.
2. A direct screening request loads the contact and selects its most recently created or updated `account_contact` link. If no link exists, it searches stored account applications for a matching customer email or normalized holder name and uses the most recently updated matching account.
3. If the selected account has an IBKR account number, the service uses the cached IBKR details backup when a matching detail exists; otherwise it uses the stored application payload.
4. The service loads OFAC, UK, and UN resource files on first use in the API worker and builds in-memory exact-name indexes. OFAC uses its `name` field, UK uses `Name 1` through `Name 6`, and UN uses primary names plus pipe-separated aliases.
5. The contact name and source names are normalized by removing accents and non-alphanumeric punctuation, converting to lowercase, and collapsing spaces. Only exact normalized-name matches are returned.
6. The service resolves a risk country from the matched IBKR associated person or the selected holder in the stored application.
7. It calculates five risk components: customer risk, jurisdiction risk, product risk, delivery-channel risk, and introducer risk.
8. The component weights are 30%, 25%, 15%, 15%, and 15%, respectively. Delivery-channel risk is always `5`; introducer risk is `3` when the account has an advisor code and `1` otherwise.
9. Customer risk considers account type, PEP or adverse regulatory indicators, investment experience, wealth-source complexity, cross-border information, employment, and organization structure. Jurisdiction and product scores use hardcoded country and product groups.
10. The weighted result is constrained to the range `1` through `10`. Any exact OFAC, UK, or UN match raises the final score to at least `9`.
11. FATF status is set to `Listed` when the resolved country matches a hardcoded FATF country code or normalized country name; otherwise it is stored as `null`.
12. The API creates a `contact_screening` row containing the contact id, risk score, FATF status, OFAC results, UK results, UN results, and a compact timestamp.
13. Account and application pages read screening history, sort it by the stored timestamp, and show counts, latest date, latest risk score, and source hit counts.

## Outputs / Records Created
- `contact_screening` database row for each completed screening
- Contact-level AML risk score
- FATF `Listed` or `null` status
- Exact-match evidence arrays for OFAC, UK, and UN
- Screening summaries shown on account and application pages

## Exception Paths / Failure Handling
- Missing contact id, contact, contact name, account context, or valid customer type causes the request to fail.
- A contact with no account link can still be screened when a stored application matches its email or normalized name.
- A Hub finalization fails if its screening call fails; there is no separate persisted onboarding screening-attempt record.
- Sanctions lists, sanctions indexes, and IBKR details are cached in each long-running API worker and are not invalidated after a resource refresh.
- A possible match does not create a separate review case and there is no stored true-match or false-positive disposition.
- Multi-account contacts are assessed using the most recent link for a direct request. The daily batch deduplicates contacts and retains the first account context encountered.

## Controls / Verification Points
- Preventive control: a screening cannot be created without contact and account context.
- Detective control: screening results and scores are retained as dated contact-level rows.
- Detective control: account and application views expose screening coverage and latest results.
- Current limitation: exact normalized-name equality is the only sanctions matching method.
- Current limitation: hardcoded FATF and jurisdiction groups have no stored effective date or source version.
- Current limitation: no reviewer disposition, escalation, override reason, or approval is retained.

## Evidence to Retain
- `contact_screening` rows
- Related `contact`, `account_contact`, and `account` records used for context
- Sanctions resource snapshots used at the time of screening, where they can be reconstructed
- API logs for direct or batch screening requests
- Account or application review exports/screenshots when retained operationally

## Related Code / Pages / Routes
- Entry surfaces: `agm-hub/src/components/hub/apply/IBKRApplicationForm.tsx`, `agm-api/src/app/clients/contacts.py`, `agm-api/src/app/tools/private/actions.py`
- Supporting modules: `agm-api/src/components/clients/contacts.py`, `agm-api/src/components/tools/private/screenings.py`
- Downstream side effects: `agm-dashboard/src/components/dashboard/clients/screenings/ContactScreenings.tsx`, account and application detail pages

## Last Reviewed
- Status: draft
- Date: 2026-07-15
- Reviewer: Codex current-state code review
