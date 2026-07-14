---
title : "Khởi tạo S3 Bucket"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.4.1. </b> "
---

Thành phần đầu tiên và cũng là nền tảng lưu trữ chính của toàn bộ hệ thống Smart Media Analytics chính là **Amazon S3 (Simple Storage Service)**. Đây sẽ là nơi tiếp nhận, lưu trữ và bảo quản toàn bộ các tệp tin video, audio gốc được tải lên từ người dùng.

Để đảm bảo tính độc nhất toàn cầu (Globally Unique) và bảo mật dữ liệu, tiến hành khởi tạo một Bucket mới.

#### 1. Quy trình Khởi tạo Amazon S3 Bucket
Truy cập dịch vụ **Amazon S3** trên AWS Console → Bấm **Create bucket**. Tiến hành cấu hình chi tiết thông qua các phân đoạn trực quan trên giao diện:

1. **Phân đoạn General configuration:**
   - **AWS Region:** Chọn `ap-southeast-1` (Singapore) - Đảm bảo cùng phân vùng với hạ tầng mạng VPC nhằm tối ưu hóa độ trễ truyền tải.
   - **Bucket name:** Nhập `cloudforge-media-upload-<tên-của-bạn>` *(Lưu ý: Tên Bucket phải là định danh duy nhất tính trên toàn cục hệ thống AWS toàn cầu).*

![General Configuration](/images/5-Workshop/5.4-Database-setup/5.4.1-s3/s3_general_configuration.png)

2. **Phân đoạn Object Ownership:** Giữ nguyên thiết lập mặc định `ACLs disabled (recommended)` để quản lý quyền truy cập tập trung thông qua chính sách IAM.

![S3 Object Ownership](/images/5-Workshop/5.4-Database-setup/5.4.1-s3/s3_object_ownership.png)

3. **Phân đoạn Block Public Access settings for this bucket:** 
   - Tích chọn **Block all public access**. Đây là quy chuẩn bảo mật bắt buộc (Best Practice) nhằm cô lập hoàn toàn tài nguyên lưu trữ khỏi môi trường Internet công cộng, chỉ cho phép các dịch vụ thuộc tầng Backend truy xuất nội bộ.

![S3 Block Public Access](/images/5-Workshop/5.4-Database-setup/5.4.1-s3/s3_block_public_access.png)

4. **Phân đoạn Bucket Versioning & Default encryption:** Giữ nguyên các thông số cấu hình mặc định (Mã hóa tự động bằng cơ chế SSE-S3).

![S3 Bucket Settings](/images/5-Workshop/5.4-Database-setup/5.4.1-s3/s3_bucket_settings.png)

Cuộn xuống cuối trang biểu mẫu và bấm **Create bucket** để hoàn tất tiến trình.

#### 2. Kích hoạt chuyển tiếp sự kiện tới Amazon EventBridge (Quan trọng)
Theo mặc định hệ thống, Amazon S3 sẽ không chủ động phát tán dữ liệu sự kiện ra ngoài. Để chuỗi đường ống điều phối bất đồng bộ tại Chương 5.6 có thể hoạt động, cần kích hoạt tính năng thông báo:

1. Tại danh sách Buckets, nhấp chọn vào tên Bucket vừa khởi tạo.
2. Điều hướng sang thẻ **Properties** trên thanh bảng chọn ngang.
3. Di chuyển xuống phân đoạn **Amazon EventBridge**, nhấp chọn **Edit**.
4. Chuyển đổi trạng thái từ *Off* sang **On** và bấm **Save changes**.

{{% notice tip %}}
**System Design Note:** Thao tác gạt trạng thái sang *On* cho phép Amazon S3 tự động đóng gói siêu dữ liệu dưới dạng JSON và đẩy thẳng vào trục Bus sự kiện mặc định của EventBridge bất cứ khi nào phát sinh hành động tải tệp tin lên (`ObjectCreated`).
{{% /notice %}}

![S3 EventBridge Config](/images/5-Workshop/5.4-Database-setup/5.4.1-s3/s3_eventbridge_enabled.png)

#### 3. Cấu hình CORS (Cross-Origin Resource Sharing)
Do frontend của hệ thống (được lưu trữ trên Amplify hoặc chạy local) sẽ truy cập trực tiếp vào các file video/audio trên S3 thông qua Presigned URL, S3 cần được cấu hình CORS để cho phép trình duyệt tải nội dung đa phương tiện.

1. Vẫn trong trang chi tiết của Bucket, chuyển sang thẻ **Permissions**.
2. Cuộn xuống phần **Cross-origin resource sharing (CORS)** và bấm **Edit**.
3. Dán đoạn cấu hình JSON sau vào ô:
```json
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "GET",
            "PUT",
            "POST",
            "HEAD"
        ],
        "AllowedOrigins": [
            "*"
        ],
        "ExposeHeaders": [
            "ETag"
        ],
        "MaxAgeSeconds": 3000
    }
]
```
4. Bấm **Save changes**. *(Lưu ý: Trong môi trường production thực tế, bạn nên thay dấu `*` ở AllowedOrigins bằng domain chính thức của frontend để bảo mật hơn).*

![S3 CORS Config](/images/5-Workshop/5.4-Database-setup/5.4.1-s3/s3_cors_enabled.png)

#### 4. Kết quả triển khai
Hạ tầng lưu trữ phi cấu trúc (Object Storage) đã được thiết lập thành công và sẵn sàng đóng vai trò là nguồn cấp phát công việc cho toàn bộ pipeline xử lý tự động phía sau.

![S3 EventBridge Config](/images/5-Workshop/5.4-Database-setup/5.4.1-s3/s3_eventbridge_enabled.png)

***

**Bước tiếp theo:** Hệ thống lưu trữ tệp (File Storage) đã sẵn sàng. Tiến hành tiếp tục thiết lập bộ nhớ dữ liệu có cấu trúc tại bài [**5.4.2: Tạo RDS PostgreSQL**](../5.4.2-create-rds-postgresql/) để phục vụ việc lưu trữ metadata và vector tìm kiếm cho AI.
