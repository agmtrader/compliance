# Market Data ETL Pipeline

## Business Purpose
Refresh AGM market-data resource files used by reporting and analytics by running the configured extract, backup, and transform stages for the market-data pipeline.

## Trigger / Frequency
- Trigger after cutover: GitHub Actions launches the `agm-market-etl` Cloud Run Job and polls its exact execution. The authenticated manual `GET`/`POST /etl/market_data` path dispatches the same job and returns HTTP 202; `GET /etl/market_data?operation=<id>` reads its operation status.
- Frequency: Daily at 06:00 UTC / 12:00 AM CST (midnight Costa Rica, UTC-6), plus manual workflow dispatch. GitHub Actions can still create the run late.

## Systems Involved
- `agm-api`
- GitHub Actions
- Cloud Run Job and private GCS terminal-overview storage
- Google Drive batch and resources folders
- External market-data source functions configured in the ETL pipeline

## Roles / Owners
- Primary owner: Andres Aguilar
- Backup owner: Hernan Castro
- Executive oversight: Hernan Castro

## Inputs / Prerequisites
- GitHub Google Cloud identity with job invocation, execution read and private overview read permissions; the API runtime identity needs job invocation/operation-read access for manual dispatch
- API auth token with existing `etl/market_data` scope for the manual route only
- Dedicated empty market staging folder, distinct from clients staging and the legacy shared inputs
- Valid `market_data` ETL config
- Access to configured market-data sources and Google Drive
- Functional transform steps for market-data output
- IBKR market-data session can be established, bootstrapped with a successful
  `/iserver/accounts` response, and closed after extraction

## Step-by-Step Workflow
1. The workflow launches one market Cloud Run execution without automatic retry or API fallback, then polls that exact execution. Manual API callers receive an operation reference immediately; acceptance is not ETL completion.
2. The job acquires a generation-owned market lock and verifies its dedicated staging folder is empty. Prior interrupted staging is retained for operator review, never blindly cleared on startup. It initializes the existing database connector and resolves the `market_data` configuration. The clients job and its coverage policy are unchanged.
3. The ETL runner executes `extract`, `backup`, and `transform` stages in order.
4. Extract pulls each configured market-data source into the batch area and summarizes step success, skip, or failure.
   For IBKR sources, each extract opens one SSO/brokerage session, queries
   `/iserver/accounts` before security or snapshot requests, and logs out in a
   `finally` path after the source completes or fails.
5. Backup renames batch files, moves them into the resource structure, and clears the batch folder.
6. Transform converts the resource files into downstream market-data outputs consumed by reporting logic.
   The background market runner enables historical daily-bar enrichment through
   per-run `historical_yields: true` for stocks and ETFs while retaining
   `DEV_MODE=false`. Direct development extracts also retain enrichment through
   `DEV_MODE=true`; foreground production extracts skip it by default.
   `Current Yield` is the existing arithmetic mean of five annual price returns
   in percentage points, using six annual close anchors. It is not dividend,
   distribution or total return. Requests, including the older daily-bar page,
   are sequential and wait 1.25 seconds before each history call.
7. The runner releases only its owned lock and publishes a durable `runs/<execution>/overview.json`, including lock-release failure and raw/operational outcomes. Publication failure exits nonzero. A killed process can leave a lock requiring verified terminal-state review before conditional operator removal.
8. GitHub Actions requires both Cloud Run terminal success and a matching execution/pipeline overview. It validates all four configured market sources, all three stages, nonzero/consistent counts and exact extract-to-transform source IDs. Partial, missing, malformed, inconsistent or provenance-free results fail the workflow; clients accepted-gap/Flex fallback rules do not apply to market. An always-running step publishes a bounded aggregate summary. Unavailable evidence stays unavailable. No whole-operation retry is performed because an interrupted execution may already have published files.

## Outputs / Records Created
- Refreshed raw market-data files in Google Drive
- Updated transformed market-data resources
- ETF and stock resource rows with populated `Current Yield` in the background
  job when five years of usable price history are available
- An empty `Current Yield` for unavailable/invalid/insufficient history, with
  per-extract requested, populated and unavailable counts in the terminal overview;
  zero and negative calculated returns are preserved
- Foreground production snapshots retain an empty `Current Yield` by default
- Per-symbol resolution, snapshot, and history failures logged with
  ticker and conid; symbols with missing or invalid snapshot rows are omitted while
  valid symbols continue processing
- Durable private GCS terminal overview; manual API dispatch returns an operation/status URL rather than the completed ETL payload
- Workflow notification email containing the overall result, extract/backup/transform stage statuses and counts, and any reported failures

## Exception Paths / Failure Handling
- Centralized API exception reports retain request/error IDs, HTTP method, route template, endpoint, status/code and exception types with stack file/line/function locations. They omit query values, client IPs, error details, exception messages and source-line text; client-facing controlled errors retain their existing message/details. This redaction covers the shared exception wrapper, not every independent integration/audit log.
- An expired IBKR trading session or the known accounts-bootstrap prerequisite remains a controlled provider error with reconnect guidance, without an exception report. In the job it becomes a failed stage/terminal outcome; the dispatch response cannot certify the later brokerage session. Stage/symbol failure outcomes remain visible. IBKR service authentication, unknown upstream failures, and missing or duplicate reporting resources remain reportable.
- Cloud authentication, job launch, polling or overview retrieval failure: workflow fails with available execution references and unavailable metrics; no synchronous fallback. A dispatch timeout is uncertain, not permission to replay.
- Missing/incomplete pipeline config, nonempty staging or lock acquisition failure: job fails before pipeline writes.
- Extract, backup, or transform failure: the overview captures the failing step and the workflow fails. A market transform cannot fall back to a prior backup when its extractor failed.
- Historical-yield enrichment is enabled independently of `DEV_MODE` in the
  background market runner. Missing/invalid history leaves the symbol's yield
  blank and does not abort the other symbols. Pipeline success alone does not
  certify complete historical-yield coverage; inspect terminal overview counts.
- A failed stock or ETF security lookup, snapshot response, or historical-yield
  request is logged with `MARKET_DATA_SYMBOL_FAILED` and the symbol/conid when
  available. Missing snapshot rows are excluded from that output; they do not abort
  the remaining symbols. The snapshot summary reports requested, returned, and
  failed counts.
- Bond duration calculation failure for an individual dated bond: the transform
  logs the exception with the row inputs and, when maturity is missing, identifies
  the affected bond by symbol, financial instrument, and company name. It leaves
  that row's duration blank, reports the failure count, and continues processing
  the remaining rows. Perpetual/non-maturing instruments are an expected
  exception: duration is not applicable, so the row remains blank and a warning
  is recorded without raising an error-group traceback.
- No automatic whole-job or dispatch retry is performed; inspect execution, private overview, staging and lock state before any manual replay. API operation completion reports Cloud Run completion only; the private overview is authoritative for pipeline/stage results.

## Controls / Verification Points
- Preventive control: pipeline name must map to a defined config.
- Detective control: each stage reports total, successful, skipped, and failed step counts.
- Detective control: backup-step counts are included in workflow summary output.
- Detective control: workflow logs and email preserve the ETL outcome for review.

## Evidence to Retain
- GitHub Actions run history for `etl_market_data.yaml`
- Terminal ETL overview JSON in private GCS, Cloud Run execution outcome and bounded Actions summary
- Resource folder outputs after the run
- Success or failure notification email

## Related Code / Pages / Routes
- Entry surfaces: `agm-api/.github/workflows/etl_market_data.yaml`, `agm-api/src/jobs/market_etl.py`, `.github/scripts/market_etl_summary.py`, `agm-api/src/app/tools/private/etl.py`
- Supporting modules: `agm-api/src/components/tools/private/etl.py`
- Downstream side effects: reporting resource files and market-data-derived outputs

## Bond Description Parsing

Bond snapshot transformation prioritizes the percentage-marked coupon in the
IBKR financial-instrument description so issue-date components are not
mistaken for coupon values. Descriptions whose maturity is represented only by
an IBKR shorthand code remain dependent on the source snapshot's explicit
maturity field. Descriptions marked ``Perpetual`` (or ``Perp``) intentionally
have no maturity; their duration is left blank because a principal repayment
date does not exist.

## Dependency Initialization

Drive and IBKR trading connectors are constructed on the first ETL operation that needs them, rather than during API startup. Their initialization failures remain within the existing ETL step/error handling. Schema compatibility is checked by the candidate-image deployment preflight before rollout; API startup no longer creates or reflects the entire database schema.

## Last Reviewed
- Status: draft
- Date: 2026-06-16
- Reviewer: Codex initial draft

## Downstream Proposal Rating Eligibility

Proposal generation validates and normalizes bond ratings from transformed
market-data resources before selection. Valid canonical S&P equivalents are
preserved, including the report merge's `_x` column. When a canonical equivalent
is unavailable, valid structured S&P/Moody's fields are preferred to free text;
when both agencies provide ratings, the lower credit rating is retained, matching
the transform's rating-level policy. Free-text parsing matches whole rating tokens
and preserves modifiers rather than matching letters embedded in words.

Only treasury, AAA/AA/A, BBB and BB bond categories are eligible for the configured
proposal buckets. BB+/BB/BB- and Moody's Ba1/Ba2/Ba3 equivalents remain eligible.
Lower-rated, missing, unrated and unrecognized candidates cannot fill a BB bucket.
A requested bucket with no eligible assets returns HTTP 422 before persistence;
existing saved proposals are not rewritten by this consumer control. The ETL
trigger, resource outputs and transformation behavior are unchanged.

## Extractor and publication controls — October 20, 2026

Extractor data and execution metadata are separate: DataFrames contribute row counts, structured outcomes retain their status, and a market source must record the Drive ID returned by its upload. Only that exact source can feed the corresponding transform in the same request. A newer unrelated backup or stale prior-run file cannot substitute. Runtime config copies discard provenance left in templates.

Final resources use verified staging: upload under a noncanonical pending name, check Drive byte length and MD5, then update the existing stable file ID or promote the verified stage when no stable resource exists. Verify the canonical result before cleaning the stage. Never delete the stable file before upload. Duplicate canonical resources fail closed; ambiguous interrupted writes are not automatically replayed. Pending files after an uncertain failure require inspection. This is not a distributed lock or a multi-file transaction.

Regression acceptance covers DataFrame return handling, exact source selection, preservation under upload/checksum/update failures, original exceptions and Actions shell outcomes. Local fixtures do not establish hosted Drive acceptance.

## Async market cutover — October 20, 2026

Implementation uses the working clients architecture with a separate market job, staging folder and generation-owned lock. Job runtime is the current immutable API image plus an exact allowlisted, generation-pinned SHA-256 source overlay; no clients runner/baseline is changed. Job timeout is 60 minutes, one task, one CPU/2 GiB and no automatic retries. The schedule is revised to 12:00 UTC / 06:00 CST (Costa Rica) alongside clients ETL; notification recipients are preserved. Deployment commands, environment and actual acceptance evidence are recorded in `agm-api/docs/market-etl-runner.md` and API #86. API main commit 4d078ff publishes the job-based workflow and dispatch source; normal CI/API deployment remains separately observable. Controlled cloud execution agm-market-etl-vb9g8 completed successfully at 18:26:28 UTC on October 20 (12:26 Costa Rica), with all four extracts and transforms successful, successful backup, durable overview, lock HTTP404 and empty staging. All four canonical outputs were independently checked for unique names, nonzero sizes/MD5 and current-execution modification times. First scheduled invocation and authenticated hosted manual-route acceptance remain separate observations.

## Shared scheduled start — October 21, 2026

Owner requested both clients and market ETLs at 07:37 CST / 13:37 UTC daily.
This supersedes earlier schedule times retained in historical cutover evidence.
Each pipeline keeps its own job, staging folder and execution lock. Scheduled
start time is a GitHub Actions target, not a guarantee of exact execution time.


## Proposal selection and count controls

The owner's final October 26 count revision requires exactly 20 assets for every
new risk-questionnaire proposal across all archetypes. This supersedes the earlier
25–37 position counts and removes the incompatible 4.5% per-position cap: 20
holdings cannot total 100% under that cap. Archetype count metadata now reports
20 and the former position limit is null. The portfolio planner passes a
user-selected positive integer to the same selection function for both preview
and save, defaulting to 20 when omitted, null or blank. Selecting a profile or
preset preserves the user's selected count. Invalid supplied counts still fail
validation. Money allocation percentages remain independent of security
counts: each active sleeve retains its assigned capital weight, split equally among
its selected securities. Within screened bond candidate pools, selection retains
the allocation-weighted return objective. ETF selection uses the breadth and
concentration policy described below; the count optimizer consumes that policy
score, so a later optimization cannot restore return-only ETF ranking.
Corporate bond rating, yield validity, outlier and issuer-exclusion controls apply.
Treasury uniqueness is by instrument. Equity ETF historical returns retain
source percentage-point values, including zero or negative returns. This
objective is a model based on source estimates, not a guarantee of future return.
Missing eligible sleeves or an infeasible exact count fail before persistence; no
silent underfilled proposal is created. Custom proposals retain the user's positive
asset weights and omit zero-weight holdings. Custom preview and save build from
the same explicitly selected holdings and source-yield interpretation; the custom
return card does not use an automatically selected portfolio estimate. Presentation and return calculations
use money weights and exclude zero-weight classes in views and exports. Existing
saved proposals are not regenerated by these code changes; historical missing
weights remain a separate reconciliation requirement.

## Shared scheduled start revised — October 23, 2026

Owner requested both clients and market ETLs daily at 03:07 CST (Costa Rica,
UTC-6) / 09:07 UTC (`7 9 * * *`). This supersedes earlier schedule times retained
in historical evidence. Each pipeline keeps its separate job, staging folder and
execution lock. GitHub Actions may start scheduled workflows late.

## Dashboard snapshot reads — October 23, 2026

Parsed bond, Treasury, stock and ETF reporting snapshots use separate entries in the API's process-local report cache. The default TTL is 30 seconds (`REPORT_CACHE_TTL_SECONDS`, 0 disables, maximum 300 seconds). Concurrent misses share one load; callers receive independent copies. Source errors are not cached, and expired values are never served as fallback. ETL publication does not broadcast invalidation, so consumers can observe the prior snapshot until the configured TTL expires. This applies to stored reporting snapshots, not live IBKR responses or orders.

## Shared scheduled start revised — October 26, 2026

Owner requested both clients and market ETLs daily at 06:00 UTC / 00:00 CST
(midnight Costa Rica, UTC-6), using `0 6 * * *`. This supersedes earlier schedule
times retained in historical evidence. GitHub Actions may start scheduled workflows
late; the configured time is a target.

## Proposal allocation categories — October 26, 2026

Risk generation, the Dashboard portfolio planner and custom proposal editor use
four separate classes: Treasuries, Investment grade (AAA–BBB), High yield (BB),
and Equity (classified equity ETFs). Individual stocks, bond ETFs and commodity
ETFs cannot enter new proposals. Planner payloads carry the four direct
percentages totaling 100%; no
separate credit suballocation or cash sleeve is accepted. Matching archetype
presets retain the archetype position count and 4.5% cap. Edited plans retain
their requested weights and count and fail explicitly when infeasible.

Custom preview and save validate each security against its selected category and
market feed; duplicate holdings and misclassified bonds or ETFs fail before save.
Custom capital weights are editable independently of archetype counts and caps.
The editor splits each class across its selected holdings. Saved proposals,
Dashboard/Hub charts, summaries and exports use the same four classes. Newly
saved records store those buckets directly.

### Custom proposal bases — October 27, 2026

The Dashboard custom editor can load an existing saved proposal as an editable
base. The picker reads compact summaries; only the selected proposal's full
holdings are fetched when the user selects **Use as base**. Loading replaces the
draft holdings, class allocations and saved starting amount, while retaining the
target contact and risk profile. No source proposal is changed. Saving continues
to create a new custom record through the existing validated assets endpoint.

Complete positive saved weights are retained at the editor's two-decimal
percentage precision, with remainder rounding keeping the total at 100%.
Allocation edits scale the imported holdings proportionally within each class;
adding a holding splits that class equally. Fresh selections retain equal splits.
Missing or invalid weights, duplicate symbols, unsupported cash/stocks, empty
proposals and holdings unavailable in their current market category prevent
loading and leave the draft intact. Errors have an explicit retry action, and
stale reads are ignored. Current market data supplies display and preview values;
the existing preview/save eligibility checks remain authoritative. Browser
acceptance and release evidence must be recorded separately from source changes.

The owner subsequently removed Gold from all portfolio calculations and selected
proportional redistribution of its former 4% across the other classes. Archetype
weights now preserve the remaining relative allocations and total 100%. Planner
and Custom presets use matching two-decimal percentages with largest-remainder
rounding, and allocation edits preserve exact totals in hundredths of a percent.
Gold (GLD and IAU) is removed from the ETF extraction universe. Existing snapshots
are not rewritten by this source change; new proposals cannot select those funds.

On read, legacy Gold buckets and GLD/IAU holdings are excluded. Fully weighted
remaining holdings are scaled proportionally to 100%; metadata is retained and
list counts omit retired holdings. Historical unweighted records keep their
existing fallback and reconciliation limits. This read view does not regenerate
securities, restore archetype position counts, reapply the per-position cap, or
rewrite stored rows. The earlier 13 risk-proposal regeneration recorded in API
issue #304 predates this removal. The owner subsequently authorized regeneration
of all existing proposals, including using linked profiles for the two planner
records with missing saved weights.

All 22 saved proposals were replaced atomically in AGM Database on October 26
at 20:33:51 UTC / 14:33:51 Costa Rica. The 13 risk-based records use the new
gold-free archetypes (446 positions); four weighted planner records retain
their saved class weights and requested counts, and two previously unweighted
planner records use their linked risk archetypes (six planner records, 147
positions). The three custom records retain their selected securities, saved
weights and saved return inputs, with canonical investment-grade grouping
(55 positions). Their legacy return inputs were not refreshed or replaced.

The operation preserved IDs, ownership links, source types, creation metadata
and starting amounts, and changed only assets and update timestamps. All-row
and linked-profile comparison guards preceded writes. The first transaction
rolled back on a metadata-check expression error; independent readback confirmed
all 22 original rows unchanged before the corrected transaction. Independent
post-commit readback verified all replacement assets, preserved metadata, 648
canonical positions, no Gold/GLD/IAU, and weights totaling 100% in every record.
Risk-derived replacements also satisfy their archetype position counts and
4.5% position cap. Before-images remain in the operation tool record.

Regeneration used stored market snapshots read from Drive: bond, Treasury and
ETF resources last modified October 25, and open-position exclusions from
October 26. No market ETL, broker transaction, or application deployment was
performed. Deployment and manual application/export acceptance of the source
removal remain separate from the completed saved-data regeneration.

Manual UI and export acceptance belongs to the owner, who requested stopping
automated application tests. Type checking and source review do not establish
that acceptance.


## ETF selection preference revised — October 26, 2026

The owner requested quality before growth and clarified that 8–12% is a planning
range, not a ceiling, a per-fund exclusion, or assured future performance. New
automatic risk and planner generation uses explicit ETF breadth/concentration
preferences in `EQUITY_ETF_SELECTION_POLICY`. Broad global, U.S. and international
market exposure receives priority over narrow sectors and innovation themes.
Nasdaq growth exposure remains eligible. Similar index wrappers share an exposure
family and receive a repeat-exposure penalty; this is a classification proxy, not
a holdings-level overlap calculation.

The base selection score gives 80% weight to the reviewed breadth preference and
20% to a smooth return-fit score around 10%. There is no hard return band, clipping
or substitution: all displayed portfolio and ETF returns still use the selected
funds' actual stored source values. Missing or nonfinite per-fund returns exclude
automatic candidates instead of receiving a shared median return. Explicit custom
security choices retain their existing behavior. Bond sorting, requested position
counts, capital allocations, uniqueness and applicable position caps are retained.

These preferences are engineering policy weights, not agency ratings or measured
fund quality scores. The current reporting snapshot lacks expense ratio, fund
assets and liquidity fields; this change does not claim to assess those factors.
It does not underwrite an 8–12% future outcome. High required ETF counts can still
require lower-priority sector or thematic funds after broader exposures.

Source references for the breadth/concentration distinction:

- [Investor.gov: Asset Allocation and Diversification](https://www.investor.gov/introduction-investing/getting-started/asset-allocation): narrowly focused ETFs and overlapping funds do not necessarily diversify a portfolio.
- [Vanguard: VTI](https://advisors.vanguard.com/investments/products/vti/vanguard-total-stock-market-etf): broad U.S. equity-market exposure.
- [Invesco: QQQM](https://www.invesco.com/us/en/financial-products/etfs/invesco-nasdaq-100-etf.html): Nasdaq-100 growth exposure.
- [VanEck: SMH fact sheet](https://www.vaneck.com/us/en/investments/semiconductor-etf-smh-fact-sheet.pdf): semiconductor-sector exposure.

No-write generation using the actual stored snapshots reproduced every existing
automatically generated proposal's requested count and capital allocation under
the new policy. For Moderate C, 35 positions and the existing allocation produce
a 14-ETF sleeve averaging 9.1528357% in source-return terms and a portfolio estimate
of 8.1213867%, versus the earlier 16.66% ETF sleeve and 12.81% portfolio estimate.
Source returns were independently checked unchanged. The 13 risk-based proposal
previews span roughly 8.30–9.15% ETF source-return averages, including the 33-ETF
Aggressive C case at 8.74%. These are snapshot-based generation results, not
future-return assurances or manual UI/export acceptance.


The owner-authorized saved-proposal follow-up completed at 20:52:50 UTC /
14:52:50 Costa Rica on October 26: all 19 automatic records (13 risk-based and
6 planner) were regenerated with the new ETF preference. The three custom rows
remain exactly unchanged, including their update timestamps. All-row guards and
linked-profile locks preceded a single transaction. Independent post-commit
readback verified every replacement asset, original IDs/ownership/source/creation
metadata/starting amounts, required counts and weights totaling 100%. Risk-derived
replacements retain the 4.5% cap. No Gold was reintroduced. Before-images remain
in the operation tool record.

The owner subsequently requested commit and deployment. API source commit
`9d2d6cef869e89fb84d8887eb57a09392f25f3ba` is published on `main`;
production release verification is recorded below when complete. No automated application/browser/export tests
were run; the saved-data operation and real-snapshot generation checks do not
establish owner UI acceptance. GitHub issue posting remains blocked by the
earlier automatic approval rejection; no alternative posting route was used.


### ETF-selection production release — October 26, 2026

API commit `9d2d6cef869e89fb84d8887eb57a09392f25f3ba` is published on
`main` and deployed as `agm-api-01066-hxh`. Fresh Cloud Run readback at
21:08:45 UTC / 15:08:45 Costa Rica confirms this revision receives 100% of
production traffic, is the latest ready revision, and uses the verified image
`sha256:a65d6ddd8a3674d41008e00e75b21e1b714e90362d2d7dc459a514fe611c3ef2`.
The previous serving revision `agm-api-propedit261026-4f74edb3` and its tag
remain available for rollback. Existing risk-profile tags were preserved.

Release evidence:

- [Compile Check 40400235194](https://github.com/agmtrader/agm-api/actions/runs/40400235194): success, including required repository checks.
- [Release workflow 40400671229](https://github.com/agmtrader/agm-api/actions/runs/40400671229): success.
- [Cloud Build da542f3e-afb4-4869-afc6-9688b971a1e0](https://console.cloud.google.com/cloud-build/builds/da542f3e-afb4-4869-afc6-9688b971a1e0?project=571127175324): success for the exact commit image.
- Exact-image schema preflight `agm-api-schema-tmp-ldwnual2yr-6ds68` passed at 21:06:00 UTC in 23.621 seconds and exited zero. The pipeline cleaned up the temporary job; the success receipt remains in Cloud Logging.
- The candidate tag `etf-quality-20261026` activated the new revision at zero production traffic. Revision readiness and candidate HTTP 200 were verified before the explicit promotion to 100%.
- The main production API URL independently returned HTTP 200 after promotion.

No frontend release was required. No additional saved-data rewrite or broker/ETL
execution was initiated during this release; the existing 19 automatic proposal
replacements remain the prior verified operation. Production image identity,
startup and routing are verified; authenticated proposal creation and manual
UI/export acceptance are not claimed by the HTTP health checks.


## Twenty-asset count release — October 26, 2026

The owner authorized commit, push and deployment of the final count correction.
Application commits are API `b716d8c`, Dashboard `4707a59` and Hub `cea2c53` on
`main`; count-control documentation is `a81a86e` on the existing `small` branch.
All three Compile Check and Trigger Cloud Build workflows succeeded. Cloud Build
receipts: API `0431aed5-3c35-4331-b5e0-7357dbca141f`, Dashboard
`ea7776b3-aa9a-474c-a414-446bc6c6d693`, Hub
`94b42d70-3a17-446d-8181-8fce79dffbf3`.

The exact API candidate image passed its required read-only schema preflight
(`agm-api-schema-tmp-gvhb78sjkm-w96kf`, 23.391 seconds); the temporary job was
removed by the release script. API and Hub candidate roots returned HTTP 200
before their explicit traffic cutovers; Dashboard's production sign-in route
returned HTTP 200. Fresh Cloud Run status at 22:32:42 UTC / 16:32:42 Costa Rica
confirmed 100% traffic on API `agm-api-01067-67c`, Dashboard
`agm-dashboard-00683-kwv` and Hub `agm-hub-00819-p84`. Existing tagged revisions
were retained for rollback. This supersedes the earlier local-only release state.

No saved proposals were regenerated by this release. No additional local
application/browser/export tests were run; the normal pushed CI checks and
release startup/schema checks passed. The owner retains manual acceptance of
new risk proposals containing 20 assets and Planner explicit/cleared counts,
profile/preset changes, and preview/save agreement. API #304 and Dashboard #41
remain open for their recorded acceptance and broader scope.
