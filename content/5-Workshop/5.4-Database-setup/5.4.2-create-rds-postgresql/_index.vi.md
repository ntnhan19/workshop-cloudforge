---
title : "Tạo RDS PostgreSQL"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.4.2. </b> "
---

Amazon RDS (Relational Database Service) giúp đơn giản hóa việc thiết lập, vận hành và mở rộng quy mô cơ sở dữ liệu quan hệ trên đám mây. Trong dự án này, tiến hành sử dụng engine **PostgreSQL** vì nó hỗ trợ cực kỳ tốt cho extension `pgvector` – thành phần cốt lõi để lưu trữ vector embeddings phục vụ tính năng tìm kiếm AI ngữ nghĩa.

#### 1. Tạo Subnet Group cho Database
Trước khi tạo database, RDS yêu cầu chúng ta phải định nghĩa một **Subnet Group** để xác định dải mạng nội bộ (Private Subnets) nào trong VPC được phép đặt máy chủ dữ liệu.

1. Truy cập **RDS Console**, chọn **Subnet groups** ở menu bên trái.
2. Bấm nút màu cam **Create DB subnet group**.
3. **Name:** `cloudforge-db-subnet-group`
4. **Description:** `Subnet group for CloudForge Databases`
5. **VPC:** Chọn `cloudforge-vpc`.
6. **Add subnets:** Chọn 2 Availability Zones (`ap-southeast-1a` và `ap-southeast-1b`), sau đó tích chọn 2 Private Subnets tương ứng dành cho Database của dự án.
7. Bấm **Create**.

![DB Subnet Group](/images/5-Workshop/5.4-Database-setup/5.4.2-create-rds-postgresql/db_subnet_group.png)

#### 2. Khởi tạo Database PostgreSQL
1. Chuyển sang menu **Databases**, bấm nút màu cam **Create database**.
2. Tại menu lựa chọn xổ xuống, click chọn **Full configuration** để mở toàn bộ cấu hình sâu hệ thống.
3. **Engine options:** Chọn **PostgreSQL** (chọn phiên bản mặc định hệ thống tự chọn).

![RDS Engine Setup](/images/5-Workshop/5.4-Database-setup/5.4.2-create-rds-postgresql/rds_engine.png)

4. **Templates & Durability:**
   - **Templates:** Chọn **Sandbox** (hoặc *Dev/Test*) để tối ưu chi phí tối đa cho môi trường workshop.
   - **Deployment options:** Đảm bảo hệ thống tự chuyển về **Single-AZ DB instance deployment**.

![RDS Template Setup](/images/5-Workshop/5.4-Database-setup/5.4.2-create-rds-postgresql/rds_template.png)

5. **Settings:**
   - **DB instance identifier:** Đổi tên mặc định thành `cloudforge-db`.
   - **Credentials management:** Tích chọn **Self managed** để tự quản lý mật khẩu.
   - **Master username:** Giữ nguyên `postgres`.
   - **Master password / Confirm password:** Nhập mật khẩu bảo mật của hệ thống (ví dụ: `CloudForge2026!`).

![RDS Settings](/images/5-Workshop/5.4-Database-setup/5.4.2-create-rds-postgresql/rds_settings.png)

6. **Instance configuration & Storage (Cấu hình phần cứng):**
   - **DB instance class:** Giữ nguyên dòng burstable nhỏ nhất hệ thống tự gợi ý (ví dụ: `db.t3.micro`).
   - **Allocated storage:** Giảm từ dung lượng lớn mặc định xuống còn **20 GiB** (mức tối thiểu tiết kiệm).
   - **Storage autoscaling:** Bỏ tích chọn *Enable storage autoscaling*.

![RDS Instance Configuration](/images/5-Workshop/5.4-Database-setup/5.4.2-create-rds-postgresql/rds_instance.png)

7. **Connectivity (Kết nối mạng nội bộ - Cực kỳ quan trọng):**
   - **Virtual private cloud (VPC):** Chọn đúng `cloudforge-vpc`.
   - **DB Subnet group:** Chọn đúng `cloudforge-db-subnet-group` vừa tạo ở bước 1.
   - **Public access:** Chọn **No** *(Đảm bảo Database tuyệt đối không bị phơi ra Internet)*.
   - **VPC security group:** Chọn **Choose existing** → Xóa nhóm `default` đi và chọn duy nhất **`cloudforge-db-redis-sg`** (Tường lửa bạn vừa tạo ở bài 5.3.4).

![RDS Connectivity Setup](/images/5-Workshop/5.4-Database-setup/5.4.2-create-rds-postgresql/rds_connectivity.png)

8. Mở rộng phần **Additional configuration** ở cuối trang:
   - **Initial database name:** Nhập `cloudforge_db` (để tự động tạo sẵn database trống).
   - **Backup:** Bỏ tích chọn *Enable automated backups* để rút ngắn thời gian khởi tạo trong workshop.

![RDS Additional Configuration](/images/5-Workshop/5.4-Database-setup/5.4.2-create-rds-postgresql/rds_additional_configuration.png)

9. Cuộn xuống dưới cùng và bấm **Create database**. *(Quá trình khởi tạo máy chủ thực tế mất từ 3-5 phút).*

***

**Bước tiếp theo:** Sau khi máy chủ RDS khởi tạo xong (trạng thái **Available**), tiến hành chuyển sang phần [**5.4.3**](../5.4.3-enable-pgvector/) để kích hoạt extension `pgvector`, chuẩn bị sẵn sàng cho việc lưu trữ embeddings.
