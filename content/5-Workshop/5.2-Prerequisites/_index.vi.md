---
title : "Chuẩn bị & Phân quyền"
date : 2026-07-08
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### 1. Tài khoản AWS & Quyền IAM

Để triển khai dự án **Smart Media Analytics**, bạn cần một tài khoản AWS đang hoạt động. Trong khuôn khổ bài thực hành này, khuyến nghị sử dụng IAM User hoặc Role có quyền `AdministratorAccess` để dễ dàng khởi tạo các dịch vụ (VPC, ECS, S3, RDS, Bedrock, v.v.).

Nếu tài khoản của bạn bị giới hạn, hãy đảm bảo bạn có quyền tạo và quản lý các nhóm dịch vụ sau:
- **Mạng & API (Networking & API):** VPC, ALB (Application Load Balancer), API Gateway, Route 53.
- **Tính toán & Điều phối (Compute & Orchestration):** ECS Fargate, Step Functions, EventBridge.
- **Lưu trữ & Dữ liệu (Storage & Data):** S3, RDS PostgreSQL, ElastiCache for Redis, SQS.
- **Dịch vụ AI (AI/ML):** Amazon Bedrock, Amazon Transcribe.
- **Frontend, Bảo mật & Khác:** Amplify, Cognito, Secrets Manager, ECR, CloudWatch, X-Ray.

*Chèn ảnh chụp màn hình cấu hình quyền IAM của bạn tại đây:*
![Quyền IAM](../../images/5-Workshop/5.2-Prerequisites/iam_permissions.png)

#### 2. Cài đặt môi trường Local

Trước khi đẩy mọi thứ lên Cloud, hãy chắc chắn máy tính của bạn đã được cài đặt các công cụ sau:
- **Git**: Để clone mã nguồn dự án.
- **Docker**: Để build các image cho Frontend và Backend trước khi đẩy lên Amazon ECR.
- **AWS CLI**: Đã được cấu hình (configure) thông tin xác thực để chạy lệnh từ Terminal.

Kiểm tra cấu hình AWS CLI của bạn bằng lệnh:
```bash
aws configure
aws sts get-caller-identity
```

*Chèn ảnh chụp màn hình Terminal hiển thị kết nối AWS CLI thành công tại đây:*
![Cấu hình AWS CLI](../../images/5-Workshop/5.2-Prerequisites/aws_cli_config.png)

#### 3. Cấp quyền truy cập Amazon Bedrock (Model Access)

Vì dự án sử dụng **Amazon Bedrock (mô hình Nova Lite & Titan Embeddings)**, bạn bắt buộc phải yêu cầu cấp quyền sử dụng các model này trên AWS Console (chúng không được bật mặc định).

1. Truy cập vào giao diện **Amazon Bedrock**.
2. Ở thanh menu bên trái, chọn **Model access**.
3. Bấm **Manage model access** và tick chọn **Nova Lite** cùng **Titan Embeddings**.
4. Bấm **Save changes** và chờ đến khi trạng thái chuyển sang `Access granted`.

*Chèn ảnh chụp màn hình trang Model Access của Amazon Bedrock hiển thị quyền đã được cấp tại đây:*
![Quyền truy cập Bedrock](../../images/5-Workshop/5.2-Prerequisites/bedrock_access.png)