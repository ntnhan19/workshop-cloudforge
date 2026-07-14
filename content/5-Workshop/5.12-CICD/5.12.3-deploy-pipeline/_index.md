---
title : "Continuous Deployment to ECS"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.12.3. </b> "
---

Following the successful packaging and pushing of the novel Docker Image to Amazon ECR in lesson 5.12.2, the conclusive phase of the CI/CD pipeline is to autonomously update the Amazon ECS server infrastructure to execute the latest source code iteration.

To guarantee that the system experiences zero-downtime during the update procedure, the project team designed a deployment workflow predicated on Amazon ECS Fargate's default **Rolling Update** mechanism.

#### Step 1: Augment the GitHub Actions Workflow
Re-open the `.github/workflows/deploy.yml` file authored in the previous lesson within VS Code and append the sequential Steps to download the current Task Definition configuration, inject the new Image ID, and command the deployment directly to the ECS Cluster.

The complete `deploy.yml` file content subsequent to CD flow integration:

```yaml
name: Deploy Backend to ECR and ECS

on:
  push:
    branches:
      - main

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
        id: build-image
        env:
          REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          REPOSITORY: cloudforge-backend
          IMAGE_TAG: ${{ github.sha }}
        run: |
          cd backend
          docker build -t $REGISTRY/$REPOSITORY:$IMAGE_TAG .
          docker push $REGISTRY/$REPOSITORY:$IMAGE_TAG
          echo "image=$REGISTRY/$REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT

      - name: Download current ECS task definition
        run: |
          aws ecs describe-task-definition \
            --task-definition cloudforge-backend-task \
            --query taskDefinition > task-definition.json

      - name: Fill in the new image ID in the Amazon ECS task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-definition.json
          container-name: backend-container
          image: ${{ steps.build-image.outputs.image }}

      - name: Deploy Amazon ECS task definition
        uses: aws-actions/amazon-ecs-deploy-task-definition@v2
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: cloudforge-backend-service
          cluster: cloudforge-compute-cluster
          wait-for-service-stability: true
```

**Supplementary Deployment Flow Breakdown:**
- **Download task definition:** Employs the AWS CLI (authenticated via a short-lived OIDC token) to download the currently active Task Definition configuration as a `task-definition.json` file, assuring the Pipeline is perpetually synchronized with the physical Cloud infrastructure configuration.
- **render-task-definition:** Scans the newly downloaded JSON definition file for a container named `cloudforge-backend-container` and replaces the `image` field with the URI path of the Docker Image bearing the commit SHA freshly pushed to ECR.
- **deploy-task-definition:** Registers a New Revision of the Task Definition to AWS and updates the ECS Service to catalyze the Rolling Update process. The `wait-for-service-stability: true` parameter compels GitHub Actions to maintain the connection and wait until the new containers pass Health Checks and stabilize before closing the successful execution session.

#### Step 2: Observe the Rolling Update Process
Save the configuration file on your local machine, execute a commit, and run `git push` to the `main` branch.

1. **On GitHub Actions:** Monitor the CI/CD pipeline. The *Deploy Amazon ECS task definition* phase will maintain a running status for approximately 3 to 5 minutes to await ECS container initialization.
2. **On AWS ECS Console:** Transition to the server cluster management interface, access the `cloudforge-backend-service`, and select the **Deployments** tab.

Here, the Rolling Update mechanism will transpire autonomously and intuitively:
- ECS calculates and provisions new Task instances in parallel with the legacy Tasks, predicated on the specified new Image.
- The new Tasks autonomously register into the Application Load Balancer (ALB) Target Group and execute the initial health check sequence.
- Upon the new Tasks attaining the RUNNING and HEALTHY status, ECS systematically directs the entirety of user traffic to the novel version whilst concurrently draining and deregistering the obsolete Tasks. This assures the system sustains 100% continuous operation devoid of interruption errors.

![ECS Rolling Update](/images/5-Workshop/5.12-CICD/5.12.3-deploy-pipeline/5.12.3-ecs-deploy.png)

***

**Chapter 5.12 Summary:** The project team has triumphantly established a CI/CD Pipeline adhering to Enterprise security paradigms. Henceforth, any modification to the Backend source code will autonomously traverse a closed loop: **Secure OIDC Authentication -> Automated Image Build & Push -> Zero-downtime Rolling Update Deployment**. The Software Development Life Cycle (SDLC) has been wholly optimized and automated.

**Next Step:** The system operates flawlessly on autopilot, but how can administrators monitor system health and trace errors if an incident transpires? In the concluding chapter ([**Chapter 5.13: Observability**](../../5.13-Observability/)), we will explore comprehensive application monitoring with AWS CloudWatch and AWS X-Ray.
