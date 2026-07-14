---
title : "Kiểm thử Real-time Flow"
date : 2026-07-10
weight : 4
chapter : false
pre : " <b> 5.9.4. </b> "
---

Sau khi hoàn tất quá trình thiết lập các thành phần phân tán (S3, SQS, ECS Fargate, AI Services, và API Gateway), nhóm dự án tiến hành kiểm thử dòng chảy dữ liệu toàn vẹn (End-to-End Data Flow).

Mục tiêu của hạng mục kiểm thử này là xác minh khả năng kết nối WebSocket có thể nhận được thông báo trạng thái từ Backend ngay khi AI Worker hoàn thành tiến trình xử lý Video.

#### Công cụ kiểm thử
Để tương tác trực tiếp với giao thức WebSocket từ môi trường dòng lệnh (CLI), dự án sử dụng công cụ **`wscat`**.
Cài đặt công cụ thông qua NPM:
```bash
npm install -g wscat
```

#### Kịch bản Kiểm thử End-to-End (E2E)

**Bước 1: Kích hoạt luồng sự kiện và lấy Job ID (Trigger)**
Trong thiết kế chuẩn của hệ thống, WebSocket route yêu cầu một UUID hợp lệ. Do đó, tiến hành kích hoạt tiến trình xử lý thông qua REST API để Backend khởi tạo một Job và cấp phát UUID (thay vì upload trực tiếp lên S3 mà không nhận được ID).

Sử dụng cURL (hoặc PowerShell) để gọi API Ingest, thay thế `[ALB-DNS]` bằng DNS của ALB:
```bash
curl -X POST http://[ALB-DNS]/api/v1/ingest \
  -H "Content-Type: application/json" \
  -d '{"source_path": "s3://cloudforge-media-upload-bucket/sample-video.mp4"}'
```
*Kết quả trả về sẽ chứa `job_id` (VD: `14492409-f583-41ac-89c4-c66a9351919c`).*

![Trigger REST API](/images/5-Workshop/5.9-API-and-realtime/5.9.4-test-realtime-flow/trigger_api.png)

**Bước 2: Thiết lập kết nối lắng nghe (Listening)**
Sử dụng `wscat` để mở kết nối WebSocket đâm xuyên trực tiếp qua Application Load Balancer (ALB). Thay thế `[ALB-DNS]` và `[job_id]` bằng chuỗi UUID bạn vừa nhận được ở Bước 1.
```bash
wscat -c ws://[ALB-DNS]/api/v1/ingest/ws/[job_id]
```
*Trạng thái hệ thống: Terminal chuyển sang trạng thái Connected. Container Backend (FastAPI) trực tiếp ghi nhận và duy trì phiên kết nối này.*

![Websocket Test](/images/5-Workshop/5.9-API-and-realtime/5.9.4-test-realtime-flow/websocket_test.png)

**Bước 3: Xác thực luồng dữ liệu thời gian thực**
Tại màn hình Terminal đang chạy `wscat`, hệ thống sẽ xuất ra các gói tin JSON ngay khi trạng thái của AI Worker thay đổi, và cuối cùng là gói tin hoàn tất (thời gian xử lý thực tế phụ thuộc vào dung lượng Video).

Kết quả trả về không yêu cầu thao tác Polling từ phía Client:

```json
< {
    "event": "processing",
    "job_id": "a99b7166-7c59-4787-9120-a4b12bf9ac3e",
    "asset_id": null,
    "status": "processing",
    "progress": 0.0,
    "assets_queued": 0,
    "assets_processed": 0,
    "error_message": null
  }
```

![Realtime Test](/images/5-Workshop/5.9-API-and-realtime/5.9.4-test-realtime-flow/realtime_test.png)

{{% notice tip %}}
**Đánh giá Kiến trúc:** Kết quả kiểm thử xác nhận kiến trúc Hướng sự kiện (Event-Driven) và Real-time API đang hoạt động đúng thiết kế. Quá trình xử lý diễn ra hoàn toàn bất đồng bộ, loại bỏ tình trạng thắt nút cổ chai (bottleneck) hay lãng phí tài nguyên kết nối do cơ chế Polling truyền thống.
{{% /notice %}}

***

**Bước tiếp theo:** Hệ thống giao tiếp API và xử lý dữ liệu nền đã được nghiệm thu. Ở chương tiếp theo ([**Chương 5.10: Semantic Search**](../../5.10-Semantic-search/)), nhóm dự án sẽ tiến hành khai thác kho dữ liệu Vector Embeddings để thiết lập công cụ tìm kiếm ngữ nghĩa.
