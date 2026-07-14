---
title : "Kích hoạt pgvector"
date : 2026-07-10
weight : 3
chapter : false
pre : " <b> 5.4.3. </b> "
---

Sau khi Amazon RDS PostgreSQL đã khởi tạo thành công, bước tiếp theo là kết nối vào cơ sở dữ liệu và kích hoạt `pgvector` - extension cốt lõi biến PostgreSQL truyền thống thành một Vector Database phục vụ lưu trữ embeddings cho AI.

Vì cụm RDS của hệ thống được đặt an toàn trong **Private Subnet** (tuân thủ mô hình Zero-Trust), chúng ta không thể kết nối trực tiếp từ Internet. Thay vào đó, thao tác này cần được thực hiện thông qua một máy chủ trung gian (Bastion Host / EC2) nằm bên trong mạng VPC.

#### 1. Lấy thông tin kết nối (Endpoint)
1. Truy cập **RDS Console** → **Databases**.
2. Click vào `cloudforge-db` (đảm bảo trạng thái đã chuyển sang *Available*).
3. Trong tab **Connectivity & security**, hãy copy đường dẫn ở mục **Endpoint** (ví dụ: `cloudforge-db.xxxxxx.ap-southeast-1.rds.amazonaws.com`).

#### 2. Kết nối và Kích hoạt extension
Tiến hành sử dụng một máy chủ EC2 (Bastion Host) được gán Security Group `cloudforge-ecs-app-sg` để đi xuyên qua tường lửa của Database. Từ Terminal của EC2, hãy thực hiện lần lượt các lệnh sau:

**Cài đặt công cụ PostgreSQL Client:**
```bash
sudo dnf install -y postgresql15
```

**Kết nối vào Database:**
```bash
psql -h <ĐIỀN_ENDPOINT_CỦA_BẠN_VÀO_ĐÂY> -U postgres -d cloudforge_db
```
*(Hệ thống sẽ yêu cầu nhập mật khẩu, hãy nhập mật khẩu mà bạn đã thiết lập ở bước khởi tạo RDS).*

Sau khi đăng nhập thành công vào PostgreSQL prompt (`cloudforge_db=>`), hãy chạy lệnh SQL sau để kích hoạt extension:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

#### 3. Kiểm tra kết quả
Để xác nhận extension đã được cài đặt thành công, bạn chạy lệnh:

```sql
SELECT extname, extversion FROM pg_extension WHERE extname = 'vector';
```

Nếu kết quả trả về hiển thị `vector` kèm theo phiên bản (ví dụ: `0.8.1`), xin chúc mừng! Cơ sở dữ liệu của hệ thống đã hoàn toàn sẵn sàng để lưu trữ các chuỗi đa chiều do Amazon Titan tạo ra.

![Enable pgvector](/images/5-Workshop/5.4-Database-setup/5.4.3-enable-pgvector/enable_pgvector.png)

***

**Bước tiếp theo:** Với lớp lưu trữ dữ liệu bền vững đã hoàn thiện, tiến hành chuyển sang phần [**5.4.4: Khởi tạo ElastiCache (Redis)**](../5.4.4-create-elasticache-redis/) để xây dựng lớp Caching tốc độ cao và Message Queue cho các AI Worker.
