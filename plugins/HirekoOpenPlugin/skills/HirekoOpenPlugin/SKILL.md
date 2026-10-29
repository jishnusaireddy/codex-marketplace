---
name: hirekoopenplugin
description: Sign in and manage your Hireko Recruiter workspace. Usage: /HirekoOpenPlugin [tenant-name]
argument-hint: "[tenant-name]"
user-invocable: true
disable-model-invocation: true
---

# HirekoOpenPlugin

The user invoked this slash command with free-form text after the command name. Do not ask questions. Do not list tenants. Do not invent URLs.

## Process model

The plugin MCP server is **stdio**, a child of the harness (Claude, Cursor, Hermes, …). When the harness quits, the OS stops that child. `up` only clears orphan detached proxies — it does not start a long-lived daemon.

## Parse args

1. **stage** — whole-word `beta` or `prod` (else omit; local installs → beta).
2. **localPort** — `local` + port / `local:8080` / `local=8080` (Vite). Not deploy stage `local`.
3. **tenant** — first remaining non-keyword word.
4. Optional trailing `username` `password` only for password-login fallback.

| User typed | Call |
|---|---|
| `test` | `{ tenant: "test" }` |
| `beta test` | `{ tenant: "test", stage: "beta" }` |
| `prod test` | `{ tenant: "test", stage: "prod" }` |
| `test local 8080` | `{ tenant: "test", localPort: 8080 }` |

**Always print the device-login URL** from tool/CLI output.

## Empty tenant / First-Time Login

1. Call `auth_status` once.
2. If authenticated: print active tenant status banner and tools count. Stop.
3. If unauthenticated: Print usage:
   > You are not signed in to Hireko yet.
   >
   > **Option 1 (Recommended - Browser Device Login):**
   > `/HirekoOpenPlugin beta [tenant_name]` (or `/HirekoOpenPlugin [tenant_name]`)
   > Claude will open Google Chrome to the secure AWS Cognito sign-in page where you enter your email and password without sharing them in chat.
   >
   > **Option 2 (Direct Prompt Box Login):**
   > `/HirekoOpenPlugin [beta|prod] [tenant_name] [username/email] [password]`
   >
   > Examples:
   > - `/HirekoOpenPlugin beta techforce`
   > - `/HirekoOpenPlugin techforce`
   Stop.

## Login Execution

1. Call `auth_status` once.
2. If already authenticated for that tenant, report active status and stop.
3. **Option 1: Browser Device Login (Preferred & Default when only tenant is supplied):**
   - Call **`login_browser` once** with `{ tenant, stage, localPort }`.
   - Claude triggers device login and automatically opens your default browser (Google Chrome) to the official Hireko sign-in page.
   - Always print the device-login URL in chat so the user can easily click or view it.
   - User enters their username/email and password in Chrome. The browser verifies credentials and hands tokens back to Claude via the local callback.
   - All 10 recruiter tools are immediately activated.
4. **Option 2: Direct Sign-In in Prompt Box (When username & password are provided):**
   - Call MCP tool `login` once with `{ tenant, username, password, stage }`.
   - Never echo the password back in chat.
   - Claude confirms active tenant status and lists active job requisitions.

5. After login, all 10 business tools are immediately active on `tools/list` — call them directly; authentication is injected automatically by the local proxy.

## Recruiter Facade Tools & Workflows

Once authenticated, Recruiter Facade tools are exposed directly as flattened tools:

### Available Tools

- **`RecruiterFacade__get_interview_candidates_count`** (or `get_interview_candidates_count`): Fetch or count candidates who took or completed an interview.
  - **Optional**: `job_id`, `job_title`, `status` (`completed`, `passed`, `in_progress`, `pending`, `failed`, `all`), `tenant`, `startDate`, `endDate`.
- **`RecruiterFacade__list_interview_candidates`** (or `list_interview_candidates`): List candidate records who took interviews with scores, interview dates, and evaluations.
  - **Optional**: `job_id`, `job_title`, `status`, `limit` (default: 20), `tenant`.
- **`RecruiterFacade__create_interview_link`** (or `create_interview_link`): Generate an interview link for candidates with automatic tenant job detection.
  - **Required**: `title`, `company_name`.
  - **Optional**: `job_id`, `description`, `location`, `start_date` (epoch ms), `end_date` (epoch ms), `interview_length` (minutes), `candidate_name`, `candidate_email`, `tenant` (e.g. `arena.ai` or `techforce`).
- **`RecruiterFacade__list_job_descriptions`** (or `list_job_descriptions`): List or search active job descriptions/requisitions created in the dashboard for a tenant.
  - **Optional**: `tenant` (defaults to active tenant, e.g. `arena.ai`), `title` (filter keyword), `limit`, `stage`.
- **`RecruiterFacade__get_assessment_details`** (or `get_assessment_details`): Retrieve comprehensive evaluation details, question-by-question breakdown, skill scores, and hiring recommendation for a candidate.
  - **Optional**: `candidate_id`, `email`, `job_id`, `tenant`.
- **`RecruiterFacade__get_tenant_entitlements`** (or `get_tenant_entitlements`): Check tenant subscription tier, allocated interview quotas, used interviews, and enabled enterprise features.
  - **Optional**: `tenant`.
- **`RecruiterFacade__search_candidates_rag`** (or `search_candidates_rag`): AI semantic candidate search across resume documents, interview transcripts, and evaluations using Hireko Universal RAG.
  - **Required**: `query` (search text).
  - **Optional**: `job_id`, `limit`, `tenant`.
- **`RecruiterFacade__get_search_score`** (or `get_search_score`): Retrieve candidate match / search score for a job posting.
  - **Required**: `job_id` (string).
  - **Optional**: `candidate_id`, `tenant`.
- **`RecruiterFacade__verify_tenant_score`** (or `verify_tenant_score`): Verify tenant score and quota status.
- **`RecruiterFacade__health`** (or `health`): Check health of the Recruiter Facade MCP endpoint.

### Common Workflows

1. **Counting or Listing Candidates Who Took Interviews**:
   - Call `RecruiterFacade__get_interview_candidates_count` (optionally filter by `job_id`, `job_title`, or `status`).
   - Call `RecruiterFacade__list_interview_candidates` to display candidate details, dates, scores, and status.
   - **MANDATORY TABLE FORMAT**:
     Always format candidate tables strictly with 6 columns, ending at Score:
     `| Name | Email | Interview date | Duration | Status | Score |`
     **CRITICAL**: NEVER add a 7th column such as "How they joined", "How they took it", "Source", or "Acquisition Channel". The table must stop strictly at Score.

2. **Creating an Interview Link**:
   - Collect or extract job/interview details: title, description, company, location, date range, duration.
   - Convert date range to epoch milliseconds (e.g. `Date.parse(...)`).
   - Call `RecruiterFacade__create_interview_link`.
   - Present the returned interview link and details to the user.

3. **Listing Job Descriptions**:
   - When asked for job descriptions for a tenant or company:
     Call `list_job_descriptions` with `{ tenant: "[tenant-name]" }`.
     Displays the active job requisitions created in the dashboard.
   - When asked generally "What are the job descriptions?":
     Call `list_job_descriptions({})` to present all active requisitions for the authenticated tenant workspace.

4. **Fetching Search Scores**:
   - Extract the `job_id`.
   - Call `RecruiterFacade__get_search_score({ job_id })`.
   - Display matching scores and relevant candidate ranking criteria.

Call `auth_status` once. Summarize authenticated/tenant/stage/`proxy.human`/`toolsIndexed`. Never invent URLs or echo tokens.

## Logout (user says hireko/recruiter logout / sign out / end session)

Call MCP `logout` once with `{}` or `{ tenant }` if specified. Prefer this over inventing CLI steps when MCP is available.

## CI / Headless mode (no browser, no prompt)

Headless authentication uses a **credential triplet**: `(tenant, username, password)`. Both headless formats supply this exact triplet to the unified authentication factory, automatically logging in at startup.

Priority: explicit tool args > `apiKey` > `HIREKO_INTERNAL_API_KEY` > plain-env triplet.

### Format 1 — Plain-env triplet (recommended for CI & Docker)

Separate environment variables prefixed with `HirekoOpenPlugin_`:

```bash
export HirekoOpenPlugin_tenant="mycompany"
export HirekoOpenPlugin_username="ci-user@example.com"
export HirekoOpenPlugin_password="secretPassword123"
```

The proxy auto-logs in at serve startup — no MCP `login` call needed.

### Format 2 — Bundled Base64 `HIREKO_INTERNAL_API_KEY`

A single environment variable containing the base64-encoded JSON representation of the exact same triplet:

1. **Plain JSON triplet:**
   ```json
   {
     "tenant": "mycompany",
     "username": "ci-user@example.com",
     "password": "secretPassword123"
   }
   ```

2. **Base64 encoded string:**
   ```bash
   export HIREKO_INTERNAL_API_KEY="eyJ0ZW5hbnQiOiJteWNvbXBhbnkiLCJ1c2VybmFtZSI6ImNpLXVzZXJAZXhhbXBsZS5jb20iLCJwYXNzd29yZCI6InNlY3JldFBhc3N3b3JkMTIzIn0="
   ```

3. **Or generated dynamically in bash:**
   ```bash
   export HIREKO_INTERNAL_API_KEY=$(echo -n '{"tenant":"mycompany","username":"ci-user@example.com","password":"secretPassword123"}' | base64)
   ```

### Docker / GitHub Actions example

```yaml
env:
  # Format 1 (Plain-env triplet):
  HirekoOpenPlugin_tenant: ${{ secrets.HIREKO_TENANT }}
  HirekoOpenPlugin_username: ${{ secrets.HIREKO_USERNAME }}
  HirekoOpenPlugin_password: ${{ secrets.HIREKO_PASSWORD }}

  # Or Format 2 (Base64 bundle):
  # HIREKO_INTERNAL_API_KEY: ${{ secrets.HIREKO_INTERNAL_API_KEY }}
```

The `auth_status` tool returns `headlessMode: true` when either source is detected, and a specific error message if credentials were set but login failed (e.g. wrong password).

Interactive users ignore these env vars entirely — they use Option 1 (browser device login) or Option 2 (direct login) as normal.
