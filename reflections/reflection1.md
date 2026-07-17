# Reflection #1

Pod Members: **Anny Dang, Eric Wong, Muhammed Enes Bilek**

## Reflection Questions

* Name at least one successful thing this week.

  We came out of Sprint 1 with a real backend behind a real UI, plus one agent running end to end in isolation. Concretely:

  - The full `/v1` API is live: targets, tests, traces, findings, and export, behind Zod validation, rate limiting, a uniform error envelope with request IDs, real DNS TXT verification, and an SSE trace stream with `Last-Event-ID` resume backed by Postgres NOTIFY.
  - GitHub OAuth via Supabase runs the full round trip: frontend redirect, provider consent, session cookie set, protected routes gated. This unblocks every other flow.
  - The Recon and Exposure agent runs locally against Laminar for tracing and Daytona for sandbox isolation, and produces findings on a local target. Not yet plugged into the frontend, but the cognition layer is proven.
  - Security posture is holding: RLS on every table, column scoped grants for the API roles, a restricted `reddy_agent` DB role, and Supabase security advisors clean. Our adversarial review pass on the OAuth PR caught a Host header spoofing surface before it shipped.
  - The frontend has a real landing page (Hero, Our Agents, Why Reddy, Footer) on the design system, and the verify flow now talks to the real backend.

* What were some challenges you and/or your group faced this week?

  The biggest one was a mid sprint architecture pivot. We had been planning EC2 plus hexstrike as the tool layer, and on 2026-07-15 we moved to Daytona sandboxes with a Kali base image plus Reddy built domain specific MCPs, and forked `0xSteph/pentest-ai-agents` (MIT) as the cognition layer starting point. The direction is correct, but any Sprint 1 work that assumed the old topology had to be re-scoped, and the top level MVP Spec Sheet in Notion is still flagged as not current pending the update.

  Beyond that:

  - Attack vector selection is still open. The ProdSec sync gave us direction (OWASP, MITRE, layer thinking) but we have not locked which two vectors join Recon for v0, which delays Web Hunter and API Security spec work.
  - The cross tier handoff is undesigned. The web tier and the agent tier share Postgres, but the actual "start a test" mechanism (shared DB row, queue, internal HTTP) is TBD. That is why the agent runs locally and not through `/v1/tests` yet.
  - Coordination overhead on a small team. Three of us on three tiers meant a lot of contract writing (HANDOFF.md, PLAN.md, SETUP.md) to keep the interfaces honest. Worth it, but a real cost.

* Did you finish all of your tasks in your sprint plan for this week? If you did not finish all of the planned tasks, how would you prioritize the remaining tasks on your list?  (i.e over planned, did not know how to implement certain features, miscommunication from the team, had to pivot from original plans, etc.)

  Partial. Measured against the Sprint 1 milestone in the Revised MVP v0 spec ("Recon streams against juice shop and produces findings"), we hit the pieces individually but not composed.

  | Sprint 1 item | Status |
  |---|---|
  | Daytona images (agent + tools) with Kali base | Local, not deployed |
  | `recon-mcp` + `findings-mcp` | Local |
  | MCP client + one tool end to end path | Local |
  | Recon and Exposure agent | Working locally with Laminar + Daytona |
  | Postgres schema + traces writer | Done |
  | SSE endpoint with `Last-Event-ID` resume | Done |
  | DNS TXT verification | Done |
  | Milestone: Recon streams against juice shop end to end through the app | Not yet, agent is not wired to the backend worker |

  Contributing factors, ranked honestly:

  1. Spec churn. The mid sprint pivot to Daytona plus Kali plus the pentest-ai-agents fork changed the shape of the agent workstream after Sprint 1 was already partly under way.
  2. Interface work took real time. The `/v1` API, HANDOFF.md, and the OAuth wiring were prerequisites for everything else. They landed clean, but consumed more of the sprint than the plan assumed.
  3. We over indexed on breadth in a few places. Landing page polish and PDF export on the frontend are done ahead of dashboard states that actually gate the pentest flow.

  Priority order for what remains:

  1. Worker plus policy gate on the app plane, so `/v1/tests` actually spawns the agent sandbox and brokers tool calls through a safety envelope. This is the gate on almost everything else.
  2. Wire the existing Recon agent through that worker so the local juice shop demo runs end to end through the app.
  3. Lock the second and third vector agents (Web Hunter and one of API Security or AI Attack Surface) and spec them in Notion before writing code.
  4. Frontend dashboard states, past tests dropdown, results page, and risk score wired to the real endpoints.
  5. PDF export and any remaining polish, only after the above.

* Did the resources provided to you help prepare you in planning and executing your capstone project sprint this week? Be specific, what resources did you find particularly helpful or which tasks did you need more support on?

  The most useful resources this sprint were the ones we authored ourselves: the API Contracts v0 and Data Model v0 pages in Notion, the in repo `HANDOFF.md` (frontend integration contract), `BACKEND_DECISIONS.md`, and `PLAN.md`. When those existed, Claude produced code that matched intent on the first pass. When they did not, we caught the divergence in review instead of at design time.

  The ProdSec pentester sync was the highest signal external resource. It reframed the "which attack vectors" question from a menu pick into a layer choice (supply chain, runtime, governance), pointed us at the OWASP GenAI red teaming guide, the OWASP LLM Top 10, and MITRE ATT&CK's adversary emulation material, and set an honest bar on what automated tools consistently miss.

  Where we needed more support: cost feasibility for the agent tier (LLM plus sandbox spend per run) and the design of the cross tier handoff between the Express web tier and the Python agents. Both are still open items.

* Which features and user stories would you consider "at risk"? How will you change your plan if those items remain "at risk"?

  Ordered by risk:

  1. Full end to end pentest through the app. Recon works locally, but the worker that starts a test, spawns the agent sandbox, brokers tool calls through the policy gate, and writes findings back is not wired. Top priority for Sprint 2.
  2. The other two vector agents. Web Hunter and API Security are specced in the Revised MVP but not started, and which two we pick against the ProdSec input is still open.
  3. Policy gate. The safety envelope (scope, hard refusal, destructive gate, rate ceiling, duration) is designed but not implemented. Nothing reaches Kali without it, so this blocks the full flow.
  4. Test results page and dashboard states. The frontend has the shells, but the past tests dropdown, risk score display, and results page still need wiring to the real endpoints (which exist).
  5. PDF export. Reserved as `501` on the backend, and the frontend has a UI stub. Fine to defer, but call it out as v0.5.

  Adjustments if these remain at risk:

  - Cut Metasploit and non essential attack surface from v0. Already confirmed in the Revised MVP.
  - Freeze frontend visual polish until dashboard states and results page are wired to real data.
  - Lock the second and third agent picks in the first two days of Sprint 2 based on ProdSec input, then spec them in Notion before any code.
  - Land the worker plus policy gate as the first two Sprint 2 PRs. Everything else waits on those.
  - If the worker plus policy gate is not landed by mid Sprint 2, drop the fourth agent (AI Attack Surface) from v0 to protect the milestone.

## Sprint retrospective additions

The course page for Week 7 asks a few extra questions the template above does not cover. Adding them here so nothing gets missed.

* Did your team update `project_plan.md` before beginning each feature?

  Not consistently. Our specification surface this sprint was Notion (MVP Spec Sheet, Revised MVP v0 spec, Data Model v0, API Contracts v0, Design Foundations v0) and the in repo `HANDOFF.md`, `BACKEND_DECISIONS.md`, and `PLAN.md` on the backend. Those stayed current, and every meaningful commit references them. The `project_plan.md` in this repo lagged.

  For Sprint 2 we will treat `project_plan.md` the way CodePath describes it: pull the issue, touch the spec section, commit both. The Notion pages will keep being the authoritative product spec, but the in repo `project_plan.md` will mirror the section being built each sprint so the code review and the spec update sit in the same diff.

* Where did your implementation diverge from spec? Was the divergence intentional? Did you update the spec to reflect it, and commit the update?

  Two kinds of divergence, both intentional.

  - Backend hardening beyond the original API Contracts. The contract said "session cookie, standard errors." The implementation added a `request_id` on every error, a globally consistent error envelope, rate limiting on the DNS verify endpoint, and a `verification_token` echo on unverified GET responses so a user who closes the tab can resume. All are documented in `HANDOFF.md` in the backend repo and in the PR bodies.
  - Agent runtime. The whole agent workstream diverged from the original spec (EC2 plus hexstrike) to the revised one (Daytona plus Kali plus forked pentest-ai-agents). We captured this in the Revised MVP v0 spec in Notion on 2026-07-15 and cross referenced it from the MVP Spec Sheet, but the MVP Spec Sheet itself still needs a "current as of" refresh.

  Sprint 2 action: when the MVP Spec Sheet gets updated to reflect the Daytona and Kali direction, do it in the same commit cycle as the first agent tier PR, not as a separate doc pass.

* Where did Claude's output require significant revision? What was missing or misaligned with the spec at the time?

  Two clear categories.

  - Greptile catches on subtle correctness. The `/v1` API PR trail included fixes for a halt race, SSE cleanup guards, and a monotonic `seq` guard. On the OAuth PR, greptile flagged that a `NODE_ENV=staging` deploy fell through to `req.get('host')`, which would have let a spoofed Host header steer `redirectTo`. Fix landed in the same PR before merge. Two P2 items are still open on that PR (`skipBrowserRedirect: true` on `signInWithOAuth`, and now redundant `getClientUrl`/`getServerUrl` wrappers), tracked for the next backend PR.
  - Spec vagueness leading to Claude confidently doing the wrong thing. The clearest example was the DNS verification record name. The frontend told the user to publish TXT at `_reddy-verify.<host>`, but the backend queries the apex. Root cause was not a code bug, it was that the spec never said "apex, no subdomain prefix." Now stated in `HANDOFF.md` with a per case table.

  Pattern we noticed: Claude's output was strongest when we handed it a concrete spec section and reviewed against it, and weakest when we asked it to design and implement in the same turn. Sprint 2 goal is to keep those two moves separate.

* Did the Decisions Log get updated during Sprint 1? What decisions felt most worth recording?

  Notion carries most of them. The ones most worth capturing in the repo's Decisions Log:

  - Daytona replaces EC2, with two sandboxes per test (agent sandbox and tool sandbox). Rationale: blast radius and clean egress separation.
  - Kali as tool sandbox base image, wrapped by Reddy built domain specific MCP servers (`recon-mcp`, `web-mcp`, `api-mcp`, `ai-mcp`, `findings-mcp`). Rationale: we get the tool inventory for free without inheriting hexstrike's coupling.
  - Cognition layer forked from `0xSteph/pentest-ai-agents` (MIT) with heavy customization. Rationale: faster start than green field prompt authoring, license is compatible, and the shapes match our tier system.
  - Two pass summarizer with Haiku 4.5 for structured extraction and Sonnet 4.5 for narrative. Rationale: cost and determinism where it matters, quality where the user actually reads.
  - Blackboard and trigger driven coordination deferred to v1. v0 is parallel fan out. Rationale: keep the coordination problem out of the critical path.
  - `SERVER_URL` unconditionally required at boot, `CLIENT_URL` and `SERVER_URL` trailing slashes normalized. Rationale: closes a Host header spoofing surface flagged in adversarial review.
  - Prisma is a query client only, and SQL migrations own the schema. Rationale: RLS, roles, and triggers are outside Prisma's model, and `prisma migrate` would silently drop them.

  Sprint 2 will keep the log in this repo alongside the section it belongs to, updated in the same commit as the code.
