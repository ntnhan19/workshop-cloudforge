---
title : "Manage config with Secrets Manager"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.5.2. </b> "
---

In previous chapters, we collected many sensitive configuration parameters, including the database password (`DB_PASSWORD`), User Pool ID (`COGNITO_USER_POOL_ID`), and App Client ID (`COGNITO_APP_CLIENT_ID`). Storing these character strings directly in Plaintext within a `.env` file or source code is a severe security vulnerability.

To definitively solve this problem according to enterprise standards, we will use **AWS Secrets Manager** to encrypt, centrally store, and manage the lifecycle of these sensitive configurations.

#### 1. Initialize the Secret to store application config
1. In the search bar of the AWS Console, type **`Secrets Manager`** and select the corresponding service. Ensure you are still in Region `ap-southeast-1`.
2. On the main interface, click the **Store a new secret** button.
3. Proceed with the configuration through the following steps:

**Step 1: Choose secret type**
- **Secret type:** Check **Other type of secret** (Suitable for bundling multiple Key/Value pairs of application configuration).
- **Key/value pairs:** Switch to the **Key/value** tab and add the following core parameters by clicking *Add row*:
  
  | Key | Value |
  | :--- | :--- |
  | `DB_PASSWORD` | *<Enter your actual RDS PostgreSQL password>* |
  | `COGNITO_USER_POOL_ID` | *<Enter the User Pool ID obtained in section 5.5.1>* |
  | `COGNITO_APP_CLIENT_ID` | *<Enter the App Client ID obtained in section 5.5.1>* |

- **Encryption key:** Keep the default `aws/secretsmanager` (AWS will automatically use the KMS service to encrypt this vault).
- Click **Next**.

**Step 2: Configure secret**
- **Secret name:** Enter the name following a hierarchical directory structure for easy management: `cloudforge/backend/config`
- **Description:** Enter a brief description (e.g., `Contains sensitive configuration for the Smart Media Analytics Backend application`).
- Leave other fields at their defaults and click **Next**.

**Step 3: Configure rotation**
- **Automatic rotation:** Select **Disable automatic rotation** (Turn off the automatic periodic password change feature for a simple Lab environment).
- Click **Next**.

**Step 4: Review**
- Scroll to the bottom to review the configuration structure and click the **Store** button to complete the initialization.

#### 2. Check the secure vault information
After successfully clicking Store, the system will redirect you back to the list. Click on the Secret named `cloudforge/backend/config` that you just created.

Here, you need to note down the following extremely important parameter:
* **Secret ARN:** The globally unique identifier string from AWS (Format: `arn:aws:secretsmanager:ap-southeast-1:xxxxxxxxxxxx:secret:cloudforge/backend/config-xxxxxx`). 

{{% notice info %}}
**Future operation mechanism:** Later on, when configuring the AWS ECS (Elastic Container Service) to run the Backend application, we will no longer need to pass the `.env` file into the Container. Instead, we only need to attach this **Secret ARN** string into the Task Definition configuration section. When the Container starts, AWS ECS will automatically use IAM permissions to read this vault and directly inject the environment variables into the Container's RAM in an absolutely secure manner.
{{% /notice %}}

![Secrets Manager Created](/images/5-Workshop/5.5-Security-setup/5.5.2-secrets-manager/secrets_manager_created.png)

***

**Next Step:** The secure configuration vault has been fully set up. We will proceed to the outermost defensive layer in **5.5.3: Configure WAF Rules to protect API** to establish a firewall that blocks destructive network attacks from the Internet!
