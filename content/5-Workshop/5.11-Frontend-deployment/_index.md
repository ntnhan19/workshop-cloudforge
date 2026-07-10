---
title : "Frontend Deployment"
date : 2026-07-10
weight : 11
chapter : false
pre : " <b> 5.11. </b> "
---

### Frontend Deployment Architecture Overview

Once the data processing ecosystem (Backend) and artificial intelligence services are operating smoothly and securely behind the internal VPC network layer, the project team executes the final integration step: Launching the User Interface of the Smart Media Analytics application onto the public Internet.

To fulfill standards for High Availability, Low Latency, and automated delivery pipelines (CI/CD), the project team selected **AWS Amplify Hosting**.

#### Why Choose AWS Amplify Hosting?
Distinct from manually provisioning traditional static server infrastructure (like configuring Amazon S3 paired with CloudFront), AWS Amplify provides a Fully Managed solution with superior architectural advantages:

- **CI/CD Automation:** Integrates directly with source code repositories (GitHub/GitLab/CodeCommit). Upon merging a new code branch, Amplify automatically triggers the Build and Deploy sequence without necessitating third-party tools.
- **Global Content Delivery Network (CDN):** Operates seamlessly on AWS's underlying CloudFront infrastructure, ensuring static assets (HTML/CSS/JS) are distributed at maximum velocity across all geographic regions.
- **Simplified Routing and Security:** Natively supports Custom Domain configuration via Route 53 and automates the provisioning and renewal of free SSL/TLS certificates (via AWS Certificate Manager).

#### Frontend Deployment Roadmap:
1.  **Provision AWS Amplify:** Connect the Frontend source code repository and configure the Build environment.
2.  **Domain Configuration (Route 53):** Map a custom domain name to the Amplify application to replace the default generated URL.
3.  **Authentication Integration (Cognito):** Configure environment variables and link the interface with the centralized Amazon Cognito authentication system.

### Hands-on Content

- [Deploy with AWS Amplify](5.11.1-deploy-amplify/)
- [Setup Route 53 Domain](5.11.2-setup-route53/)
- [Connect Cognito and Frontend](5.11.3-connect-cognito-frontend/)
