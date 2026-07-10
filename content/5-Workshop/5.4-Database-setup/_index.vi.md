---
title : "Database Setup"
date : 2026-07-10
weight : 4 
chapter : false
pre : " <b> 5.4. </b> "
---

### Tổng quan Hệ thống Dữ liệu Cốt lõi

Trong chương này, chúng ta sẽ tiến hành xây dựng nền tảng dữ liệu vững chắc và bảo mật cho ứng dụng Smart Media Analytics. Để tuân thủ nghiêm ngặt kiến trúc bảo mật **Zero-Trust**, toàn bộ các dịch vụ cơ sở dữ liệu sẽ được triển khai hoàn toàn bên trong vùng mạng nội bộ (**Private Subnets**) và được bảo vệ bởi các lớp tường lửa chuyên biệt.

Các thành phần cốt lõi được thiết lập bao gồm:
1. **Amazon S3:** Lưu trữ đối tượng (Object Storage) không giới hạn, là nơi tiếp nhận và bảo quản toàn bộ các tệp tin video/audio gốc.
2. **Amazon RDS PostgreSQL:** Đóng vai trò là cơ sở dữ liệu quan hệ chính, được tích hợp sẵn tiện ích mở rộng `pgvector` nhằm phục vụ tính năng tìm kiếm ngữ nghĩa (Semantic Search) cho các mô hình AI.
3. **Amazon ElastiCache (Redis):** Đóng vai trò làm bộ nhớ đệm (Caching) tốc độ siêu cao và quản lý hàng đợi (Message Broker) để phân phối tác vụ cho các AI Worker xử lý bất đồng bộ.

Kết thúc chương này, hệ thống dữ liệu không chỉ được kiểm chứng khả năng giao tiếp an toàn, mà bạn còn thu thập được toàn bộ các thông số kết nối cần thiết để đóng gói thành biến môi trường (`.env`), sẵn sàng cho việc triển khai ứng dụng.

### Nội dung thực hành

- [Khởi tạo S3 Bucket](5.4.1-create-s3-bucket/)
- [Tạo RDS PostgreSQL](5.4.2-create-rds-postgresql/)
- [Kích hoạt pgvector](5.4.3-enable-pgvector/)
- [Khởi tạo ElastiCache (Redis)](5.4.4-create-elasticache-redis/)
- [Kiểm tra Kết nối & Tổng hợp Endpoint](5.4.5-test-connection/)
