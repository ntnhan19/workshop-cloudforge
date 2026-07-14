---
title : "Triển khai ECS AI Worker"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.7.3. </b> "
---

Trái ngược với hệ thống Backend API chịu trách nhiệm tiếp nhận và phản hồi luồng yêu cầu trực tiếp từ người dùng cuối, hệ thống **AI Worker** hoạt động như một trình xử lý nền (Background Processor) của toàn bộ dự án. Hệ thống này hoạt động hoàn toàn tách biệt, không mở bất kỳ cổng kết nối nào ra ngoài Internet. Thay vào đó, nó liên tục thực hiện cơ chế dò tìm (Polling) thông điệp sự kiện từ hàng đợi Amazon SQS, từ đó kích hoạt các chu trình phân tích Media và tương tác với các mô hình trí tuệ nhân tạo.

Trong phân đoạn này, tiến hành tiến hành đóng gói mã nguồn AI Worker, đẩy lên kho lưu trữ Amazon ECR Private và triển khai dưới dạng một **Background Service** cô lập trên hạ tầng không máy chủ AWS Fargate.

#### 1. Đóng gói và đẩy mã nguồn lên Amazon ECR
Quá trình đóng gói và biên dịch này được thực hiện trực tiếp tại thư mục mã nguồn chuyên trách của AI Worker và ánh xạ về kho lưu trữ `cloudforge-ai-worker` đã tạo.

1. Mở giao diện Terminal tích hợp trong IDE tại thư mục gốc của dự án.
2. Thực hiện đăng nhập xác thực vào Amazon ECR (Nếu phiên làm việc trước đó của AWS CLI đã hết hạn):
   ```bash
   aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 236320489525.dkr.ecr.ap-southeast-1.amazonaws.com
   ```
3. Di chuyển vào thư mục chứa mã nguồn của AI Worker:
   ```bash
   cd ai-worker
   ```
4. Biên dịch Docker Image dựa trên cấu hình Dockerfile chuyên dụng và gắn thẻ chỉ định:
   ```bash
   docker build -t 236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-ai-worker:latest .
   ```
5. Thực hiện đẩy Image hoàn chỉnh lên đám mây ECR:
   ```bash
   docker push 236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-ai-worker:latest
   ```

![ECR Worker Image Pushed](/images/5-Workshop/5.7-Compute-setup/5.7.3-deploy-ecs-ai-worker/ecr_worker_image_pushed.png)

#### 2. Thiết lập cấu hình tác vụ (Task Definition)
Do đặc thù phải thực thi các tác vụ tính toán liên quan đến xử lý luồng dữ liệu Media và chạy các thuật toán mô hình học máy, cấu hình phần cứng ảo của AI Worker sẽ được thiết lập cao hơn so với tầng API thông thường.

1. Truy cập dịch vụ **Amazon ECS** → **Task definitions** → Bấm **Create new task definition**.
2. **Task definition configuration:** Thiết lập tên định danh `cloudforge-ai-worker-task`.
3. **Infrastructure requirements:**
   - **Launch type:** Chọn **AWS Fargate**.
   - **Task size:** Cấp phát tài nguyên tối ưu xử lý bao gồm `1 vCPU` cho bộ vi xử lý và `2 GB` cho dung lượng RAM.
   - **Task execution role:** Chỉ định vai trò `ecsTaskExecutionRole` sở hữu chính sách đính kèm quyền truy cập Secrets Manager nhằm nạp các biến môi trường hệ thống.
   - **Task role (Quan trọng):** Bấm chọn chính xác IAM Role **`ECS-Worker-TaskRole`** đã được cấu hình cấp sẵn các quyền bảo mật cốt lõi bao gồm: Quyền giao tiếp hàng đợi Amazon SQS, xử lý file trên Amazon S3, cập nhật Database, và đặc biệt là quyền gọi các dịch vụ AI như Amazon Bedrock và Transcribe.

![Worker Task Role](/images/5-Workshop/5.7-Compute-setup/5.7.3-deploy-ecs-ai-worker/worker_task_role.png)
4. **Container configuration:**
   - **Container name:** Định danh tên `worker-container`.
   - **Image URI:** Dán liên kết chính xác từ ECR: `236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-ai-worker:latest`.
   - **Port mappings:** BỎ TRỐNG HOÀN TOÀN. Do đặc thù Worker hoạt động theo mô hình Outbound (Chủ động gọi ra ngoài hệ thống để lấy việc), việc không mở cổng Inbound giúp triệt tiêu hoàn toàn bề mặt tấn công mạng (Attack Surface).
5. **Environment variables (Biến môi trường):**
   - Khai báo các biến môi trường cấu hình bắt buộc cho AI Worker hoạt động:
     - `AWS_DEFAULT_REGION`: `ap-southeast-1`
     - `AWS_S3_BUCKET`: `cloudforge-media-upload-ntnhan19`
     - `DATABASE_URL`: `postgresql+psycopg://postgres:<mật-khẩu-của-bạn>@<endpoint-rds-của-bạn>:5432/cloudforge_db`
     - `PYTHONPATH`: `/app`
     - `STORAGE_BACKEND`: `s3`

![Worker Environment](/images/5-Workshop/5.7-Compute-setup/5.7.3-deploy-ecs-ai-worker/worker_environment.png)
6. Bấm **Create** để lưu cấu hình phiên bản V1.

![Worker Task Definition](/images/5-Workshop/5.7-Compute-setup/5.7.3-deploy-ecs-ai-worker/worker_task_definition.png)

#### 3. Triển khai vận hành AI Worker (Event-Driven)

Khác với Backend, **AI Worker** trong kiến trúc này được thiết kế theo mô hình **Event-Driven (Hướng sự kiện)**. Tiến hành **KHÔNG TẠO ECS SERVICE** cho AI Worker.

Lý do: AI Worker là một tác vụ dạng kịch bản (script) sẽ tự động tắt ngay sau khi xử lý xong một video (chạy một lần rồi thoát). Nếu bạn cố tình tạo ECS Service, hệ thống sẽ cố gắng giữ nó chạy liên tục, dẫn đến lỗi khởi động lại liên tục (Task exited with code 1 hoặc 0) và sẽ bị AWS kích hoạt tính năng **Service deployment circuit breaker**.

Thay vì chạy nền liên tục, AI Worker sẽ được gọi (Invoke) tự động thông qua **AWS Step Functions** (`ecs:RunTask`) mỗi khi có video mới tải lên S3 (như bạn đã cấu hình ở bước 5.6).

Bạn đã hoàn tất việc tạo **Task Definition** cho AI Worker. Hệ thống hạ tầng dành cho Worker hiện tại đã hoàn toàn sẵn sàng!

***

**Bước tiếp theo:** Toàn bộ hệ thống hạ tầng xử lý cốt lõi bao gồm Mạng lưới kết nối, Cơ sở dữ liệu lưu trữ, Điều phối sự kiện bất đồng bộ và Cụm Compute tính toán (ECS Backend & Worker) của dự án Smart Media Analytics đã được xây dựng và liên kết hoàn chỉnh. Tiến hành chuyển sang bài học tiếp theo để cấu hình khả năng mở rộng hệ thống với [**Thiết lập Auto Scaling**](../5.7.4-configure-auto-scaling/).
