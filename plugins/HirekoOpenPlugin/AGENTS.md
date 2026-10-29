# HirekoOpenPlugin — agent notes

This plugin is a **local auth-injecting MCP router**. Plugin name: **HirekoOpenPlugin** (not `hireko`). The harness talks to one stdio MCP server; downstream tools are flattened onto `tools/list` as `server__tool`.

## Hard rules

1. **Never invent or pass `Authorization` headers, Bearer tokens, Cognito passwords, or API keys** to downstream tools. The proxy injects tenant auth on every outbound call.
2. Prefer **calling flattened tools directly** from `tools/list` when the name/description matches the task.
3. Use **`filter_tools_with_query`** only when the catalog is large or you need a ranked shortlist (deterministic text scoring — no LLM inside the proxy).
4. If unauthenticated, call **`login_browser`** or the `/HirekoOpenPlugin` slash skill. Do not ask the user to paste a password into chat.
5. To sign out, call MCP **`logout`** (optional `tenant`; defaults to active). Prefer this over inventing CLI steps when MCP is available.
6. After `/human` adds a server, call **`reload_registry`** so `tools/list_changed` refreshes the flattened surface.

## Routing

| Intent | Action |
|--------|--------|
| Status / “am I logged in?” / “hireko|recruiter status” | MCP `auth_status` |
| Browser / device login / “hireko|recruiter login” | `/HirekoOpenPlugin` → `login_browser` |
| Sign out / “hireko|recruiter logout” | MCP `logout` (optional `tenant`; default active) |
| Rank tools by English query | `filter_tools_with_query` |
| Create candidate interview link | Flattened `RecruiterFacade__create_interview_link` directly |
| Get job search / match score | Flattened `RecruiterFacade__get_search_score` directly |
| Recruiter Facade health check | Flattened `RecruiterFacade__health` directly |
| Call any business API | Flattened `server__tool` directly |

Only one tenant session may be active: if already logged in, logout before logging into a different tenant (same-tenant re-login is allowed).

## Recruiter Facade Tools & Workflows

The proxy auto-registers the downstream **Recruiter Facade** server (`RecruiterFacade`). Its tools are flattened onto `tools/list` as `RecruiterFacade__<tool>`:

### 1. `RecruiterFacade__create_interview_link`
Creates candidate interview links via Hireko Recruiter Facade API. Tenant auth is automatically injected.
- **Required parameters**:
  - `title` (string): Interview title (e.g. `"Senior Frontend Engineer - Screening"`)
  - `description` (string): Detailed interview description / expectations
  - `company_name` (string): Name of the hiring company
  - `location` (string): Job / interview location (e.g. `"Remote"`, `"Seattle, WA"`)
  - `start_date` (integer): Epoch timestamp in milliseconds (e.g. `1741737600000`)
  - `end_date` (integer): Epoch timestamp in milliseconds (e.g. `1744329600000`)
  - `interview_length` (string): Duration in minutes (e.g. `"30"`, `"45"`)
- **Optional parameters**:
  - `status` (string, default: `"active"`)
  - `employment_type` (string, default: `"Full Time"`)
  - `language` (string, default: `"en"`)
  - `stage` (string, default: `"prod"`)
  - `tenant` (string, default: active tenant)
  - `funnel` (string, default: `"AllFunnels"`)
  - `company_culture` (string): Company culture or candidate guidance notes

### 2. `RecruiterFacade__get_search_score`
Retrieves candidate match scores and search relevance for a specific job posting.
- **Required parameters**:
  - `job_id` (string): The ID of the job posting
- **Optional parameters**:
  - `stage` (string, default: `"prod"`)
  - `tenant` (string, default: active tenant)

### 3. `RecruiterFacade__list_interview_candidates`
Lists candidates for domain company TechForce or any subcompany (e.g. `arena.ai`), organized cleanly into completed and pending interview groups:
- **Optional parameters**:
  - `company` (string): Filter by company (`"TechForce"`, `"arena.ai"`, or `"all"`)
  - `status` (string, default: `"all"`): Filter by status (`"all"`, `"completed"`, `"pending"`)
  - `job_id` (string): Filter by job requisition ID

### 4. `RecruiterFacade__health`
Health check endpoint for the downstream Recruiter Facade service.

### Agent Workflow Guidelines
- **Candidate Listing (Completed & Not-Completed Sections)**: When a recruiter asks to view candidates or asks who has completed the interview:
  - Call `list_interview_candidates({ company: ... })`.
  - In your reply, ALWAYS present BOTH groups for recruiter clarity using the recommended presentation standard:
    1. **Candidates Who Have Completed the Interview (TABLE FORMAT)**: Present in a Markdown Table strictly with 6 columns: Name, Email, Interview date, Duration, Status, and Score (`| Name | Email | Interview date | Duration | Status | Score |`).
    2. **Candidates Who Have NOT Completed the Interview**: Present in a clean Markdown Table (or bullet list if large roster) with 6 columns: Name, Email, Interview date, Duration, Status, and Score (`| Name | Email | Interview date | Duration | Status | Score |`).
    - **CRITICAL HARD RULE — NO SOURCE COLUMN**: NEVER add a "Source" column, "Acquisition Channel" column, or "How they attended / took it" column in candidate tables or replies. Recruiters do not want a source column. Keep tables strictly to 6 columns: Name, Email, Interview date, Duration, Status, Score.
  - Even if the recruiter asks "who completed the interview", show who completed first, and immediately follow with who has not completed yet.
  - **High-Volume Scale Handling (100+ to 1,000 candidates)**: For high candidate counts, present an Executive Pipeline Summary card, a table of the Top 10 completed scorers, a grouped text breakdown of pending candidates, and offer the clean 2-tab spreadsheet export (`spreadsheet_export`).
- **Spreadsheet & CSV Exports (Clean Data Separation)**: When a recruiter asks to create a spreadsheet, export to Google Sheets/Excel, or generate a CSV table:
  - **For Completed Candidates Spreadsheet**: Call `list_interview_candidates({ company: ..., status: "completed" })` or use `spreadsheet_export.completed_sheet` / `completed_candidates`. Never include pending candidates in this spreadsheet; keeping it pure prevents zero-scores from corrupting averages and evaluation metrics.
  - **For Pending Candidates Follow-up Spreadsheet**: Call `list_interview_candidates({ company: ..., status: "pending" })` or use `spreadsheet_export.pending_sheet` / `not_completed_candidates`. This spreadsheet is dedicated for candidate outreach and sending reminder interview links.
  - **For Multi-Tab Workbooks (All Candidates)**: Structure the spreadsheet with TWO separate tabs/sheets:
    - Tab 1: **"Completed Interviews"** (from `spreadsheet_export.completed_sheet`)
    - Tab 2: **"Pending Invitations"** (from `spreadsheet_export.pending_sheet`)
    This cleanly separates completed scoring data from outreach tasks without confusion.
- **Creating Interview Links**: When a recruiter asks to create or schedule an interview link:
  1. Confirm authentication status using `auth_status` if needed.
  2. Parse the interview details (convert dates/times into epoch milliseconds for `start_date` and `end_date`).
  3. Call `RecruiterFacade__create_interview_link`.
  4. Present the resulting link clearly to the user.
- **Candidate Match / Search Scores**: When a recruiter asks for candidate scores or job matching scores:
  1. Identify the target `job_id`.
  2. Call `RecruiterFacade__get_search_score({ job_id })`.
  3. Summarize and format the scoring results for the user.

## Process model

MCP is **stdio**, owned by the harness (Claude, Cursor, Hermes, Codex, …). When the harness exits, the proxy exits. `up` only clears orphan detached processes.

**Hermes (incl. Amazon Linux Lightsail Docker):** `./install.sh --only=hermes` copies the package to `~/.hermes/plugins/<Name>`, registers MCP via `hermes mcp add` (absolute `node` path + `HIREKO_CREDENTIAL_STORE=file`), and installs the skill under `~/.hermes/skills/<Name>`.

- Credentials on Linux containers: `~/.<Name>/credentials/<tenant>.json` (no Keychain).
- Auth in Docker / Headless: Both headless options inject the exact same **credential triplet** `(tenant, username, password)`:
  - **Format 1 — Plain-env triplet (recommended for CI/Docker — no base64):**
    ```bash
    export HirekoOpenPlugin_tenant="testcompany"
    export HirekoOpenPlugin_username="recruiter@example.com"
    export HirekoOpenPlugin_password="mySecretPassword"
    ```
  - **Format 2 — Base64 JSON bundle (`HIREKO_INTERNAL_API_KEY` or CLI `login --apikey`):**
    Plain JSON triplet:
    ```json
    {"tenant": "testcompany", "username": "recruiter@example.com", "password": "mySecretPassword"}
    ```
    Base64 encoded:
    ```bash
    export HIREKO_INTERNAL_API_KEY="eyJ0ZW5hbnQiOiJ0ZXN0Y29tcGFueSIsInVzZXJuYW1lIjoicmVjcnVpdGVyQGV4YW1wbGUuY29tIiwicGFzc3dvcmQiOiJteVNlY3JldFBhc3N3b3JkIn0="
    ```
  - Priority: `HIREKO_INTERNAL_API_KEY` > plain-env triplet. The proxy auto-logs in at startup.
  - Device/`login_browser` posts to host loopback — use only with `ssh -L 87xx:127.0.0.1:87xx` from your laptop, or run Hermes on the same machine as the browser.
- Persist across container restarts: volume-mount `$HOME/.<Name>` (or set `HIREKO_HOME`).
- Hermes does **not** load Cursor-style `agents/` — skills + MCP only.
