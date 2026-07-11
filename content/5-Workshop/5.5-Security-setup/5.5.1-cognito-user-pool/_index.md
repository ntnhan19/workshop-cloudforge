---
title : "Set up Cognito User Pool"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.5.1. </b> "
---

In the architecture of Smart Media Analytics, we will not build an account management system from scratch (as it is time-consuming and carries high security risks). Instead, we will delegate all identity and authentication tasks to **Amazon Cognito**.

The Backend (or Frontend) system will communicate with Cognito to handle sign-up, sign-in, and receive standard JWT (JSON Web Token) tokens to authenticate API requests.

#### 1. Initialize Cognito User Pool (New UI)
Navigate to the **Amazon Cognito Console**, ensuring you are in the correct Region `ap-southeast-1` (Singapore), and follow the quick setup interface (**Set up resources for your application**):

**Step 1: Define your application**
- **Application type:** Select **Single-page application (SPA)**. *(Selecting SPA helps Cognito automatically configure a Public Client without a Client Secret, which is perfectly suited for Frontend React/Vue architectures calling APIs).*
- **Name your application:** Enter `cloudforge-app`.

**Step 2: Configure options**
- **Options for sign-in identifiers:** Check **Email** (Users will log in using their Email).
- **Self-registration:** Check **Enable self-registration** (Allow users to register themselves).
- **Required attributes for sign-up:** Click the box and select the **name** attribute (Requires users to enter their name upon registration).

**Step 3: Add a return URL (Optional)**
- **Return URL:** Although marked as optional, AWS recommends providing it. You can enter a temporary address to serve local testing: `https://localhost`

Finally, scroll to the bottom and click the **Create user directory** button for AWS to automatically provision all resources.

#### 2. Retrieve Integration Details (App Client ID)
Once successfully created, the system will take you to the management page of the newly created user directory. You need to note down 2 extremely important parameters in preparation to inject them into the application's environment variables (`.env`):

1. **User Pool ID:** Located right at the top of the overview page (Format: `ap-southeast-1_xxxxxxxxx`).
2. **App Client ID:** Switch to the **App integration** tab, scroll down to the *App clients and analytics* section, and copy the ID string for `cloudforge-app` (e.g., `3abc123xyz...`).

![Cognito App Client](/images/5-Workshop/5.5-Security-setup/5.5.1-cognito-user-pool/cognito_app_client.png)

***

**Next Step:** The user identity system is complete. We will proceed to section **5.5.2: Manage configuration with Secrets Manager** to set up an ultra-secure vault, definitively solving the problem of hiding the `DB_PASSWORD`!
