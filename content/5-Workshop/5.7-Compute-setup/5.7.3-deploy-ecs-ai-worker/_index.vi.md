---
title : "Triển khai ECS AI Worker"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.7.3. </b> "
---

Trái ngược với hệ thống Backend API chịu trách nhiệm tiếp nhận và phản hồi luồng yêu cầu trực tiếp từ người dùng cuối, hệ thống **AI Worker** hoạt động như một trình xử lý nền (Background Processor) của toàn bộ dự án. Hệ thống này hoạt động hoàn toàn tách biệt, không mở bất kỳ cổng kết nối nào ra ngoài Internet. Thay vào đó, nó liên tục thực hiện cơ chế dò tìm (Polling) thông điệp sự kiện từ hàng đợi Amazon SQS, từ đó kích hoạt các chu trình phân tích Media và tương tác với các mô hình trí tuệ nhân tạo.

Trong phân đoạn này, chúng ta sẽ tiến hành đóng gói mã nguồn AI Worker, đẩy lên kho lưu trữ Amazon ECR Private và triển khai dưới dạng một **Background Service** cô lập trên hạ tầng không máy chủ AWS Fargate.

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
   - **Task role (Quan trọng):** Bấm chọn IAM Role dành riêng cho ứng dụng ngầm (ví dụ: `cloudforge-ecs-app-role`) đã được cấp sẵn các quyền bảo mật cốt lõi bao gồm: Quyền đọc/xóa thông điệp từ Amazon SQS (`sqs:ReceiveMessage`, `sqs:DeleteMessage`), quyền ghi/đọc tài nguyên số từ Amazon S3, và quyền cập nhật trạng thái vào cơ sở dữ liệu.
4. **Container configuration:**
   - **Container name:** Định danh tên `worker-container`.
   - **Image URI:** Dán liên kết chính xác từ ECR: `236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-ai-worker:latest`.
   - **Port mappings:** BỎ TRỐNG HOÀN TOÀN. Do đặc thù Worker hoạt động theo mô hình Outbound (Chủ động gọi ra ngoài hệ thống để lấy việc), việc không mở cổng Inbound giúp triệt tiêu hoàn toàn bề mặt tấn công mạng (Attack Surface).
5. **Environment variables:**
   - Tiến hành nạp các liên kết động bao gồm đường dẫn hàng đợi `SQS_QUEUE_URL` và tên tài nguyên lưu trữ `S3_BUCKET_NAME` để mã nguồn ứng dụng tự động nhận diện điểm đến.
6. Bấm **Create** để lưu cấu hình phiên bản V1.

![Worker Task Definition](/images/5-Workshop/5.7-Compute-setup/5.7.3-deploy-ecs-ai-worker/worker_task_definition.png)

#### 3. Triển khai vận hành ECS Service (Background Worker)
Vì AI Worker không sử dụng bộ cân bằng tải Load Balancer để tiếp nhận Traffic, chu trình khởi tạo cấu hình Service trên mạng lưới sẽ được tinh giản tối đa, tập trung vào tính cô lập hạ tầng.

1. Quay lại trang điều khiển **Amazon ECS**, truy cập Cluster dữ liệu `cloudforge-compute-cluster`.
2. Tại phân hệ **Services**, bấm chọn hành động **Create**.
3. **Environment:** Đảm bảo tùy chọn hạ tầng là Launch type kết hợp với **FARGATE**.
4. **Deployment configuration:**
   - **Application type:** Chọn **Service**.
   - **Task definition:** Chọn Family tác vụ `cloudforge-ai-worker-task` vừa thiết lập với phiên bản mới nhất (`latest`).
   - **Service name:** Nhập tên dịch vụ định danh `cloudforge-ai-worker-service`.
   - **Desired tasks:** Đặt cấu hình khởi chạy mặc định ban đầu là `1` (Hệ thống có thể tự động mở rộng - Auto Scaling dựa trên số lượng tin nhắn tồn đọng trong hàng đợi SQS ở giai đoạn vận hành cao điểm).
5. **Networking (Thiết lập mạng cốt lõi):**
   - **VPC:** Chọn chính xác mạng nội bộ `cloudforge-vpc`.
   - **Subnets:** Tích chọn chính xác hệ thống các vùng mạng kín **Private Subnets**.
   - **Security group:** Bấm chọn nhóm bảo mật tập trung `cloudforge-ecs-app-sg` (Nhóm này chặn toàn bộ dữ liệu gọi vào từ bên ngoài nhưng cho phép tự do gọi Outbound ra Internet qua mạng NAT Gateway để kết nối tới SQS và S3).
   - **Public IP:** Chuyển trạng thái bắt buộc sang **Turned off**.
6. **Load balancing:** Tại phân hệ này, chọn trạng thái **None** (Không áp dụng bộ cân bằng tải).
7. Cuộn xuống cuối bảng điều khiển và bấm nút màu cam **Create**.

![Worker Service Success](/images/5-Workshop/5.7-Compute-setup/5.7.3-deploy-ecs-ai-worker/worker_service_success.png)

{{% notice tip %}}
**Architectural Note (Decoupled Architecture):** Mô hình thiết kế này phản ánh đặc tính của kiến trúc Hướng sự kiện (Event-Driven Architecture) với tính chất liên kết lỏng lẻo (Loosely Coupled). Hệ thống tiếp nhận (S3/EventBridge) và hệ thống xử lý tính toán (AI Worker) không có kết nối trực tiếp về mặt hạ tầng mạng. Amazon SQS đóng vai trò bộ đệm (Buffer) trung gian. Nếu hệ thống tiếp nhận hàng ngàn tệp tin đa phương tiện cùng một thời điểm, SQS sẽ lưu trữ an toàn trong hàng đợi để AI Worker lấy dữ liệu và xử lý tuần tự, loại bỏ rủi ro nghẽn mạch hay sập hệ thống do quá tải tài nguyên.
{{% /notice %}}

***

**Bước tiếp theo:** Toàn bộ hệ thống hạ tầng xử lý cốt lõi bao gồm Mạng lưới kết nối, Cơ sở dữ liệu lưu trữ, Điều phối sự kiện bất đồng bộ và Cụm Compute tính toán (ECS Backend & Worker) của dự án Smart Media Analytics đã được xây dựng và liên kết hoàn chỉnh. Chúng ta sẽ chuyển sang chương tiếp theo để thực hiện **Triển khai Frontend** và cấu hình giao diện tương tác trực quan cho người dùng cuối.
