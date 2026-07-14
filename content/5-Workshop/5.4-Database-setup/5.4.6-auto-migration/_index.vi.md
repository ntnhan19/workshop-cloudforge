---
title : "Khởi tạo Database Schema (Auto-Migration)"
date : 2026-07-12
weight : 6
chapter : false
pre : " <b> 5.4.6. </b> "
---

Một trong những thách thức phổ biến nhất khi triển khai hệ thống lên Môi trường Đám mây (Cloud) chính là việc **khởi tạo bảng và các quan hệ (Database Schema)**. 

Trong các mô hình triển khai thực tế trên AWS, do cơ sở dữ liệu (RDS) thường được giấu kín bên trong Private Subnet nhằm bảo mật tối đa, chúng ta **không thể kết nối trực tiếp từ máy tính cá nhân** để chạy các lệnh tạo bảng (ví dụ như `alembic upgrade head`).

### Làm thế nào để giải quyết bài toán này?
Các kiến trúc sư Cloud thường chọn một trong ba phương án sau:
1. **Dùng Bastion Host / VPN kết hợp CI/CD:** GitHub Actions hoặc GitLab CI kết nối thông qua một cầu nối mạng để chạy lệnh.
2. **Migration Task độc lập:** Sử dụng AWS Step Functions hoặc AWS CodeBuild kích hoạt một ECS Task chạy một lần duy nhất với nhiệm vụ khởi tạo DB.
3. **Container Auto-Migration:** Gắn trực tiếp logic tạo bảng vào quá trình khởi động (Startup / Lifespan) của ứng dụng. Container vừa bật lên sẽ tự dò tìm Database và tạo bảng nếu chưa có.

Trong khuôn khổ Workshop này, nhằm mang lại trải nghiệm mượt mà và tập trung vào kiến trúc thay vì các thao tác Command Line phức tạp, chúng ta áp dụng **Phương án 3 (Auto-Migration)**. 

### Cấu hình FastAPI Auto-Migration
Nếu kiểm tra mã nguồn tại file `backend/main.py`, sẽ thấy tính năng này đã được cài cắm sẵn vào sự kiện `lifespan` của ứng dụng:

```python
@asynccontextmanager
async def lifespan(app: FastAPI):
    # Auto-migrate database tables for the workshop
    try:
        from database import engine, Base
        import models.asset
        import models.ingest_job
        import models.scene
        async with engine.begin() as conn:
            await conn.run_sync(Base.metadata.create_all)
        logger.info("Database tables initialized successfully via Auto-Migration.")
    except Exception as e:
        logger.error(f"Failed to auto-migrate database: {e}")
```

Bất cứ khi nào ECS Task bắt đầu chạy (khi bạn triển khai ứng dụng ở chương sau), đoạn mã này sẽ tự động liên lạc với RDS và thực thi lệnh khởi tạo toàn bộ bảng (`ingest_jobs`, `assets`, `search`, v.v.).

![Auto Migration Result](/images/5-Workshop/5.4-Database-setup/5.4.6-auto-migration/auto_migration.png)

{{% notice tip %}}
**Best Practice cho Production:** Mặc dù Auto-Migration cực kỳ tiện lợi cho các bài Lab, POC hoặc Workshop, nhưng khi triển khai ứng dụng ở quy mô Doanh nghiệp (Enterprise), các DevOps Engineer thường ưu tiên sử dụng **Phương án 1 hoặc 2** kết hợp các công cụ quản lý phiên bản Database (như Alembic, Flyway, Liquibase) để kiểm soát chặt chẽ những thay đổi trên Database thay vì tạo bảng tự động.
{{% /notice %}}

***

**Bước tiếp theo:** Hệ thống mạng và cơ sở dữ liệu nền tảng đã sẵn sàng hoàn toàn (Bảng sẽ được tự động tạo khi Backend khởi chạy). Tiến hành chuyển sang phần [**5.5: Security Setup**](../../5.5-Security-setup/) để thiết lập kho quản lý Secrets an toàn.
