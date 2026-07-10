---
title : "CI/CD Pipeline"
date : 2026-07-10
weight : 12
chapter : false
pre : " <b> 5.12. </b> "
---

### Overview of CI/CD Pipeline Architecture

Following the stable operation of the entire Smart Media Analytics system—encompassing the Database, Backend, AI/ML, and Frontend components—the project team proceeds to establish a Continuous Integration and Continuous Deployment (CI/CD) pipeline. This optimizes the Software Development Life Cycle (SDLC) in alignment with modern DevOps standards.

Manual source code deployment from local workstations to the AWS infrastructure not only induces temporal bottlenecks but also harbors significant risks associated with human error. Consequently, the project team resolved to architect a fully automated CI/CD pipeline utilizing **GitHub Actions**.

#### Why use GitHub Actions with OIDC?
The CI/CD ecosystem is engineered with strict adherence to AWS security standards:

- **Keyless Authentication:** Instead of persistently storing static IAM Access Keys within GitHub Secrets, which carries significant security risks, GitHub Actions securely authenticates its identity with AWS via **OpenID Connect (OIDC)**, completely negating the necessity for static credentials.
- **Least Privilege:** The IAM Role granted to GitHub Actions is stringently constrained to exclusively execute ECR Push and ECS update operations, and can solely be triggered from the precise `main` branch of the project's Repository.

#### Deployment Roadmap:
1. **Configure OIDC Authentication:** Configure the Identity Provider on AWS IAM and provision a dedicated Role for GitHub Actions.
2. **Automate Docker Image Push:** Author a GitHub Actions workflow to autonomously Build the source code and Push the Image to the private Amazon ECR repository.
3. **Continuous Deployment to ECS:** Update the configuration record (Task Definition) on Amazon ECS and execute a Rolling Update without incurring service interruptions (Zero-downtime).

### Hands-on Content

- [Configure OIDC Authentication](5.12.1-github-actions-oidc/)
- [Automate Image Push to ECR](5.12.2-ecr-push/)
- [Continuous Deployment to ECS](5.12.3-deploy-pipeline/)
