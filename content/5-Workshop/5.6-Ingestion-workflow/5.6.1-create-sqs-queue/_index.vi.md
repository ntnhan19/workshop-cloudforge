---
title : "Khởi tạo Hàng đợi SQS"
date : 2026-07-10
weight : 1
chapter : false
pre : " <b> 5.6.1. </b> "
---

Thành phần cốt lõi đầu tiên trong kiến trúc điều phối bất đồng bộ của Smart Media Analytics là **Amazon SQS (Simple Queue Service)**. SQS đóng vai trò như một bộ đệm (buffer) an toàn, tiếp nhận các yêu cầu xử lý đa phương tiện và phân phối chúng tuần tự cho các cụm máy chủ AI Worker.

Hạ tầng SQS cần được khởi tạo trước để làm đích đến (Target) cho các quy tắc định tuyến sự kiện, đồng thời làm nguồn cấp việc cho phân lớp Compute (máy chủ ứng dụng).

#### 1. Khởi tạo Dead-Letter Queue (DLQ)
Theo tiêu chuẩn thiết kế hệ thống phân tán (Distributed Systems Best Practices), một hàng đợi phụ (Dead-Letter Queue - DLQ) cần được thiết lập trước nhằm cô lập các thông điệp xử lý lỗi (file hỏng, lỗi model suy luận), ngăn chặn việc hệ thống rơi vào vòng lặp xử lý vô hạn (poison pill).

Từ AWS Console, truy cập dịch vụ **Amazon SQS** → **Queues** → chọn **Create queue**. Tiến hành cấu hình các phân đoạn trực tiếp trên giao diện một trang duy nhất:

- **Phân đoạn Details:**
  - **Type:** Chọn `Standard`.
  - **Name:** Nhập `cloudforge-media-dlq`.
- **Phân đoạn Configuration & Encryption & Access policy:** Giữ nguyên các thông số cấu hình mặc định của hệ thống.
- **Phân đoạn Dead-letter queue - Optional:** Mặc định chọn `Disabled` (hàng đợi DLQ không cần trỏ đến một DLQ khác).

![Create DLQ](/images/5-Workshop/5.6-Ingestion-workflow/5.6.1-create-sqs-queue/create_dlq.png)

Cuối cùng, di chuyển xuống góc dưới cùng bên phải và bấm **Create queue** để hoàn tất khởi tạo.

#### 2. Khởi tạo Hàng đợi tác vụ chính (Task Queue)
Sau khi có DLQ, quay lại giao diện danh sách Queues, bấm **Create queue** một lần nữa để thiết lập hàng đợi điều phối công việc chính:

- **Phân đoạn Details:**
  - **Type:** Chọn `Standard` (Đảm bảo thông lượng xử lý tối đa, phù hợp với kiến trúc Media xử lý bất đồng bộ).
  - **Name:** Nhập `cloudforge-media-task-queue`.

- **Phân đoạn Configuration (Cấu hình tối ưu cho tác vụ AI/Media):**
  - **Visibility timeout:** Thay đổi từ mặc định sang **15 Minutes** (Tương đương 900 Seconds).

{{% notice tip %}}
**System Design Note:** Do đặc thù các mô hình AI xử lý file video/audio tiêu tốn nhiều thời gian tính toán, thông số *Visibility timeout* cần được thiết lập đủ dài (15 phút). Khi một AI Worker nhận thông điệp từ hàng đợi, thông điệp này sẽ tạm ẩn đối với các Worker khác. Điều này đảm bảo tiến trình Worker hiện tại có đủ thời gian hoàn thành tác vụ mà không bị các máy chủ khác giành giật hoặc xử lý trùng lặp.
{{% /notice %}}

- **Phân đoạn Dead-letter queue - Optional:**
  - **Set this queue to receive undeliverable messages:** Chuyển sang trạng thái **Enabled**.
  - **Choose queue:** Chọn đúng hàng đợi `cloudforge-media-dlq` vừa khởi tạo ở mục 1.
  - **Maximum receives:** Thiết lập giá trị bằng `3`. (Nếu một tác vụ Media bị Worker xử lý lỗi vượt quá 3 lần, thông điệp sẽ tự động chuyển sang DLQ để đội ngũ kỹ sư kiểm tra thủ công).

![Configure Task Queue](/images/5-Workshop/5.6-Ingestion-workflow/5.6.1-create-sqs-queue/configure_task_queue1.png)

![Configure Task Queue](/images/5-Workshop/5.6-Ingestion-workflow/5.6.1-create-sqs-queue/configure_task_queue2.png)

Cuối cùng, kiểm tra lại toàn bộ thông tin cấu hình và bấm nút **Create queue**.

#### 3. Kết quả triển khai
Sau khi khởi tạo thành công, hệ thống sẽ cấp phát một định danh URL duy nhất cho từng hàng đợi. Chuỗi URL này sẽ được ghi nhận để cấu hình vào các biến môi trường của phân lớp ứng dụng.

![SQS Queue Created](/images/5-Workshop/5.6-Ingestion-workflow/5.6.1-create-sqs-queue/sqs_queue_created.png)

***

**Bước tiếp theo:** Sau khi hoàn tất phân lớp lưu trữ và điều phối tin nhắn (SQS), tiến hành tiến hành cấu hình bộ định tuyến tại bài [**5.6.2: Cấu hình EventBridge**](../5.6.2-create-eventbridge-rule/) để tự động bắt các sự kiện thay đổi dữ liệu từ S3 và đẩy vào hàng đợi này.
