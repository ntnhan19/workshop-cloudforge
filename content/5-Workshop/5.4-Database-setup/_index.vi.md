---
title : "Database Setup"
date : 2026-07-09
weight : 4 
chapter : false
pre : " <b> 5.4. </b> "
---

#### Tổng quan Lưu trữ & Dữ liệu

Trong phần này, chúng ta sẽ cấu hình các lớp cơ sở dữ liệu và lưu trữ mạnh mẽ cho Smart Media Analytics. Chúng ta sẽ thiết lập Amazon RDS PostgreSQL (với pgvector để tìm kiếm ngữ nghĩa) và Amazon ElastiCache (Redis) để theo dõi tiến trình xử lý theo thời gian thực.

#### Nội dung

- [Tạo RDS PostgreSQL](5.4.1-create-rds-postgresql/)
- [Kích hoạt pgvector](5.4.2-enable-pgvector/)
- [Tạo ElastiCache Redis](5.4.3-create-elasticache-redis/)
- [Kiểm tra Kết nối](5.4.4-test-connection/)
