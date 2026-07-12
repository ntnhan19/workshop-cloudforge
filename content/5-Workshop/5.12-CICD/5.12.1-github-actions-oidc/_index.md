---
title : "Configure OIDC Authentication"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.12.1. </b> "
---

The paramount security challenge when integrating CI/CD pipelines via third-party platforms (such as GitHub Actions) lies in granting access to the AWS environment. Instead of provisioning an IAM User and perpetually storing static keys (`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`) within GitHub Secrets—a methodology fraught with extreme data leakage risks—the project team implements the **OpenID Connect (OIDC)** security standard.

OIDC empowers GitHub Actions to exchange a short-lived JWT token with the AWS Security Token Service (STS) to retrieve Temporary Credentials, which are automatically revoked upon pipeline completion.

#### Step 1: Create an OIDC Identity Provider in AWS
The project team configures AWS IAM to recognize GitHub as a legitimate identity provider.

1. Access the **AWS IAM** Console and select **Identity providers** from the left navigation pane.
2. Click **Add provider** and select the **OpenID Connect** type.
3. Supply the configuration parameters:
   - **Provider URL:** `https://token.actions.githubusercontent.com`
   - **Audience:** `sts.amazonaws.com`
4. Click **Get thumbprint** allowing the system to autonomously verify the certificate from GitHub.
5. Click **Add provider** to finalize.

![Configure OIDC Provider](/images/5-Workshop/5.12-CICD/5.12.1-github-actions-oidc/5.12.1-oidc-provider.png)

#### Step 2: Provision an IAM Role for GitHub Actions
Subsequent to provider establishment, the system requires a specific IAM Role that GitHub can temporarily assume.

1. In the IAM Console, navigate to **Roles** -> **Create role**.
2. Trusted entity type: Select **Web identity**.
3. Choose the Identity provider as `token.actions.githubusercontent.com` and the Audience as `sts.amazonaws.com`. Click **Next**.
4. Attach the requisite Permission policies imperative for the CI/CD flow:
   - `AmazonEC2ContainerRegistryPowerUser` (Authorization to push Images to ECR).
   - `AmazonECS_FullAccess` (Authorization to update service configurations on ECS).
5. Assign a nomenclature to the Role, for instance: `GitHubActionsDeployRole`.

#### Step 3: Configure Least Privilege Trust Policy
To unequivocally prevent any extraneous GitHub repository from assuming this Role, the project team establishes an extremely stringent **Condition** within the Trust Policy.

1. Open the newly provisioned Role and transition to the **Trust relationships** tab.
2. Select **Edit trust policy** and append a `StringLike` condition targeting the `sub` (Subject) field. Ensure the repository name is correctly designated as `smart_media_analytics_cloudforge`.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::[ACCOUNT_ID]:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:ntnhan19/smart_media_analytics_cloudforge:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

Governed by this configuration, the IAM Role will summarily reject all authorization requests unless the CI/CD pipeline is executed precisely from the `main` branch of the project repository.

![IAM Trust Relationship](/images/5-Workshop/5.12-CICD/5.12.1-github-actions-oidc/5.12.1-trust-policy.png)

***

**Next Step:** The OIDC security authentication foundation has been robustly established. In lesson 5.12.2, the project team will proceed to author the GitHub Actions workflow file to compile the source code and push the Docker Image to Amazon ECR.
