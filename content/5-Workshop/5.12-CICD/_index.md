---
title : "CI/CD Pipeline"
date : 2026-07-10
weight : 12
chapter : false
pre : " <b> 5.12. </b> "
---

### CI/CD Overview

Manual deployment is not only time-consuming but also prone to human error. To achieve DevOps standards, we need to establish a Continuous Integration and Continuous Deployment (CI/CD) pipeline.

In this chapter, we will use **GitHub Actions** (or AWS CodePipeline) to:
1. Automatically run Unit Tests whenever new source code is pushed.
2. Automatically build Docker Images and push them to Amazon ECR.
3. Automatically update the Task Definition on ECS to perform zero-downtime Rolling Updates.

*(Detailed content will be updated in the sub-articles).*
