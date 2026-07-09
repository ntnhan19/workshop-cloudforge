---
title : "Chuẩn bị & Phân quyền"
date : 2026-07-09
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### 1. Tài khoản AWS & Các Quyền hạn (IAM Roles)

Để hệ thống hoạt động tự động, chúng ta cần tạo trước các "thẻ bài" (IAM Roles) để các dịch vụ có thể tự do nói chuyện với nhau một cách bảo mật nhất. Trong khuôn khổ bài thực hành này, khuyến nghị user của bạn có quyền `AdministratorAccess` để dễ dàng khởi tạo các dịch vụ, và sau đó tạo các Role chuyên biệt sau:

- **ECS-Backend-TaskRole:** Cấp quyền cho cụm API Backend được gọi Bedrock (để chuyển câu query tìm kiếm thành Vector), đọc/ghi S3 và truy cập mạng nội bộ.
- **ECS-Worker-TaskRole:** Cấp quyền cho tác vụ xử lý AI nặng được phép gọi Bedrock, Transcribe, đọc/ghi S3 và Publish tiến độ vào Redis.
- **StepFunctions-Orchestrator:** Cấp quyền cho Step Functions được phép kích hoạt cụm ECS Worker (API RunTask) và đọc thông điệp từ SQS.

*📸 Góc chụp: Màn hình danh sách Roles trong IAM khi gõ tìm kiếm tên dự án.*
![IAM Roles](../../images/5-Workshop/5.2-Prerequisites/iam_permissions.png)

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