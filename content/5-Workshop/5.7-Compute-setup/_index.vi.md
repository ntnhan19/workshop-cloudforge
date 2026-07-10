---
title : "Triển khai Compute (ECS)"
date : 2026-07-10
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

### Tổng quan Khối Tính toán (Compute Setup)

Sau khi có mạng nội bộ, cơ sở dữ liệu, bảo mật và hàng đợi sự kiện, đây là lúc chúng ta mang "trái tim" của hệ thống lên Cloud: Các mã nguồn ứng dụng Backend và AI Worker.

Thay vì quản lý máy chủ ảo EC2 truyền thống nặng nề, chúng ta sẽ đóng gói ứng dụng bằng Docker và chạy chúng trên nền tảng **Amazon ECS (Elastic Container Service)** kết hợp với công cụ tính toán không máy chủ **AWS Fargate**.

Lợi ích của Fargate:
- **Không quản lý máy chủ:** AWS tự lo việc cấp phát tài nguyên CPU/RAM bên dưới.
- **Bảo mật tuyệt đối:** Các Container sẽ chạy hoàn toàn trong vùng mạng **Private Subnet** mà chúng ta đã vạch ra ở Chương 5.3.
- **Tự động mở rộng:** Dễ dàng tăng giảm số lượng Container dựa trên lượng tin nhắn trong hàng đợi SQS hoặc tải CPU.

Trong chương này, chúng ta sẽ thiết lập:
1. **Amazon ECR:** Đẩy Docker Image của Backend và AI Worker lên kho lưu trữ đám mây.
2. **ECS Cluster & Task Definitions:** Định nghĩa cấu hình phần cứng và nạp các thông số môi trường (`.env`) từ Secrets Manager.
3. **Application Load Balancer (ALB):** Tạo cổng giao tiếp từ Internet vào Backend API (đã được bọc bởi WAF).

*(Nội dung chi tiết các bước thiết lập sẽ được cập nhật ở các bài viết con trong phần này).*
