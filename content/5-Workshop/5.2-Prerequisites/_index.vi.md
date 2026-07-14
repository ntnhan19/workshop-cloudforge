---
title : "Chuẩn bị & Phân quyền"
date : 2026-07-09
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

Để tuân thủ nguyên tắc **Đặc quyền tối thiểu (Least Privilege)** và đảm bảo giao tiếp an toàn giữa các Microservices, tiến hành thiết lập các Nhóm người dùng (IAM Groups) và các Roles (JSON-based) cực kỳ chi tiết cho các container chạy trên ECS Fargate cũng như Step Functions.

#### 0. Tạo IAM Groups & Users (Môi trường làm việc)

Trước khi tạo các roles cho dịch vụ, hãy thiết lập các nhóm IAM để quản lý quyền truy cập Console cho các thành viên trong đội.

1. Vào **IAM > User groups > Create group**.
2. Tạo nhóm quản trị viên (Admin):
   - **Tên nhóm (User group name):** `CloudForge-Admins`
   - **Quyền hạn (Permissions policy):** `AdministratorAccess`
   
   ![Create Admin Group](/images/5-Workshop/5.2-Prerequisites/iam_group_admin.png)

3. Tạo nhóm lập trình viên (Dev - cho các thành viên cần deploy nhưng không có quyền xóa các tài nguyên lõi):
   - **Tên nhóm (User group name):** `CloudForge-Developers`
   - **Quyền hạn (Permissions policy):** `PowerUserAccess`
   
   ![Create Dev Group](/images/5-Workshop/5.2-Prerequisites/iam_group_dev.png)

4. Vào **IAM > Users > Create user**, tạo tài khoản cho các thành viên (ví dụ: `cf-vy`, `cf-luan`) và đưa họ vào các group tương ứng.

   ![Assign Users](/images/5-Workshop/5.2-Prerequisites/iam_assign_users.png)

#### 1. Khởi tạo IAM Roles cho Tính toán & Điều phối

Trong AWS ECS Fargate, một container đòi hỏi 2 loại Role: **Execution Role** (Để bản thân cái máy ảo Fargate tải Image và gửi logs) và **Task Role** (Để code bên trong Container được phép gọi tới các dịch vụ AWS khác).

**Role 1: ECS Task Execution Role (Cấp hệ thống)**
1. Vào **IAM > Roles > Create role**. Chọn **AWS service > Elastic Container Service > Elastic Container Service Task**.
2. Gắn policy có sẵn: `AmazonECSTaskExecutionRolePolicy`.
3. Đặt tên là `ecsTaskExecutionRole` và bấm Create.

   ![ECS Execution Role](/images/5-Workshop/5.2-Prerequisites/iam_ecs_execution_role.png)

**Role 2: ECS-Backend-TaskRole (API & Vector Search)**
Role này cho phép ứng dụng FastAPI/Node.js chuyển văn bản thành Vector qua Bedrock và đọc/ghi S3.
1. Vào **IAM > Policies > Create policy (Tab JSON)**.
2. Dán đoạn JSON sau:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowS3Access",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::cloudforge-media-storage/*"
        },
        {
            "Sid": "AllowBedrockVectorGeneration",
            "Effect": "Allow",
            "Action": "bedrock:InvokeModel",
            "Resource": "*"
        }
    ]
}
```
3. Đặt tên Policy là `cloudforge-backend-policy` và lưu lại.
4. Vào **Roles > Create role** (Dịch vụ: Elastic Container Service Task). Gắn `cloudforge-backend-policy` vào.
5. Đặt tên Role là `ECS-Backend-TaskRole`.

![ECS Backend Role](/images/5-Workshop/5.2-Prerequisites/iam_ecs_backend_role.png)

**Role 3: ECS-Worker-TaskRole (Xử lý AI Nặng)**
Role này cho phép Worker chạy ngầm cắt cảnh, gọi Bedrock (phân tích đa phương thức) và gọi Transcribe (âm thanh).

1. Vào **IAM > Policies > Create policy (Tab JSON)**.
2. Dán đoạn JSON sau:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowFullS3MediaAccess",
            "Effect": "Allow",
            "Action": ["s3:GetObject", "s3:PutObject", "s3:ListBucket"],
            "Resource": ["arn:aws:s3:::cloudforge-media-storage", "arn:aws:s3:::cloudforge-media-storage/*"]
        },
        {
            "Sid": "AllowAIAndMLServices",
            "Effect": "Allow",
            "Action": [
                "bedrock:InvokeModel",
                "transcribe:StartTranscriptionJob",
                "transcribe:GetTranscriptionJob"
            ],
            "Resource": "*"
        }
    ]
}
```
3. Đặt tên Policy là `cloudforge-ai-worker-policy`. Tạo xong thì gắn vào một Role mới tên là `ECS-Worker-TaskRole`.

![ECS Worker Role](/images/5-Workshop/5.2-Prerequisites/iam_ecs_worker_role.png)

**Role 4: StepFunctions-Orchestrator**
Role này cho phép Step Functions State Machine kích hoạt (trigger) ECS AI Worker và lấy thông điệp từ SQS.

1. Vào **IAM > Policies > Create policy (Tab JSON)**.
2. Dán đoạn JSON sau:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowRunECSTask",
            "Effect": "Allow",
            "Action": "ecs:RunTask",
            "Resource": "arn:aws:ecs:*:*:task-definition/cloudforge-ai-worker:*"
        },
        {
            "Sid": "AllowPassRoleToECS",
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": [
                "arn:aws:iam::*:role/ecsTaskExecutionRole",
                "arn:aws:iam::*:role/ECS-Worker-TaskRole"
            ]
        },
        {
            "Sid": "AllowSQSReceive",
            "Effect": "Allow",
            "Action": [
                "sqs:ReceiveMessage",
                "sqs:DeleteMessage",
                "sqs:GetQueueAttributes"
            ],
            "Resource": "arn:aws:sqs:*:*:cloudforge-ingest-queue"
        }
    ]
}
```
3. Đặt tên Policy là `cloudforge-orchestrator-policy`.
4. Tạo một Role (Trusted Entity: Step Functions) tên là `StepFunctions-Orchestrator` và gắn policy này vào.

![Final IAM Roles](/images/5-Workshop/5.2-Prerequisites/iam_final_roles.png)

#### 2. Cài đặt môi trường Local
Trước khi triển khai lên Cloud, hãy chắc chắn máy tính của hệ thống đã được cài đặt các công cụ sau:

- **Git:** Để clone mã nguồn dự án.
- **Docker:** Để build các image cho Frontend và Backend trước khi đẩy lên Amazon ECR.
- **AWS CLI:** Đã cấu hình xác thực để gọi API đến AWS.

Kiểm tra cấu hình AWS CLI của hệ thống:
```bash
aws configure
aws sts get-caller-identity
```
![AWS CLI Config](/images/5-Workshop/5.2-Prerequisites/aws_cli_config.png)

#### 3. Quyền truy cập Amazon Bedrock Model (Bản cập nhật mới của AWS)

Trước đây, Amazon Bedrock yêu cầu người dùng phải kích hoạt thủ công cho các mô hình nền tảng (foundation models) cụ thể. Tuy nhiên, theo bản cập nhật mới nhất từ AWS, trang Model Access đã chính thức bị loại bỏ.

Các mô hình nền tảng Serverless (bao gồm **Nova Lite** và **Titan Embeddings** được sử dụng trong dự án của hệ thống) giờ đây sẽ được **tự động kích hoạt (automatically enabled)** trên tất cả các khu vực AWS thương mại ngay trong lần gọi API đầu tiên của tài khoản. Do đó, bạn không cần phải thực hiện bất kỳ thao tác thủ công nào trên AWS Console ở bước này nữa.

![Bedrock Model Access Auto](/images/5-Workshop/5.2-Prerequisites/bedrock_access_retired.png)

---
**Bước tiếp theo:** Chuyển sang phần **[VPC & Networking](../5.3-network-vpc/)** để xây dựng nền móng mạng nội bộ an toàn.
