---
title : "Khởi tạo ElastiCache (Redis)"
date : 2026-07-10
weight : 4
chapter : false
pre : " <b> 5.4.4. </b> "
---

Trong kiến trúc AI Pipeline, Amazon ElastiCache (Redis) đóng hai vai trò cực kỳ quan trọng:
1. **Tốc độ (Caching):** Lưu trữ tạm thời các kết quả truy vấn thường xuyên để giảm tải cho Database chính.
2. **Quản lý hàng đợi (Message Broker):** Đóng vai trò làm trung gian phân phối các tác vụ nặng (như tạo embeddings bằng AI hoặc xử lý media) cho các AI Worker xử lý bất đồng bộ.

Tương tự như RDS, chúng ta sẽ đặt cụm Redis này vào vùng Private Subnet để đảm bảo tuân thủ kiến trúc bảo mật Zero-Trust.

#### 1. Tạo Subnet Group cho ElastiCache
1. Truy cập **ElastiCache Console**, chọn **Subnet groups** ở thanh menu bên trái (nằm dưới mục *Configurations*).
2. Bấm nút **Create subnet group**.
3. **Name:** `cloudforge-redis-subnet-group`
4. **Description:** `Subnet group for CloudForge Redis`
5. **VPC ID:** Chọn `cloudforge-vpc`.
6. **Availability Zones:** Chọn `ap-southeast-1a` và `ap-southeast-1b`.
7. **Subnet ID:** *Lưu ý quan trọng:* Hệ thống có thể sẽ tự động chọn cả 4 subnets. Bạn hãy bấm nút **Manage**, **bỏ chọn 2 Public Subnets**, chỉ giữ lại dấu tích ở 2 Private Subnets của dự án.
8. Bấm **Create**.

![Redis Subnet Group](/images/5-Workshop/5.4-Database-setup/5.4.4-create-elasticache-redis/redis_subnet_group.png)

#### 2. Khởi tạo cụm Redis
Vì giao diện AWS ElastiCache mới sử dụng dạng trình tự từng bước, bạn hãy thực hiện cấu hình như sau:

1. Chuyển sang menu **Redis OSS caches** (dưới mục *Resources*), bấm nút màu cam **Create cache**. *(Nếu có bảng thông báo giới thiệu Valkey, hãy chọn **Continue with Redis OSS**).*
2. **Step 1: Settings (Cài đặt cơ bản):**
   * **Configuration:** Chọn **Redis OSS** → **Node-based cluster** → **Cluster cache**.
   * **Cluster mode:** Chọn **Disabled**.
   * **Cluster info:**
     * **Name:** `cloudforge-redis`
     * **Description:** `Redis cluster for Caching and Message Queue`
     * **Node type:** Chọn `cache.t3.micro` hoặc `cache.t4g.micro` (Tối ưu chi phí).
     * **Number of replicas:** `0`.
   * **Connectivity:** Ở mục *Subnet groups*, chọn **Choose existing subnet group** → Chọn `cloudforge-redis-subnet-group` vừa tạo.
   * Bấm **Next** để sang bước 2.
3. **Step 2: Advanced settings (Cài đặt nâng cao - Rất quan trọng):**
   * **Security groups:** Bấm nút **Manage** → Bỏ chọn nhóm `default`, tích chọn duy nhất nhóm **`cloudforge-db-redis-sg`** → Bấm **Save**.
   * **Backup:** Cuộn xuống dưới, **bỏ chọn** *Enable automatic backups* để rút ngắn thời gian tạo.
   * Bấm **Next**.
4. **Step 3: Review and create:**
   * Kiểm tra lại các thông số và bấm **Create** ở cuối trang. *(Quá trình này sẽ mất khoảng 3-5 phút).*

![Redis Connectivity](/images/5-Workshop/5.4-Database-setup/5.4.4-create-elasticache-redis/redis_connectivity.png)

#### 3. Lấy thông tin Endpoint
Sau khi trạng thái của cụm Redis chuyển sang **Available**, bạn hãy click vào tên `cloudforge-redis`.
Tại phần **Cluster details**, hãy copy lại đường dẫn ở mục **Primary endpoint** (có dạng `cloudforge-redis.xxxxxx.0001.apse1.cache.amazonaws.com:6379`) để lưu lại cấu hình kết nối cho ứng dụng.

***

**Bước tiếp theo:** Hệ thống dữ liệu lõi đã hoàn tất. Chúng ta sẽ tiến tới phần **5.4.4** để tổng hợp lại các Endpoint cần thiết, chuẩn bị hành trang đưa mã nguồn Backend lên Cloud.
