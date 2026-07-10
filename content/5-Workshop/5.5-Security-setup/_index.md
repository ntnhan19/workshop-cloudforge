---
title : "Security Setup"
date : 2026-07-10
weight : 5 
chapter : false
pre : " <b> 5.5. </b> "
---

### Security Setup Overview

Security is not a bolted-on "feature" but a mandatory foundation of any solid Cloud architecture. For the Smart Media Analytics system, which handles media data and integrates AI, we need to establish a Defense in Depth strategy.

In this chapter, we will implement the most rigorous security standards through 3 main pillars:

1. **Amazon Cognito (Identity & Access):** Build a secure user identity system. This service will shoulder the entire authentication lifecycle (sign-up, sign-in, forgot password) and issue standard OAuth 2.0 / JWT tokens without us having to build it from scratch.
2. **AWS Secrets Manager (Data Protection):** Actualize the security warning from the previous chapter. All sensitive information such as `DB_PASSWORD` or API Keys will be encrypted and centrally managed here, completely eliminating the risk of exposure from source code or plaintext environment variables.
3. **AWS WAF (Edge Protection):** Deploy a Web Application Firewall "shield" at the outermost edge to intercept malicious traffic, automatically preventing common vulnerabilities like SQL Injection, Cross-Site Scripting (XSS), and DDoS attacks.

Completing this chapter will elevate our project's architecture from a "Proof of Concept" level to a "Production-ready" standard with the highest reliability.

### Hands-on Content

- [Set up Cognito User Pool](5.5.1-cognito-user-pool/)
- [Manage configuration with Secrets Manager](5.5.2-secrets-manager/)
- [Configure WAF Rules to protect API](5.5.3-waf-rules/)
