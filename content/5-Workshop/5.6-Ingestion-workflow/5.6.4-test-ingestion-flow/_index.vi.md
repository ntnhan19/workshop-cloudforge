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
- (Tùy chọn) Ở phần **Details**, bạn hãy bấm vào nút **`► More`** để mở rộng. Bạn sẽ thấy chỉ số **Messages available** tăng lên (ví dụ: 1) báo hiệu có sự kiện mới được đẩy vào hàng đợi nhưng chưa có ai lấy ra xử lý.

![SQS Queue Details](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/sqs_queue_list.png)

- Chọn **Send and receive messages** → Di chuyển xuống mục *Receive messages* và bấm **Poll for messages**.
- Click vào ID thông điệp vừa xuất hiện để kiểm tra tab **Body**. Gói tin JSON được EventBridge gửi qua phải chứa chính xác các thông tin:
  - Sự kiện phát sinh: `ObjectCreated:Put`.
  - Tên tệp tin gốc nằm trong trường `object.key`.

![SQS Message Received](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/sqs_message_received.png)

#### 2. Xác thực tiến trình điều phối (Step Functions Workflow)
Mục tiêu của giai đoạn này là kiểm tra tính hợp lệ và khả năng rẽ nhánh logic nghiệp vụ của máy trạng thái khi tiếp nhận yêu cầu điều phối.

**Bước 1: Kích hoạt thực thi State Machine**
- Truy cập dịch vụ **[AWS Step Functions](https://console.aws.amazon.com/states/home#/statemachines)** → Chọn máy trạng thái `cloudforge-media-workflow`.
- Bấm nút **Start execution**. Tại hộp thoại Input, bạn có thể nhập thử một đoạn JSON giả lập (ví dụ: `{"status": "success"}`) và chọn **Start execution** một lần nữa.

![Step Functions Input Mock](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/step_functions_input_mock.png)

![Step Functions Execution List](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/step_functions_execution_list.png)

**Bước 2: Giám sát trực quan luồng công việc (Graph View)**
- Dựa trên điều kiện giữ chỗ (`{% true %}`) đã thiết lập tại phân đoạn xây dựng bộ khung, luồng thực thi sẽ tự động chuyển trạng thái tuần tự từ `Start Processing` → vượt qua điểm đánh giá `Check Status` → hội tụ chính xác về nhánh tác vụ thành công `Update Metadata`.
- Thanh trạng thái tổng thể hiển thị màu xanh lá kèm nhãn **Succeeded**.

![Step Functions Success](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/step_functions_success.png)

**Bước 3: Phân tích Dữ liệu Đầu vào / Đầu ra (Input/Output)**
- Click vào một bước (step) bất kỳ trên Graph View (ví dụ: `Update Metadata`).
- Ở thanh bên phải, kiểm tra tab **Step input** và **Step output** để thấy rõ cách dữ liệu được luân chuyển giữa các bước. *(Lưu ý: Vì đây mới chỉ là các bước "Pass State" làm bộ khung giả lập, dữ liệu bạn nhập ở Bước 1 sẽ được truyền thẳng từ Input sang Output mà không bị biến đổi).*

![Step Functions Details](/images/5-Workshop/5.6-Ingestion-workflow/5.6.4-test-ingestion-flow/step_functions_details.png)

#### 3. Kết luận Phân lớp Điều phối Workflow
Kết quả thực nghiệm cho thấy toàn bộ đường ống Ingestion Workflow đã vận hành hoàn toàn đúng với thiết kế kiến trúc ban đầu. Dữ liệu từ tầng lưu trữ đã được định tuyến, cô lập an toàn tại bộ đệm hàng đợi và kịch bản điều phối sẵn sàng kết nối với các ứng dụng tính toán thực tế.

***

**Bước tiếp theo:** Hệ thống điều phối và đường ống tin nhắn hướng sự kiện của Chương 5.6 đã hoàn thành trọn vẹn. Chúng ta đã có đầy đủ nền tảng vững chắc để bước sang **Chương 5.7: Triển khai Compute (ECS)** nhằm cấu hình các cụm máy chủ Container thực thi mô hình AI kéo dữ liệu từ SQS.
