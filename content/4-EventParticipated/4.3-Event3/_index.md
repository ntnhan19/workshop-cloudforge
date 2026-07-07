---
title: "Event 3"
date: 2026-06-27
weight: 1
chapter: false
pre: " <b> 4.3. </b> "
---

# Summary Report: FCAJ Community Day - June 2026

### Event Objectives
- Share practical experiences from the Proof of Concept (PoC) phase to Production deployment by tech startups.
- Analyze optimal architectures for Voice AI and how to solve specific problems related to the Vietnamese language.
- Introduce modern AI tools (AWS DevOps Agent, Amazon Q) that help automate system operations and Human Resources (HR) management.
- Provide advanced security solutions when integrating AI with internal enterprise data.

### Speakers
- **Steve Tran** – Founder & CTO, CloudThinker
- **Trung Vu** – Founder, Revve AI
- **Kiet Tran** – Core Team, AWS Student Builder Group HUFLIT
- **Danh Hoang Hieu Nghi** – AI Engineer Ambassador, AWS Study Group
- **Bao Phan Kim & Nguyen Nguyen** – Cloud Engineers, Cloud Kinetics
- **Truong Tran (Wayne)** – Solution Sales, Noventiq Vietnam
- **Dang Cao Minh Anh** – Solution Sales Internship, Noventiq Vietnam
- **Nguyen Duc Toan** – AWS Security Builder

### Key Highlights

#### 1. Deep Response Engine: From Detection to Autonomous Resolution (Steve Tran)
![Session 1](../../images/4-EventParticipated/4.3-Event3/session1.jpg)

- **From PoC to Production:** Do not spend too much time overthinking theories. Start executing and look for "champion customers" (like F88, FPT) to ensure your idea solves real-world enterprise problems.
- **Agentic Platform for Cloud Infrastructure:** Applying AI Agents to assist engineering teams in operating complex Cloud infrastructures, automating Incident resolution, optimizing costs (FinOps), and performing security checks before Production. AI will not replace humans, but highly skilled AI users will be needed.

#### 2. Voice Agents: Building Human-Like AI Conversations at Scale (Trung Vu - Hieu Nghi - Kiet Tran)
![Session 2](../../images/4-EventParticipated/4.3-Event3/session2.jpg)

- **Limitations of Speech-to-Speech models:** Currently, they do not well support Vietnamese (a low-resource language), and it's hard to control whether the AI says inappropriate things (Guardrails).
- **Optimal 3-step architecture:** Use *Speech-to-Text -> LLM (Text) -> Text-to-Speech*. This architecture handles Vietnamese well, accurately performs Tool Calling (like locking a bank card), and easily passes the call to a human agent naturally when the AI cannot handle it.
- **Language challenges:** The model must be trained to understand barge-in contexts, distinguish genders (anh/chị) via voice, and handle regional accents.

#### 3. AWS DevOps Agent: Your Always-Available Operations Teammate (Bao Phan - Nguyen Nguyen)
![Session 3](../../images/4-EventParticipated/4.3-Event3/session3.jpg)

- **Pain points of operations engineers:** Having to manually retrieve scattered logs/traces from multiple places, which wastes time and easily disrupts context during troubleshooting.
- **DevOps Agent Solution:** An automated system that categorizes, investigates Root Causes, and draws the system Topology diagram.
- **Optimal 4-step process:** Categorize -> Investigate -> Propose Mitigation Plan -> Improve the system, aiming to minimize Mean Time To Detect (MTTD) and Mean Time To Recovery (MTTR).

#### 4. AI-Powered Productivity: Workforce Planning For Enterprise (Truong Tran - Minh Anh)
![Session 4](../../images/4-EventParticipated/4.3-Event3/session4.jpg)

- **Manual recruitment problems:** HR spends too much time filtering CVs, makes subjective evaluations, and often loses Key Talent.
- **Amazon Q AI Assistant:** Capable of reading Job Descriptions (JD), automatically extracting data from PDF/Image CVs, and matching them.
- **Visual evaluation:** AI scores candidates based on standard benchmarks and creates a Talent Review Report indicating clear reasons for passing/failing, helping HR make quick decisions and reduce workload.

#### 5. Building Secure Private MCP Connection with Amazon Quick (Hieu Nghi - Toan Nguyen)
![Session 5](../../images/4-EventParticipated/4.3-Event3/session5.jpg)

- **Public Endpoint Risks:** When Amazon Q (on Cloud) needs to retrieve data from an MCP Server (containing third-party APIs like Jira, Zalo, HR System), routing through the Public Internet can lead to data leaks or Man-in-the-Middle attacks.
- **Private VPC Connection Solution:** Integrate Amazon Q into the internal VPC via an Interface Endpoint, combined with an Application Load Balancer (ALB) and Route 53 Resolver.
- **Result:** Creates a completely closed internal data transmission tunnel, strictly adhering to the enterprise's Zero-Trust security policies.

### Key Takeaways

#### On AI Mindset & App Development
- **Solving core problems:** The value of AI lies not in flashy technology, but in its ability to optimize costly repetitive processes (Cloud operations, CV screening).
- **Control & Customization:** For a specific language like Vietnamese, breaking down the process (e.g., converting Voice to Text for the LLM) is more important than using an imperfect end-to-end model.

#### Technical Architecture
- **AI Security First:** Never let internal data pass through the public Internet when integrating AI. Always use Private Links and VPC Connections.
- **Agentic Workflow:** The future of operations lies in multitasking Agents, from reading infrastructure logs to reviewing HR CVs.

### Applying to Work
- **Experiment with Voice AI Architecture:** Deploy Speech-to-Text combined with LLM for internal customer support switchboard scenarios or advisory bots.
- **Apply AI in HR/Admin:** Build Agents to automatically read and extract data from documents, forms, and CVs to reduce manual labor for the Back-office department.
- **Review integrated security architecture:** Ensure any third-party AI plugged into the company's data system passes through internal secure channels (VPC/ALB).

### Event Experience
Attending the FCAJ Community Day in June 2026 provided a highly practical perspective on how AI is reshaping every corner of the enterprise, from dry IT departments (DevOps, Cloud Infra) to human-centric departments (HR).

The most impressive aspect was how the speakers shared very "real-life" barriers (like a switchboard AI being scolded for using the wrong gender pronouns, or engineers burning out from reading logs manually) and provided thorough solutions using Agents. The boundary between a "demo-for-fun" AI product and a "money-making" one is the ability to control data (Guardrails) and the accompanying security architecture (Private Connection).

### Event Photos

![Audience Photo 1](../../images/4-EventParticipated/4.3-Event3/event_27_06%281%29.jpg)

![Audience Photo 2](../../images/4-EventParticipated/4.3-Event3/event_27_06%282%29.jpg)

![Audience Photo 3](../../images/4-EventParticipated/4.3-Event3/event_27_06%283%29.jpg)
