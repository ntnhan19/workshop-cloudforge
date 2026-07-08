---
title: "Workshop"
date: 2024-01-01
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

1. [Tổng quan Kiến trúc mạng](5.1-Architecture-overview/)
2. [Chuẩn bị & Phân quyền IAM](5.2-Prerequisites/)
3. [Triển khai VPC & S3 Gateway](5.3-Network-vpc/)
4. [Triển khai Database (RDS & Redis)](5.4-Database-setup/)
5. [Triển khai Ứng dụng (ECS & App Runner)](5.5-Compute-setup/)
6. [Dọn dẹp tài nguyên](5.6-Cleanup/)