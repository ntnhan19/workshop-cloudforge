---
title : "Tạo RDS PostgreSQL"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

Amazon RDS (Relational Database Service) giúp đơn giản hóa việc thiết lập, vận hành và mở rộng quy mô cơ sở dữ liệu quan hệ trên đám mây. Trong dự án này, chúng ta sẽ sử dụng engine **PostgreSQL** vì nó hỗ trợ cực kỳ tốt cho extension `pgvector` – thành phần cốt lõi để lưu trữ vector embeddings phục vụ tính năng tìm kiếm AI ngữ nghĩa.

#### 1. Tạo Subnet Group cho Database
Trước khi tạo database, RDS yêu cầu chúng ta phải định nghĩa một **Subnet Group** để xác định dải mạng nội bộ (Private Subnets) nào trong VPC được phép đặt máy chủ dữ liệu.

1. Truy cập **RDS Console**, chọn **Subnet groups** ở menu bên trái.
2. Bấm nút màu cam **Create DB subnet group**.
3. **Name:** `cloudforge-db-subnet-group`
4. **Description:** `Subnet group for CloudForge Databases`
5. **VPC:** Chọn `cloudforge-vpc`.
6. **Add subnets:** Chọn 2 Availability Zones (`ap-southeast-1a` và `ap-southeast-1b`), sau đó tích chọn 2 Private Subnets tương ứng dành cho Database của dự án.
7. Bấm **Create**.

*📸 Ảnh minh họa: Tạo Subnet Group cho Database*
![DB Subnet Group](../../../../images/5-Workshop/5.4-Database-setup/5.4.1-create-rds-postgresql/db_subnet_group.png)

#### 2. Khởi tạo Database PostgreSQL
1. Chuyển sang menu **Databases**, bấm nút màu cam **Create database**.
2. **Choose a database creation method:** Chọn **Standard create**.
3. **Engine options:** Chọn **PostgreSQL** (chọn phiên bản mới nhất được hỗ trợ, ví dụ 15.x hoặc 16.x).
4. **Templates:** Chọn **Free tier** hoặc **Dev/Test** (để tối ưu chi phí cho môi trường workshop).
5. **Settings:**
   - **DB instance identifier:** `cloudforge-db`
   - **Master username:** `postgres`
   - **Master password:** *Nhập mật khẩu an toàn của bạn (ví dụ: CloudForge2026!)*. *(Lưu ý: Nhớ kỹ mật khẩu này để lát nữa chúng ta đăng nhập)*
6. **Connectivity (Kết nối - Cực kỳ quan trọng):**
   - **Virtual private cloud (VPC):** Chọn `cloudforge-vpc`.
   - **DB Subnet group:** Chọn `cloudforge-db-subnet-group` vừa tạo ở bước 1.
   - **Public access:** Chọn **No** *(Đảm bảo Database tuyệt đối không bị phơi ra Internet)*.
   - **VPC security group:** Chọn **Choose existing** $\rightarrow$ Xóa nhóm `default` đi và chọn **`cloudforge-db-redis-sg`** (Tường lửa Zero-Trust bạn vừa tạo ở bài 5.3.4).
7. Mở rộng phần **Additional configuration**:
   - **Initial database name:** Nhập `cloudforge_db` (để tự động tạo sẵn database trống).
   - Bạn có thể bỏ chọn *Enable automated backups* để rút ngắn thời gian khởi tạo trong workshop.
8. Cuộn xuống dưới cùng và bấm **Create database**. *(Lưu ý: Quá trình khởi tạo máy chủ thực tế có thể mất từ 5-10 phút).*

*📸 Ảnh minh họa: Thiết lập phần Connectivity khóa chặt DB trong Private Subnet và sử dụng Security Group chuẩn Zero-Trust.*
![RDS Connectivity Setup](../../../../images/5-Workshop/5.4-Database-setup/5.4.1-create-rds-postgresql/rds_connectivity.png)

***

**Bước tiếp theo:** Trong thời gian chờ máy chủ RDS khởi tạo hoàn tất, chúng ta sẽ chuyển sang phần **5.4.2** để kích hoạt extension `pgvector`, chuẩn bị sẵn sàng cho việc lưu trữ embeddings.
