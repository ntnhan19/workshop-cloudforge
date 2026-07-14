---
title : "Tổng quan Kiến trúc"
date : 2026-07-09
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Kiến trúc hệ thống Smart Media Analytics (Team CloudForge)

Dự án **Smart Media Analytics** là một giải pháp phân tích video toàn diện, kết hợp sức mạnh của Generative AI và kiến trúc Serverless / Event-Driven trên AWS. Hệ thống được thiết kế mở rộng tự động, xử lý video dung lượng lớn, trích xuất dữ liệu thông minh và cho phép tìm kiếm theo ngữ nghĩa (Semantic Search).

Dưới đây là sơ đồ kiến trúc tổng thể của hệ thống:

![Tổng quan kiến trúc nền tảng](/images/5-Workshop/5.1-Architecture-overview/diagram1.png)

#### Các luồng xử lý chính (Key Workflows)

1. **Frontend & Authentication (Luồng Truy cập & Xác thực):**
   - Ứng dụng Web (React) được host trên **AWS Amplify**, kết hợp **Route 53** để quản lý tên miền toàn cầu.
   - Người dùng đăng nhập thông qua **Amazon Cognito** để nhận JWT Token an toàn.
   - Các API request được xác thực bởi **API Gateway** và định tuyến qua **Application Load Balancer (ALB)** tới dịch vụ Backend API chạy trên **Amazon ECS (AWS Fargate)**.

2. **Ingestion & Orchestration (Luồng Tải lên & Điều phối Job):**
   - Người dùng upload video trực tiếp lên **Amazon S3** (Secure Upload).
   - S3 tạo *Event Notification* gửi tới hàng đợi **Amazon SQS**, sau đó tiếp tục được chuyển qua **Amazon EventBridge** để kích hoạt **AWS Step Functions**.
   - **Step Functions** đóng vai trò nhạc trưởng (Orchestrator), điều phối toàn bộ quá trình chia tách và xử lý video tự động.

3. **AI Processing Pipeline (Luồng Xử lý AI):**
   - Các tác vụ xử lý nặng được tự động khởi chạy trên **ECS AI Worker (AWS Fargate)** nằm trong vùng an toàn (Private Subnet) và có khả năng Auto Scaling trên đa vùng (AZ A & B).
   - ECS Worker kết nối tới S3 thông qua **S3 Gateway Endpoint (PrivateLink)** để lấy video và lưu keyframes mà không tốn chi phí truyền tải qua Internet.
   - Quá trình phân tích sử dụng các dịch vụ AI Managed mạnh mẽ của AWS:
     - **Amazon Transcribe**: Trích xuất giọng nói thành văn bản (Speech-to-Text).
     - **Amazon Bedrock (Nova Lite & Titan Embeddings)**: Phân tích đa phương thức (multimodal analysis) và tạo vector nhúng (Embeddings).
   - Trạng thái tiến trình (Job Progress) liên tục được Worker đẩy vào **ElastiCache for Redis** trong Private Subnet. Dịch vụ **ECS Backend API** sẽ đăng ký theo dõi (subscribe) các sự kiện này và gửi cập nhật tiến độ theo thời gian thực trực tiếp về Frontend thông qua **kết nối WebSocket (WSS)** được quản lý bởi ALB.

4. **Data Persistence & Search (Luồng Lưu trữ & Tìm kiếm):**
   - Sau khi phân tích xong, ECS AI Worker ghi trực tiếp Metadata và Vector vào cơ sở dữ liệu **RDS PostgreSQL** (triển khai Multi-AZ với Primary/Standby giúp đảm bảo độ sẵn sàng cao).
   - Khi người dùng tìm kiếm, **ECS Backend API** gọi Bedrock tạo vector truy vấn, sau đó truy xuất trực tiếp dữ liệu từ RDS để thực hiện tìm kiếm Vector (Vector Search).

5. **CI/CD & Observability (Triển khai & Giám sát):**
   - Quá trình CI/CD hoàn toàn tự động qua **GitHub Actions**, build và đẩy Docker Image lên **Amazon ECR**.
   - Toàn bộ log, số liệu đo lường và theo dõi luồng (tracing) được quản lý bởi **CloudWatch** và **AWS X-Ray**.

---
**Bước tiếp theo:** Chuyển sang phần **[Chuẩn bị & Phân quyền (Prerequisites)](../5.2-prerequisites/)** để cấu hình IAM Roles và môi trường làm việc.
