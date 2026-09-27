# CareerCompass AI

When the 08:00 schedule or a manual run starts, score eligible jobs and save strong matches, then send one daily digest.

## Production operations

**Purpose:** When the 08:00 schedule or a manual run starts, score eligible jobs and save strong matches, then send one daily digest.

- **Scope:** Configured Hacker News search/item API, Job Matches and Digest Runs tabs, OpenAI, personal inbox. No applications are submitted.
- **Success:** Job Matches upserted by HN Item ID; at most one Digest Runs marker per Manila date and a digest if matches exist.
- **Inputs:** JOBHUNT_PROFILE and URLs in environment; public HN jobs; OpenAI scoring.
- **Philippine use:** Provide a truthful Philippines-based profile. Reject roles whose location or remote rules exclude the candidate; preserve posted salary currency and never invent PHP conversion.

### Configure and run

1. Import this workflow and `Failure Alert.json` into n8n. Connect OpenAI, Google Sheets and Gmail credentials. For a webhook workflow, also connect a Header Auth credential and configure the caller to send it. Use a dedicated Google account with access only to the named spreadsheet and mail account.
2. Create the tabs and exact headers: Job Matches: Date, Company, Role, Location, Work Mode, Salary, Match Score, Match Reasons, Missing Skills, Email Subject, Cover Letter, Apply URL, Company URL, HN Item ID, Status. Digest Runs: Date, Run ID, Count, Status.
3. Set required environment values: `JOBHUNT_SHEET_ID, JOBHUNT_EMAIL_TO, JOBHUNT_PROFILE, JOBHUNT_SEARCH_URL, JOBHUNT_ITEM_API_BASE`. For CareerCompass, use `JOBHUNT_SEARCH_URL=https://hn.algolia.com/api/v1/search_by_date?query=%22Ask%20HN%3A%20Who%20is%20hiring%22&tags=story&hitsPerPage=30` and `JOBHUNT_ITEM_API_BASE=https://hacker-news.firebaseio.com/v0/item` as starting URLs. Set `CAREER_ALERT_EMAIL_TO` to the operator's real inbox.
4. Optional config: CAREER_MAX_JOBS=10; CAREER_MIN_SCORE=75. Set `CAREER_ENABLED=true` to enable runs. Dry run is on unless `CAREER_DRY_RUN=false` is explicitly set. The kill switch is `CAREER_ENABLED=false`; deactivate the workflow too for an immediate stop.
   On self-hosted n8n, set `N8N_BLOCK_ENV_ACCESS_IN_NODE=false` or Code nodes cannot read these values. Run these workflows on a dedicated instance; keep API secrets in n8n credentials and expose only workflow config through environment values.
   Set `N8N_CONCURRENCY_PRODUCTION_LIMIT=1` on that dedicated instance to serialize Sheet lookups and writes.
5. Open this workflow's n8n Settings and select the imported Failure Alert workflow as **Error Workflow**. Send its Gmail node from an account the operator monitors. Keep execution retention appropriate for sensitive data.
6. Run `python smoke_test.py` after every edit; GitHub Actions runs it on push and pull request. With dry run on, run the n8n workflow once and confirm no Google/AI/Gmail nodes executed. Then use a test sheet, test inbox, and representative input before setting `CAREER_DRY_RUN=false`.

Manual or daily 08:00 Asia/Manila. Owner: candidate. If the instance is offline, scheduled events are missed and webhook callers must retry with the same request ID. Manually replay missed scheduled runs after recovery. Review this workflow each quarter; retire it when its owner, input source, or business process no longer exists.

### Reliability, audit, and failure response

- **Reruns:** Job Matches upserts by HN Item ID. Digest Runs records Attempted before Gmail and blocks another digest for the same Manila date. Google Sheets lookup plus upsert is sequential idempotency; Sheets has no atomic uniqueness constraint, so concurrent runs can still duplicate rows. Keep one active run per workflow and reconcile after interruptions.
- **Partial failure:** An Attempted digest can represent failed delivery. Inspect Gmail and the execution, then clear that marker only after deciding a resend is needed. Never blindly retry a Gmail node after a timeout.
- **Timeouts/retries:** The workflow has a 900 second execution limit; HTTP calls have 15 second timeouts and three attempts with a two second wait. AI calls use a 30 second timeout. n8n's generic retry also retries some permanent HTTP errors, so disable that per node if your source returns persistent 4xx responses. Writes are not automatically retried because an ambiguous timeout can follow a successful write.
- **Audit:** n8n execution history and JSON console logs include timestamp, execution ID, and result without raw customer content. Sheet rows carry business keys and run IDs where available. Inspect failed executions and the `Failure Alert` email. The alert workflow depends on n8n being up; use an external uptime monitor for instance outages.
- **Security:** Store OAuth/API secrets in n8n credentials, not JSON or git. Protect webhook URLs with authenticated ingress and rate limits. Restrict who can view execution history, Sheets, and Gmail. Do not feed sensitive customer or financial data to OpenAI without your organization's approval and retention policy.
- **Exit status/health:** `python smoke_test.py` exits 0 on pass and nonzero on failure. In n8n, the execution status is the scheduler signal; monitor failed and missing scheduled executions. The latest timestamp in the relevant sheet is the simple status indicator.

### Go-live preflight

- [ ] Operator and failure inbox assigned; Error Workflow selected and alert tested.
- [ ] Dry run checked on test input; no live side effect occurred.
- [ ] Required variables and least-privilege credentials configured; no secrets in the export.
- [ ] Same request/lead/invoice rerun checked in a test sheet; no second mail or draft.
- [ ] Network timeout and failure alert checked; partial-write recovery rehearsed.
- [ ] `CAREER_ENABLED=false` demonstrated as the kill switch.

**Production gate:** Offline smoke tests do not prove n8n import compatibility or live Google/Gmail/OpenAI behavior. Complete the test-account run above and review the Sheets concurrency and ambiguous-email limits before activating on real data.

These exports target self-hosted n8n. n8n Cloud may block `$env` access inside Code nodes; verify compatibility before importing there.
