# 🎯 Hireko AI Recruiter Plugin

Welcome to the **Hireko AI Recruiter Plugin** for Claude! This plugin integrates your Hireko recruiter workspace directly into Claude, enabling recruiters, hiring managers, and talent acquisition teams to track candidate interviews, generate AI interview invitation links, review detailed candidate scorecards, list job descriptions, and search candidate resumes with natural language.

---

## 🚀 How the Application Actually Works

When you install and run the Hireko AI Recruiter Plugin in Claude Desktop:
1. **Local Secure Bridge**: Claude Desktop launches the Hireko MCP server as a local stdio process. It connects to Hireko cloud services without exposing passwords or tokens to chat context.
2. **Authentication Flow**: You connect your company workspace using one of the two sign-in options below.
3. **Instant Tool Activation**: Once signed in, Claude automatically receives full access to all 10 Recruiter Facade tools.
4. **Natural Language Execution**: You can chat with Claude in plain English (or use slash commands) to view candidate interview metrics, generate interview links, inspect candidate scorecards, and search talent.

---

## ⚡ Authentication Options: How to Sign In

You can authenticate your company's recruiter workspace in two ways. **Option 1 (Browser Device Login)** is the recommended primary method.

### 🌟 Option 1: Browser Device Login via Chrome (Recommended - Most Secure)

Instead of typing your sensitive password directly into the chat prompt box, Claude opens your default browser (Google Chrome) to sign in securely through AWS Cognito:

```text
/HirekoOpenPlugin [beta|prod] [tenant_name]
```
*(Stage defaults to `prod` if omitted, or specify `beta` for testing environments).*

*Examples:*
- `/HirekoOpenPlugin beta techforce`
- `/HirekoOpenPlugin techforce`

#### 🔄 Step-by-Step Execution:
1. **Enter Command**: In the Claude Desktop chat box, type `/HirekoOpenPlugin beta techforce` (or your company tenant name).
2. **Automatic Browser Redirect**: Claude immediately redirects and opens your default web browser (e.g., Google Chrome) to the official Hireko AWS Cognito sign-in page.
3. **Enter Credentials in Chrome**: On the login page, enter your **Username (Email)** and **Password**, then click **Sign In**.
4. **Instant Verification**: Once you click Sign In, the browser securely passes authentication tokens back to the local plugin bridge via a loopback callback.
5. **Ready to Work**: You can close Chrome and return to Claude! All 10 recruiter tools are instantly active and ready to use. Your credentials remain safe and are never stored or displayed in the chat.

---

### 💬 Option 2: Direct Sign-In in the Prompt Box (Username & Password)

If you prefer to sign in directly without opening a browser window, you can provide your credentials directly in the chat prompt box in a single message:

```text
/HirekoOpenPlugin [beta|prod] [tenant_name] [username/email] [password]
```

*Example:*
```text
/HirekoOpenPlugin beta techforce recruiter@techforce.com MySecretPassword123
```

#### 🔄 Step-by-Step Execution:
1. **Enter Command with Credentials**: Type the slash command along with your tenant name, username/email, and password.
2. **Direct Authentication**: Claude's local MCP process verifies your credentials with Hireko authentication services directly.
3. **Session Activation**: Claude confirms your signed-in tenant workspace. Your password is never echoed back or repeated in chat.

---

### 🗣️ Natural Conversational Sign-In

You can also simply prompt Claude in natural conversational English:
- *"Sign in to Hireko tenant techforce"*
- *"Check my Hireko login status"*
- *"Log out from Hireko"*

Claude will check your status and automatically trigger the Chrome browser login or guide you.

> **🔒 Note on Security**: Once signed in, credentials and tokens are stored securely in your local environment. Passwords and tokens are never exposed in chat.

---

### 🖥️ Interactive Web Console
A local visual dashboard is available while Claude is running:
- **URL**: [http://127.0.0.1:8400/human](http://127.0.0.1:8400/human) (or port 8401)
- View real-time recruiter tool execution, tenant quotas, candidate records, and active sessions.

---

## 🛠️ Recruiter Tools & Capabilities

The plugin equips Claude with 10 powerful recruiter tools:

| Tool Name | Core Capability | When to Use |
| :--- | :--- | :--- |
| **`get_interview_candidates_count`** | Count and analyze candidates who took or completed interviews. | Get fast metrics, pass rates, average scores, and status breakdowns by job or company. |
| **`list_interview_candidates`** | List candidate records with scores, durations, and summaries. | Review candidate interview results, status (`completed`, `passed`, `pending`), and evaluations. |
| **`create_interview_link`** | Generate candidate AI interview invitation links. | Send automated interview links with custom durations (e.g. 15 min), dates, and job roles. |
| **`list_job_descriptions`** | List active job requisitions and interview templates. | View open requisitions created in your recruiter dashboard. |
| **`get_assessment_details`** | Fetch in-depth candidate assessment scorecards. | Review question-by-question scoring, competency analysis, and hiring recommendations. |
| **`search_candidates_rag`** | AI semantic talent search across resumes & transcripts. | Find qualified candidates matching specific tech stacks, keywords, or job requirements. |
| **`get_search_score`** | Retrieve candidate match scores for specific job requisitions. | Rank and compare candidate fit against specific job postings. |
| **`get_tenant_entitlements`** | Check interview quotas and subscription balance. | Check remaining interview credits and active enterprise features. |
| **`verify_tenant_score`** | Verify tenant subscription and backend connectivity. | Ensure company tenant account is active and verified. |
| **`health`** | Check recruiter gateway connection health. | Verify end-to-end connectivity with Hireko cloud services. |

---

## 💬 Recruiter Prompts: What You Can Ask Claude

Recruiters can copy and paste any of these example prompts directly into their chat with Claude:

### 📊 1. Candidate Analytics & Interview Counts
- *"How many candidates took an interview for Full Stack Java Developer?"*
- *"Show me the candidate interview count and pass rate for our active jobs."*
- *"How many candidates completed interviews this month?"*
- *"Give me a breakdown of all completed vs pending interviews."*

### 📋 2. Candidate Listings & Scorecards
- *"List all candidates who completed interviews this week."*
- *"Show candidates who have an interview status of completed or passed."*
- *"List the top scoring candidates for the DevOps Engineer requisition."*
- *"Show me candidate interview scores and evaluation summaries."*

### 🔗 3. Generating AI Interview Links
- *"Create an interview link for a Senior Full Stack Java Developer for 15 minutes."*
- *"Generate an AI interview invitation link for candidate John Doe (john@example.com) for DevOps Engineer."*
- *"Create an interview link for React Frontend Engineer starting next Monday for 30 days."*

### 📂 4. Job Requisitions & Descriptions
- *"What active job requisitions do we have in Hireko?"*
- *"List all job descriptions created in the dashboard."*
- *"Show me the job requisition details for Full Stack Java Developer."*

### 🔍 5. In-Depth Candidate Assessments
- *"Get the detailed assessment evaluation for candidate."*
- *"Show me the question-by-question breakdown and hiring recommendation for the candidate."*

### 🧠 6. AI Semantic Candidate Search (RAG)
- *"Search candidates with hands-on experience in Kubernetes, Docker, and CI/CD pipelines."*
- *"Find candidates who have strong expertise in Java Spring Boot and Microservices."*
- *"Search resumes for candidates with experience leading DevOps teams."*

### 💳 7. Tenant Quota & Status
- *"What is our remaining interview quota?"*
- *"Check our tenant subscription tier and entitlements."*

---

## 🔒 Security & Privacy

- **Zero Credential Exposure**: Your credentials are never shared with Claude or included in prompt context.
- **Secure PKCE Authentication**: Handled directly through AWS Cognito in your local browser.
- **Granular Session Control**: Easily connect to your tenant workspace or sign out at any time.
"# codex-marketplace" 
