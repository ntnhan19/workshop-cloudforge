---
title : "Điều phối Workflow"
date : 2026-07-10
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

### Tổng quan Điều phối (Ingestion Workflow)

Trong các chương trước, chúng ta đã xây dựng xong tầng lưu trữ, cơ sở dữ liệu và bảo mật. Ở chương 5.6 này, chúng ta sẽ thiết lập hệ thống **tin nhắn và điều phối sự kiện (Event-driven)** để liên kết các thành phần lại với nhau một cách lỏng lẻo (decoupled) nhưng cực kỳ bền bỉ.

Hệ thống Smart Media Analytics phải xử lý khối lượng lớn file đa phương tiện. Quá trình xử lý AI mất nhiều thời gian, do đó chúng ta không thể gọi API đồng bộ. Thay vào đó, chúng ta sử dụng kiến trúc bất đồng bộ (Asynchronous) dựa trên hàng đợi.

Trong chương này, chúng ta sẽ tập trung vào:
1. **Amazon SQS (Simple Queue Service):** Hàng đợi chứa các tác vụ (task) xử lý video/âm thanh đang chờ được AI Worker lấy ra xử lý.
2. **Amazon EventBridge:** Trung tâm định tuyến sự kiện (Event Bus), lắng nghe các sự kiện trạng thái (thành công/thất bại) và kích hoạt luồng xử lý tiếp theo.
3. **AWS Step Functions:** (Tùy chọn) Điều phối quy trình xử lý phức tạp gồm nhiều bước nối tiếp nhau một cách trực quan.

*(Nội dung chi tiết các bước thiết lập sẽ được cập nhật ở các bài viết con trong phần này).*
