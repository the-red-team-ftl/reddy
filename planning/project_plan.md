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

## Pages/Screens

1. **Landing and Authentication Page**: Marketing copy, user sign-up, and sign-in via OAuth or Better Auth.
2. **Dashboard Home**: Greeting, Website title, last risk score, past tests dropdown, and a button to initiate a new test.
3. **Pentest Sequence Modal/Page**: Form to input a target URL, instructions for verification (DNS/file), and policy configuration.
4. **Active Pentest Traces Page ("Camera")**: Real-time view streaming data from agents to the frontend using Laminar Signals / SSE. Includes a kill switch button.
5. **Expert Report Page**: The final deliverable showing conclusions from the Swarm Leader, validated exploits, risk scores, step-by-step reproduction, and markdown/PDF export buttons.
6. **User Profile Page**: Interface to manage account details and subscription.

## Data Model

* **Users**: `id`, `email`, `auth_provider`, `created_at`
* **Targets**: `id`, `user_id`, `domain_url`, `verification_status`, `verification_method`, `dns_record_value`
* **Tests**: `id`, `target_id`, `status` (running/halted/completed), `policy_settings` (JSON), `start_time`, `end_time`, `risk_score`
* **Findings**: `id`, `test_id`, `severity`, `title`, `description`, `reproduction_steps`, `payload_data`, `remediation`
* **Agent Traces**: `id`, `test_id`, `agent_role`, `action_type`, `timestamp`, `log_content`, `embedding` (pgvector for self-learning/recall)

## Endpoints

*(Note: These represent the Node/Express BFF layer endpoints that the React frontend calls. The BFF proxies domain logic to the Python/FastAPI core).*

* `POST /api/auth/login` : Handle user authentication and session management.
* `GET /api/tests` : Fetch the user's past tests and overall risk score for the dashboard.
* `POST /api/targets/verify` : Trigger backend (dig/nslookup/DNS API) to check for domain ownership.
* `POST /api/tests/start` : Initiate a new pentest run and spawn the Python orchestrator.
* `POST /api/tests/:id/halt` : Trigger the kill switch to immediately terminate all agent processes.
* `GET /api/tests/:id/stream` : SSE endpoint relaying live agent traces from Python to the frontend.
* `GET /api/tests/:id/report` : Fetch the aggregated and validated vulnerability report.
