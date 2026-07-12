---
title : "Setup Route 53 Domain"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.11.2. </b> "
---

Following the deployment of the static interface onto the AWS Amplify infrastructure, the system provisions a default URL. To adhere to the standards of a genuine Production system, the project team proceeds to configure a Custom Domain via Amazon Route 53.

Establishing a Custom Domain not only elevates brand identity but is also a mandatory prerequisite for formulating stringent CORS (Cross-Origin Resource Sharing) policies on the API Gateway and Amazon Cognito in subsequent steps.

#### Step 1: Domain Management in Amplify
The integration process occurs directly within the AWS Amplify console, leveraging its native linkage capabilities with Route 53.

1. Access the Smart Media Analytics application within the **AWS Amplify** console.
2. In the left navigation pane, locate the **Hosting** section and select **Custom domains**.
3. Click the **Add domain** button situated in the top right corner.

#### Step 2: Configure Domain Mapping
The Amplify system automatically retrieves the list of existing Hosted Zones within the AWS account's Amazon Route 53.

1. **Select Domain:** In the *Domain* field, select the domain (e.g., `cloudforge.com`) that is registered or managed by Route 53.
2. **Subdomain Setup:** Click on **Configure domain** to establish routing rules.
   - Point the root domain directly (e.g., `cloudforge.com`) or a subdomain (e.g., `app.cloudforge.com`) to the `main` source code branch deployed in lesson 5.11.1.
   - Configure automatic Redirects from `www.cloudforge.com` to the root domain to ensure user access flows remain unfragmented.
3. Click **Save** to persist the configuration.

![Amplify Custom Domain](/images/5-Workshop/5.11-Frontend-deployment/5.11.2-setup-route53/5.11.2-amplify-domain.png)

#### Step 3: Automated SSL/TLS Verification (ACM)
Distinct from traditional architectures demanding administrators to manually provision and configure security certificates, AWS Amplify handles this entire procedure completely autonomously.

- Immediately upon saving the configuration, Amplify transmits a request to **AWS Certificate Manager (ACM)** to provision a free SSL/TLS certificate for the newly established domain.
- The Amplify system also automatically injects the requisite CNAME validation records into the Route 53 Hosted Zone without requiring manual intervention.
- The Domain's status will transition through: *Pending verification* -> *Provisioning* -> *Available*.

Once the status shifts to **Available**, the Frontend application is officially fortified by an HTTPS encrypted connection and is ready to serve users via the Custom Domain.

{{% notice info %}}
**Technical Note:** The SSL certificate verification process and global DNS (Domain Name System) record propagation can take anywhere from 5 to 30 minutes depending on network infrastructure.
{{% /notice %}}

***

**Next Step:** Both Backend and Frontend systems are now identified and secured with SSL. In the final step within lesson 5.11.3, the project team will configure environment variables on the Frontend to connect directly with the API Gateway and the Amazon Cognito authentication system.
