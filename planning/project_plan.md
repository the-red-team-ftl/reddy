# Project Plan

Pod Members: **Anny Dang, Eric Wong, Muhammed Enes Bilek**

> **Note (updated 2026-07-09):** The stack, data model, and endpoints below were revised after this deliverable was first submitted to match the current [Notion source of truth](https://app.notion.com/p/38e0ffa387f680b49700cd7b77050810) ([Data Model v0](https://app.notion.com/p/3960ffa387f681a49b74e15a0864006c), [API Contracts v0](https://app.notion.com/p/3970ffa387f681cbbac3dee75d04aaad)). Key changes: Supabase (managed Postgres) + Supabase Auth replace self-hosted Postgres and Better Auth; the Node/Express tier owns the REST API (PERN web tier) rather than being a thin proxy to FastAPI; `pgvector` self-learning is deferred to v1.

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

## Pages/Screens

1. **Landing and Authentication Page**: Marketing copy, user sign-up, and sign-in via Supabase Auth.
2. **Dashboard Home**: Greeting, Website title, last risk score, past tests dropdown, and a button to initiate a new test.
3. **Pentest Sequence Modal/Page**: Form to input a target URL, instructions for verification (DNS TXT), and policy configuration.
4. **Active Pentest Traces Page ("Camera")**: Real-time view streaming data from agents to the frontend using SSE. Includes a kill switch button.
5. **Expert Report Page**: The final deliverable showing conclusions from the Swarm Leader, validated exploits, risk scores, step-by-step reproduction, and markdown/PDF export buttons.
6. **User Profile Page**: Interface to manage account details and subscription.

## Data Model

Matches [Data Model v0](https://app.notion.com/p/3960ffa387f681a49b74e15a0864006c) (Supabase Postgres). Field names/types are kept 1:1 with that page.

* **users**: `id` (uuid PK), `email` (text), `auth_provider` (text), `created_at` (timestamptz)
* **targets**: `id` (uuid PK), `user_id` (uuid FK → users), `base_url` (text), `label` (text), `verification_token` (text), `verified_at` (timestamptz, null = not yet verified). Verification is DNS TXT for v0.
* **tests**: `id` (uuid PK), `target_id` (uuid FK → targets), `status` (enum: running/completed/halted), `policy` (jsonb — rate ceiling, destructive toggle, max duration), `risk_score` (int, null until the run finishes), `started_at` (timestamptz), `ended_at` (timestamptz)
* **findings**: `id` (uuid PK), `test_id` (uuid FK → tests), `severity` (enum: info/low/medium/high/critical), `title` (text), `description` (text), `reproduction_steps` (text), `remediation` (text), `evidence` (jsonb — payload + request/response that proved it)
* **traces**: `id` (uuid PK), `test_id` (uuid FK → tests), `agent` (text), `seq` (bigint, monotonic per test for ordered replay/reconnect), `event_type` (enum: thought/tool_call/result/breach/error), `summary` (text), `detail` (jsonb)

*Deferred to later versions (additive): `category` on findings (until the 2–3 v0 attack vectors are picked), `pgvector` embeddings on traces for self-learning (v1), attack-chain tables (v1).*

## Endpoints

Matches [API Contracts v0](https://app.notion.com/p/3970ffa387f681cbbac3dee75d04aaad). These are the Node/Express web-tier endpoints the React frontend calls. Auth is a Supabase session cookie; the server derives `user_id` from the session and never accepts it in the body. Base path is `/v1`.

* `GET /auth/session` : Return the current user, or 401 if no session.
* `POST /auth/signout` : Clear the session cookie (204).
* `POST /targets` : Create a new (unverified) target; response includes the DNS TXT value to publish.
* `GET /targets`, `GET /targets/{id}` : List / fetch the caller's targets (owner-scoped by session).
* `POST /targets/{id}/verify` : Synchronously perform a DNS TXT lookup and, if the token matches, set `verified_at`. Idempotent.
* `POST /tests` : Start a new run (requires a verified target); accepts `policy` (rate ceiling, destructive toggle, max duration).
* `GET /tests/{id}`, `GET /tests?target_id={id}` : Fetch a run / list runs for a target (powers the "past tests" dropdown).
* `POST /tests/{id}/halt` : Kill switch — request a graceful halt (idempotent, 202).
* `GET /tests/{id}/traces/stream` : SSE endpoint relaying live agent traces; supports `Last-Event-ID` for reconnect.
* `GET /tests/{id}/traces` : Paged historical trace view for the results page.
* `GET /tests/{id}/findings`, `GET /findings/{id}` : Read validated findings (no mutation endpoints in v0).
* `GET /tests/{id}/export?format=markdown|pdf` : Single-call report bundle for the "Export Report" button.
