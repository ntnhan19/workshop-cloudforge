---
title : "Kiểm tra Kết nối"
date : 2026-07-10
weight : 5
chapter : false
pre : " <b> 5.4.5. </b> "
---

Sau khi khởi tạo thành công RDS PostgreSQL và ElastiCache Redis, bước cuối cùng của phần Database Setup là **kiểm tra và xác thực kết nối**. Bước này nhằm đảm bảo hệ thống tường lửa (Security Groups) và cấu hình định tuyến trong mạng nội bộ (Private Subnets) đang hoạt động chính xác theo đúng thiết kế.

Đồng thời, chúng ta sẽ tổng hợp lại toàn bộ thông số kết nối để đóng gói thành các biến môi trường (`.env`) sẵn sàng nạp cho ứng dụng Backend ở các chương sau.

#### 1. Kiểm tra kết nối Redis từ EC2
Do cụm Redis được cô lập hoàn toàn trong vùng Private Subnet, chúng ta bắt buộc phải sử dụng máy chủ trung gian EC2 (Bastion Host) nằm trong cùng mạng VPC để thực hiện kiểm thử.

Tại Terminal của máy chủ EC2, tiến hành các bước sau:

**Bước 1: Cài đặt công cụ Redis CLI**
Trên hệ điều hành Amazon Linux 2023, gói công cụ dòng lệnh được AWS tối ưu và định danh dưới tên `redis6`:
```bash
sudo dnf install -y redis6
```

**Bước 2: Thực hiện Ping đến Redis Cluster**
Sử dụng Primary Endpoint đã thu thập được ở bài trước (lưu ý bỏ phần định tuyến cổng `:6379` ở cuối chuỗi URL khi gán vào tham số `-h`):
```bash
redis6-cli -h <ĐIỀN_ENDPOINT_REDIS_CỦA_BẠN> -p 6379 --tls ping
```

**Kinh nghiệm thực tế (Troubleshooting):**
- **Tên lệnh hệ thống:** Hãy đảm bảo sử dụng chính xác lệnh `redis6-cli` thay vì `redis-cli` truyền thống để phù hợp với cơ chế quản lý package mới của Amazon Linux 2023.
- **Cờ bảo mật `--tls`:** ElastiCache mặc định bật tính năng mã hóa đường truyền (Encryption in transit). Nếu thiếu cờ `--tls`, kết nối từ EC2 đến Redis sẽ rơi vào trạng thái im lặng và bị treo lệnh (Timeout).
- **Thông chốt Tường lửa:** Đảm bảo Security Group của Redis (`cloudforge-db-redis-sg`) đã được cấu hình mở Port 6379 với Nguồn (Source) chấp nhận lưu lượng đến từ Security Group của ứng dụng (`cloudforge-ecs-app-sg`).

Nếu cấu hình mạng hoàn toàn chính xác, Terminal sẽ phản hồi kết quả:
```plaintext
PONG
```

#### 2. Kết quả xác thực kiến trúc tổng thể
Để minh chứng cho việc lớp hạ tầng mạng và bảo mật Zero-Trust đã thông suốt hoàn toàn, chúng ta thực hiện chạy đồng thời hai lệnh kiểm tra trạng thái kết nối đến cả dịch vụ Caching (Redis) và Cơ sở dữ liệu lõi (RDS PostgreSQL) ngay trên cùng một cửa sổ dòng lệnh.

![Test Connection Result](/images/5-Workshop/5.4-Database-setup/5.4.5-test-connection/test_connection.png)

Kết quả trả về trên Terminal là minh chứng kỹ thuật khẳng định:
- Máy chủ ứng dụng (ECS Tasks / AI Workers) trong tương lai hoàn toàn có quyền truy cập hợp lệ và an toàn vào các phân lớp dữ liệu cốt lõi.
- Các vòng bảo vệ của Security Groups hoạt động đúng thiết kế: Chặn đứng mọi truy cập trái phép từ Internet nhưng mở cửa chính xác cho các tài nguyên nội bộ giao tiếp với nhau.

#### 3. File cấu hình môi trường mẫu (.env)
Sau khi xác thực các Endpoint hoạt động ổn định, chúng ta tiến hành cấu trúc các thông số này thành dạng tệp tin cấu hình môi trường mẫu chuẩn để chuẩn bị nạp vào các Container ứng dụng:
```env
# Database Configuration
DB_HOST=cloudforge-db.xxxxxx.ap-southeast-1.rds.amazonaws.com
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=<NHẬP_MẬT_KHẨU_CỦA_BẠN_VÀO_ĐÂY>
DB_NAME=cloudforge_db

# Redis Caching & Queue Configuration
REDIS_HOST=master.cloudforge-redis.xxxxxx.apse1.cache.amazonaws.com
REDIS_PORT=6379
```

{{% notice warning %}}
**Khuyến nghị Bảo mật (Security Best Practice):** Không lưu trữ trực tiếp các thông tin nhạy cảm (như `DB_PASSWORD`) dưới dạng bản rõ (plaintext) trong mã nguồn hoặc các kho lưu trữ công khai (GitHub). Đối với kiến trúc Production, các thông số này cần được cô lập, quản lý tập trung và nạp động vào ứng dụng thông qua các dịch vụ chuyên biệt như AWS Secrets Manager.
{{% /notice %}}

***

**Bước tiếp theo:** Hệ thống mạng và cơ sở dữ liệu nền tảng đã sẵn sàng. Chúng ta sẽ chuyển sang phần **5.5: Security Setup** để thiết lập kho quản lý Secrets an toàn, đưa cảnh báo bảo mật ở trên vào thực tế!
