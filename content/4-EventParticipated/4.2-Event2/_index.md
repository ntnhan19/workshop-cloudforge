---
title: "Event 2"
date: 2026-05-23
weight: 1
chapter: false
pre: " <b> 4.2. </b> "
---

# Summary Report: AWS Vietnam Community Day & First Cloud Journey 2026

### Event Objectives
- Share in-depth knowledge on applying Generative AI (GenAI), fine-tuning, and building Multi-Agent systems.
- Provide methods to optimize the AI user experience through Context building and Memory integration.
- Guide the construction of a secure, cost-optimized, and highly performant cloud platform architecture using Amazon CloudFront.
- Inspire attendees through real-world product development journeys in Hackathon competitions.

### Speakers
- **Duc Dao** – Solution Architect, Cloud Kinetics
- **Vy Lam** – Senior Business Systems Analyst, VPBank
- **Pham Ng Hai Anh** – Cloud Consultant (G-AsiaPacific Vietnam) & AWS Community Builder
- **Nguyen Tuan Thinh** – DevOps Engineer, First Cloud AI Journey
- **Tinh Truong** – Platform Engineer, GoTymeX
- **Team VIB** – Track Winner at LotusHacks 2026

### Key Highlights

#### 1. The Non-determinism of LLM Settings (Duc Dao)
- **The Reality of Temperature = 0:** Theoretically, Temp=0 should yield deterministic results (100% identical outputs). In reality, however, no LLM guarantees consistency across all tasks.
- **Technical Causes:** GPU architectures perform floating-point operations that lack associativity, and API providers often batch user requests to optimize costs, leading to minor errors that alter the final output.
- **Mitigation Strategies:** Build systems with an "accept variability" mindset using majority voting (running multiple times and picking the most common result), enforcing output types (JSON, YAML), and using temp=0.1 to prevent the model from getting stuck in repetitive loops.

#### 2. Enterprise-Grade Multi-Agent Systems - Startup Scoring (Vy Lam)
- **Limitations of a Single Agent:** Using a single AI Agent for credit approvals often encounters context limits, diluted expertise, a lack of cross-checking, and becoming a single point of failure.
- **The Power of Multi-Agent:** An architecture simulating a virtual credit committee comprised of specialized Agents (Financial Analysis, Market, Team, Risk, and Compliance) helps make more accurate decisions.
- **Enterprise-Grade Mindset:** To transition from POC to Production, the system must be equipped with Guardrails (input/output protection), security (MFA, encryption), Data Governance (handling PII), and compliance with frameworks like SOC 2 and GDPR.

#### 3. The Importance of Context in AI (Tinh Truong)
- **Common Mistakes:** AI giving poor answers is often due to weak input (context) rather than the model itself. Cramming too much garbage information (Internet Puller) reduces accuracy and wastes tokens.
- **Context Building Formula:** A good context needs 4 essential elements: Goal, Relevant info, Constraints, and Success criteria.
- **The Evolution of AI:** How we interact with AI is shifting from simply using Prompts -> Building Context -> Forming Memory (A second AI brain) to support long-term personalization.

#### 4. Platform Architecture Optimization with Amazon CloudFront (Nguyen Tuan Thinh)
- **Cost and Security Issues:** The pay-as-you-go model can sometimes result in unexpected CDN bills. Routing and protection packages are needed to mitigate financial risks.
- **Origin Cloaking:** Utilizing Origin Access Control (OAC) to lock direct access to S3 and Lambda ensures that traffic must route through CloudFront.
- **High Availability:** CloudFront supports Failover configuration when the primary origin fails, maintaining an uninterrupted user experience and boosting speed with HTTP Compression (reducing load times by up to 81%).

#### 5. AI Assistants and Practical Applications (Pham Ng Hai Anh & Team VIB)
- **Amazon Q:** Introduced the Amazon Q Suite which helps automate PM tasks like summarizing meeting minutes (MoM), sending emails, and scheduling, via its internal data connection capabilities.
- **Lessons from LotusHacks:** Building the UTMorpho project in 36 hours. The hard-learned lesson is that the best ideas stem from real-world frustrations. In teamwork, synchronization is paramount, and one must avoid "scope creep" (overloading features).

### Key Takeaways

#### On AI Mindset & App Development
- **Quality over Quantity:** Provide high-quality Context for AI instead of a pile of garbage data.
- **Multi-Agent Mechanisms:** Complex enterprise problems should be broken down and cross-processed by multiple AI Agents instead of relying on a single agent.
- **Understanding Technology Basics:** Do not blindly trust the deterministic setting (temperature=0) of LLMs for high-risk applications.

#### Technical Architecture
- Secure the system from the network edge to the origin server using CloudFront and Guardrails.
- Transition from manual processes to integrating AI Agents and "Second Brain" systems with memory retrieval capabilities.

### Applying to Work
- **Optimize internal LLM systems:** Shift mindset when writing Prompts, utilize the 4-step framework (Goal, Info, Constraints, Criteria), and experiment with temp=0.1 if repetitive responses occur.
- **Experiment with Multi-Agent Architectures:** Build automated evaluation workflows instead of using one long, single prompt.
- **CDN Security:** Re-evaluate access permissions (bucket policies) on AWS S3, and implement OAC instead of the legacy OAI to enhance static content security.

### Event Experience
Attending the AWS Vietnam Community Day 2026 provided me with an incredibly realistic perspective on the challenges of deploying AI in an Enterprise environment. From deep technical issues like GPU floating-point errors skewing LLMs, to designing a multi-layered security architecture with Multi-Agent and CloudFront.

The event didn't just impart technical knowledge but also reminded me of product-thinking: Always start from a real "pain point", maintain focus, and establish guardrails before scaling. The biggest highlight was realizing that the boundary between "knowing how to use AI" and "making AI work effectively" lies entirely in our ability to provide and manage Context.

### Event Photos

![Event Photo 1](/images/4-EventParticipated/4.2-Event2/event_23_05%281%29.jpg)

![Event Photo 2](/images/4-EventParticipated/4.2-Event2/event_23_05%282%29.jpg)
