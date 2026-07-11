---
title: "Workshop"
date: 2026-07-09
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Triển khai Kiến trúc Smart Media Analytics trên AWS

#### Tổng quan

**Smart Media Analytics** (phát triển bởi nhóm **CloudForge**) là một hệ thống phân tích media thông minh End-to-End. Để đưa một hệ thống đồ sộ như thế này lên Cloud, chúng ta cần một hạ tầng mạng linh hoạt, bảo mật cao và tự động mở rộng bằng cách kết hợp các dịch vụ Serverless và Container của AWS.

Trong bài thực hành này, bạn sẽ học cách triển khai toàn bộ nền tảng Smart Media Analytics lên AWS. Chúng ta sẽ bắt đầu bằng việc thiết kế mạng VPC bảo mật, sau đó triển khai cụm pipeline xử lý AI (AI Pipeline) trên **Amazon ECS (Fargate)**, thiết lập cơ sở dữ liệu vector với **RDS PostgreSQL**, và cuối cùng là khởi chạy giao diện người dùng thông qua **AWS App Runner**.

Điểm nhấn của workshop này là các mô hình bảo mật như việc đặt toàn bộ Compute ở **Private Subnet** và truy cập Amazon S3 hoàn toàn thông qua mạng nội bộ nhờ vào **S3 Gateway Endpoint**.

#### Nội dung

1. [Tổng quan Kiến trúc](5.1-Architecture-overview/)
2. [Chuẩn bị & Phân quyền IAM](5.2-Prerequisites/)
3. [Nền móng Mạng nội bộ (VPC)](5.3-Network-vpc/)
4. [Lưu trữ & Dữ liệu (S3, RDS, Redis)](5.4-Database-setup/)
5. [Bảo mật (Cognito, Secrets Manager, WAF)](5.5-Security-setup/)
6. [Điều phối Workflow (SQS, EventBridge, Step Functions)](5.6-Ingestion-workflow/)
7. [Tính toán (ECS Backend & AI Worker)](5.7-Compute-setup/)
8. [Tích hợp AI/ML (Bedrock & Transcribe)](5.8-AI-ML-integration/)
9. [API & Real-time (API Gateway, ALB, WebSocket)](5.9-API-and-realtime/)
10. [Tìm kiếm Ngữ nghĩa (Semantic Search)](5.10-Semantic-search/)
11. [Triển khai Frontend (Amplify, Route53)](5.11-Frontend-deployment/)
12. [CI/CD (GitHub Actions, ECR)](5.12-CICD/)
13. [Giám sát Hệ thống (CloudWatch, X-Ray)](5.13-Observability/)
14. [Dọn dẹp tài nguyên](5.14-Cleanup/)
