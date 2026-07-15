---
title : "System Walkthrough (Test the Application)"
date : 2026-07-14
weight : 14
chapter : false
pre : " <b> 5.14. </b> "
---

After successfully deploying all frontend and backend components, comprehensive end-to-end testing is conducted to validate the system from an end-user's perspective.

This section demonstrates how the Smart Media Analytics platform operates in a real-world scenario.

### 1. Registration & Login
1. The [Amplify Frontend URL](https://main.d10clqxr7o0tzf.amplifyapp.com/) deployed in Chapter 5.11 is accessed via a web browser.
2. In the **Sign Up** tab, a new account is created by filling in the required email, password, and personal details.
3. A **Confirmation Code** is securely sent by Amazon Cognito to the provided email for account verification.
4. Once verified, a login is performed to access the main Dashboard.

![Cognito Login and Signup](/images/5-Workshop/5.14-User-guide/5.14-login-ui.png)

### 2. Uploading Media (Ingestion)
1. The **Upload** section is navigated to in order to select a sample video file.
2. The frontend securely uploads the file directly to the S3 bucket using Pre-signed URLs, displaying a visual progress bar.

![Upload Media Progress](/images/5-Workshop/5.14-User-guide/5.14-upload-progress.png)

3. Once the upload finishes, the video appears in the gallery with a **Processing** status. Behind the scenes, EventBridge triggers Step Functions orchestration, and the ECS AI Worker spins up to extract and analyze the video frames.

![Video Processing Status](/images/5-Workshop/5.14-User-guide/5.14-processing-status.png)

4. After the AI pipeline finishes the analysis and indexing, the status automatically updates to **Completed**, and the media details become available.

![Video Processing Completed](/images/5-Workshop/5.14-User-guide/5.14-completed-status.png)

### 3. Asset Details & Editor Features
1. By clicking on a fully processed video, the Asset Detail view is opened.
2. A comprehensive interface is provided for editors, featuring automatic **Tag Classification** derived from the AI analysis.
3. A full **Transcript** of the video is also displayed, enabling quick review and extraction of spoken content.

![Asset Details, Tags and Transcript](/images/5-Workshop/5.14-User-guide/5.14-asset-detail.png)

### 4. Semantic Search (Search in Video)
1. Within the Asset Detail view, the **Search in Video** functionality allows for deep semantic exploration inside a specific video.
2. When a natural language query is entered (e.g., *"a tiger running in the snow"* or *"sunset over the ocean"*), the backend leverages Amazon Titan vector embeddings and PostgreSQL `pgvector` to identify semantic similarities.
3. The system instantly returns the exact scenes along with precise **timestamps** matching the description.
4. Clicking on a returned scene seamlessly plays the video right from that exact moment.

![Search in Video Results](/images/5-Workshop/5.14-User-guide/5.14-search-in-video.png)

---

**Next Step:** With the intelligent, event-driven architecture successfully built and validated, the final phase is cleaning up the environment. [**Chapter 5.15: Resource Cleanup**](../5.15-Cleanup/) outlines the steps taken to decommission the infrastructure and prevent unintended AWS charges.
