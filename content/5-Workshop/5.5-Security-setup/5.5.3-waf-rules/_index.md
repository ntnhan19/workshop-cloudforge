---
title : "Configure WAF Rules to protect API"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.5.3. </b> "
---

After setting up identity (Cognito) and protecting sensitive data (Secrets Manager), the final step in our multi-layered security strategy is to build a "shield" at the outermost layer: **AWS WAF (Web Application Firewall)**.

AWS WAF will act as a gatekeeper force, analyzing all access traffic (HTTP/HTTPS) entering the system and automatically intercepting common destructive attacks before they can reach the application source code or Database.

#### 1. Initialize Protection Pack (Web ACL)
From the AWS Console, access the **WAF & Shield** → **Protection packs (web ACLs)** service and select **Create protection pack (web ACL)**. Since AWS continuously updates the interface, please configure carefully following these steps to optimize costs:

**Step 1: Tell us about your app**
- **App category:** Select **API & integration services** (Or **Other**).
- **App focus:** The system will automatically select **API** (Suitable for our Backend API architecture).

**Step 2: Select resources to protect**
- Select **Skip for now**. We will attach this Web ACL to the Application Load Balancer in a later chapter.

**Step 3: Choose protection pack (web ACL) resource types**
- Check the regional resource type: **Amazon API Gateway REST API, Application Load Balancer...**

**Step 4: Choose initial protections (Note: Avoid cost traps)**
By default, AWS will suggest the *Recommended* pack, which carries a relatively high cost. To optimize costs for the Lab environment, operate as follows:
- You must select the 3rd option: **Build your own pack from all of the protections AWS WAF offers**.

**Step 5: Name and describe**
- **Name:** Enter `cloudforge-api-waf`.

**Step 6: Add rules**
On the **Add rules** panel on the right side of the screen, proceed to add the AWS-managed rule groups to protect the application:
1. Add the **Core rule set** (AWSManagedRulesCommonRuleSet): Intercepts basic web attacks according to the OWASP Top 10.
2. Add the **SQL database** rule set (AWSManagedRulesSQLiRuleSet): Absolutely prevents database malicious code insertion techniques (SQL Injection).
*(Ensure both rule sets display a green **Saved** status).*

Finally, scroll to the bottom of the main screen and click the **Create protection pack (web ACL)** button to complete.

#### 2. Application Firewall Configuration Results
Once the system successfully initializes, the Web ACL is ready in an active state with optimal rule sets.

![WAF Rules Created](/images/5-Workshop/5.5-Security-setup/5.5.3-waf-rules/waf_rules_created.png)

{{% notice tip %}}
**Cost Optimization:** Proactively choosing to build your own rule set (Build your own pack) instead of using the Recommended pack helps reduce WAF maintenance costs from ~$55/month down to only ~$5/month (basic Web ACL maintenance fee). This setup is the most appropriate and optimal for a proof-of-concept (PoC) environment while still ensuring critical risks are blocked.
{{% /notice %}}

***

**Next Step:** The Smart Media Analytics system is now deployed with foundational security layers (Cognito, Secrets Manager, AWS WAF), meeting core safety standards. Get ready to move on to [**Chapter 5.6: Ingestion Workflow**](../../5.6-Ingestion-workflow/) to begin setting up the asynchronous event orchestration architecture!
