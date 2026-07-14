---
title : "Cấu hình Security Groups"
date : 2026-07-10
weight : 4
chapter : false
pre : " <b> 5.3.4. </b> "
---

Security Groups đóng vai trò như những bức tường lửa ảo (virtual firewalls) ở cấp độ tài nguyên. Để hệ thống đạt chuẩn bảo mật cao nhất, tiến hành áp dụng mô hình **Zero-Trust**: các thành phần trong hệ thống chỉ chấp nhận lưu lượng truy cập (traffic) từ những nguồn được ủy quyền cụ thể, tuyệt đối không mở cổng tràn lan ra Internet.

Trong phần này, tiến hành tiến hành tạo 3 Security Groups cốt lõi bên trong `cloudforge-vpc`:

#### 1. ALB-SG (Tường lửa cho Application Load Balancer)
**Mục đích:** Đây là cánh cửa duy nhất mở ra Internet để đón lưu lượng người dùng.
1. Truy cập **EC2 Console** hoặc **VPC Console**, chọn mục **Security Groups** ở menu bên trái.
2. Bấm nút màu cam **Create security group**.
3. **Security group name:** `cloudforge-alb-sg`
4. **Description:** `Allow HTTP/HTTPS from Internet`
5. **VPC:** Chọn `cloudforge-vpc`.
6. Tại phần **Inbound rules**, cấu hình như sau:
   - Type: `HTTP` | Port: `80` | Source: `0.0.0.0/0` (Anywhere IPv4)
   - Type: `HTTPS` | Port: `443` | Source: `0.0.0.0/0` (Anywhere IPv4)
7. **Outbound Rules:** Giữ nguyên Default (Cho phép tất cả).
8. Bấm **Create security group**.

![ALB Security Group](/images/5-Workshop/5.3-Network-vpc/5.3.4-security-groups/alb_sg.png)

#### 2. ECS-App-SG (Tường lửa cho Backend & AI Worker)
**Mục đích:** Chỉ cho phép nhận requests đã được lọc qua Load Balancer và cho phép các containers giao tiếp nội bộ với nhau.
1. Tạo một security group mới tên là `cloudforge-ecs-app-sg` thuộc `cloudforge-vpc`.
2. **Inbound rules:**
   - Type: `Custom TCP` | Port: `8000` (Cổng Backend API) | Source: Gắn ID của `cloudforge-alb-sg`. *(Bảo vệ API, chỉ nhận traffic từ ALB)*.
   - Type: `All TCP` | Port: `0-65535` | Source: Gắn ID của chính `cloudforge-ecs-app-sg`. *(Cho phép các container nội bộ gọi chéo nhau)*.

![ECS App Security Group](/images/5-Workshop/5.3-Network-vpc/5.3.4-security-groups/ecs_app_sg.png)

#### 3. DB-Redis-SG (Tường lửa cho Database & Cache)
**Mục đích:** Khóa chặt lớp Dữ liệu (Data tier), chỉ cho phép các máy chủ xử lý (ECS containers) truy cập vào để đọc/ghi dữ liệu.
1. Tạo một security group mới tên là `cloudforge-db-redis-sg` thuộc `cloudforge-vpc`.
2. **Inbound rules:**
   - Type: `PostgreSQL` | Port: `5432` | Source: Gắn ID của `cloudforge-ecs-app-sg`.
   - Type: `Custom TCP` | Port: `6379` (Cổng của Redis) | Source: Gắn ID của `cloudforge-ecs-app-sg`.

![DB Security Group](/images/5-Workshop/5.3-Network-vpc/5.3.4-security-groups/sg_db_redis.png)

***

**Bước tiếp theo:** Với hệ thống mạng nền tảng và các lớp khóa bảo mật Zero-Trust đã hoàn tất, tiến hành chuyển sang phần [**5.4: Database Setup**](../../5.4-Database-setup/) để khởi tạo các cụm cơ sở dữ liệu.
