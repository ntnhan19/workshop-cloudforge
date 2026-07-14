---
title : "Triển khai ECS Backend"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.7.2. </b> "
---

Tầng Backend API đóng vai trò là lõi điều hướng, chịu trách nhiệm tiếp nhận luồng yêu cầu HTTP/HTTPS từ người dùng, thực thi logic nghiệp vụ và tương tác với các phân lớp cơ sở dữ liệu, hàng đợi thông điệp. Trong phân đoạn này, chúng tiến hành thực hiện đóng gói mã nguồn Backend thành Docker Image, đẩy lên kho lưu trữ Amazon ECR, cấu hình bộ cân bằng tải Application Load Balancer (ALB) và triển khai ứng dụng trên nền tảng Serverless AWS Fargate.

#### 1. Đóng gói và đẩy mã nguồn lên Amazon ECR
Để chuyển giao mã nguồn cục bộ lên hạ tầng Cloud, chúng ta tiến hành biên dịch ứng dụng thành Docker Image và đẩy vào kho lưu trữ `cloudforge-backend` đã khởi tạo ở bài trước thông qua công cụ dòng lệnh (AWS CLI).

1. Mở Terminal tại thư mục gốc của toàn bộ dự án trên máy tính cục bộ.
2. Thực hiện đăng nhập và xác thực quyền truy cập vào Amazon ECR Private Registry của tài khoản:
   ```bash
   aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 236320489525.dkr.ecr.ap-southeast-1.amazonaws.com
   ```
3. KHÔNG CẦN di chuyển vào thư mục backend, hãy đứng ngay tại thư mục gốc của dự án (nơi thấy 2 thư mục `backend` và `ai_pipeline`).
4. Tiến hành biên dịch (Build) Docker Image bằng cách chỉ định đường dẫn file Dockerfile và context là thư mục gốc:
   ```bash
   docker build -t 236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-backend:latest -f backend/Dockerfile .
   ```
5. Đẩy (Push) sản phẩm đã đóng gói lên đám mây:
   ```bash
   docker push 236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-backend:latest
   ```

![ECR Image Pushed](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/ecr_image_pushed.png)

#### 2. Khởi tạo Application Load Balancer (ALB)
Do các Container của ứng dụng Backend sẽ được cô lập hoàn toàn bên trong vùng mạng Private Subnet nhằm bảo mật, cần thiết lập một bộ cân bằng tải Application Load Balancer (ALB) nằm tại vùng mạng Public Subnets đóng vai trò làm lá chắn trung gian tiếp nhận và phân phối lưu lượng từ Internet.

**Bước 1: Khởi tạo Target Group**
1. Truy cập dịch vụ **EC2** → **Target Groups** → **Create target group**.
2. **Choose a target type:** Chọn **IP addresses** (Đây là bắt buộc đối với kiến trúc AWS Fargate vì mỗi Task sẽ sở hữu một địa chỉ IP nội bộ riêng biệt thuộc Elastic Network Interface - ENI).
3. **Target group name:** Nhập `cloudforge-backend-tg`.
4. **Protocol & Port:** Chọn HTTP và cấu hình cổng `8000`.
5. **VPC:** Chọn đúng `cloudforge-vpc`.
6. Phần **Health checks**, thay đổi **Health check path** từ `/` thành `/health` (vì Backend kiểm tra sức khỏe ở đường dẫn này). Sau đó bấm **Next** → **Create target group** (Bỏ qua bước đăng ký IP thủ công vì ECS Service sẽ tự động quản lý luồng này).

![Target Group Created](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/target_group_created.png)

**Bước 2: Cấu hình Load Balancer**
1. Tại menu trái EC2, chọn **Load Balancers** → Bấm **Create load balancer** → Chọn **Application Load Balancer**.
2. **Load balancer name:** Nhập `cloudforge-backend-alb`.
3. **Scheme:** Tích chọn **Internet-facing** để cho phép tiếp nhận traffic công cộng.
4. **Network mapping:**
   - Chọn VPC của dự án (`cloudforge-vpc`).
   - Tại mục Mappings, tích chọn cả 2 vùng Availability Zone (`ap-southeast-1a` và `ap-southeast-1b`), đồng thời ánh xạ chính xác vào các **Public Subnets** tương ứng của từng Zone.
5. **Security groups:** Chọn Security Group **`cloudforge-alb-sg`** (Đây là SG đã cấu hình từ trước cho phép tiếp nhận lưu lượng HTTP/HTTPS từ mọi nguồn Internet).
6. **Listeners and routing:** Tại giao thức HTTP Port 80, mục Default action chọn chuyển tiếp (Forward to) về Target Group `cloudforge-backend-tg` vừa tạo ở Bước 1.

![ALB Routing Config](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/alb_routing_config1.png)

![ALB Routing Config](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/alb_routing_config2.png)
7. Bấm **Create load balancer**.

![ALB Created](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/alb_created.png)

#### 3. Thiết lập cấu hình tác vụ (Task Definition)
Task Definition đóng vai trò là một bản thiết kế kiến trúc (Blueprint) quy định chi tiết về giới hạn phần cứng, hình ảnh Docker sử dụng và các tham số bảo mật của ứng dụng.

1. Truy cập dịch vụ **Amazon ECS** → **Task definitions** → Bấm **Create new task definition**.
2. **Task definition configuration:** Định danh tên `cloudforge-backend-task`.
3. **Infrastructure requirements:**
   - **Launch type:** Chọn **AWS Fargate**.
   - **Task size:** Chỉ định tài nguyên tối ưu chi phí bao gồm `0.5 vCPU` và `1 GB` Memory.
   - **Task role:** Chọn IAM Role dành riêng cho ứng dụng Backend (ví dụ: `ECS-Backend-TaskRole`). *(Lưu ý cực kỳ quan trọng: Đảm bảo IAM Role này đã được đính kèm Policy cấp quyền `s3:PutObject`, `s3:DeleteObject` vào đúng bucket của hệ thống. Nếu thiếu, tiến trình upload file sẽ báo lỗi AccessDenied)*.
   - **Task execution role:** Chọn tạo mới hoặc chỉ định `ecsTaskExecutionRole` (Cấp quyền cho tác vụ kéo ảnh từ ECR và đẩy logs về CloudWatch).
4. **Container configuration:**
   - **Container name:** `backend-container`.
   - **Image URI:** Dán chuỗi liên kết ECR chính xác: `236320489525.dkr.ecr.ap-southeast-1.amazonaws.com/cloudforge-backend:latest`.
   - **Port mappings:** Cấu hình Cổng Container là `8000`, Giao thức `TCP`, App protocol chọn `HTTP`.
5. **Environment variables (Biến môi trường):**
   - Khai báo các biến môi trường cấu hình bắt buộc cho Backend hoạt động:
     - `AWS_DEFAULT_REGION`: `ap-southeast-1`
     - `AWS_S3_BUCKET`: `cloudforge-media-upload-ntnhan19`
     - `DATABASE_URL`: `postgresql+psycopg://postgres:<mật-khẩu-của-bạn>@<endpoint-rds-của-bạn>:5432/cloudforge_db`
     - `REDIS_URL`: `redis://master.cloudforge-redis...` *(Hoặc `rediss://` nếu bạn đang bật In-Transit Encryption ở Redis)*.
     - `STORAGE_BACKEND`: `s3`

![Task Def Environment 1](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/task_def_environment1.png)
![Task Def Environment 2](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/task_def_environment2.png)
![Task Def Environment 3](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/task_def_environment3.png)
6. Bấm **Create**.

![Task Definition Created](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/task_definition_created.png)

#### 4. Triển khai vận hành ECS Service
Service đóng vai trò giữ điều phối, đảm bảo duy trì liên tục số lượng Container hoạt động ổn định và tự động kết nối chúng với bộ cân bằng tải ALB.

1. Tại bảng điều khiển **Amazon ECS**, chọn Cluster `cloudforge-compute-cluster`.
2. Chuyển sang tab **Services** → Bấm **Create**.
3. **Service details:**
   - **Task definition family:** Chọn `cloudforge-backend-task`.
   - **Task definition revision:** Để trống (Hệ thống sẽ tự động sử dụng phiên bản mới nhất - LATEST).
   - **Service name:** Đặt tên `cloudforge-backend-service`.

   ![ECS Service Details](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/ecs_service_details.png)
4. **Environment:**
   - **Compute options:** Chọn **Capacity provider strategy** với nhà cung cấp là **FARGATE** (hoặc cứ để nguyên mặc định của Cluster).

   ![ECS Service Environment](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/ecs_service_environment.png)
5. **Deployment configuration:**
   - **Desired tasks:** Nhập `2` (Khởi chạy 2 container đồng thời để dự phòng và chia tải).
   ![ECS Service Configuration](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/ecs_service_configuration.png)
6. **Networking:**
   - **VPC:** Chọn `cloudforge-vpc`.
   - **Subnets:** Bắt buộc chọn 2 **Private Subnets** (Bảo mật: Backend không nên tiếp xúc trực tiếp với Internet).
   - **Security group:** Chọn **Use an existing security group** và TÌM CHỌN CHÍNH XÁC **`cloudforge-ecs-app-sg`** (Đây là SG dành riêng cho ứng dụng, chỉ cho phép nhận traffic từ Load Balancer ở cổng 8000).

   ![ECS Service Networking](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/ecs_service_networking.png)
7. **Load balancing:**
   - Chọn loại **Application Load Balancer**.
   - **Application Load Balancer:** Chọn **Use an existing load balancer** và trỏ vào `cloudforge-backend-alb` (ALB vừa tạo ở Bước 2).
   - **Listener:** Chọn **Use an existing listener** và trỏ vào Listener cổng `80:HTTP` đã cấu hình.
   - **Target group:** Chọn **Use an existing target group** và trỏ vào `cloudforge-backend-tg`.

   ![ECS Service Load Balancing 1](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/ecs_service_loadbalancing1.png)

8. Bấm **Create** để tiến hành triển khai. Quá trình này có thể mất 2-3 phút để trạng thái Service chuyển sang `Running`.

![ECS Service Success](/images/5-Workshop/5.7-Compute-setup/5.7.2-deploy-ecs-backend/ecs_service_success.png)

{{% notice tip %}}
**Architectural Note (Isolate by Design):** Với mô hình thiết kế phân tách này, hệ thống Backend API sở hữu hai lớp bảo vệ nghiêm ngặt. Toàn bộ lưu lượng tấn công tiềm ẩn từ Internet sẽ bị chặn lại hoặc lọc bớt tại bộ cân bằng tải ALB thuộc vùng Public Subnet (Có thể bọc thêm AWS WAF). Luồng dữ liệu sạch sau đó mới được định tuyến âm thầm qua mạng nội bộ để đi vào các Container nằm sâu trong vùng mạng Private Subnet, triệt tiêu hoàn toàn rủi ro bị khai thác lỗ hổng trực diện từ môi trường bên ngoài.
{{% /notice %}}

***

**Bước tiếp theo:** Hệ thống Backend API đã được triển khai hoàn tất và có thể tiếp nhận yêu cầu điều phối thông qua endpoint của ALB. Tiến hành tiếp tục thiết lập thành phần "não bộ" phân tích của dự án tại bài [**5.7.3: Triển khai ECS AI Worker**](../5.7.3-deploy-ecs-ai-worker/) để cấu hình cụm máy chủ chuyên trách xử lý các tác vụ xử lý ngầm từ hàng đợi SQS.
