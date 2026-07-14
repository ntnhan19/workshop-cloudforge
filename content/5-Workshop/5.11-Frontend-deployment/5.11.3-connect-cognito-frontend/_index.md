---
title : "Connect Cognito and Frontend"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.11.3. </b> "
---

To enable the Frontend application to communicate securely with the Backend system (API Gateway) and authenticate users (Amazon Cognito) constructed in preceding chapters, the project team proceeds to supply connection parameters via Environment Variables in AWS Amplify.

Storing these parameters in AWS Amplify instead of hard-coding them into the source code is a fundamental security principle, isolating environment-specific data (Dev/Staging/Prod) from the GitHub repository.

#### Step 1: Configure Environment Variables in Amplify
The project team proceeds to harvest the parameter values from API Gateway and Amazon Cognito to declare them within Amplify.

1. Access the application on the **AWS Amplify** console.
2. On the left navigation bar, under **Hosting**, select **Environment variables**.
3. Click **Manage variables** and add the following core keys:
   - `VITE_API_ENDPOINT`: The Invoke URL of the **API Gateway** appended with the `/api/v1` suffix (Example: `https://xxxx.execute-api.ap-southeast-1.amazonaws.com/api/v1`).
   - `VITE_WS_URL`: The WebSocket connection URL via the **Application Load Balancer** (Example: `ws://cloudforge-alb-123456.ap-southeast-1.elb.amazonaws.com/api/v1/ingest/ws`).
   - `VITE_COGNITO_USER_POOL_ID`: The ID of the Cognito User Pool (Example: `ap-southeast-1_xxxxxxxxx`).
   - `VITE_COGNITO_USER_POOL_CLIENT_ID`: The ID of the corresponding App Client.
4. Click **Save** to persist the changes.

#### Step 2: Inject Environment Variables into the Build Pipeline (amplify.yml)
For static frameworks like React/Vite, environment variables only take effect if they are "injected" into the application during the source code compilation process. Given the project's **Monorepo** architecture, file writing commands must be executed precisely within the Frontend application directory.

1. Switch to the **Build settings** section under **App settings** on the left menu.
2. Locate the *App build specification* configuration (content of the `amplify.yml` file) and click **Edit**.
3. Modify the `preBuild` phase to auto-generate an internal `.env` file prior to executing the compilation command:

```yaml
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - echo "VITE_API_ENDPOINT=$VITE_API_ENDPOINT" >> .env
        - echo "VITE_WS_URL=$VITE_WS_URL" >> .env
        - echo "VITE_COGNITO_USER_POOL_ID=$VITE_COGNITO_USER_POOL_ID" >> .env
        - echo "VITE_COGNITO_USER_POOL_CLIENT_ID=$VITE_COGNITO_USER_POOL_CLIENT_ID" >> .env
        - npm install
    build:
      commands:
        - npm run build
  artifacts:
    baseDirectory: dist
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
```

4. Click **Save** to update the Build pipeline configuration.

#### Step 3: Integrate Authentication Component into React Source Code
To solve the challenge of constructing a user management interface in a short timeframe, the project team integrates AWS Amplify's pre-built interface library. This library automatically provisions complete Enterprise-standard Login, Registration, Forgot Password, and OTP Authentication forms without necessitating manual coding from scratch.

Install the supplementary libraries at the Frontend source code directory:

```bash
npm install aws-amplify @aws-amplify/ui-react
```

Initialize the configuration and utilize the `<Authenticator>` Component to wrap the application's primary flow in the `src/App.jsx` file:

```javascript
import { Amplify } from 'aws-amplify';
import { Authenticator } from '@aws-amplify/ui-react';
import '@aws-amplify/ui-react/styles.css';

// Configure SDK connection with Amazon Cognito via environment variables injected by Amplify
Amplify.configure({
  Auth: {
    Cognito: {
      userPoolId: import.meta.env.VITE_COGNITO_USER_POOL_ID,
      userPoolClientId: import.meta.env.VITE_COGNITO_USER_POOL_CLIENT_ID,
    }
  }
});

export default function App() {
  return (
    <Authenticator>
      {({ signOut, user }) => (
        <main style={{ padding: '20px' }}>
          <h1>Welcome {user?.username} to Smart Media Analytics!</h1>
          <button onClick={signOut}>Sign Out</button>
          
          {/* The application's main Dashboard interface will render here after a valid login */}
          
        </main>
      )}
    </Authenticator>
  );
}
```

#### Step 4: Trigger Redeployment (Redeploy)
For all new environment variable configurations and integrated source code to take effect on the Cloud environment, the Build process must be restarted.

1. Return to the **Deployments** screen of the application on the Amplify Console.
2. Click on the **`main`** branch.
3. Looking at the top right corner, click the **Redeploy this version** button to re-trigger the CI/CD process.
4. Wait for the system to complete all 4 phases and transition the status to a green **Deployed**.

![Amplify Redeploy Success](/images/5-Workshop/5.11-Frontend-deployment/5.11.3-connect-cognito-frontend/5.11.3-amplify-success.png)

At this juncture, when accessing the application's domain URL, the entire interface is protected by the authentication system. Users are required to log in or create an account via Cognito's auto-generated UI Form before they can access the primary Dashboard.

![Cognito Login UI](/images/5-Workshop/5.11-Frontend-deployment/5.11.3-connect-cognito-frontend/5.11.3-login-ui.png)

***

**Chapter 5.11 Summary:** By combining AWS Amplify Hosting, Amazon Route 53, and Amazon Cognito, the project team has finalized a comprehensive Serverless Frontend architecture. The system not only possesses automated deployment capabilities (CI/CD) but also ensures absolute access security at the end-user level with a professional authentication interface.

***

**Next Step:** The Frontend system has successfully interconnected with the Backend. However, the current CI/CD Build procedure relies entirely on Amplify's rudimentary flow. In the subsequent chapter ([**Chapter 5.12: CI/CD Pipeline**](../../5.12-CICD/)), the project team will standardize the Continuous Delivery pipeline for the entire ecosystem utilizing AWS CodePipeline.
