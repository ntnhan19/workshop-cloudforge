---
title : "Kiểm tra luồng Ingestion"
date : 2026-07-10
weight : 4
chapter : false
pre : " <b> 5.6.4. </b> "
---

Sau khi hoàn tất cấu hình toàn bộ các thành phần cốt lõi của kiến trúc điều phối hướng sự kiện (S3, SQS, EventBridge, Step Functions), bước cuối cùng là thực hiện kiểm thử tích hợp (Integration Test). Quá trình này nhằm xác thực tính thông suốt của đường ống dữ liệu (Data Pipeline) từ lúc tiếp nhận file cho đến khi kích hoạt luồng xử lý nghiệp vụ.

Kịch bản kiểm thử được chia làm hai giai đoạn độc lập nhằm minh chứng cho tính phân tách cao (Loose Coupling) của hệ thống.

#### 1. Kiểm thử luồng định tuyến sự kiện (S3 → EventBridge → SQS)
Mục tiêu của giai đoạn này là xác nhận Amazon EventBridge có bắt đúng sự kiện thay đổi trạng thái từ Amazon S3 và chuyển giao gói tin siêu dữ liệu (Metadata) an toàn vào hàng đợi Amazon SQS hay không.

**Bước 1: Tải lên tệp tin dữ liệu mẫu**
- Truy cập vào giao diện console của **[Amazon S3](https://s3.console.aws.amazon.com/s3/home)** và chọn Bucket `cloudforge-media-upload-ntnhan19`.
- Bấm **Add files**, chọn một tệp tin đa phương tiện mẫu (ví dụ: `sample-audio.mp3` hoặc `demo-video.mp4`) từ máy tính và bấm **Upload**.

![S3 Upload Success](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/s3_upload_success.png)

**Bước 2: Kiểm tra và xác thực thông điệp tại hàng đợi SQS**
- Truy cập dịch vụ **[Amazon SQS](https://console.aws.amazon.com/sqs/v2/home#/queues)** → Chọn hàng đợi tác vụ chính `cloudforge-media-task-queue`.
- (Tùy chọn) Ở phần **Details**, hãy bấm vào nút **`► More`** để mở rộng. Sẽ thấy chỉ số **Messages available** tăng lên (ví dụ: 1) báo hiệu có sự kiện mới được đẩy vào hàng đợi nhưng chưa có ai lấy ra xử lý.

![SQS Queue Details](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/sqs_queue_list.png)

- Chọn **Send and receive messages** → Di chuyển xuống mục *Receive messages* và bấm **Poll for messages**.
- Click vào ID thông điệp vừa xuất hiện để kiểm tra tab **Body**. Gói tin JSON được EventBridge gửi qua phải chứa chính xác các thông tin:
  - Sự kiện phát sinh: `ObjectCreated:Put`.
  - Tên tệp tin gốc nằm trong trường `object.key`.

![SQS Message Received](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/sqs_message_received.png)

#### 2. Xác thực tiến trình điều phối (Step Functions Workflow)
Mục tiêu của giai đoạn này là kiểm tra xem EventBridge có kích hoạt thành công Step Functions và truyền đúng siêu dữ liệu từ S3 hay không.

**Bước 1: Kiểm tra lịch sử tự động kích hoạt**
- Truy cập dịch vụ **[AWS Step Functions](https://console.aws.amazon.com/states/home#/statemachines)** → Chọn máy trạng thái `cloudforge-media-workflow`.
- Bạn KHÔNG CẦN bấm nút "Start execution". Vì chúng ta đã cấu hình EventBridge trỏ tới Step Functions, hành động tải file lên S3 ở bước trước đã tự động tạo ra một lượt chạy (execution) mới!
- Tại tab **Executions**, hãy click vào lượt chạy gần nhất ở trên cùng.

![Step Functions Executions](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/step_functions_executions.png)

**Bước 2: Giám sát trực quan luồng công việc (Graph View)**
- Graph View lúc này sẽ hiển thị trọn vẹn logic điều phối của hệ thống: `Launch AI Worker` và tiếp theo là `Call Backend Webhook`.
- Luồng thực thi sẽ chuyển sang trạng thái `Launch AI Worker` và chủ đích báo lỗi (do chúng ta chưa triển khai cụm ECS Compute Cluster).

![Step Functions Graph View](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/step_functions_graph.png)

**Bước 3: Phân tích Dữ liệu Đầu vào (Input)**
- Click vào bước `Launch AI Worker` (đang báo lỗi) trên Graph View.
- Ở thanh bên phải, kiểm tra tab **Step input**. Sẽ thấy toàn bộ gói tin JSON sự kiện từ S3, chứng minh rằng EventBridge đã truyền siêu dữ liệu thành công cho Step Functions.

#### 3. Kết luận Phân lớp Điều phối Workflow
Kết quả thực nghiệm cho thấy toàn bộ đường ống Ingestion Workflow đã vận hành hoàn toàn đúng với thiết kế kiến trúc Fan-out ban đầu. Dữ liệu từ tầng lưu trữ đã được định tuyến tới SQS (để lưu vết) và đồng thời kích hoạt thành công kịch bản điều phối (Step Functions).

***

**Bước tiếp theo:** Hệ thống điều phối và đường ống tin nhắn hướng sự kiện của Chương 5.6 đã hoàn thành trọn vẹn. Chúng ta đã có đầy đủ nền tảng vững chắc để bước sang [**Chương 5.7: Triển khai Compute (ECS)**](../../5.7-Compute-setup/) nhằm cấu hình các Container thực thi mô hình AI thật sự.
