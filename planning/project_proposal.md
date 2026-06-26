# Project Proposal

Pod Members: **Anny Dang, Eric Wong, Muhammed Enes Bilek**

> See the [main README](../README.md) for the project overview, AI-powered feature summary, architecture stack, and the open-source libraries used. This proposal covers the problem framing, feature scope, competitive landscape, and open questions that aren't already in the README.

## Problem Statement

**Target audience:** solo founders and small startup teams who ship a live web app but cannot afford a real pentest.

The security market gives them three options, none of which fit:

| Option | Cost | Why it fails for founders |
|---|---|---|
| Free / cheap scanners (e.g. ~$200/mo signature scanners) | Low | Signature/pattern matching only. Flags potential CVEs but does not validate them, can't chain findings, drowns a non-expert in noise. |
| Learn to pentest yourself | 300–500 hrs | A founder shipping daily does not have those hours. |
| Hire a human pentest firm | ~$20k per webapp | Point-in-time snapshot; outdated by the time it lands. Out of reach for pre-revenue startups. |

Meanwhile real attackers are running 24/7 — adaptive, chaining exploits, and dynamic. The defender's tooling is static and periodic. We sit in the gap between "free scanner you can't trust" and "$20k engagement you can't afford": a robust, exploit-validating red-team in software, priced for a founder.

## Description

The product is a coordinated swarm of Claude agents (Leader + attack subagents) that drives the same open-source pentesting tools a human red team uses — Nmap, Nikto, sqlmap, Metasploit, plus a native HTTP proxy and headless browser — against a live, authorized URL. Every tool is exposed through its own containerized MCP server, so the agents call them like first-class tools without the orchestrator inheriting their attack surface.

**How a founder uses it:**

1. Sign up, add the domain they want tested.
2. Prove ownership (DNS TXT record or a hosted verification file) and set a policy (rate ceiling, destructive actions on/off, time window).
3. Launch a run. The Leader probes → plans → spawns parallel attack agents → reports.
4. Watch live agent traces in the dashboard, hit the kill switch any time, then read a founder-readable report at the end — validated findings only, with reproduction steps and concrete remediation.

The differentiator is **proof over noise**. A normal scanner flags "potential" issues and leaves the sorting to you. Our agents do the sorting themselves — when one suspects a vulnerability, it attempts the exploit; only confirmed breaches reach the report, each with the payload, response, and reproduction steps. A non-security-expert can act on the report directly, without triaging.

## Expected Features List

Grouped by the v1 product surface. Detailed mechanics live in the PRD; this is the feature inventory.

**Engagement & orchestration**
- Authorized run lifecycle: PROBE → PLAN → SWARM → REPORT, driven by the Leader agent.
    - PROBE: reconnaissance only — map URLs, forms, APIs, tech stack. No attacks.
    - PLAN: Leader reviews the probe and decides which subagents to spawn against which surfaces.
    - SWARM: parallel attack subagents execute, one per surface, with Leader spawning follow-ups when findings chain.
    - REPORT: aggregate validated exploits into the founder-readable report.
- Subagent coordination framework — Leader (Opus) dispatches one focused attack agent (Sonnet) per surface, can spawn follow-ups when one finding unlocks another, and aggregates outcomes. Design TBD between a custom FastAPI orchestrator vs. a thinner harness over the Claude Agent SDK; whatever we pick must make leader→subagent message passing, tool-call routing, and trace emission first-class.
- Live agent traces streamed to the dashboard (per-agent action log, tool calls, decisions, breach markers).
- Kill switch that halts an entire run within seconds.

**Target authorization & safety**
- Domain ownership verification via DNS TXT record and/or hosted verification file.
- Per-target policy: rate ceiling, destructive-action toggle, allowed time window.
- Sandbox "controlled doorway" — scope allow-list, rate limiting, egress control, destructive-action gating, full audit log of every request and tool call.

**Attack capabilities (elementary vectors covered in v1)**
- Surface discovery — forms, API routes, auth endpoints, file uploads, integrations.
- Exposed endpoint analysis — URL endpoints that leak data, unauthenticated routes, IDOR-style access-control gaps.
- Middleware / auth integrity — session handling, authz boundaries, CORS, header misconfigurations.
- Injection family — SQLi (via sqlmap MCP server), command/template/header injection probes.
- Network/service surface — Nmap MCP server for port/service discovery; Nikto for web-server-level misconfigurations.
- Exploitation chaining — Metasploit Framework MCP server for validated post-discovery exploitation.
- AI chatbot exploitation — prompt injection, jailbreaks, tool-abuse, and data exfiltration probes against any LLM/chat widget the agent finds on the target. This is a first-class vector, not an afterthought.
- Load / rate behavior under attack — controlled load testing against each identified vector to confirm exploitability (and to detect rate-limit gaps) without crossing into DoS.

**Tooling layer (MCP)**
- Tier 0 (native, ships first): HTTP proxy, headless browser — the two tools agents lean on most in black-box mode.
- Tier 1 (open source, in v1): Nmap, Nikto, sqlmap, Metasploit Framework — each as its own MCP server in its own container.
- Tier 2 (post-v1, BYOL): Burp Suite Pro, Nessus, Cobalt Strike — customer brings their own license; we orchestrate, never resell. Deferred from v1 entirely.

**Reporting**
- Founder-readable attack report: plain-language narrative, validated findings only, risk score per finding, step-by-step reproduction, concrete remediation guidance, breach timeline.
- Exports: PDF / HTML / JSON (sharing with investors, customers, auditors).

## Related Work

We're positioning between two existing tiers, both of which leave the founder underserved:

| Class | Examples | What they do well | Where they fail the founder |
|---|---|---|---|
| **Free / OSS scanners** | OWASP ZAP, Nikto standalone, free tiers of online scanners | Free, easy to run, surface known CVEs and misconfigurations | No validation — every finding is a "potential issue." Single-tool, no chaining. A non-expert can't tell which of the 200 alerts actually matter. |
| **Mid-tier SaaS scanners** | Detectify, Intruder.io, Probely (~$100–500/mo) | Polished, scheduled scans, decent UI | Still signature-based at the core. Can't chain low-severity findings into a real breach. Reports are noisy. No LLM/chatbot vector coverage. |
| **DAST + bug bounty hybrids** | HackerOne, Bugcrowd | Human-validated findings | Requires a public program, attracts a crowd of strangers, pays per finding — wrong shape for a pre-revenue founder. |
| **Human pentest firms** | Boutique consultancies | True exploit validation, exploit chaining, expert judgment | ~$20k+/engagement, point-in-time, weeks of lead time. Out of reach for solo founders. |
| **Emerging AI-pentest** | XBOW, RunSybil, early entrants | Closest peers; also using agents | Mostly enterprise-priced or research demos. We are explicitly built for the solo-founder price point, with founder-readable reports and exploit-only findings as a hard rule. |

**How we stand out:**
- **Exploit-validated findings only** — no "potential" alerts. If the agent didn't break in, it doesn't go in the report.
- **Leader + swarm**, not one agent with a checklist. The Leader chains exploits across surfaces the way a human red team does.
- **Real industry-standard tooling** (Nmap, sqlmap, Metasploit) driven by agents — not a reimplementation of those tools in LLM prompts.
- **AI chatbot vectors as a first-class attack surface** — most scanners don't test the LLM features founders are shipping in 2026.
- **Founder-priced and founder-readable.** Reports written for someone who is not a security expert.

## Open Questions

These are the items we still need to answer through research or vetting before/during build. Items 1–3 are the "how do these tools actually work + how do we give them to Claude" questions the team flagged.

1. **Is MCP-server-per-tool sufficient for every tool we want?** Lightweight tools (Nmap, Nikto, sqlmap) wrap cleanly as MCP servers exposing parameterized commands. The open question is **Metasploit** — it ships as `msfrpcd`, a long-running RPC daemon with stateful sessions. Open sub-questions: do we run one `msfrpcd` per run (ephemeral container, cold-start latency, simpler isolation) or pool warm instances (faster, harder to isolate between tenants)? How do we expose Metasploit's stateful session model through MCP's request/response shape? Is there an existing community MCP server for Metasploit, or do we build one?
2. **What's the baseline workflow each tool expects, and can an agent drive it?** Need a short "how this tool works" write-up per tool (Nmap, Nikto, sqlmap, Metasploit, headless browser) covering: inputs, outputs, failure modes, typical multi-step flows a human would run. This determines the MCP server's tool surface — too thin and the agent has to chain too many calls; too thick and we've hidden the capability the agent needs.
3. **Provisioning & licensing reality.** Tier 0 (our own) and Tier 1 (open source: NPSL / GPL / BSD — Nmap, Nikto, sqlmap, Metasploit Framework) are legally fine to bundle. Confirm: (a) license compatibility of bundling these into our deployment, (b) no liability shifts when we drive them against customer-authorized targets, (c) any GPL distribution implications if we ship MCP-server images publicly.
4. **Target verification UX.** DNS TXT record, hosted verification file, or both? Which is lowest friction for a non-technical founder while still being forgery-resistant?
5. **Destructive-action default.** Off by default with explicit opt-in (safest for non-experts) vs. on for fully owned-and-verified targets (more realistic attack simulation)?
6. **Live-trace transport.** SSE-through-BFF vs. direct WebSocket — confirm one single stream the frontend consumes, and that it survives long runs and reconnects cleanly.
7. **Pricing model.** Per-run, per-target subscription, or usage-based on agent compute? Has to land below the $200/mo shallow-scanner price floor to credibly serve the target audience.
8. **Findings ground truth & evaluation.** How do we measure the < 5% false-positive target during development — a benchmark suite of intentionally-vulnerable apps (DVWA, OWASP Juice Shop, etc.) with known-good answers we score against?
