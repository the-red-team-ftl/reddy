# Project Plan

Pod Members: **Anny Dang, Eric Wong, Muhammed Enes Bilek**

## Problem Statement and Description

**Problem:** Software is shipped faster than ever. Solo founders can launch a web app in a weekend, but security tooling remains priced and paced for enterprises. Founders must choose between shallow $200/month scanners that flag false positives, spending 300 hours learning to pentest themselves, or hiring a human red team for $20k. There is a gap for founders who need rigorous, exploit-validated security testing at an affordable price.

**Description:** Reddy is an autonomous, agentic red-team swarm for sandboxed web-app security testing. The app uses a Leader agent (Claude Opus) that coordinates a swarm of specialized attack agents (Claude Sonnet). These agents drive real, open-source pentesting tools via a containerized Model Context Protocol (MCP) layer against a user-authorized URL. The product prioritizes validated exploits over noisy warnings. Findings only make the final report if the agent successfully exploited them, providing founders with actionable remediation steps.

## User Roles and Personas

### Role: startup_founder
* **Lisa**: A 29-year-old founder with a business background who just raised a seed round. She relies on AI-generated code to speed up development. She does not understand technical security jargon and needs plain text reports detailing business risks and specific remediation steps.
* **Marcus**: A 35-year-old solo founder who is pre-revenue. He cannot afford an external cybersecurity firm. His motivation is getting a reliable baseline security check to avoid launching with critical security flaws.

### Role: serial_product_developer
* **Arman**: A 24-year-old indie developer who ships 3 to 4 micro-SaaS products a year. He is technical but not a security expert. His motivation is catching basic security misconfigurations across multiple projects before deployment.
* **Sarah**: A 34-year-old lead developer at a digital studio that builds web apps for clients. Her motivation is proving to clients that the delivered software is secure. She needs a tool that can run a test in a few hours and export a professional report for client deliverables.

## User Stories

1. **As a startup_founder**, I want to authorize my web application for testing, so that the platform only scans infrastructure I legally own.
2. **As a startup_founder**, I want to set strict limits on the test's aggressiveness, so that my app remains online and stable for my customers during the scan.
3. **As a startup_founder**, I want to view a real-time dashboard of the test's progress, so that I know what surfaces of my app are currently being evaluated.
4. **As a startup_founder**, I want to be able to immediately halt an ongoing test, so that I can intervene if my servers start struggling under load.
5. **As a startup_founder**, I want to export the final vulnerability report as a PDF, so that I can share it with investors or clients.
6. **As a serial_product_developer**, I want the final report to only include validated findings, so that I do not waste time chasing false positives.
7. **As a serial_product_developer**, I want step-by-step reproduction instructions for each finding, so that I can recreate the vulnerability locally and verify my patch.
8. **As a serial_product_developer**, I want specific remediation guidance for each vulnerability, so that I know exactly how to fix the underlying configuration or code issue.
9. **As a serial_product_developer**, I want to see a visual timeline of how the system was breached, so that I can understand how different minor vulnerabilities were chained together to create an exploit.
10. **As a startup_founder**, I want a synthesized text summary of all the raw security findings, so that I can quickly understand my overall risk level without having to interpret technical logs.

## Wireframes

Our wireframes live in Figma and cover the six key screens of the application:

[View wireframes in Figma](https://www.figma.com/design/nPVEJcPSU6rNCGubk8LJ0U/reddy_for_life?node-id=0-1&t=Kzd4bvnB5SgElQ8U-1)

**Screen inventory (3+ screens, component hierarchy noted):**

1. **Landing and Authentication Page** -- Marketing copy, social proof, and OAuth login buttons. Components: HeroSection, FeatureGrid, AuthButtons (GitHub OAuth via Supabase Auth).

2. **Dashboard Home** -- Shows the user's registered targets with last risk score, past tests as a dropdown, and a button to start a new test. Components: TargetCard (label, base_url, last score, verified status), TestHistoryDropdown, NewTestButton.

3. **Pentest Setup Modal** -- Form to register a new target URL, DNS TXT verification instructions with the generated token, and policy configuration (rate ceiling, destructive toggle, max duration). Components: TargetForm, VerificationInstructions, PolicyConfig, StartTestButton.

4. **Active Test / Traces Page ("Camera")** -- Real-time SSE stream of agent actions as they happen. Each trace shows agent name, event type, summary, and timestamp. A prominent kill switch halts the test. Components: TraceStream (scrolling list of TraceEvent items), AgentGraph (visual of which agents are active), KillSwitchButton, TestStatusBadge.

5. **Report Page** -- The final deliverable. Shows overall risk score, validated findings sorted by severity, each with title, description, reproduction steps, remediation, and evidence. Export buttons for markdown and PDF. Components: RiskScoreHeader, FindingsList > FindingCard (severity badge, title, expandable detail), ExportButtons.

6. **User Profile** -- Account details, connected OAuth providers, list of all targets. Components: ProfileHeader, ConnectedProviders, TargetList.

## Data Model

Matches [Data Model v0 on Notion](https://app.notion.com/p/3960ffa387f681a49b74e15a0864006c). Schema is defined as SQL migrations in the `reddy-backend` repo (`supabase/migrations/`). Prisma introspects the live DB for the type-safe query client but never owns the schema.

### users

| Column | Type | Description |
|--------|------|-------------|
| id | uuid PK | References auth.users(id), auto-created on signup via trigger |
| email | text | User's email from OAuth provider |
| auth_provider | text | e.g. 'github', 'google' |
| created_at | timestamptz | Row creation time |

### targets

| Column | Type | Description |
|--------|------|-------------|
| id | uuid PK | Auto-generated |
| user_id | uuid FK -> users | Owner of this target |
| base_url | text, NOT NULL | Canonical URL under test. UNIQUE per user. |
| label | text | User-facing display name |
| verification_token | text, NOT NULL, UNIQUE | DNS TXT value user must publish to prove ownership |
| verified_at | timestamptz | NULL means not yet verified |
| created_at | timestamptz | Row creation time |

### tests

| Column | Type | Description |
|--------|------|-------------|
| id | uuid PK | Auto-generated |
| target_id | uuid FK -> targets | Which domain this run is against |
| status | enum (queued, running, completed, halted) | Lifecycle state. Defaults to 'queued'. |
| policy | jsonb, NOT NULL | Rate ceiling, destructive flag, max duration |
| risk_score | int, CHECK 0-100 | NULL until status = completed |
| started_at | timestamptz | When the test was created |
| ended_at | timestamptz | NULL while queued/running, required when completed/halted |

### findings

| Column | Type | Description |
|--------|------|-------------|
| id | uuid PK | Auto-generated |
| test_id | uuid FK -> tests | Which run produced this finding |
| severity | enum (info, low, medium, high, critical) | CVSS-aligned rating |
| title | text, NOT NULL | One-line finding name |
| description | text | Founder-readable explanation |
| reproduction_steps | text | How to reproduce the vulnerability |
| remediation | text | How to fix it |
| evidence | jsonb | Payload, request/response proving exploitation |
| created_at | timestamptz | When the finding was written |

### traces

| Column | Type | Description |
|--------|------|-------------|
| id | uuid PK | Auto-generated |
| test_id | uuid FK -> tests | Which run this trace belongs to |
| agent | text, NOT NULL | Agent name that emitted the trace |
| seq | bigint, NOT NULL | Monotonic per test for ordered replay and SSE reconnect |
| event_type | enum (thought, tool_call, result, breach, error) | What kind of agent event |
| summary | text | One-liner for the dashboard |
| detail | jsonb | Full payload |
| created_at | timestamptz | When the trace was written |

**Security model:** Row Level Security is enabled on all tables. Users can only see their own data. The agent swarm uses a restricted DB role (`reddy_agent`) that can only read verified targets, read tests, write traces/findings, and update test status. It cannot access users, cannot delete anything, cannot bypass RLS, and cannot read verification tokens.

## Endpoints

All endpoints are served by the Node/Express web tier (`reddy-backend`). Auth is via Supabase session cookie (httpOnly, PKCE flow). The server derives user_id from the session and never accepts it in the request body. Resource routes are prefixed with `/v1` and require authentication.

### Auth (no prefix, no auth required except where noted)

| Method | Path | Description | Request | Response | Errors |
|--------|------|-------------|---------|----------|--------|
| GET | /auth/callback | OAuth code exchange (Supabase redirects here) | query: code, next | 303 redirect to frontend | Redirect to /auth/error on failure |
| GET | /auth/session | Return current user (requires auth) | -- | `{ user: { id, email, auth_provider } }` | 401 if no session |
| POST | /auth/signout | Clear session (requires auth) | -- | 204 | 401 if no session |

### Targets

| Method | Path | Description | Request | Response | Errors | Stories |
|--------|------|-------------|---------|----------|--------|---------|
| POST | /v1/targets | Create a new unverified target | `{ base_url, label? }` | `{ id, base_url, label, verification_token, verified_at: null, created_at }` | 409 duplicate base_url, 401 | 1 |
| GET | /v1/targets | List caller's targets | -- | `[{ id, base_url, label, verified_at, created_at }]` | 401 | 1 |
| GET | /v1/targets/:id | Fetch single target | -- | `{ id, base_url, label, verification_token, verified_at, created_at }` | 404, 401 | 1 |
| POST | /v1/targets/:id/verify | DNS TXT lookup; sets verified_at if token matches | -- | `{ id, verified_at }` | 404, 422 token mismatch, 401 | 1 |
| DELETE | /v1/targets/:id | Remove a target | -- | 204 | 404, 401 | -- |

### Tests

| Method | Path | Description | Request | Response | Errors | Stories |
|--------|------|-------------|---------|----------|--------|---------|
| POST | /v1/tests | Start a new run (target must be verified) | `{ target_id, policy }` | `{ id, target_id, status: 'queued', policy, started_at }` | 422 unverified target, 404, 401 | 2 |
| GET | /v1/tests/:id | Fetch a run | -- | Full test object | 404, 401 | 3 |
| GET | /v1/tests | List runs (filterable by target_id) | query: target_id? | `[{ id, target_id, status, risk_score, started_at, ended_at }]` | 401 | 3 |
| POST | /v1/tests/:id/halt | Kill switch (idempotent) | -- | 202 `{ status: 'halted' }` | 404, 401 | 4 |

### Traces

| Method | Path | Description | Request | Response | Errors | Stories |
|--------|------|-------------|---------|----------|--------|---------|
| GET | /v1/tests/:id/traces/stream | SSE live trace stream; supports Last-Event-ID | -- | text/event-stream | 404, 401 | 3, 9 |
| GET | /v1/tests/:id/traces | Paged historical traces | query: cursor?, limit? | `{ traces: [...], next_cursor }` | 404, 401 | 9 |

### Findings

| Method | Path | Description | Request | Response | Errors | Stories |
|--------|------|-------------|---------|----------|--------|---------|
| GET | /v1/tests/:id/findings | List findings for a test | -- | `[{ id, severity, title, description, created_at }]` | 404, 401 | 6, 7, 8 |
| GET | /v1/findings/:id | Fetch single finding with full detail | -- | Full finding object (includes evidence, reproduction_steps, remediation) | 404, 401 | 7, 8 |

### Export

| Method | Path | Description | Request | Response | Errors | Stories |
|--------|------|-------------|---------|----------|--------|---------|
| GET | /v1/tests/:id/export | Download report | query: format (markdown or pdf) | File download (appropriate Content-Type) | 404, 401 | 5 |

## State Architecture

| State Variable | Type | Initial Value | Owner | What Triggers Updates |
|----------------|------|---------------|-------|----------------------|
| currentUser | object or null | null | App (context) | Successful OAuth callback, signout, page load check via GET /auth/session |
| targets | array | [] | App (context) | Fetch on dashboard load, after target creation or deletion |
| selectedTarget | object or null | null | Dashboard | User clicks a target card |
| activeTest | object or null | null | App (context) | After POST /tests, updated by SSE status events and polling |
| testHistory | array | [] | Dashboard | Fetch when a target is selected (GET /tests?target_id=X) |
| traces | array | [] | TracesPage | SSE stream appends in real-time; historical fetch on page load |
| findings | array | [] | ReportPage | Fetch when test completes (GET /tests/:id/findings) |
| isLoading | boolean | false | App (context) | API call start/end |
| error | object or null | null | App (context) | API errors; cleared on next successful request |
| testPolicy | object | `{ rate_limit: 10, destructive: false, max_duration: 3600 }` | PentestSetup | User adjusts policy sliders/toggles before starting a test |
| sseConnected | boolean | false | TracesPage | SSE connection open/close events |

**Key decisions:**
- Authentication state lives in a top-level React context so any component can check login status and redirect.
- Real-time trace data arrives via SSE (Server-Sent Events) on GET /v1/tests/:id/traces/stream. The frontend holds an EventSource connection that appends to the traces array. On disconnect, it reconnects with Last-Event-ID for seamless resume.
- Test status updates come from the SSE stream (a 'status' event type) so the UI reacts immediately when a test completes or is halted.
- State does not persist across page reloads. The session cookie handles auth persistence; all other state is re-fetched from the API on mount.

## AI Feature Specification

**What it does for the user:** After a pentest run completes, the system generates a plain-language executive summary that explains the user's overall security posture, highlights the most critical risks in business terms, and provides a prioritized action plan -- all without requiring the user to read individual findings or understand technical jargon.

**Where in the app it lives:** The Report Page, rendered as the top section above the detailed findings list. Triggered automatically when a test transitions to "completed" status.

**Input:** All findings from the completed test (severity, title, description, reproduction steps, remediation) plus the target's base_url and the test's risk_score. Sent to Claude as a structured prompt on the backend.

**Output shape:**
```json
{
  "executive_summary": "2-3 paragraph plain-language overview",
  "risk_level": "critical | high | moderate | low | clean",
  "top_priorities": [
    { "finding_id": "uuid", "action": "one-sentence what to do first" }
  ],
  "business_impact": "one paragraph explaining what these vulnerabilities mean for the business"
}
```

**Validation criteria:** A good response (1) references only findings that actually exist in the test results, (2) uses language a non-technical founder can understand, (3) correctly ranks priorities by severity, and (4) does not hallucinate findings or remediation steps that were not produced by the agents. We validate by checking that every finding_id in top_priorities exists in the test's findings table.

**Endpoint:** `GET /v1/tests/:id/export` with format=summary (returns the AI-generated summary JSON), or embedded in the PDF/markdown export as the opening section. The backend calls Claude with the findings as context and caches the result so repeated requests do not re-generate.

**Fallback:** If the AI call fails or returns an invalid response (missing fields, hallucinated finding IDs), the Report Page displays the findings list without the executive summary section and shows a notice: "Summary generation failed. Your detailed findings are available below." The user can retry via a "Regenerate Summary" button.

### AI Feature Decisions Log

| Decision | Sprint | What Changed | Why |
|----------|--------|--------------|-----|
| Placed AI summary generation on the backend, not frontend | Sprint 1 | Architecture | API key security; backend already has DB access to findings. Matches the pattern from Week 4 capstone. |
| Cache AI summaries in a `test_summaries` column or table | Sprint 1 | Data model | Avoid re-calling Claude on every page load. Summaries are deterministic for a given set of findings. |

## Decisions Log

| Decision | Context | Alternatives Considered | Tradeoffs |
|----------|---------|------------------------|-----------|
| Use Supabase Auth instead of Better Auth | Needed auth with automatic Row Level Security integration. Supabase Auth gives us RLS for free since user IDs flow into policies. | Better Auth (more control, self-hosted), Auth0 (enterprise-grade but expensive), custom JWT | More coupled to Supabase. Acceptable for v0 since we are already all-in on Supabase Postgres. |
| Node/Express owns the REST API (PERN web tier) | The frontend needs a BFF (backend-for-frontend) that handles session cookies, calls Supabase with the service role, and relays SSE. Python agents are a separate tier. | FastAPI as the single backend, Next.js API routes | Extra tier to maintain, but clear separation: web team writes JS, agent team writes Python. Both share one DB. |
| Prisma as query-only client (never prisma migrate) | Our schema carries RLS policies, the reddy_agent role, and the signup trigger. Prisma cannot represent these. Running prisma migrate would silently drop them. | Let Prisma own the schema, use raw SQL alongside Prisma | Slightly more manual (write SQL migrations + run db pull), but we never risk losing security-critical DB objects. |
| Agents get a restricted DB role, not the service-role key | The Python agents run on EC2. If the instance is compromised and the credential leaks, the blast radius must be minimal. Service-role key bypasses all RLS. | Give agents the service-role key and enforce scoping in application code | More setup (dedicated role, column-level grants, RLS policies for the agent), but a leaked key only gives INSERT on traces/findings and SELECT on verified targets. |
| GitHub OAuth only for v0 (skip Google for now) | Target users are developers. GitHub is the lowest-friction option. Google OAuth adds complexity (consent screen review, different token shapes) with minimal uplift for our audience. | Both GitHub and Google from day one | Slightly limits reach (non-developer founders might prefer Google), but we can add Google in a day when needed. |
| DNS TXT verification for domain ownership | Proves the user controls the domain's DNS. Cannot be faked by someone who just has a page on the domain. | Hosted file verification (/.well-known/reddy-verify.txt), email verification | DNS TXT is harder for non-technical users but more secure. We will add clear step-by-step instructions and eventually support both methods. |
| SSE for live trace streaming (not WebSockets) | Unidirectional server-to-client flow. SSE is simpler, works through CDNs, supports automatic reconnect with Last-Event-ID, and does not require a separate protocol upgrade. | WebSocket, long polling | Cannot send client-to-server messages over SSE (not needed for traces). Slight limitations with HTTP/1.1 connection limits (6 per domain), but fine for our use case. |
| 'queued' status in test lifecycle | Tests should not show "running" before an agent picks them up. The server creates the row, agents poll for queued tests and transition them to running. | Start with 'running' immediately, let the agent update | More accurate UX; dashboard can show "waiting for agent" vs "actively testing". |
| Column-level grants on the API role | RLS scopes rows but not columns. Without column-level grants, a logged-in user could write to server-controlled columns (verified_at, risk_score) via PostgREST directly. | Rely only on RLS, disable PostgREST | Belt and suspenders. Even if someone bypasses our Express server and hits Supabase directly, they cannot forge verification or fake test results. |
