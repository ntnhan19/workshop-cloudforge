---
title: "Workshop"
date: 2026-07-09
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Triển khai Kiến trúc Smart Media Analytics trên AWS

#### Tổng quan

**Smart Media Analytics** (phát triển bởi nhóm **CloudForge**) là một hệ thống phân tích media thông minh End-to-End. Để đưa một hệ thống đồ sộ như thế này lên Cloud, cần một hạ tầng mạng linh hoạt, bảo mật cao và tự động mở rộng bằng cách kết hợp các dịch vụ Serverless và Container của AWS.

Bài workshop này là tài liệu ghi nhận lại **toàn bộ quá trình thực tế** mà đội ngũ CloudForge đã trải qua khi thiết kế và đưa dự án này lên môi trường AWS. Xuyên suốt workshop, chúng tiến hành thuật lại cách nhóm xây dựng một nền móng mạng VPC bảo mật, triển khai cụm pipeline xử lý AI (AI Pipeline) trên **Amazon ECS (Fargate)**, thiết lập cơ sở dữ liệu vector với **RDS PostgreSQL**, và cuối cùng là khởi chạy giao diện người dùng tự động thông qua **AWS Amplify**.

Điểm nhấn của hệ thống là các mô hình bảo mật và tối ưu chi phí, chẳng hạn như việc đặt toàn bộ các dịch vụ tính toán (Compute) ở **Private Subnet** và truy cập Amazon S3 hoàn toàn thông qua mạng nội bộ nhờ vào **S3 Gateway Endpoint**.

#### Tài nguyên dự án (Resources)
- 🌐 **Live Demo:** [https://main.d10clqxr7o0tzf.amplifyapp.com/](https://main.d10clqxr7o0tzf.amplifyapp.com/)
- 💻 **Source Code (GitHub):** [https://github.com/ntnhan19/smart_media_analytics_cloudforge](https://github.com/ntnhan19/smart_media_analytics_cloudforge)

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
12. [CI/CD tự động (GitHub Actions, ECR)](5.12-CICD/)
13. [Quan sát hệ thống (CloudWatch, X-Ray)](5.13-Observability/)
14. [Hướng dẫn Sử dụng (Test the Application)](5.14-User-guide/)
15. [Dọn dẹp tài nguyên](5.15-Cleanup/)
