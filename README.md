# SITE Capstone Project

SITE Course Year: **2026**

Cohort: **Salesforce FTL**

Team Member Names: **Anny Dang, Eric Wong, Muhammed Enes Bilek**

Mentors Names: **Adam Ambrose, Dallas Dias, Lakshmi Subramanian**

Project Code Repository Links

* Frontend Repo Link — _TBD (to be added once the repo is created)_
* Backend Repo Link — _TBD (to be added once the repo is created)_

## Project Overview

**Agentic Pen-testing — an autonomous red-team swarm for sandboxed web-app security testing.** *(Product name TBD.)*

Software is shipped faster and by fewer people than ever — a solo founder can launch a revenue-generating web app in a weekend — but security tooling is still priced and paced for enterprises. Today a founder's only options are shallow ~$200/month scanners, 300–500 hours of learning to pentest themselves, or a ~$20k human engagement that's outdated the moment it lands. Meanwhile real attackers probe, adapt, and chain exploits 24/7.

Our capstone closes that gap with a coordinated team of AI agents that behave like a real hacker crew rather than a single checklist scanner:

- **Leader agent (Claude Opus)** runs the engagement — plans, decides what to probe, interprets results, and chains findings across surfaces.
- **Swarm of attack agents (Claude Sonnet)** each work a single surface (login form, API route, chat/LLM widget) in parallel toward a concrete objective.
- **Real, industry-standard pentesting tools** exposed through a controlled tool layer, so agents drive the same utilities human red teams use.
- **Black-box by design** — agents only attack a live URL the user has authorized and verified, exactly as an outside attacker would. They never see source code.
- **Live traces + a validated report** — every agent action streams to a dashboard, and each run ends in a founder-readable report containing only findings backed by proof of exploitation, with risk scores and step-by-step reproduction.

The product's core differentiator is **proof over noise**: a finding is only reported if an agent actually exploited it in a controlled way.

### AI-Powered Feature

The AI *is* the product. The Leader agent autonomously plans a multi-step attack engagement (PROBE → PLAN → SWARM → REPORT), spawns and coordinates attack agents, calls external pentesting tools, and synthesizes results into a validated vulnerability report — all orchestrated server-side via Claude Opus and Claude Sonnet.

### Architecture

| Layer | Choice | Role |
|-------|--------|------|
| Frontend | React / Next.js (TypeScript) | Real-time dashboard, agent graph, live trace view |
| Edge / BFF | Node + Express (thin) | Serves the app, auth/session, proxies to Python, relays the live trace stream — **zero domain logic** |
| Core services | Python + FastAPI | Agent orchestration, tool calls, sandbox/policy, reporting — where the value lives |
| AI | Claude Opus (Leader) + Claude Sonnet (swarm) | Frontier agentic reasoning, planning, and tool-use |
| Tools | MCP server per tool (containerized) | Isolation + extensibility |
| Data | Postgres + pgvector | Relational data and the agents' learned-behavior vector store |

**Safety model:** every run is scoped to user-verified domains (DNS/hosted-file ownership proof), routed through a single controlled doorway that enforces a scope allow-list, rate limiting, egress control, destructive-action gating, a full audit log, and a kill switch.

Deployment Website: _TBD (not yet deployed)_

### Open-source libraries used

- [React](https://react.dev/) / [Next.js](https://nextjs.org/) — frontend dashboard and live trace view
- [Express](https://expressjs.com/) — thin Node BFF (auth, session, proxy, SSE relay)
- [FastAPI](https://fastapi.tiangolo.com/) — Python core services (agents, sandbox, reporting)
- [PostgreSQL](https://www.postgresql.org/) + [pgvector](https://github.com/pgvector/pgvector) — relational store and vector store
- [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) — tool-server layer
- [Nmap](https://nmap.org/) — network/port discovery (open source)
- [Nikto](https://github.com/sullo/nikto) — web server scanning (open source)
- [sqlmap](https://sqlmap.org/) — SQL injection testing (open source)
- [Metasploit Framework](https://github.com/rapid7/metasploit-framework) — exploitation framework (open source)
- [Laminar](https://github.com/lmnr-ai/lmnr) — agent trace observability _(under evaluation)_
