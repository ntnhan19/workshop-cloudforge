---
title : "Deploy with AWS Amplify"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.11.1. </b> "
---

This document records the technical procedure for initializing and establishing continuous deployment (CI/CD) for the Frontend source code of the Smart Media Analytics application via the AWS Amplify Hosting service.

#### Step 1: Initialize Amplify Application
The deployment process commences with the establishment of a static Web App on the AWS Amplify platform.

1. Access the **AWS Amplify** service from the AWS Management Console.
2. On the Amplify console, scroll down to the **Amplify Hosting** section and select **Host your web app**.
3. The system requires authorization to connect with the source code repository platform (Version Control System). The project team selects **GitHub** and clicks **Continue**.
4. Perform the Authorization to grant AWS Amplify permission to read data from the respective GitHub account.

![Amplify Connect GitHub](/images/5-Workshop/5.11-Frontend-deployment/5.11.1-deploy-amplify/5.11.1-amplify-github.png)

#### Step 2: Configure Build Settings
Following a successful connection, AWS Amplify requires the specification of the particular repository and Branch that will serve as the data source.

1. **Designate Repository:** Under the *Repository and branch* section, select the exact repository housing the Frontend source code (e.g., `cloudforge-frontend`) and designate the **`main`** branch as the Production deployment branch. Click Next.
2. **Configure Build:** Under the *Build settings* section, Amplify automatically detects the framework to generate a default `amplify.yml` configuration file. The project team adjusts the `preBuild` phase to inject environment variables connecting to the Backend system (These variables will be detailed in lesson 5.11.3).
3. **Automatic Authorization:** Check the option *Allow AWS Amplify to automatically deploy all files hosted in your project root directory* to ensure frictionless CI/CD execution. Click Next.

#### Step 3: Review & Deploy
1. On the Review screen, verify all Repository linkage parameters and Build settings.
2. Click **Save and deploy**. 

*System state: At this juncture, Amplify initiates an automated 4-phase procedure:*

- **Provision:** Allocates a virtual container environment to prepare for compilation.
- **Build:** Pulls the source code from GitHub, downloads dependencies (`npm install`), and compiles the application into static assets (`npm run build`).
- **Deploy:** Pushes the compiled static assets to AWS's underlying CDN system (CloudFront + S3).
- **Verify:** Checks the operational status of the application.

![Amplify Build Pipeline](/images/5-Workshop/5.11-Frontend-deployment/5.11.1-deploy-amplify/5.11.1-amplify-pipeline.png)

When all 4 steps transition to green (Success), AWS Amplify provisions a default URL formatted as `https://[branch-name].[app-id].amplifyapp.com`. 

{{% notice tip %}}
**Architectural Evaluation:** Amplify's default URL is entirely serviceable for Staging/Testing environments. However, for a professional Production application, the project team must mandate replacing this URL with a Custom Domain to elevate brand identity and establish stricter CORS security policies.
{{% /notice %}}

***

**Next Step:** The Frontend source code has been successfully compiled and distributed. The project team will proceed to attach a Custom Domain to the application via the Amazon Route 53 service in lesson 5.11.2.
