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

**Bước 1: Thiết lập kết nối lắng nghe (Listening)**
Sử dụng `wscat` để mở kết nối đâm xuyên trực tiếp qua Application Load Balancer (ALB) của Backend. Thay thế `[ALB-DNS]` bằng DNS Name thực tế của ALB, và `[job_id]` bằng một chuỗi định danh ngẫu nhiên (VD: `test-job-123`).
```bash
wscat -c ws://[ALB-DNS]/api/v1/ingest/ws/[job_id]
```
*Trạng thái hệ thống: Terminal chuyển sang trạng thái Connected. Container Backend (FastAPI) trực tiếp ghi nhận và duy trì phiên kết nối này.*

**Bước 2: Kích hoạt luồng sự kiện (Trigger)**
Sử dụng AWS CLI để tải lên một tệp Video mẫu vào Bucket S3 (đã cấu hình ở Chương 5.6).
```bash
aws s3 cp sample-video.mp4 s3://cloudforge-media-upload-bucket/
```
*Trạng thái hệ thống: Thao tác tải tệp hoàn tất sẽ kích hoạt EventBridge, gửi Message vào SQS, từ đó kích hoạt AI Worker trên ECS Fargate để gọi các dịch vụ Transcribe và Bedrock.*

**Bước 3: Xác thực luồng dữ liệu thời gian thực**
Tại màn hình Terminal đang chạy `wscat`, hệ thống sẽ xuất ra một gói tin JSON ngay khi AI Worker hoàn tất vòng đời xử lý (thời gian xử lý thực tế phụ thuộc vào dung lượng Video). 

Kết quả trả về không yêu cầu thao tác Polling từ phía Client:

```json
< {
    "action": "ANALYSIS_COMPLETED",
    "videoId": "sample-video.mp4",
    "status": "SUCCESS",
    "summary": "Đoạn video thảo luận về các giải pháp Cloud Native trên AWS...",
    "timestamp": "2026-07-10T10:00:00Z"
  }
```

![Realtime Test](/images/5-Workshop/5.9-API-and-realtime/5.9.4-test-realtime-flow/realtime_test.png)

{{% notice tip %}}
**Đánh giá Kiến trúc:** Kết quả kiểm thử xác nhận kiến trúc Hướng sự kiện (Event-Driven) và Real-time API đang hoạt động đúng thiết kế. Quá trình xử lý diễn ra hoàn toàn bất đồng bộ, loại bỏ tình trạng thắt nút cổ chai (bottleneck) hay lãng phí tài nguyên kết nối do cơ chế Polling truyền thống.
{{% /notice %}}

***

**Bước tiếp theo:** Hệ thống giao tiếp API và xử lý dữ liệu nền đã được nghiệm thu. Ở chương tiếp theo (**Chương 5.10: Semantic Search**), nhóm dự án sẽ tiến hành khai thác kho dữ liệu Vector Embeddings để thiết lập công cụ tìm kiếm ngữ nghĩa.
