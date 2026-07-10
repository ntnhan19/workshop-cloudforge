---
title : "Kích hoạt pgvector"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.4.2. </b> "
---

Để hỗ trợ tính năng tìm kiếm ngữ nghĩa AI (semantic search), cơ sở dữ liệu PostgreSQL của chúng ta cần khả năng lưu trữ và tính toán khoảng cách giữa các vector embeddings. May mắn là Amazon RDS for PostgreSQL hỗ trợ sẵn extension `pgvector`.

#### 1. Kết nối vào Database
Vì database của chúng ta được đặt an toàn trong Private Subnet (tuân thủ mô hình Zero-Trust), bạn không thể kết nối trực tiếp từ Internet. Để thao tác, bạn cần sử dụng một công cụ SQL client (như `psql`, DBeaver, hoặc pgAdmin) từ bên trong mạng VPC.

*Các cách phổ biến để kết nối trong workshop này:*
- Sử dụng một máy chủ **EC2 Bastion Host** hoặc môi trường **AWS Cloud9** được đặt ở Public Subnet.
- Sử dụng tính năng port forwarding của **AWS Systems Manager (SSM) Session Manager** để kết nối an toàn từ máy tính cá nhân.

Sử dụng thông tin đăng nhập bạn đã tạo ở bài trước:
- **Host:** *[Endpoint URL của RDS]*
- **Port:** 5432
- **Database:** `cloudforge_db`
- **Username:** `postgres`
- **Password:** *[Mật khẩu Master của bạn]*

#### 2. Kích hoạt Extension
Sau khi kết nối thành công vào database `cloudforge_db`, hãy chạy câu lệnh SQL sau để kích hoạt extension:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

#### 3. Kiểm tra cài đặt
Để xác nhận extension đã được cài đặt và hoạt động tốt, bạn có thể chạy truy vấn sau:

```sql
SELECT * FROM pg_extension WHERE extname = 'vector';
```

Nếu kết quả trả về có chứa dòng dữ liệu của `vector`, xin chúc mừng! Database của bạn hiện đã được "nâng cấp" thành một Vector Database, sẵn sàng lưu trữ các chuỗi đa chiều do Amazon Titan tạo ra.

*📸 Ảnh minh họa: Chạy lệnh CREATE EXTENSION thành công trên SQL client.*
![Enable pgvector](../../../../images/5-Workshop/5.4-Database-setup/5.4.2-enable-pgvector/enable_pgvector.png)

***

**Bước tiếp theo:** Với lớp lưu trữ dữ liệu AI đã sẵn sàng, chúng ta sẽ chuyển sang phần **5.4.3: Tạo ElastiCache Redis** để thiết lập hệ thống đệm tốc độ cao và quản lý hàng đợi.
