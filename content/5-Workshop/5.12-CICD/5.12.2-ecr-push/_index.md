---
title : "Automate Image Push to ECR"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.12.2. </b> "
---

Having successfully established the secure OIDC authentication flow in lesson 5.12.1, the subsequent phase in the CI/CD pipeline is to automate the Build (source code packaging) and Push (Image uploading) processes to the Amazon Elastic Container Registry (ECR) via GitHub Actions.

Amazon ECR functions as a secure Private Registry hosting Docker Image iterations of the Backend application, primed for deployment onto the server infrastructure.

#### Step 1: Initialize the Amazon ECR Repository
Before GitHub Actions can upload the Image, a vacant repository must be provisioned on AWS.

1. Access the **Amazon ECR** Console.
2. From the left-hand menu, navigate to **Repositories** and click **Create repository**.
3. Configure the fundamental parameters:
   - **Visibility settings:** Select `Private`.
   - **Repository name:** Assign a repository name as `cloudforge-backend`.
4. Click **Create repository** at the bottom of the page to finalize.

Note and copy the **URI** of the newly generated Repository (e.g., `236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-backend`).

![Create ECR Repository](/images/5-Workshop/5.12-CICD/5.12.2-ecr-push/5.12.2-create-ecr.png)

#### Step 2: Formulate the GitHub Actions Workflow
The project team authors a Workflow definition file situated at the project root directory in VS Code: `.github/workflows/deploy.yml`.

This configuration file dictates the sequential Steps to be executed whenever novel source code is pushed to the `main` branch.

```yaml
name: Deploy Backend to ECR

on:
  push:
    branches:
      - main

# Mandatory permissions allowing GitHub Actions to receive OIDC Tokens
permissions:
  id-token: write 
  contents: read 

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Source Code
        uses: actions/checkout@v4

      - name: Configure AWS Credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::236320489525:role/GitHubActionsDeployRole
          aws-region: ap-southeast-1

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build, tag, and push Docker image to ECR
        env:
          REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          REPOSITORY: cloudforge-backend
          IMAGE_TAG: ${{ github.sha }}
        run: |
          # Navigate to the backend directory since the project is a monorepo
          cd backend
          docker build -t $REGISTRY/$REPOSITORY:$IMAGE_TAG .
          docker push $REGISTRY/$REPOSITORY:$IMAGE_TAG
```

**Execution Flow Breakdown:**
- **permissions (`id-token: write`):** This is the paramount configuration. Devoid of this directive, GitHub will fail to solicit a JWT token from STS, culminating in an OIDC authentication failure.
- **configure-aws-credentials:** This step assumes the `GitHubActionsDeployRole` (created in 5.12.1) to extract Temporary Credentials.
- **amazon-ecr-login:** This step autonomously authenticates into the internal ECR repository.
- **Build & Push:** Packages the Docker Image utilizing the `Dockerfile` in the `backend` directory, subsequently tags it utilizing the commit's SHA code (`github.sha`) for streamlined versioning, and pushes it to ECR.

#### Step 3: Validate the Automated Build & Push Process
After creating the `.github/workflows/deploy.yml` file in VS Code, save it and execute a `git push` command to the `main` branch. Next, navigate to the **Actions** tab on your GitHub repository interface.

You will observe a process entitled `Deploy Backend to ECR` executing autonomously. Assuming all OIDC configurations and source code are valid, the workflow will terminate with a green checkmark (Success).

Re-access the `cloudforge-backend` repository on the Amazon ECR Console; a pristine Docker Image will have been pushed, tagged with the commit's SHA string.

![GitHub Actions Success](/images/5-Workshop/5.12-CICD/5.12.2-ecr-push/5.12.2-actions-success.png)

***

**Next Step:** The Docker Image has been provisioned on the private registry. In the conclusive lesson 5.12.3, the project team will append a final Step to the existing Workflow to instigate the deployment command of the novel Image onto Amazon ECS employing a Rolling Update strategy (Zero-downtime).
