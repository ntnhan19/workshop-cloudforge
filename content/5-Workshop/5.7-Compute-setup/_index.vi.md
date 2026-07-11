---
title : "Triển khai Compute (ECS)"
date : 2026-07-10
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

### Tổng quan Khối Tính toán (Compute Setup)

Sau khi hoàn thiện phân lớp mạng nội bộ bảo mật, cơ sở dữ liệu bền bỉ và đường ống điều phối tin nhắn hướng sự kiện, hệ thống Smart Media Analytics cần một nền tảng tính toán mạnh mẽ để vận hành "trái tim" của toàn bộ dự án: Các ứng dụng Backend API và các tác vụ AI Worker thực thi tác vụ nặng.

Thay vì sử dụng kiến trúc quản lý máy chủ ảo EC2 truyền thống vốn đòi hỏi nhiều công sức vận hành (OS patching, scaling group quản lý thủ công), dự án áp dụng giải pháp đóng gói ứng dụng bằng Docker Container và triển khai trên nền tảng **Amazon ECS (Elastic Container Service)** kết hợp với công cụ tính toán không máy chủ **AWS Fargate**.

#### Lợi ích cốt lõi của Kiến trúc ECS Fargate:
- **Trừu tượng hóa hạ tầng (Serverless Compute):** AWS tự động chịu trách nhiệm cấp phát, quản lý và tối ưu hóa tài nguyên phần cứng (CPU/RAM) bên dưới. Kỹ sư chỉ cần tập trung vào việc tối ưu hóa mã nguồn ứng dụng.
- **Bảo mật chuyên sâu (Isolate by Design):** Các tác vụ tính toán (Tasks) được cấu hình để khởi chạy hoàn toàn trong phân vùng **Private Subnets** đã thiết lập ở Chương 5.3, cách ly hoàn toàn khỏi Internet công cộng.
- **Khả năng co giãn linh hoạt (Elastic Scalability):** Hệ thống có khả năng tự động tăng giảm số lượng tác vụ xử lý (Containers) dựa trên số lượng thông điệp tích tụ trong hàng đợi SQS hoặc ngưỡng tải phần cứng, giúp tối ưu hóa chi phí vận hành tối đa.

#### Lộ trình triển khai phân lớp Compute:
1. **Amazon ECR (Elastic Container Registry):** Khởi tạo kho lưu trữ đám mây bảo mật để quản lý và lưu trữ các bản phân phối Docker Images của hệ thống.
2. **ECS Backend & Application Load Balancer:** Thiết lập bộ cân bằng tải ứng dụng làm cổng giao tiếp trung gian và triển khai ứng dụng API công khai trên nền tảng AWS Fargate.
3. **Background AI Worker:** Triển khai dịch vụ chạy ngầm hoàn toàn cô lập trong Private Subnet để xử lý các tác vụ phân tích nặng từ hàng đợi SQS.
4. **Auto Scaling:** Cấu hình chính sách tự động mở rộng và thu hẹp tài nguyên thông minh dựa trên thông số CPU nhằm tối ưu hiệu năng và chi phí.

Hoàn thành chương này sẽ cung cấp cho hệ thống một năng lực tính toán linh hoạt, bảo mật tuyệt đối và tự động tối ưu hóa theo quy quy mô tải thực tế của doanh nghiệp.

### Nội dung thực hành

- [Khởi tạo ECS Cluster](5.7.1-create-ecs-cluster/)
- [Triển khai ECS Backend](5.7.2-deploy-ecs-backend/)
- [Triển khai ECS AI Worker](5.7.3-deploy-ecs-ai-worker/)
- [Cấu hình Auto Scaling](5.7.4-configure-auto-scaling/)
