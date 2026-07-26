---
title : "Tích hợp Amazon Transcribe"
date : 2026-07-10
weight : 2
chapter : false
pre : " <b> 5.8.2. </b> "
---

Để hiểu và bóc tách được nội dung của một tệp Video hay Audio, bước đầu tiên và quan trọng nhất trong đường ống xử lý (Pipeline) là phải trích xuất được luồng hội thoại thành văn bản thô. Trong kiến trúc tổng thể của Smart Media Analytics, chúng ta sử dụng dịch vụ **Amazon Transcribe** - giải pháp nhận dạng tiếng nói tự động (Automatic Speech Recognition - ASR) dựa trên công nghệ học máy chuyên sâu của AWS.

Không giống như Amazon Bedrock yêu cầu cơ chế tự động kích hoạt hoặc khai báo Use Case từ phía người dùng, Amazon Transcribe được AWS mở khóa sẵn mặc định trên mọi tài khoản. Trong phân đoạn này, tiến hành tìm hiểu sâu về cơ chế tương tác tự động giữa AI Worker và dịch vụ xử lý âm thanh này.

#### Luồng xử lý âm thanh tự động (Audio Processing Flow)

Ứng dụng AI Worker được lập trình để thực hiện một chu trình tự động hóa hoàn toàn bằng Python (thông qua thư viện SDK `boto3`) để tương tác với Amazon Transcribe:

1. **Trích xuất dữ liệu âm thanh:** Khi AI Worker tiêu thụ một thông điệp (Metadata của Video vừa tải lên) từ hàng đợi Amazon SQS, nó sẽ gọi công cụ `FFmpeg` tích hợp sẵn trong Docker Container để bóc tách riêng luồng âm thanh ra khỏi tệp Video gốc.
2. **Lưu trữ trung gian:** Tệp âm thanh sau khi trích xuất sẽ được tải ngược lên một thư mục tạm thời chuyên biệt trên **Amazon S3** theo cấu trúc đường dẫn `s3://[Bucket-Name]/audio-temp/`.
3. **Kích hoạt Transcribe Job:** AI Worker gửi một lệnh gọi API `StartTranscriptionJob` tới Amazon Transcribe, truyền vào liên kết S3 của tệp âm thanh đồng thời cấu hình tính năng tự động nhận dạng ngôn ngữ (Auto Language Identification).
4. **Kiểm tra trạng thái bất đồng bộ (Polling):** Do tiến trình chuyển đổi giọng nói cần thời gian tính toán và diễn ra bất đồng bộ, AI Worker sẽ liên tục thực hiện vòng lặp thăm dò (Poll) qua API `GetTranscriptionJob` để cập nhật trạng thái xử lý của hệ thống.
5. **Đồng bộ kết quả:** Ngay khi trạng thái chuyển sang nhãn `COMPLETED`, Amazon Transcribe sẽ xuất ra một tệp JSON chứa toàn bộ văn bản (Transcript) đã được bóc tách, đi kèm mốc thời gian chi tiết (Timestamps) của từng từ ngữ.

![Transcribe Flow](/images/5-Workshop/5.8-AI-ML-integration/5.8.2-setup-transcribe/transcribe_flow.png)

#### Yêu cầu quyền hạn hạ tầng (IAM Role Permissions)
Để quy trình tự động hóa trên chạy trôi chảy không gặp lỗi phân quyền, **ECS Task Role** của dịch vụ AI Worker bắt buộc phải được đính kèm các chính sách quyền hạn sau:
- `transcribe:StartTranscriptionJob` và `transcribe:GetTranscriptionJob` để điều khiển tiến trình chuyển đổi văn bản.
- `s3:GetObject` và `s3:PutObject` trên Bucket chỉ định nhằm mục đích ghi tệp âm thanh đầu vào và đọc kết quả phân tích JSON đầu ra.

{{% notice tip %}}
**Tối ưu hóa chi phí (Cost Optimization):** Việc AI Worker chủ động tách riêng và nén luồng âm thanh đầu vào (như định dạng MP3/OGG) trước khi đẩy lên đám mây không chỉ làm giảm băng thông mạng nội bộ mà còn tiết kiệm đáng kể chi phí lưu trữ. Do Amazon Transcribe tính toán chi phí dựa trên từng giây xử lý của tệp, tệp tin gọn nhẹ sẽ giúp tăng tốc độ xử lý I/O và tối ưu tài nguyên hóa đơn.
{{% /notice %}}

***

### 2. Thực hành Cấu hình (Hands-on)

Để mã nguồn của hệ thống kích hoạt luồng xử lý AWS Serverless thay vì chạy Local, cũng như được phép thao tác với các dịch vụ đám mây, chúng ta cần thực hiện cập nhật cấu hình cho ECS Task.

#### 2.1. Cập nhật quyền hạn (IAM Role)
Như đã đề cập trong lý thuyết, quá trình gọi AI được chuyển giao cho Backend tiếp tục xử lý sau khi Worker chạy xong. Do đó, **ECS-Backend-TaskRole** cần được bổ sung các quyền tương tác với dịch vụ Amazon Transcribe và Amazon S3.

- Truy cập dịch vụ **[IAM Console](https://us-east-1.console.aws.amazon.com/iam/home)**.
- Chọn mục **Roles** ở thanh menu bên trái và tìm kiếm role `ECS-Backend-TaskRole`.
- Đảm bảo Role này đã được đính kèm các chính sách (Policy) cấp quyền sau:
  - `transcribe:StartTranscriptionJob`
  - `transcribe:GetTranscriptionJob`
  - `s3:PutObject`
  - `s3:DeleteObject`

![Update IAM Role](/images/5-Workshop/5.8-AI-ML-integration/5.8.2-setup-transcribe/iam_role_permissions.png)

#### 2.2. Khai báo Biến môi trường (Environment Variables)

- Truy cập dịch vụ **[Amazon ECS](https://console.aws.amazon.com/ecs/home)**,
- Chọn Task Definition của **Backend Task** (`cloudforge-backend-task`), tiến hành tạo một bản sửa đổi mới (Create new revision).
- Tìm đến mục Environment variables, bạn cần bổ sung hoặc đảm bảo hệ thống có biến môi trường sau:
  - `AI_PROVIDER`: `aws`
  - `AWS_S3_BUCKET`: `cloudforge-media-upload-<tên-của-bạn>` *(Bucket đã tạo ở mục 5.4.1)*

![Update Task Definition](/images/5-Workshop/5.8-AI-ML-integration/5.8.2-setup-transcribe/task_def_env_vars.png)

#### 2.3. Cấp quyền Bucket Policy cho S3 (Quan trọng)
Amazon Transcribe là một dịch vụ độc lập, nó cần được cấp quyền đọc tệp âm thanh từ S3 Bucket của bạn để tiến hành phân tích. Do đó, bạn cần bổ sung Bucket Policy:

1. Truy cập **Amazon S3**, chọn Bucket `cloudforge-media-upload-<tên-của-bạn>`.
2. Chuyển sang tab **Permissions**, cuộn xuống phần **Bucket policy** và chọn **Edit**.
3. Dán đoạn JSON sau (nhớ thay đổi `<tên-của-bạn>` cho khớp với bucket của bạn):
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowTranscribe",
            "Effect": "Allow",
            "Principal": {
                "Service": "transcribe.amazonaws.com"
            },
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::cloudforge-media-upload-<tên-của-bạn>",
                "arn:aws:s3:::cloudforge-media-upload-<tên-của-bạn>/*"
            ]
        }
    ]
}
```

![Update Task Definition](/images/5-Workshop/5.8-AI-ML-integration/5.8.2-setup-transcribe/bucket_policy.png)

4. Chọn **Save changes**.

#### 2.4. Kiểm thử tiến trình trên AWS (Testing)
Sau khi cập nhật cấu hình ECS Task, Bucket Policy và tự động triển khai mã nguồn mới qua CI/CD, chúng ta sẽ xác minh luồng thực thi thông qua quá trình tải lên video thực tế:

1. Tải lên một video thông qua giao diện Web của hệ thống.
2. Truy cập vào bucket S3. Bạn sẽ quan sát thấy một tệp Audio tạm thời được tạo ra trong thư mục `audio-temp/`:
   ![Audio Temp in S3](/images/5-Workshop/5.8-AI-ML-integration/5.8.2-setup-transcribe/s3_audio_temp.png)
3. Chuyển sang dịch vụ **Amazon Transcribe**, chọn mục **Transcription jobs**. Bạn sẽ thấy một tiến trình mang tên `sma-transcribe-...` đang trong trạng thái **In progress** hoặc **Completed**.
   ![Transcribe Job Progress](/images/5-Workshop/5.8-AI-ML-integration/5.8.2-setup-transcribe/transcribe_job_status.png)

***

**Bước tiếp theo:** Sau khi đã sở hữu toàn bộ dữ liệu văn bản thô được trích xuất từ Video/Audio, tiến hành chuyển sang phân đoạn cuối cùng của luồng phân tích tri thức: Sử dụng mô hình nhúng toán học nhằm tạo lập các Vector (Embeddings), phục vụ trực tiếp cho tính năng tìm kiếm ngữ nghĩa chuyên sâu (Semantic Search).
