# Capstone Project: User Stories

Pod Members: **Anny Dang, Eric Wong, Muhammed Enes Bilek**

## 1. User Roles

* **startup_founder**: A non-technical or semi-technical business creator who needs to ensure their application is secure from breaches but lacks the budget for a professional $20k pentest.
* **serial_product_developer**: A technical builder or agency engineer who rapidly ships multiple web apps and needs to catch vulnerabilities without wasting time on false-positive scanners.

## 2. User Personas

### Role: startup_founder
* **Lisa**: A 29-year-old founder with a business background who just raised a seed round. She relies on AI-generated code to speed up development. She does not understand technical security jargon and needs plain text reports detailing business risks and specific remediation steps for her team.
* **Marcus**: A 35-year-old solo founder who is pre-revenue. He cannot afford an external cybersecurity firm. His motivation is getting a reliable baseline security check to avoid launching with critical security flaws.

### Role: serial_product_developer
* **Arman**: A 24-year-old indie developer who ships 3 to 4 micro-SaaS products a year. He is technical but not a security expert and does not have the 300 hours required to manually pentest every app he launches. His motivation is catching basic security misconfigurations across multiple projects before deployment.
* **Sarah**: A 34-year-old lead developer at a digital studio that builds web apps for clients. Her motivation is proving to clients that the delivered software is secure. Human pentests take weeks to schedule and delay handoffs. She needs a tool that can run a test in a few hours and export a professional report for client deliverables.

## 3. User Stories

1. **As a startup_founder**, I want to authorize my web application for testing, so that the platform only scans infrastructure I legally own.
2. **As a startup_founder**, I want to set strict limits on the test's aggressiveness, so that my app remains online and stable for my customers during the scan.
3. **As a startup_founder**, I want to view a real-time dashboard of the test's progress, so that I know what surfaces of my app are currently being evaluated.
4. **As a startup_founder**, I want to be able to immediately halt an ongoing test, so that I can intervene if my servers start struggling under load.
5. **As a startup_founder**, I want to export the final vulnerability report as a PDF, so that I can share it with investors or clients.
6. **As a serial_product_developer**, I want the final report to only include validated findings, so that I do not waste time chasing false positives.
7. **As a serial_product_developer**, I want step-by-step reproduction instructions for each finding, so that I can recreate the vulnerability locally and verify my patch.
8. **As a serial_product_developer**, I want specific remediation guidance for each vulnerability, so that I know exactly how to fix the underlying configuration or code issue.
9. **As a serial_product_developer**, I want to see a visual timeline of how the system was breached, so that I can understand how different minor vulnerabilities were chained together to create an exploit.
10. **As a serial_product_developer**, I want to review a history of my previous security tests, so that I can track if the security posture of my apps is improving over time.

## 4. AI Feature User Story

* **As a startup_founder**, I want a synthesized text summary of all the raw security findings, so that I can quickly understand my overall risk level without having to interpret technical logs. 

## 5. Decisions Log: User Stories

- **Story we debated the scope of**: *"As a startup_founder, I want to verify ownership of my domain via a DNS TXT record or hosted file."*
  **How we resolved it**: We realized this was prescribing a specific technical implementation. We revised it to *"As a startup_founder, I want to authorize my web application for testing"* and decided to leave the verification mechanism for our component spec next week.
- **Story we cut (and why)**: *"As a serial_product_developer, I want to schedule recurring weekly tests in my CI/CD pipeline."*
  **Why**: We cut this because continuous monitoring and CI/CD integration are explicitly out of scope for v1 in our PRD. We need to focus on manually triggered runs for the MVP.
- **Story that changed after Claude's feedback**: 
  **Original**: *"As a serial_product_developer, I want the system to use Nmap and Metasploit to find vulnerabilities without reporting false positives."*
  **Revision**: Claude flagged this as too tool-specific. The user cares about accuracy, not the backend MCP tools. We revised it to: *"As a serial_product_developer, I want the final report to only include validated findings, so that I don't waste time chasing false positives."*
- **AI feature story: user benefit we landed on**: We described the benefit as clarity and time savings by synthesizing complex logs into plain text, rather than detailing the Swarm Leader agent structure or the Claude LLM prompts.
